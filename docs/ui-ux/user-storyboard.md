# AnotaAI — Storyboard Textual da Jornada do Usuário

Este documento apresenta o **Storyboard Textual** do aplicativo **AnotaAI**, narrando quadro a quadro (Cena por Cena) a experiência real do usuário interagindo com todas as telas projetadas no Google Stitch.

---

## 👤 Persona Principal

* **Nome:** Lucas (23 anos)
* **Perfil:** Estudante de Engenharia de Computação e Desenvolvedor Backend Júnior.
* **Contexto:** Assiste aulas presenciais repletas de fórmulas matemáticas na lousa e participa de reuniões de arquitetura/sprint com post-its e diagramas desenhados em quadros brancos. Usa **Obsidian** no computador e quer ter todas as anotações físicas organizadas em Markdown sem precisar redigitar nada.

---

## 🎬 Ato 1: Configuração Inicial (BYOK — Bring Your Own Key)

### Quadro 1.1 — O Primeiro Acesso e Configuração do Motor Gemini
* **Cenário:** Lucas acabou de instalar o AnotaAI no celular antes do início da aula de Termodinâmica.
* **Tela:** `AI & Model Settings`
* **Ação do Usuário:**
  1. Lucas toca na aba **Settings** (ícone de engrenagem) na barra inferior.
  2. No card *"Gemini Vision & OCR Engine"*, ele cola sua chave gratuita gerada no Google AI Studio (`AIzaSyD-...`) no campo **Gemini API Key**.
  3. Ele toca no ícone de olho (`visibility`) rapidamente para conferir se colou todos os caracteres e lê o aviso tranquilizador: *"Stored client-side in encrypted local storage. Zero telemetry"*.
  4. Toca no botão amarelo manteiga **"Verify & Save Credentials"**.
* **Resposta do Sistema:**
  - O aplicativo faz um ping leve na API do Gemini.
  - Exibe `Latency: xx ms` com um ponto pulsante verde/azul e muda o badge superior para **Ready**.
    - Mostra pro usuario a latencia e se a conexão aconteceu de maneira correta.
  - O card inferior *"API Quota & Usage"* mostra `0% USED (0 / 1,500)` com reset agendado para `00:00 UTC`.

---

## 🎬 Ato 2: Capturando uma Aula no Quadro Negro (Nova Nota)

### Quadro 2.1 — Visão do Dashboard Matinal
* **Cenário:** O professor acabou de encher o quadro com as equações da Primeira Lei da Termodinâmica e da Lei de Fourier. Antes que o professor apague a lousa, Lucas pega o celular.
* **Tela:** `Home Dashboard`
* **Visualização:**
  - O topo saúda Lucas com o card Azul Névoa: *"Good morning! X notes digitized today"*.
  - O gráfico de atividade semanal mostra seu progresso (`XX/XX pages digitized`).
* **Ação do Usuário:**
  - Lucas toca no grande card tátil Amarelo Manteiga **"Scan your note (Whiteboard or index card)"**.

### Quadro 2.2 — Enquadramento e Rotação: Scanner Horizontal (Landscape) para Lousa Panorâmica
* **Cenário:** O professor utilizou toda a extensão da lousa da sala, que é muito larga horizontalmente. Na orientação vertical tradicional, as extremidades das equações seriam cortadas.
* **Tela:** `Live Camera Scanner - Horizontal (Landscape)` (ID: `ce87f21f9b264234bd3a586ed02132a6`)
* **Ação do Usuário:**
  1. Ao abrir o scanner inicialmente em modo retrato, Lucas percebe a amplitude do quadro e gira o smartphone 90° para a posição horizontal.
  2. O AnotaAI reorganiza a interface instantaneamente: a área central se expande em um **Viewfinder Widescreen**, e os botões se concentram no **Dock Lateral Direito (`CameraControlsDock`)**, ao alcance natural do polegar direito.
  3. No viewfinder panorâmico, os quatro cantos angulares amarelos luminosos com brilho neon (`shadow-glow-bracket`) rastreiam os quatro cantos da lousa.
  4. Lucas ativa o **Grid 3x3** (`grid_on`) para alinhar a base do quadro e nota que o filtro de alto contraste (`filter_b_and_w`) realça perfeitamente o giz branco e amarelo sobre a ardósia escura.
  5. Segurando o aparelho com as duas mãos para máxima estabilidade, ele dispara o grande **Botão Obturador Amarelo (CAPTURE)** com o polegar direito no dock lateral.
* **Resposta do Sistema:**
  - O anel pulsante externo se contrai (`active:scale-90`), o flash de captura ilumina o visor por 150ms e o frame widescreen é congelado e comprimido com precisão.

### Quadro 2.3 — Decisão de Destino Pós-Captura
* **Cenário:** Um segundo após o disparo, um modal desliza suavemente de baixo para cima sobre a câmera.
* **Tela:** `Camera Capture - Save Note Destination (Modal)`
* **Visualização:**
  - Lucas vê a miniatura nítida da foto com um selo `Ready`.
  - Caso a foto tivesse saído tremida, ele poderia tocar em `Retake` no rodapé.
* **Ação do Usuário:**
  - Como é o início de um novo tema da matéria, Lucas toca na primeira opção destacada com o selo *Recommended*: **"Create New Note (Transcribe with AI into a fresh markdown note)"**.

---

## 🎬 Ato 3: Revisão, Estudo e Exportação da Nota Transcrita

### Quadro 3.1 — Leitura do Markdown Sintetizado pela IA
* **Cenário:** O Gemini 2.5 Flash processa a imagem em ~3 segundos e abre automaticamente a nota estruturada.
* **Tela:** `Transcribed Markdown Note` (Aba *Synthesized Markdown* ativa)
* **Visualização e Interação:**
  1. No topo, o card hero amarelo mostra o título gerado automaticamente: **"Physics Lecture: Thermodynamics"** e o resumo do que a IA detectou (*4 seções principais, 1 bloco de fórmula e 2 itens de ação*).
  2. Logo abaixo, uma caixa azul destaca o **Key Takeaway**: *"Heat transfer rate is strictly proportional to the negative temperature gradient..."*.
  3. Lucas rola a página e vê as equações perfeitamente formatadas em blocos matemáticos: $\Delta U = Q - W$ com a tag `Joule [J]` e $q = -k \cdot (dT / dx)$.
  4. Na seção `03 Detected Action Items`, a IA converteu os avisos anotados no canto do quadro pelo professor em dois **checkboxes interativos**:
     - `[x] Re-derive conduction equation with radial cylindrical coordinates.` (Lucas toca no checkbox e ele fica marcado em amarelo com texto riscado!).
     - `[ ] Submit Problem Set #3 thermodynamics derivations before Friday 17:00.`

### Quadro 3.2 — Conferência com a Foto Original (Original Scan)
* **Cenário:** Lucas fica em dúvida se o professor escreveu um sinal de negativo na fórmula de condução ou se era apenas um risco de giz.
* **Tela:** `Transcribed Markdown Note` (Alternando para a aba *Original Scan*)
* **Ação do Usuário:**
  1. No controlador segmentado superior, Lucas toca na aba **"Original Scan"**.
  2. A tela alterna instantaneamente para a fotografia original de alta resolução (`4032 × 3024 px`) com correção de perspectiva aplicada (`Perspective Keystone Corrected`).
  3. Lucas confirma visualmente que era mesmo um sinal de menos (`-k`), volta para a aba **Synthesized Markdown** e toca no botão **".md"** (ou **Copy**) para exportar a nota para seu vault do Obsidian.

---

## 🎬 Ato 4: Anexando Post-its de Trabalho a uma Nota Existente (Fluxo de Append)

### Quadro 4.1 — Captura Rápida no Trabalho
* **Cenário:** À tarde, no estágio, a equipe de Lucas termina uma reunião de retrospectiva da Sprint 24 cheia de post-its colados na parede. Ele quer anexar essas novas decisões ao documento da Sprint que ele já havia criado ontem no app.
* **Tela:** `Home Dashboard` $\rightarrow$ `Live Camera Scanner` $\rightarrow$ `Save Destination Modal`
* **Ação do Usuário:**
  1. Lucas abre o app, toca diretamente no botão flutuante central **`+`** na barra de navegação inferior e tira a foto dos post-its.
  2. No modal que sobe após a foto, em vez de criar uma nova nota, ele toca na segunda opção: **"Add to Existing Note (Append transcription to an existing page)"**.

### Quadro 4.2 — Escolhendo a Nota Destino na Árvore de Pastas
* **Cenário:** Lucas precisa escolher em qual nota existente os novos 348 vocábulos extraídos serão inseridos.
* **Tela:** `Select Destination Note`
* **Visualização:**
  - O banner no topo lembra qual captura está pendente: miniatura da foto + *"Sprint Whiteboard · 348 words parsed · Append mode"*.
* **Ação do Usuário:**
  1. Na seção **Notebook Folders**, a pasta **Work & sprints** já aparece expandida no acordeão mostrando suas 3 notas mais recentes.
  2. Lucas toca na primeira nota da lista: **"Sprint 24 Retro & Action Items"** (que informa *"Ends at: Section 4 · Tasks"*).
  3. O seletor circular (`radio-indicator`) daquela nota fica amarelo com um ícone de `check`, e o rodapé fixo atualiza o alvo: `Target: Sprint 24 Retro & Action Items`.
  4. Lucas pressiona o botão principal amarelo no rodapé: **"Append Transcription"**.
* **Resposta do Sistema:**
  - O botão exibe um spinner (`Appending 348 words...`), um toast confirma no topo (*"Appended into Sprint 24 Retro & Action Items!"*) e o app abre a nota completa já com a nova seção mesclada ao final do arquivo `.md`.

---

## 🎬 Ato 5: Organização de Pastas e Auditoria de Consumo

### Quadro 5.1 — Organizando os Cadernos por Categoria
* **Cenário:** No fim do dia, no ônibus voltando para casa, Lucas quer organizar suas pastas de estudo e criar uma pasta dedicada para seu TCC.
* **Tela:** `Folders & Categories`
* **Ação do Usuário:**
  1. Lucas toca na aba **Folders** na barra inferior.
  2. Ele visualiza todos os seus diretórios organizados com ícones temáticos e contadores (*Personal notes: 16 notes*, *Work & sprints: 21 notes*, *Whiteboard snaps: 48 notes*, *Lecture transcriptions: 4 notes*, *Research papers: 8 notes*).
  3. Ele toca no botão flutuante amarelo no canto inferior direito (`create_new_folder`) para criar um novo caderno para suas referências bibliográficas.

### Quadro 5.2 — Conferindo o Consumo da Quota Gratuita
* **Cenário:** Curioso para saber quantas chamadas à API do Gemini ele gastou ao longo da semana intensa de provas e sprints, Lucas vai até as configurações.
* **Tela:** `AI & Model Settings`
* **Visualização:**
  - No card **API Quota & Usage**, o gráfico circular mostra que ele utilizou apenas **3%** da sua cota diária gratuita (`42 / 1,500`, restando `1,458` requisições hoje).
  - No gráfico de barras **7-Day Usage History**, ele vê seu ritmo ao longo dos dias da semana (`Mon: 35`, `Tue: 68`, `Wed: 92`, `Thu: 54`, `Fri: 79`, `Sat: 48`, `Today: 42`, totalizando `418 calls`), com custo zero e todas as notas salvas localmente em seu dispositivo.

---

## 📊 Matriz Resumo: Intenção do Usuário $\times$ Tela Acionada

| Momento da Jornada | Intenção do Usuário | Tela do Stitch Acionada | Resultado Esperado |
| :--- | :--- | :--- | :--- |
| **Setup Inicial** | Configurar chave própria do Gemini e checar saúde da API | `AI & Model Settings` | Chave salva em storage criptografado e latência validada |
| **Abertura Diária** | Ver resumo do dia e acionar câmera com 1 toque | `Home Dashboard` | Acesso imediato ao scanner ou às últimas notas |
| **Captura Física (Vertical)** | Fotografar folhas A4, cadernos ou post-its individuais | `Live Camera Scanner (Retrato)` | Imagem 3:4 com cantos delimitados e filtros rápidos |
| **Captura Física (Panorâmica)** | Fotografar lousas largas e quadros brancos compridos | `Live Camera Scanner - Horizontal` | Viewfinder widescreen e disparo ergonômico no dock lateral direito |
| **Triagem Rápida** | Decidir entre criar arquivo novo ou continuar existente | `Save Destination Modal` | Roteamento para criação direta ou seletor de nota |
| **Continuidade** | Juntar foto da segunda parte da aula na mesma nota | `Select Destination Note` | Transcrição anexada ao final do `.md` escolhido |
| **Estudo e Uso** | Ler resumo, fórmulas LaTeX, ticar tarefas e exportar | `Transcribed Markdown Note` | Estudo produtivo, conferência com foto raw e exportação |
| **Arquivamento** | Separar notas da faculdade, trabalho e projetos pessoais | `Folders & Categories` | Repositório limpo, pesquisável e categorizado |
