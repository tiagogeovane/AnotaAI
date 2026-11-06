# AnotaAI 📝✨

> Aplicativo mobile 100% local para transcrição inteligente de anotações físicas e quadros brancos em Markdown estruturado com Google Gemini (BYOK).

---

## 📚 Documentação Completa do Projeto (`docs/`)

A documentação está organizada em 4 pastas temáticas:

### 📦 1. Produto & Negócio (`docs/product/`)
- **[Visão Geral do Projeto](file:///home/tiago/Projects/AnotaAI/docs/product/project-overview.md):** Proposta de valor, resumo da identidade visual, casos de uso e filosofia Local-First.
- **[Requisitos e Regras de Negócio](file:///home/tiago/Projects/AnotaAI/docs/product/requirements-and-rules.md):** Requisitos Funcionais (`RF01–RF18`), Não-Funcionais (`RNF01–RNF06`) e Regras de Negócio (`RN01–RN08`).

### 🏛️ 2. Arquitetura & Engenharia (`docs/architecture/`)
- **[Arquitetura e Modelagem Técnica](file:///home/tiago/Projects/AnotaAI/docs/architecture/architecture.md):** Padrão em camadas, Provider/ChangeNotifier, schema SQLite (`anota_ai.db`), sistema de arquivos e árvore de diretórios Flutter.
- **[Especificação de IA e Engenharia de Prompts](file:///home/tiago/Projects/AnotaAI/docs/architecture/ai-integration-spec.md):** Contrato `AiVisionProvider` (Strategy Pattern), integração REST com Gemini 2.5 Flash, prompts de *Create* e *Append*, JSON Schema e matriz de resiliência.

### 🎨 3. UI/UX & Design System (`docs/ui-ux/`)
- **[Mapeamento de Telas e Fluxos (Google Stitch)](file:///home/tiago/Projects/AnotaAI/docs/ui-ux/screens-and-flows.md):** Descrição minuciosa de cada tela projetada no Stitch (incluindo câmera Retrato e Horizontal), elementos visuais e diagrama de navegação.
- **[Design System Técnico (Obsidian Note Engine)](file:///home/tiago/Projects/AnotaAI/docs/ui-ux/design-system.md):** Tradução completa de cores, tipografia (*Space Grotesk* e *JetBrains Mono*), espaçamentos, glows e catálogo de widgets para Flutter.
- **[Storyboard Textual do Usuário](file:///home/tiago/Projects/AnotaAI/docs/ui-ux/user-storyboard.md):** Narrativa cena a cena (5 atos) da jornada real do usuário utilizando o aplicativo.

### 🗺️ 4. Planejamento & Testes (`docs/planning/`)
- **[Roadmap de Implementação e Testes](file:///home/tiago/Projects/AnotaAI/docs/planning/implementation-roadmap.md):** Plano de execução incremental em 7 Fases (Milestones) com critérios de pronto (*Definition of Done*).

---

## 🛠️ Stack Tecnológica

- **Framework:** Flutter (Dart)
- **Gerenciamento de Estado:** Provider (`ChangeNotifier`)
- **Persistência Local:** Híbrida — SQLite (`sqflite`) + Arquivos locais (`.md` / `.jpg` via `path_provider`)
- **Segurança de Credenciais:** `flutter_secure_storage` (Keystore / Keychain)
- **Motor de IA:** Google Gemini 2.5 Flash (REST Multimodal + Structured JSON)
- **Design System:** Obsidian Note Engine (Dark `#131316`, Destaques Pastéis `#F3F7A2`, *Space Grotesk* e *JetBrains Mono*)
