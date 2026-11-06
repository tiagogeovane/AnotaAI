# AnotaAI — Design System Técnico para Flutter (Obsidian Note Engine)

Este documento traduz o design system **Obsidian Note Engine** (extraído do projeto no Google Stitch) diretamente para tokens, temas e especificações de widgets reutilizáveis em **Flutter**.

---

## 1. Filosofia Estética

O **Obsidian Note Engine** combina dois pilares visuais:
1. **Dark Minimalist Utilitarianism:** Fundos em preto-obsidiana profundo (`#131316`) com empilhamento tonal sutil e zero sombras pesadas em cartões escuros, reduzindo a fadiga visual durante estudos ou reuniões.
2. **Playful Tactile Modernism (Metáfora de Papel Pastel):** Cartões de ação, resumos de IA e selos de destaque usam cores sólidas inspiradas em papelaria física (post-its amarelo manteiga, cartões de índice azul névoa e marcadores lavanda), sempre combinados com texto escuro de alto contraste (`#131316`).

---

## 2. Paleta de Cores e Tokens (`AppColors`)

Arquivo alvo: `lib/core/constants/app_colors.dart`

### 2.1 Superfícies Escuras (Tonal Stacking)
| Token no Stitch | Constante Flutter (`AppColors`) | Hexadecimal | Uso Principal |
| :--- | :--- | :--- | :--- |
| `surface` / `background` | `surface` | `#131316` | Fundo principal (Scaffold background) |
| `surface-container-lowest` | `surfaceContainerLowest` | `#0E0E11` | Fundo do viewfinder, Dynamic Island e caixas de código |
| `surface-container-low` | `surfaceContainerLow` | `#1B1B1E` | Cartões secundários, seções de Settings e listas agrupadas |
| `surface-container` | `surfaceContainer` | `#1F1F22` | Cards de notas recentes, blocos de fórmulas e inputs |
| `surface-container-high` | `surfaceContainerHigh` | `#2A2A2D` | Chips ativos, botões secundários, hover/tap state |
| `surface-container-highest`| `surfaceContainerHighest` | `#353438` | Barras inativas de gráficos, checkboxes desmarcados, divisores |
| `surface-bright` | `surfaceBright` | `#39393C` | Toasts flutuantes e estados pressionados |

### 2.2 Superfícies Pastéis Táteis (Destaques de Papelaria)
| Token no Stitch | Constante Flutter (`AppColors`) | Hexadecimal | Cor do Texto Sobreposto | Uso Principal |
| :--- | :--- | :--- | :--- | :--- |
| `primary-fixed` / `primary-container` | `butterYellow` | `#E4E895` / `#F3F7A2` | `#131316` (`onPrimaryFixed`) | Botão de Scan, Shutter da câmera, Hero de Síntese da Nota, Checkboxes marcados e barra de "Hoje" |
| `secondary-fixed` | `mistBlue` | `#C2E9F2` / `#BCE3EC` | `#001F25` (`onSecondaryFixed`) | Banner de boas-vindas (*"Good morning!"*), badges de status e pasta Whiteboard |
| `secondary-container` | `mistBlueDarkContainer` | `#294E56` | `#99BFC8` (`onSecondaryContainer`) | Callout box de *"Key Takeaway"* e indicadores *Ready* |
| `tertiary-fixed` | `softLavender` | `#E8DDFF` / `#D6C7FB` | `#1F143D` (`onTertiaryFixed`) | Colunas de histórico semanal, pasta *Personal notes* e tags de IA |
| `accent-coral` / `error` | `coralMelon` | `#FF9F80` / `#FFB4AB` | `#690005` (`onError`) | Alertas, pasta *Research papers* e erros de validação |
| `accent-mint` | `mintGreen` | `#A7F3D0` | `#0C353D` | Confirmações de sincronização e latência baixa |

### 2.3 Tipografia, Bordas e Ícones
| Token no Stitch | Constante Flutter (`AppColors`) | Hexadecimal | Uso Principal |
| :--- | :--- | :--- | :--- |
| `primary` | `textPrimary` | `#FFFFFF` | Títulos principais e ícones ativos |
| `on-surface` | `textOnSurface` | `#E4E1E6` | Corpo de texto padrão do Markdown |
| `on-surface-variant` | `textSecondary` | `#C9C7B6` | Subtítulos, descrições e abas inativas |
| `outline` | `textMuted` | `#929181` | Timestamps (`2m ago`), contadores secundários e placeholders |
| `outline-variant` | `borderSubtle` | `#47483A` | Bordas finas de 1px (usar com `withOpacity(0.35)`) |

---

## 3. Sistema Tipográfico (`AppTypography`)

Arquivo alvo: `lib/core/theme/app_theme.dart` (via pacote `google_fonts`).

* **Família Display & Corpo (`Space Grotesk`):** Transmite modernidade geométrica e excelente legibilidade em telas pequenas.
* **Família Técnica & Metadados (`JetBrains Mono`):** Transmite precisão de OCR, engenharia, contadores de tokens, fórmulas e tags.

| Estilo no Stitch | Fonte | Tamanho (`sp`/`px`) | Peso (`FontWeight`) | Altura de Linha (`height`) | Tracking (`letterSpacing`) | Aplicação |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `headline-xl` | Space Grotesk | `40px` | `w600` (SemiBold) | `1.20` (`48px`) | `-1.2px` (`-0.03em`) | Métrica principal do gráfico (`42/50`) |
| `headline-xl-mobile`| Space Grotesk | `30px` | `w600` (SemiBold) | `1.20` (`36px`) | `-0.6px` (`-0.02em`) | Saudação Hero (*"Good morning!"*) |
| `headline-lg` | Space Grotesk | `26px` | `w600` (SemiBold) | `1.23` (`32px`) | `-0.5px` (`-0.02em`) | Títulos de modais e seções amplas |
| `headline-md` | Space Grotesk | `20px` | `w500` (Medium) | `1.30` (`26px`) | `-0.2px` (`-0.01em`) | Denominadores de métricas e subtítulos |
| `headline-sm` | Space Grotesk | `17px` | `w500` (Medium) | `1.29` (`22px`) | `-0.17px` (`-0.01em`)| Títulos da AppBar, nomes de pastas e notas |
| `body-lg` | Space Grotesk | `16px` | `w400` (Regular) | `1.50` (`24px`) | `0.0px` | Parágrafos destacados |
| `body-md` | Space Grotesk | `14px` | `w400` (Regular) | `1.43` (`20px`) | `0.0px` | Texto padrão das notas Markdown e checklists |
| `body-sm` | Space Grotesk | `12px` | `w400` (Regular) | `1.50` (`18px`) | `0.0px` | Descrições curtas de cartões e legendas |
| `label-lg` | JetBrains Mono| `13px` | `w500` (Medium) | `1.38` (`18px`) | `+0.26px` (`0.02em`) | Equações em destaque (`ΔU = Q - W`) |
| `label-md` | JetBrains Mono| `11px` | `w500` (Medium) | `1.45` (`16px`) | `+0.44px` (`0.04em`) | Badges de acurácia (`98%`), botões `Copy`/`.md` |
| `label-sm` | JetBrains Mono| `10px` | `w400` (Regular) | `1.40` (`14px`) | `+0.60px` (`0.06em`) | Tags (`#thermo`), timestamps e rótulos `LIVE` |

---

## 4. Geometria, Espaçamento e Efeitos (`AppDimens`)

### 4.1 Raios de Borda (`BorderRadius`)
* `radiusSm`: `8.0` (`0.5rem`) — Checkboxes, barras de gráfico, badges internos.
* `radiusDefault`: `16.0` (`1.0rem`) — Cards de notas em listas, inputs de formulário, botões quadrados do dock.
* `radiusMd`: `24.0` (`1.5rem`) — Cards de configurações e containers de visualização.
* `radiusLg`: `32.0` (`2.0rem`) — Hero cards pastéis, gráfico semanal e moldura do viewfinder.
* `radiusFull`: `9999.0` — Pílulas de filtro, botões de ação rápida, abas segmentadas e obturador.

### 4.2 Espaçamentos Padrão
* `marginMobile`: `20.0` (`1.25rem`) — Padding horizontal padrão das telas.
* `spaceXs`: `4.0` (`0.25rem`)
* `spaceSm`: `8.0` (`0.5rem`)
* `spaceMd`: `16.0` (`1.0rem`)
* `spaceLg`: `24.0` (`1.5rem`)
* `spaceXl`: `32.0` (`2.0rem`)

### 4.3 Efeitos de Brilho e Glassmorphism
* **TopBar & BottomNav Glassmorphism:** `BackdropFilter` com `ImageFilter.blur(sigmaX: 16, sigmaY: 16)` + cor `#131316` com opacidade `0.88`.
* **Glow do Obturador (`shadow-glow-yellow`):**
  ```dart
  BoxShadow(
    color: Color(0xFFF3F7A2).withOpacity(0.35),
    blurRadius: 25,
    spreadRadius: 0,
  )
  ```
* **Glow dos Cantos do Scanner (`shadow-glow-bracket`):**
  ```dart
  BoxShadow(
    color: Color(0xFFF3F7A2).withOpacity(0.55),
    blurRadius: 14,
    spreadRadius: 0,
  )
  ```

---

## 5. Catálogo de Componentes Customizados (Widgets Flutter)

Para manter consistência visual absoluta entre todas as telas, os seguintes widgets base devem ser construídos em `lib/presentation/widgets/`:

### 5.1 `ObsidianTopBar`
* **Uso:** AppBar fixa translúcida em todas as telas.
* **Variantes:**
  - `TabShell`: Exibe a pílula superior estilo *Dynamic Island* (com ponto pulsante azul e ícone `auto_awesome`), logo do AnotaAI e título da aba.
  - `StackShell`: Exibe botão circular `arrow_back_ios_new`, logo, título da tela e ações opcionais à direita (ex: botão `Portrait` no scanner horizontal).

### 5.2 `TactileScaleWrapper`
* **Uso:** Reproduz o script de micro-interação do Stitch (`transform: scale(0.985)` no `pointerdown`).
* **Implementação:** `GestureDetector` + `AnimatedScale(scale: _isPressed ? 0.98 : 1.0, duration: Duration(milliseconds: 100))` envolvendo cartões e botões interativos.

### 5.3 `PastelFlashcard`
* **Uso:** Hero de boas-vindas (`Home`), botão *"Scan your note"*, e card de síntese da nota (`Transcribed Markdown Note`).
* **Propriedades:** `backgroundColor` (`butterYellow` ou `mistBlue`), `showGraphWatermark` (desenha via `CustomPainter` a grade tracejada de papel milimetrado no canto inferior direito com `opacity: 0.10`), `child`.

### 5.4 `ScannerReticleOverlay`
* **Uso:** Moldura inteligente sobre o preview da câmera tanto no modo Retrato (`3:4`) quanto no modo Paisagem (`Widescreen`).
* **Implementação:** Desenha os 4 cantos luminosos amarelos (`border-t-2 border-l-2` com `shadow-glow-bracket`), a mira central em cruz (`center-crosshair`) e a grade opcional (`4x4` ou `3x3`).

### 5.5 `FormulaCalloutCard`
* **Uso:** Renderizador customizado de blocos matemáticos na tela `Transcribed Markdown Note`.
* **Visual:** Container `#1F1F22` (`surfaceContainer`), ícone `functions` à esquerda, equação renderizada via `flutter_math_fork` em fonte monospaçada/LaTeX clara e badge opcional de unidade/referência à direita (`Joule [J]`, `Fourier '22`).

### 5.6 `FloatingBottomNav`
* **Uso:** Barra de navegação inferior nas telas principais (`Home`, `Folders`, `Settings`) com o botão flutuante central elevado (`-top-3`, `56x56px`) que dispara diretamente o `Live Camera Scanner`.
