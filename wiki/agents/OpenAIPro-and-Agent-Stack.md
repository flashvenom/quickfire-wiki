# OpenAIPro & Agent Stack

Quickfire ships with two complementary AI layers:

1. **OpenAIAgent + Semantic Kernel** – the conversational / navigation / SQL / task brain that powers Enhanced AI Chat and SmartPaste buttons on feature pages.
2. **OpenAIPro** – a lower-level HTTP client optimized for long-running, strict-schema automations (Proposler, SmartPaste intake, attachment enrichment).

## OpenAIAgent (Semantic Kernel)
- **Files**: `src/Quickfire.Blazor/Domain/Agents/Services/OpenAIAgent.cs`, `NavigationAgent.cs`, `ParameterExtractionService.cs`, `TaskAgents/*`
- **Models**: `Domain/Agents/Models/UnifiedRequest`, `UnifiedResponse`, `IntentClassificationResult`
- **Key dependencies**: `Microsoft.SemanticKernel`, `QdrantVectorStore`, `OpenAIEmbeddingService`, `NavigationAgent`
- **Capabilities**:
  - Intent classification (GeneralAI, DatabaseQuery, Navigation, TaskAgent, SmartAction)
  - SQL generation with schema loading (`wwwroot/schema/extractor.json`) and validation (blocks DDL/DML)
  - Vector-powered navigation by resolving client/policy names via embeddings
  - Streaming status updates (UI shows progress spinner; streaming output wiring ready via `ProcessRequestStreamingAsync`)
  - Task agent orchestration (`ITaskAgent` implementations live in `Domain/Agents/TaskAgents`)

### Request flow
1. `EnhancedAIChat.ChatHandlers.SendMessage` builds a `UnifiedRequest` (input, sessionId, context)
2. `OpenAIAgent.ProcessRequest{Streaming}` validates input, appends to chat history, classifies intent (unless `manualIntent` supplied)
3. Intent-specific handler executes (database query via EF, navigation via `NavigationAgent`, TaskAgent execution, general AI completion)
4. Response serialized back to the page, which renders Markdown/HTML and triggers navigation or toast events

### Configuration
- Models use `gpt-4.1` by default (centralized in `OpenAIAgent` ctor). Change there if needed.
- API keys resolved from `builder.Configuration["OpenAI:ApiKey"]` populated in `Program.cs` from `OPENAI` env var
- Embeddings use `text-embedding-3-small` via `OpenAIEmbeddingService`; vectors stored/fetched via `QdrantVectorStore`
- `NavigationAgent` also builds its own Semantic Kernel instance – keep model IDs in sync with `OpenAIAgent`

## OpenAIPro (Strict-schema automations)
- **File**: `src/Quickfire.Blazor/Domain/Shared/Services/OpenAIPro.cs`
- **Use cases**: SmartPaste, Proposler JSON cleanup, attachment tagging, PDF/Word data extraction
- **Features**:
  - Uses named HttpClient `"OpenAI"` with custom timeout and SSE support
  - Reads prompt templates (`wwwroot/prompts/*.txt`) and schema definitions (`wwwroot/schema/*.json`)
  - Streams progress through `StateService.UpdateStatus` so the UI can show user-friendly updates
  - Persists intermediary and final outputs as `Attachment` records via `AttachmentService`
  - Supports long-running tasks (up to ~230s timeout) and handles retry/backoff manually when needed

### Pattern
1. Collect attachments (JSON, TXT, PDF) and pass them to `OpenAIPro`
2. Build the full prompt from template + schema + file contents
3. Call OpenAI completions endpoint (model default `gpt-4.1` but overrideable per method)
4. Parse/validate JSON before saving as attachments or merging into domain models
5. Update status + log events through `ILoggingService`

## Task Agent Framework
- **Docs**: `src/Quickfire.Blazor/Domain/Agents/README.md`
- **Interfaces**: `ITaskAgent`, `ITaskAgentRegistry`, `IParameterExtractionService`
- **Usage**:
  - Agents expose metadata (triggers, required parameters, outcome type) and implement `ExecuteAsync`
  - Parameter extraction uses AI prompts defined per parameter with optional entity extraction (client, carrier, policy type)
  - Agents can be invoked from Enhanced AI Chat or via action buttons on renewal/client pages

## Voice & Tray tie-ins
- `Domain/Agents/Pages/EnhancedAIChat.Voice.cs` wires microphone capture through `wwwroot/js/enhanced-ai-chat.js`
- `TranscriptionService` sends audio to OpenAI Whisper endpoints; responses feed right back into the chat pipeline
- Tray + Word helpers (`Quickfire.Tray/Methods/WordControl.cs`) capture structured data for AI prompts (e.g., Word doc contents for Business Details editors)

## Debugging & Telemetry
- Increase logging via `appsettings.Development.json` (e.g., set `Surefire.Domain.Agents.Services.OpenAIAgent` to `Debug`)
- Database query debugging is covered in [[agents/Database-Debug-Guide]]
- To test agent health, use the `/agents/enhanced/chat` page and watch the status ring in the layout; errors bubble through `_statusbar`

## When to use which API
| Scenario | Use this |
| --- | --- |
| Conversational chat, SQL answers, nav, parameter extraction | `OpenAIAgent` |
| Strict JSON transformation, long-running multi-file prompts, attachment persistence | `OpenAIPro` |
| Simple "complete this text" or legacy prompt | `OpenAISimpleService` (wrapper around HttpClient, useful for lightweight calls)

Whenever you add new automations, document the prompt templates under `/wwwroot/prompts`, cite them here, and keep sample payloads in `/ref` for quick regression checks.
