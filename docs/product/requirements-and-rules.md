# AnotaAI — Requisitos do Sistema e Regras de Negócio

Este documento especifica os **Requisitos Funcionais (RF)**, **Requisitos Não-Funcionais (RNF)** e **Regras de Negócio (RN)** do aplicativo **AnotaAI**, servindo como contrato base para a implementação e validação de testes.

---

## 1. Requisitos Funcionais (RF)

### 1.1 Configuração de IA e Credenciais (BYOK)
* **RF01 — Gerenciamento de API Key:** O usuário deve poder inserir, visualizar/ocultar, substituir e remover sua chave de API do Google Gemini na tela `AI & Model Settings`.
* **RF02 — Customização de Endpoint (Base URL):** O usuário deve poder configurar uma URL base customizada (padrão: `https://generativelanguage.googleapis.com/v1beta`) para suportar proxies ou gateways corporativos.
* **RF03 — Arquitetura Multi-Provider (UI & Extensibilidade):** A interface de configurações deve exibir a seleção de provedores conforme projetado no Stitch (*Gemini*, *OpenAI*, *Anthropic*, *Other Provider*), sendo o **Google Gemini** totalmente funcional no MVP e os demais estruturados sob a interface `AiVisionProvider`.
* **RF04 — Verificação e Teste de Latência:** O botão *"Verify & Save Credentials"* deve realizar uma chamada leve de validação à API configurada, medir o tempo de resposta em milissegundos (ex: `28ms`), atualizar o indicador visual para `Ready` e persistir a chave somente se válida (ou mediante confirmação).
* **RF05 — Monitoramento de Quota e Histórico:** O sistema deve exibir um gráfico circular (gauge) com o consumo diário de requisições em relação ao limite configurado (padrão: `1.500 req/dia`), o horário de reset (`00:00 UTC`) e um gráfico de barras com o volume de chamadas dos últimos 7 dias.

### 1.2 Captura de Imagem e Scanner Inteligente
* **RF06 — Scanner em Modo Retrato (Portrait 3:4):** A tela `Live Camera Scanner` deve exibir o feed da câmera em proporção 3:4 com HUD de enquadramento (`detection-box`), botão de disparo tátil, flash toggle, grade 4x4 e filtro anti-reflexo (*de-glare*).
* **RF07 — Scanner em Modo Paisagem (Horizontal Widescreen):** Ao girar o dispositivo para a horizontal (ou tocar no botão de alternância de orientação), a interface deve adaptar-se para `Live Camera Scanner - Horizontal`, exibindo um viewfinder panorâmico widescreen com grade 3x3, retículo neon e um **Dock Lateral Direito (`CameraControlsDock`)** ergonômico para disparo com o polegar.
* **RF08 — Modo Single vs. Batch (Múltiplas Páginas):** O usuário deve poder alternar entre captura única (`Single`) e captura em lote (`Batch`, de 1 até 5 fotos sequenciais) antes de prosseguir para a transcrição.
* **RF09 — Importação da Galeria:** O usuário deve poder importar 1 ou até 5 imagens existentes da biblioteca de fotos do dispositivo para transcrição.

### 1.3 Triagem, Criação e Anexação (Append) de Notas
* **RF10 — Modal de Destino Pós-Captura:** Imediatamente após capturar a(s) foto(s) ou importar da galeria, o app deve exibir o `Save Note Destination Modal` com a miniatura da captura, opção de refazer (`Retake`), opção **"Create New Note"** e opção **"Add to Existing Note"** (com atalhos para pastas recentes).
* **RF11 — Criação de Nova Nota com Auto-Classificação:** Ao selecionar *"Create New Note"*, o sistema deve enviar a(s) imagem(ns) e a lista de pastas disponíveis ao Gemini, que retornará o documento Markdown estruturado e a pasta sugerida onde a nota será salva automaticamente.
* **RF12 — Seleção de Nota Destino e Append Contextual:** Ao selecionar *"Add to Existing Note"*, o usuário deve navegar pela tela `Select Destination Note` (com busca, filtros *Recent/Starred/Tags* e árvore de pastas em acordeão), escolher a nota alvo e confirmar em *"Append Transcription"*.

### 1.4 Visualização, Edição e Exportação de Notas
* **RF13 — Visualizador Dual (Markdown Sintetizado vs. Original Scan):** A tela `Transcribed Markdown Note` deve possuir um seletor de abas permitindo alternar instantaneamente entre a nota renderizada (*Synthesized Markdown*) e a visualização da(s) foto(s) original(is) (*Original Scan* em carrossel quando houver múltiplas imagens).
* **RF14 — Renderização Rica de Conteúdo:** O visualizador Markdown deve renderizar de forma nativa e estilizada:
  - Card de síntese superior (título, resumo estrutural e metadados de tokens).
  - Callout em destaque para o **Key Takeaway**.
  - Seções numeradas, listas, tabelas e blocos de código/diagramas **Mermaid JS**.
  - Fórmulas matemáticas em **LaTeX** (inline `$...$` e bloco `$$...$$`).
  - Lista de **Detected Action Items** com checkboxes interativos.
  - Chips de **Tags de Contexto** (`#tag`).
* **RF15 — Edição Inline e Checkboxes Sincronizados:** O usuário deve poder editar manualmente o título, a pasta associada e o corpo Markdown da nota para eventuais ajustes finos, bem como marcar/desmarcar checkboxes de tarefas, refletindo a alteração tanto no banco SQLite quanto no arquivo `.md` físico.
* **RF16 — Cópia e Exportação de Arquivo `.md`:** O usuário deve poder copiar todo o conteúdo Markdown para a área de transferência (*Clipboard*) com um toque ou exportar/compartilhar o arquivo `.md` físico usando o share sheet nativo do sistema operacional.

### 1.5 Organização de Pastas e Dashboard
* **RF17 — Gestão de Pastas e Categorias:** Na tela `Folders & Categories`, o usuário deve visualizar suas pastas com ícone temático, cor de destaque, quantidade de notas e tempo desde a última atualização, podendo filtrar, buscar e criar novas pastas pelo botão flutuante (`+`).
* **RF18 — Home Dashboard Analítico:** A tela inicial deve apresentar saudação contextual conforme o horário do dia, contagem de notas digitalizadas hoje, cartão de acesso rápido ao scanner, gráfico semanal de páginas convertidas (segunda a domingo) e lista das últimas notas processadas.

---

## 2. Requisitos Não-Funcionais (RNF)

* **RNF01 — Arquitetura 100% Local-First e Privacidade:** Nenhum dado pessoal, nota, foto ou chave de API deve transitar por servidores intermediários. A única comunicação externa permitida é diretamente entre o dispositivo do usuário e o endpoint de IA configurado pelo próprio usuário.
* **RNF02 — Segurança de Credenciais:** Todas as chaves de API devem ser armazenadas exclusivamente no cofre seguro do sistema operacional (`Android Keystore` / `iOS Keychain`) via `flutter_secure_storage`.
* **RNF03 — Otimização e Compressão de Imagem:** Antes do envio à API e salvamento em disco, toda foto capturada deve ser comprimida localmente em background (resolução máxima de `1920px` no maior lado, formato JPEG com qualidade `80%`), mantendo legibilidade de OCR e reduzindo o payload para `~250KB–500KB`.
* **RNF04 — Operação Offline para Leitura e Gestão:** Todas as notas já transcritas, fotos originais, pastas, buscas, edições manuais de Markdown e marcações de checkboxes devem funcionar 100% offline, sem necessidade de conexão com a internet.
* **RNF05 — Performance de UI (60/120 FPS):** As transições de tela, alternância entre abas *Markdown/Original Scan* e rotação entre o scanner Retrato e Horizontal devem ocorrer sem travamentos da thread principal de UI (uso de `Isolates` ou chamadas assíncronas nativas para compressão e I/O de disco).
* **RNF06 — Fidelidade ao Design System:** A interface deve respeitar rigorosamente a paleta **Obsidian Note Engine** (`#131316`, `#F3F7A2`, `#BCE3EC`, `#D6C7FB`), tipografia (*Space Grotesk* e *JetBrains Mono*) e micro-interações táteis definidas no Google Stitch.

---

## 3. Regras de Negócio (RN)

### RN01 — Disparo Tardio da Transcrição (Economia de Quota)
A requisição para a API do Gemini **não** deve ser disparada no instante do clique no botão obturador da câmera. Ela só é executada quando o usuário:
1. Confirma **"Create New Note"** no modal pós-captura; ou
2. Confirma **"Append Transcription"** na tela `Select Destination Note`.
* **Motivação:** Evita consumo desnecessário da quota diária caso o usuário decida descartar a foto (`Retake`) e permite injetar o contexto da nota destino no prompt quando a opção for *Append*.

### RN02 — Processamento Unificado em Modo Batch
Quando o usuário captura múltiplas imagens no modo `Batch` (de 2 a 5 fotos), o sistema envia **todas as imagens em uma única requisição multimodal** ao Gemini, instruindo o modelo a sintetizar o conteúdo em sequência lógica em um único documento Markdown.
* **Contabilização:** Conta como **1 chamada de API** em `api_usage_logs`, mas incrementa o contador de **"Pages Converted"** no Dashboard pelo número exato de fotos enviadas no lote (ex: +4 páginas convertidas).

### RN03 — Semeadura Inicial de Pastas (Database Seed)
Na primeira inicialização do aplicativo (banco de dados recém-criado), o sistema deve pré-cadastrar automaticamente as **5 pastas padrão do Google Stitch**:
1. `Personal notes` (Cor: Lavanda `#D6C7FB`, Ícone: `vpn_key`)
2. `Work & sprints` (Cor: Branco/Menta `#FFFFFF`, Ícone: `precision_manufacturing`)
3. `Whiteboard snaps` (Cor: Azul Névoa `#BCE3EC`, Ícone: `draw`)
4. `Lecture transcriptions` (Cor: Amarelo Manteiga `#F3F7A2`, Ícone: `menu_book`)
5. `Research papers` (Cor: Coral `#FF9F80`, Ícone: `movie`)

### RN04 — Auto-Classificação Inteligente de Pasta
Ao criar uma nova nota via *"Create New Note"*, o prompt enviado ao Gemini inclui a lista de pastas atuais do usuário (`id` e `name`). O Gemini devolve no JSON o `suggested_folder_id` que melhor se encaixa no tema da nota. Caso o ID retornado não exista ou seja nulo, a nota é associada à pasta `Personal notes` como fallback seguro.

### RN05 — Regras de Anexação Contextual (Append Mode)
Ao anexar uma captura a uma nota existente:
1. O serviço lê o título e os últimos 500 caracteres do arquivo `.md` da nota destino e os envia como contexto ao Gemini.
2. O novo conteúdo transcrito é concatenado ao final do arquivo `.md` existente precedido por um separador horizontal (`\n\n---\n\n`) e comentário de timestamp.
3. As novas imagens capturadas são salvas e vinculadas àquela nota na tabela `note_images` com `order_index` incremental.
4. Os novos `action_items` e `tags` extraídos são mesclados aos existentes da nota sem duplicação.

### RN06 — Sincronização Bidirecional de Action Items
Quando o usuário marca ou desmarca um checkbox na seção *Detected Action Items* da tela `Transcribed Markdown Note`:
1. O campo `is_completed` na tabela `action_items` do SQLite é atualizado imediatamente.
2. O arquivo físico `.md` correspondente tem a linha da tarefa atualizada de `- [ ] Tarefa` para `- [x] Tarefa` (ou vice-versa), garantindo que ao exportar o `.md` o status das tarefas esteja preservado.

### RN07 — Exclusão Segura de Pastas e Notas
* **Exclusão de Nota:** Remove o registro em `notes` (e em cascata `action_items` e `note_images` no SQLite) **e** apaga do sistema de arquivos o arquivo `.md` e as imagens `.jpg` vinculadas para não deixar arquivos órfãos ocupando espaço no celular.
* **Exclusão de Pasta:** Caso o usuário exclua uma pasta que contenha notas, o sistema deve perguntar se deseja **mover as notas para `Personal notes`** (padrão recomendado) ou excluir tudo.

### RN08 — Cálculo de Quota Diária e Reset UTC
* O limite diário padrão para cálculo percentual na tela de Settings é de **1.500 requisições/dia** (referência do Free Tier do Gemini Flash), podendo ser alterado pelo usuário nas configurações.
* O contador diário contabiliza os registros em `api_usage_logs` cujo `timestamp` pertença ao dia corrente em **UTC** (a partir das `00:00:00 UTC`).
* Se uma requisição falhar por erro de rede ou credencial inválida (status != 200), ela não é contabilizada como página convertida no Dashboard.
