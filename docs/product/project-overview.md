# AnotaAI — Visão Geral do Projeto

> **Digitalização inteligente de anotações físicas para Markdown com IA local.**

---

## 1. O que é o AnotaAI?

O **AnotaAI** é um aplicativo mobile focado em produtividade para estudantes, engenheiros, pesquisadores e profissionais do conhecimento. O aplicativo permite que o usuário fotografe quadros brancos (whiteboards), cadernos, post-its ou anotações em papel e transforme esse conteúdo manuscrito/visual em documentos estruturados em **Markdown** com alta fidelidade.

O aplicativo opera no modelo **100% Local-First** utilizando a chave de API do Google Gemini do próprio usuário (BYOK - *Bring Your Own Key*), garantindo privacidade total, custo zero de infraestrutura e sem qualquer dependência de backend centralizado.

---

## 2. Proposta de Valor

- **Zero Latência de Backend Próprio:** Comunicação direta do celular com a API Gemini do Google.
- **Privacidade Absoluta:** As fotos e notas nunca passam por servidores de terceiros; a chave de API e os dados ficam armazenados exclusivamente no dispositivo de forma criptografada.
- **Fidelidade Multimodal:** Reconhecimento não apenas de texto simples, mas de:
  - Fórmulas matemáticas (renderizadas em LaTeX).
  - Diagramas e fluxogramas (convertidos em sintaxe Mermaid JS).
  - Tarefas e listas de pendências (checkboxes interativos de action items).
  - Destaques estruturais (títulos, bullet points e takeaways).
- **Visualização Dual:** Capacidade de alternar a qualquer momento entre o Markdown sintetizado pela IA e a foto original em alta resolução.

---

## 3. Telas e Fluxos do Usuário (Baseado no Google Stitch)

> 📖 **Consulte a pasta `docs/ui-ux/` para os documentos detalhados de [Mapeamento de Telas e Fluxos](file:///home/tiago/Projects/AnotaAI/docs/ui-ux/screens-and-flows.md) e [Storyboard Textual do Usuário](file:///home/tiago/Projects/AnotaAI/docs/ui-ux/user-storyboard.md).**

O design foi projetado sob a estética **Obsidian Note Engine** no Stitch:

```
┌──────────────────────────────────────────────────────────┐
│                     Home Dashboard                       │
│  - Saudação & métricas diárias (ex: 4 notas hoje)        │
│  - Quick Action tátil: "Scan your note"                  │
│  - Gráfico semanal de páginas convertidas (Live)         │
│  - Lista de scans recentes com badge de acurácia         │
└────────────────────────────┬─────────────────────────────┘
                             │ (Toque no botão + ou scan)
                             ▼
┌──────────────────────────────────────────────────────────┐
│              Live Camera Scanner (Retrato & Paisagem)    │
│  - Modo Retrato: Viewfinder 3:4 com HUD & Anti-reflexo   │
│  - Modo Paisagem: Widescreen + Dock Lateral Ergonômico   │
│  - Controles: Flash, Filtro B&W, Grid e Modo Batch       │
└────────────────────────────┬─────────────────────────────┘
                             │ (Foto capturada)
                             ▼
┌──────────────────────────────────────────────────────────┐
│            Modal: Save Note Destination                  │
│  - Miniatura da foto capturada com status "Ready"        │
│  - Opção A: Create New Note (Nova nota .md)              │
│  - Opção B: Add to Existing Note (Anexar a existente)    │
└────────────────────────────┬─────────────────────────────┘
                             │
            ┌────────────────┴────────────────┐
            ▼                                 ▼
┌───────────────────────────┐   ┌───────────────────────────┐
│ Transcribed Markdown Note │   │  Select Destination Note  │
│ - Abas: Markdown vs Foto  │   │ - Busca & Filtros         │
│ - Key Takeaway callout    │   │ - Acordeão de pastas      │
│ - Seções estruturadas     │   │ - Confirmação de append   │
│ - Fórmulas LaTeX          │   │ - Append inline ou seção  │
│ - Checkboxes interativos  │   └─────────────┬─────────────┘
│ - Copiar & Exportar .md   │                 │
└───────────────────────────┘                 ▼
                                ┌───────────────────────────┐
                                │ Nota Destino Atualizada   │
                                └───────────────────────────┘
```

### Outras Telas de Suporte:
1. **Folders & Categories (`/folders`):**
   - Gestão de pastas de cadernos (*Personal notes*, *Work & sprints*, *Whiteboard snaps*, *Lecture transcriptions*, etc.).
   - Chips de filtragem de categoria e criação de novas pastas.
2. **AI & Model Settings (`/settings`):**
   - Configuração da Gemini API Key com armazenamento criptografado local.
   - Endpoint / Base URL customizável (para proxies ou enterprise).
   - Teste de conexão e indicador de latência em tempo real.
   - Gauge circular de consumo de quota diária (ex: 42/1.500 requisições) e histórico de 7 dias.

---

## 4. Identidade Visual & Design System

- **Tema:** Dark Minimalist Utilitarianism + Tactile Paper Modernism.
- **Cores Principais:**
  - Base / Canvas: `#131316` (Obsidian Deep Dark).
  - Superfícies Elevadas: `#1B1B1E`, `#1F1F22`, `#2A2A2D`.
  - Amarelo Manteiga (Destaque Principal): `#F3F7A2` (para CTAs, status concluído e cards primários).
  - Azul Névoa: `#BCE3EC` (para resumos, boas-vindas e status).
  - Lavanda Suave: `#D6C7FB` (para tags de IA e gráficos).
  - Coral Melão: `#FF9F80` (para marcações urgentes).
- **Tipografia:**
  - **Space Grotesk:** Títulos, cabeçalhos e corpo de texto das notas.
  - **JetBrains Mono:** Metadados, tags, contadores, latência, LaTeX e status de OCR.
