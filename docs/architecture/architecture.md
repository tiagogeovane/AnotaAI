# AnotaAI — Arquitetura de Software & Modelagem Técnica

Este documento define a arquitetura técnica, modelo de dados, pipeline de inteligência artificial e estrutura do código do **AnotaAI**.

---

## 1. Princípios Arquiteturais

1. **100% Local-First:** Os dados do usuário residem integralmente no dispositivo. Não há servidor central ou backend dedicado.
2. **Provider & ChangeNotifier:** Gerenciamento de estado direto, reativo, com baixo boilerplate e alta testabilidade.
3. **Persistência Híbrida:**
   - **Banco SQLite (sqflite):** Indexação de metadados, pastas, histórico, estatísticas de uso e busca rápida.
   - **Sistema de Arquivos Local (path_provider):** Arquivos físicos `.md` (legíveis e fáceis de exportar) e fotos comprimidas `.jpg`.
   - **Armazenamento Criptografado (flutter_secure_storage):** Chave de API do Google Gemini (Keystore no Android / Keychain no iOS).
4. **Comunicação Direta com Gemini:** Chamadas HTTP REST seguras diretamente do dispositivo para a API oficial do Google Gemini com saída estruturada (*Structured JSON Output*).

---

## 2. Diagrama da Arquitetura em Camadas

```
┌────────────────────────────────────────────────────────────────────────┐
│                        Camada de Apresentação (UI)                     │
│  - Screens: Dashboard, CameraViewfinder, NoteViewer, Folders, Settings │
│  - Widgets: MetricCards, ViewSwitcher, BoundingBoxHUD, ActionSheets    │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Assina (Consumer / context.watch)
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                  Camada de Estado (ChangeNotifier Providers)           │
│  - NotesProvider: Criação, listagem, append e remoção de notas         │
│  - ScannerProvider: Controle de câmera, disparo e processamento OCR    │
│  - SettingsProvider: API Key, quota de requisições, endpoints e tema   │
│  - FoldersProvider: Gestão de pastas e contadores                      │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Executa comandos e regras
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                     Camada de Serviços & Repositórios                  │
│  - GeminiVisionService: Formatação de payload, prompt e chamadas HTTP  │
│  - ImageProcessingService: Compressão, redimensionamento de JPEG       │
│  - NotesRepository: CRUD sincronizado entre SQLite e FileSystem        │
│  - SecurityService: Leitura e escrita de credenciais seguras           │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Acessa
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                         Camada de Dados & Driver                       │
│  - SQLite (sqflite): Tabelas `folders`, `notes`, `api_usage_logs`      │
│  - File System: `/app_data/notes/*.md`, `/app_data/images/*.jpg`       │
│  - FlutterSecureStorage: Encrypted SharedPreferences / Keychain        │
│  - Google Gemini API Endpoint: `generativelanguage.googleapis.com`     │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Modelo de Dados e Persistência

### 3.1 Esquema do Banco SQLite (`anota_ai.db`)

```sql
-- Tabela de Pastas
CREATE TABLE folders (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    color_accent TEXT NOT NULL,
    icon_name TEXT NOT NULL,
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL
);

-- Tabela de Notas
CREATE TABLE notes (
    id TEXT PRIMARY KEY,
    folder_id TEXT,
    title TEXT NOT NULL,
    summary TEXT,
    key_takeaway TEXT,
    tags TEXT, -- JSON array serializado de strings: ["fourier", "physics"]
    file_path TEXT NOT NULL, -- Caminho relativo para o arquivo .md
    ocr_accuracy REAL,       -- Estimativa de acurácia (ex: 0.98)
    tokens_parsed INTEGER DEFAULT 0,
    formulas_detected INTEGER DEFAULT 0,
    mermaid_detected INTEGER DEFAULT 0,
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL,
    FOREIGN KEY(folder_id) REFERENCES folders(id) ON DELETE SET NULL
);

-- Tabela de Imagens Originais da Nota (Suporta Modo Batch 1..5 e Append)
CREATE TABLE note_images (
    id TEXT PRIMARY KEY,
    note_id TEXT NOT NULL,
    image_path TEXT NOT NULL, -- Caminho relativo para a foto .jpg otimizada
    width INTEGER,
    height INTEGER,
    order_index INTEGER NOT NULL DEFAULT 0,
    captured_at INTEGER NOT NULL,
    FOREIGN KEY(note_id) REFERENCES notes(id) ON DELETE CASCADE
);

-- Tabela de Ações / To-dos detectados
CREATE TABLE action_items (
    id TEXT PRIMARY KEY,
    note_id TEXT NOT NULL,
    content TEXT NOT NULL,
    is_completed INTEGER NOT NULL DEFAULT 0,
    order_index INTEGER NOT NULL DEFAULT 0,
    FOREIGN KEY(note_id) REFERENCES notes(id) ON DELETE CASCADE
);

-- Histórico de Consumo da API (para os gráficos da tela de Settings & Dashboard)
CREATE TABLE api_usage_logs (
    id TEXT PRIMARY KEY,
    provider TEXT NOT NULL DEFAULT 'gemini',
    pages_count INTEGER NOT NULL DEFAULT 1, -- Quantidade de fotos convertidas na chamada
    tokens_prompt INTEGER NOT NULL,
    tokens_response INTEGER NOT NULL,
    latency_ms INTEGER NOT NULL,
    status_code INTEGER NOT NULL,
    timestamp INTEGER NOT NULL
);
```

### 3.2 Estrutura do Sistema de Arquivos

```
<AppDocumentDirectory>/
  ├── notes/
  │   ├── note_1730890000_thermodynamics.md
  │   └── note_1730891200_sprint_retro.md
  └── images/
      ├── scan_1730890000_thermodynamics.jpg
      └── scan_1730891200_sprint_retro.jpg
```

---

## 4. Pipeline de Inteligência Artificial (Multimodal)

### 4.1 Fluxo de Captura e Processamento

1. **Captura:** O usuário tira a foto ou seleciona da galeria via `camera` ou `image_picker`.
2. **Otimização Local:** A imagem é comprimida e redimensionada via `flutter_image_compress` (máx. 1920px de largura, qualidade JPEG 80%), reduzindo o tamanho de ~12MB para ~400KB sem perder legibilidade da escrita.
3. **Conversão Base64:** A imagem comprimida é codificada em Base64.
4. **Requisição ao Gemini:** Chamada `POST` para `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key={USER_API_KEY}`.
5. **Configuração da Saída:** `responseMimeType: "application/json"` com o `responseSchema` rigorosamente tipado.
6. **Persistência Atômica:**
   - Salva a foto otimizada na pasta `images/`.
   - Salva o corpo markdown na pasta `notes/`.
   - Insere o registro e os action items no SQLite.
   - Incrementa o contador de uso em `api_usage_logs`.

### 4.2 Schema JSON da Resposta do Gemini

```json
{
  "title": "Título resumido e descritivo da nota",
  "summary": "Breve parágrafo sumarizando o conteúdo do quadro/anotação",
  "key_takeaway": "O principal aprendizado ou conclusão da nota",
  "tags": ["conceito1", "materia", "topico"],
  "action_items": [
    "Tarefa detectada 1",
    "Resolver exercício da lista"
  ],
  "markdown_content": "# Seção 1\nTexto estruturado com equações $$\\Delta U = Q - W$$...",
  "ocr_accuracy_estimate": 0.95
}
```

### 4.3 System Instruction do Modelo

> *"Você é um especialista em OCR inteligente e transcrição de anotações manuscritas e quadros brancos. Analise a imagem fornecida, remova ruídos de iluminação e manchas, e transcreva todo o conteúdo em um documento Markdown elegante e didático. Se houver equações matemáticas, formate-as em LaTeX ($...$ ou $$...$$). Se houver diagramas conceituais ou fluxogramas, gere a sintaxe Mermaid JS correspondente. Extraia claramente itens de ação/tarefas em formato de lista."*

---

## 5. Estrutura de Pastas do Projeto Flutter

```
lib/
├── core/
│   ├── constants/
│   │   ├── api_endpoints.dart
│   │   └── app_colors.dart        # Paleta Obsidian (#131316, #F3F7A2, #BCE3EC, etc.)
│   ├── theme/
│   │   └── app_theme.dart         # Space Grotesk & JetBrains Mono typography
│   ├── utils/
│   │   ├── date_formatter.dart
│   │   └── file_helper.dart
│   └── errors/
│       └── app_exception.dart
│
├── data/
│   ├── datasources/
│   │   ├── local_db_datasource.dart       # sqflite implementation
│   │   ├── file_storage_datasource.dart   # path_provider read/write
│   │   └── secure_storage_datasource.dart # flutter_secure_storage
│   ├── models/
│   │   ├── folder_model.dart
│   │   ├── note_model.dart
│   │   └── gemini_response_model.dart
│   └── repositories/
│       ├── notes_repository_impl.dart
│       └── settings_repository_impl.dart
│
├── domain/
│   ├── entities/
│   │   ├── note_entity.dart
│   │   ├── folder_entity.dart
│   │   └── action_item_entity.dart
│   └── services/
│       ├── gemini_vision_service.dart
│       └── image_processing_service.dart
│
├── presentation/
│   ├── providers/
│   │   ├── notes_provider.dart
│   │   ├── scanner_provider.dart
│   │   ├── settings_provider.dart
│   │   └── folders_provider.dart
│   ├── screens/
│   │   ├── dashboard/
│   │   │   ├── dashboard_screen.dart
│   │   │   └── widgets/
│   │   │       ├── metric_card.dart
│   │   │       ├── weekly_activity_chart.dart
│   │   │       └── recent_scans_list.dart
│   │   ├── scanner/
│   │   │   ├── camera_scanner_screen.dart
│   │   │   ├── widgets/
│   │   │   │   ├── bounding_hud_overlay.dart
│   │   │   │   └── shutter_button.dart
│   │   │   └── destination_modal.dart
│   │   ├── note_viewer/
│   │   │   ├── note_viewer_screen.dart
│   │   │   └── widgets/
│   │   │       ├── markdown_renderer.dart
│   │   │       ├── original_image_viewer.dart
│   │   │       └── action_items_checklist.dart
│   │   ├── folders/
│   │   │   ├── folders_screen.dart
│   │   │   └── select_destination_screen.dart
│   │   └── settings/
│   │       ├── settings_screen.dart
│   │       └── widgets/
│   │           ├── api_key_input_card.dart
│   │           └── quota_circular_gauge.dart
│   └── widgets/
│       ├── custom_app_bar.dart
│       └── bottom_navigation_deck.dart
│
└── main.dart
```

---

## 6. Dependências Principais (`pubspec.yaml`)

| Categoria | Pacote | Função |
| :--- | :--- | :--- |
| **Estado** | `provider` | Gerenciamento de estado via `ChangeNotifierProvider` |
| **Persistência** | `sqflite`, `path_provider` | Banco de dados relacional e manipulação de arquivos `.md` e `.jpg` |
| **Segurança** | `flutter_secure_storage` | Criptografia local da chave de API do Gemini |
| **Câmera & Mídia** | `camera`, `image_picker` | Captura fotográfica no viewfinder e acesso à galeria |
| **Compressão** | `flutter_image_compress` | Otimização e redução de peso de imagens antes do envio |
| **Rede HTTP** | `http` | Comunicação direta e veloz com a API REST do Gemini |
| **Renderização** | `flutter_markdown` | Renderização visual de notas Markdown |
| **Matemática** | `flutter_math_fork` | Suporte à renderização de equações em LaTeX |
| **Tipografia** | `google_fonts` | Carregamento de *Space Grotesk* e *JetBrains Mono* |
| **Ícones** | `material_symbols_icons` | Ícones oficiais Google Material Symbols Outlined |

---

## 7. Segurança e Proteção de Dados

1. **Armazenamento Seguro da Chave:** A API Key do Gemini nunca é escrita em SharedPreferences planas ou arquivos de log. Usa-se Keystore/Keychain.
2. **Zero Log de Imagens:** Fotos originais nunca são transmitidas para nenhum servidor intermediário além da API oficial do Google.
3. **Sandbox do Sistema Operacional:** Tanto o banco SQLite quanto a pasta de notas residem no diretório privado da aplicação, isolados de outros apps do celular.
