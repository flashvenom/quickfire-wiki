# System-Architecture

This overview stitches together the four projects that ship in the main solution and highlights the services/components you will see referenced throughout the wiki.

## Solution Layout
| Project | Purpose | Key folders |
| --- | --- | --- |
| `src/Quickfire.Blazor` (`Surefire.csproj`) | Blazor Server host, REST APIs, SignalR hubs, background workers | `/Domain/*` feature areas, `/Data` EF Core contexts/migrations, `/wwwroot` assets/prompts/schemas |
| `src/Quickfire.Desktop` (`Quickfire.csproj`) | .NET MAUI wrapper that boots the published Blazor host locally | `/Services/SurefireHostService.cs`, `/MainPage.xaml`, `/Resources` |
| `src/Quickfire.Gateway` | Lightweight ASP.NET Core API for external webhooks (RingCentral, SMS, Blastmail) | `/Controllers/*`, `/Services/*`, `/Hubs` |
| `src/Quickfire.Tray` | Windows tray utility for deep desktop integrtions (Outlook, Word, system commands) | `/System`, `/Methods`, `/Resources` |

## Runtime Modes
- **Server/Web** – Standard ASP.NET Core host, SQL Server by default. Enables background services (Blastmail, TaskAgents, Gateway).
- **Desktop** – Controlled via `SUREFIRE_DESKTOP_RUNTIME`/environment detection. Publishes into `Build/Desktop/Surefire/` and runs under MAUI WebView; some services (Blastmail, Gateway) are disabled for offline use.
- **Hybrid** – Desktop MAUI + Gateway service for telephony/webhooks (RingCentral) and Tray for OS-level tasks. `RuntimeFeatureFlags` (registered in `Program.cs`) toggle features per environment.

## Core Services
- **StateService** (`src/Quickfire.Blazor/Domain/Shared/Services/StateService.cs`) keeps long-lived caches (carriers, policies, DocuSign, payments) and surfaces status rings in the UI.
- **IntegrationKeyValueService** (`Domain/Integrations/Services/IntegrationKeyValueService.cs`) stores encrypted integration secrets. Providers (e.g., `DocuSignSettingsProvider`, `RingCentralSettingsProvider`) cache per-integration configs.
- **OpenAIAgent** + **NavigationAgent** (`Domain/Agents/Services`) run Semantic Kernel with `gpt-4.1`, Qdrant embeddings, streaming handlers, and database query safeguards.
- **OpenAIPro** (`Domain/Shared/Services/OpenAIPro.cs`) powers SmartPaste, Proposler extraction, and long-running JSON-first LLM tasks with `AttachmentService` persistence.
- **Task Agents Framework** (`Domain/Agents/TaskAgents/*`) orchestrates declarative business actions triggered from Enhanced AI Chat or action buttons.
- **AttachmentService** and **AttachmentUploaderApi** handle network drive/Azure/local storage with hashed file names + thumbnails.

## Data & Integrations
- EF Core context lives in `Data/ApplicationDbContext*.cs` with dual migrations: `MigrationsLocal` (SQLite) and `MigrationsRemote` (SQL Server).
- Integrations register in `Program.cs`: DocuSign, ePayPolicy, RingCentral, Azure Form Recognizer (`AddFormRecognitionIntegration`), Microsoft Graph (`AddGraphIntegration`), Embedding/Qdrant clients, and Outlook interop.
- `Quickfire.Gateway` exposes `/api/sms/webhook`, `/api/blastmail/track`, etc., forwarding to the Blazor host via SignalR hubs (`Surefire.Hubs` in the main app).
- `Quickfire.Tray` listens to Ember commands (SignalR) and executes Word/Outlook/Windows automations (see `Methods/WordControl.cs`, `System/SystemTray.cs`).

## Request Lifecycles
1. **UI** – Razor components (e.g., `Domain/Clients/Pages/Clients.razor`) use Fluent UI + Syncfusion controls wired with binding conventions in [[reference/Binding-Events]].
2. **Services** – Domain services (Clients, Renewals, Proposals, Accounting) live under `Domain/*/Services`. They depend on `ApplicationDbContext`, integration providers, and state caches.
3. **Background** – Hosted services live in `/Infrastructure` and `Program.cs` (e.g., `AttachmentWatcher`, `BlastmailWorker`). Desktop mode disables long-running workers via feature flags.
4. **Desktop Loop** – MAUI host starts, unpacks `Build/Desktop/Surefire`, boots `Surefire.exe`, and displays the WebView once `/` responds. Tray + Gateway bridge native OS events back into the Blazor UI.

## Observability
- Logging wired through `Serilog`? (if configured) and built-in `ILogger`. See `appsettings.Development.json` for agent-specific log levels (e.g., `Surefire.Domain.Agents.Services.OpenAIAgent`).
- `StateService.UpdateStatus("...", isBusy)` surfaces user-facing progress, notably for long AI runs.
- `Quickfire.Tray/System/SystemTray.cs` writes to `%LOCALAPPDATA%\Surefire\TrayLog.txt` for troubleshooting Word/Outlook commands.

Keep this page handy when updating docs: cite the relevant file paths from this map so engineers can jump from wiki to code immediately.
