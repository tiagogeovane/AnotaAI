# AnotaAI — Especificação de Integração de IA & Engenharia de Prompts

Este documento detalha a arquitetura de comunicação direta (client-side) com modelos de Inteligência Artificial Multimodal, o contrato da interface extensível (`AiVisionProvider`), os schemas de saída estruturada e os prompts do **Google Gemini 2.5 Flash**.

---

## 1. Arquitetura Multi-Provider (Strategy Pattern)

Embora o **Google Gemini 2.5 Flash** seja o motor oficial e completo do MVP (aproveitando sua janela de contexto, velocidade em OCR multimodal e tier gratuito generoso), o Google Stitch prevê telas de configuração para *OpenAI*, *Anthropic* e *Other Provider*.

Para refletir isso com código limpo, a camada de domínio define o contrato abstrato `AiVisionProvider`:

```dart
enum AiProviderType { gemini, openai, anthropic, custom }

abstract class AiVisionProvider {
  AiProviderType get providerType;

  /// Testa a validade da API Key e retorna a latência em milissegundos.
  Future<int> verifyCredentials({
    required String apiKey,
    required String baseUrl,
  });

  /// Transcreve de 1 a 5 imagens criando uma nova nota estruturada.
  Future<AiTranscriptionResult> transcribeNewNote({
    required List<Uint8List> compressedImages,
    required List<FolderSummary> availableFolders,
    required String apiKey,
    required String baseUrl,
  });

  /// Transcreve de 1 a 5 imagens para anexar ao final de uma nota existente.
  Future<AiAppendResult> transcribeForAppend({
    required List<Uint8List> compressedImages,
    required String existingNoteTitle,
    required String existingNoteTailContent,
    required String apiKey,
    required String baseUrl,
  });
}
```

### Status de Suporte por Provedor
| Provedor | Classe de Implementação | Modelo Padrão | Status no MVP |
| :--- | :--- | :--- | :--- |
| **Google Gemini** | `GeminiVisionProvider` | `gemini-2.5-flash` | **100% Implementado (Padrão)** |
| **OpenAI** | `OpenAiVisionProvider` | `gpt-4o-mini` | Contrato pronto / UI preparada |
| **Anthropic** | `AnthropicVisionProvider` | `claude-3-5-sonnet-latest` | Contrato pronto / UI preparada |
| **Other (Custom)** | `CustomOpenAiProvider` | Configurável pelo usuário | Contrato pronto / UI preparada |

---

## 2. Integração REST com Google Gemini (`gemini-2.5-flash`)

### 2.1 Endpoint Padrão
* **Base URL Padrão:** `https://generativelanguage.googleapis.com/v1beta`
* **Rota de Geração:** `POST {baseUrl}/models/gemini-2.5-flash:generateContent?key={apiKey}`
* **Headers:** `Content-Type: application/json`

### 2.2 Verificação de Credenciais e Medição de Latência (`verifyCredentials`)
Quando o usuário toca em **"Verify & Save Credentials"** na tela de Settings:
1. Inicia-se um `Stopwatch` em Dart.
2. Faz-se uma requisição `GET {baseUrl}/models/gemini-2.5-flash?key={apiKey}` (consome zero tokens de geração e valida instantaneamente a chave e o endpoint).
3. Para-se o `Stopwatch` ao receber HTTP `200 OK` e registra-se `stopwatch.elapsedMilliseconds` (ex: `28ms`).
4. Salva-se a chave via `flutter_secure_storage`.

---

## 3. Fluxo 1: Criar Nova Nota (`transcribeNewNote`)

### 3.1 System Instruction (Create Mode)
```text
You are AnotaAI, an expert multimodal OCR and academic/technical note synthesis engine.
Your role is to analyze 1 to 5 photographs of handwritten notebooks, blackboards, whiteboards, or sticky notes and convert them into a rich, well-structured Markdown document.

Follow these strict rules:
1. LANGUAGE: Always write the output (title, summary, key_takeaway, action_items, and markdown_content) in the SAME language used in the handwritten image(s).
2. ARTIFACT REMOVAL: Ignore camera glare, chalk dust, background objects, or crossed-out mistakes.
3. STRUCTURE: Organize `markdown_content` with clear hierarchical headings (`## 01 Title`, `## 02 Title`), bullet points, and bold key terms. Do NOT repeat the main document `# Title` at the top of `markdown_content` because the UI renders the title in a dedicated header card.
4. MATHEMATICS (LaTeX): Convert all mathematical equations, physical formulas, and chemical notations into clean LaTeX (`$inline$` or `$$block$$`).
5. DIAGRAMS (Mermaid JS): If the image contains flowcharts, architecture diagrams, sequence diagrams, or mind maps, represent them using valid ```mermaid code blocks inside `markdown_content`.
6. ACTION ITEMS: Extract any tasks, homework, deadlines, or next steps into the `action_items` array (plain text strings without `- [ ]` prefixes) AND also include them as a checklist section at the end of `markdown_content` using `- [ ] Task description`.
7. FOLDER CLASSIFICATION: Select the most appropriate `suggested_folder_id` from the user's provided folder list based on the subject matter.
```

### 3.2 User Prompt Dinâmico (Create Mode)
```text
Transcribe and synthesize the attached {{IMAGE_COUNT}} image(s) into a structured note.

Available User Folders for classification (pick the best matching `id` for `suggested_folder_id`):
{{FOLDERS_JSON_LIST}}
```
*(Exemplo de `{{FOLDERS_JSON_LIST}}`: `[{"id": "folder_lectures", "name": "Lecture transcriptions"}, {"id": "folder_work", "name": "Work & sprints"}, ...]`)*

### 3.3 Payload JSON e `responseSchema` (Create Mode)

```json
{
  "systemInstruction": {
    "parts": [{ "text": "<SYSTEM_INSTRUCTION_CREATE_MODE>" }]
  },
  "contents": [
    {
      "role": "user",
      "parts": [
        {
          "inlineData": {
            "mimeType": "image/jpeg",
            "data": "<BASE64_IMAGE_1>"
          }
        },
        {
          "text": "<USER_PROMPT_CREATE_MODE>"
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.2,
    "responseMimeType": "application/json",
    "responseSchema": {
      "type": "OBJECT",
      "required": [
        "title",
        "summary",
        "key_takeaway",
        "tags",
        "action_items",
        "markdown_content",
        "suggested_folder_id",
        "ocr_accuracy_estimate",
        "formulas_detected",
        "mermaid_detected"
      ],
      "properties": {
        "title": {
          "type": "STRING",
          "description": "Concise, descriptive title of the note (max 60 chars)."
        },
        "summary": {
          "type": "STRING",
          "description": "1-2 sentence structural summary (e.g. 'Blackboard diagram resolved into 4 core sections, 2 formulas, and 2 action items')."
        },
        "key_takeaway": {
          "type": "STRING",
          "description": "The single most important principle, conclusion, or insight from the note."
        },
        "tags": {
          "type": "ARRAY",
          "items": { "type": "STRING" },
          "description": "3 to 5 lowercase topic tags without the # symbol (e.g. ['thermo', 'fourier-law'])."
        },
        "action_items": {
          "type": "ARRAY",
          "items": { "type": "STRING" },
          "description": "List of actionable tasks or follow-ups detected in the scan."
        },
        "markdown_content": {
          "type": "STRING",
          "description": "Full synthesized Markdown body including sections, LaTeX formulas, Mermaid diagrams, and the `- [ ]` action items section."
        },
        "suggested_folder_id": {
          "type": "STRING",
          "description": "The exact ID of the best matching folder from the provided list."
        },
        "ocr_accuracy_estimate": {
          "type": "NUMBER",
          "description": "Estimated OCR confidence between 0.0 and 1.0 based on handwriting legibility and lighting."
        },
        "formulas_detected": {
          "type": "BOOLEAN",
          "description": "True if LaTeX formulas were generated."
        },
        "mermaid_detected": {
          "type": "BOOLEAN",
          "description": "True if Mermaid diagrams were generated."
        }
      }
    }
  }
}
```

---

## 4. Fluxo 2: Anexar a Nota Existente (`transcribeForAppend`)

Quando o usuário escolhe **"Add to Existing Note"** e confirma a nota destino em `Select Destination Note`:

### 4.1 System Instruction (Append Mode)
```text
You are AnotaAI, continuing an existing Markdown note with newly scanned handwritten/whiteboard material.
Analyze the attached image(s) and generate ONLY the new Markdown section(s) to be appended at the end of the existing note.

Follow these strict rules:
1. CONTINUITY: Read the provided existing note title and tail content so you continue section numbering and terminology seamlessly.
2. NO DUPLICATE HEADERS: Do NOT repeat the main document title. Start directly with the next logical section heading (`## ...`).
3. FORMATS: Use LaTeX (`$...$` / `$$...$$`) for math and ```mermaid blocks for diagrams.
4. ACTION ITEMS: Return newly discovered tasks in `new_action_items` and include them as `- [ ] Task` at the end of `appended_markdown`.
```

### 4.2 `responseSchema` (Append Mode)
```json
{
  "type": "OBJECT",
  "required": [
    "appended_markdown",
    "new_action_items",
    "new_tags",
    "updated_summary",
    "ocr_accuracy_estimate"
  ],
  "properties": {
    "appended_markdown": {
      "type": "STRING",
      "description": "The Markdown content to append to the end of the existing note."
    },
    "new_action_items": {
      "type": "ARRAY",
      "items": { "type": "STRING" }
    },
    "new_tags": {
      "type": "ARRAY",
      "items": { "type": "STRING" }
    },
    "updated_summary": {
      "type": "STRING",
      "description": "Updated 1-2 sentence summary reflecting the combined note."
    },
    "ocr_accuracy_estimate": {
      "type": "NUMBER"
    }
  }
}
```

---

## 5. Contabilização de Tokens e Auditoria Local

Toda resposta HTTP `200 OK` do Gemini retorna o bloco `usageMetadata`:
```json
"usageMetadata": {
  "promptTokenCount": 310,
  "candidatesTokenCount": 412,
  "totalTokenCount": 722
}
```
O `GeminiVisionProvider` extrai `promptTokenCount` e `candidatesTokenCount` e os grava na tabela `api_usage_logs` juntamente com a latência da chamada (`latency_ms`).
Esses dados alimentam:
1. O **bandeja expansível** do card pastel na tela `Transcribed Markdown Note` (*"Tokens parsed: 412"*).
2. O **Gauge Circular de Quota** e o **Gráfico de 7 Dias** na tela `AI & Model Settings`.

---

## 6. Matriz de Tratamento de Erros e Resiliência

| Cenário / Código HTTP | Causa Provável | Comportamento do App & Feedback na UI |
| :--- | :--- | :--- |
| **Sem API Key configurada** | Usuário tentou escanear antes de configurar a chave | Exibe modal amigável redirecionando para `AI & Model Settings`, mantendo a foto salva temporariamente para não perder a captura. |
| **HTTP 400 (Bad Request)** | Chave malformada ou payload inválido | Exibe alerta *"Chave de API ou requisição inválida. Verifique suas configurações."* |
| **HTTP 401 / 403** | API Key revogada, inválida ou sem permissão | Marca status do provedor como `Error` em Settings e solicita revisão da chave. |
| **HTTP 429 (Rate Limit)** | Limite de requisições por minuto (RPM) ou diário atingido | Realiza 1 retentativa automática após `3 segundos`; se persistir, informa *"Limite de velocidade da API atingido. Aguarde alguns segundos e tente novamente."* |
| **HTTP 500 / 503** | Instabilidade temporária nos servidores do Google | Retentativa automática com *Exponential Backoff* (`1s`, `2s`). |
| **Timeout / Sem Internet** | Celular sem sinal no momento da transcrição | Oferece opção **"Salvar Rascunho para Transcrever Depois"** ou **"Tentar Novamente"**, preservando a foto capturada localmente. |
| **Imagem Ilegível / Vazia** | Foto totalmente escura, borrada ou sem texto/diagrama | O modelo retorna `ocr_accuracy_estimate < 0.25`; o app exibe um aviso sugerindo *"Baixa legibilidade detectada — deseja refazer a foto (Retake)?"*. |
