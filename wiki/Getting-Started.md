# Getting-Started

Use this guide when cloning the wiki into the `quickfire.wiki.git` repo or when onboarding a new machine. It mirrors the setup that `README.md` documents but adds the wiki-specific context.

## Prerequisites
- **.NET SDK 9.0.100+** with ASP.NET, EF Core, and MAUI workloads (`dotnet workload install maui blazor wasm-tools`)
- **Node/WebView2** – WebView2 runtime is required for the MAUI desktop shell
- **SQL Server LocalDB** *or* SQLite (set via `DEFAULTCONNECTION` or skip for local SQLite)
- **Syncfusion License** – register at syncfusion.com and keep the license string handy
- **OpenAI + Qdrant access** – `OPENAI` API key is required for OpenAIAgent/OpenAIPro, Qdrant URL/API key optional but recommended for embeddings
- **RingCentral / ePayPolicy / DocuSign credentials** (optional) entered later through the Integration Settings pages

## Environment Variables / .env
Create an `.env` next to `Surefire.sln`:

```
DEFAULTCONNECTION=Server=.;Database=Quickfire;Trusted_Connection=True;TrustServerCertificate=true;
OPENAI=sk-...
SYNCFUSION=YOUR-LICENSE
QDRANT_BASE_URL=http://localhost:6333/
QDRANT_API_KEY=local-dev
SUREFIRE_DATA_DIRECTORY=C:\SurefireData (optional desktop storage)
SUREFIRE_SEED_DEMO=true (optional demo data on first run)
```

> The app will fall back to SQLite (`Data/MigrationsLocal`) when `DEFAULTCONNECTION` is empty.

## Clone + Restore
```bash
git clone https://github.com/flashvenom/surefire.git
cd surefire
# Restore solution
dotnet restore Surefire.sln
```

## Database
```bash
# Pick the provider that matches your env var
# SQLite (Local):
dotnet ef database update --project src/Quickfire.Blazor/Surefire.csproj --context ApplicationDbContext -- --migrations "Data/MigrationsLocal"

# SQL Server (Remote/Desktop flavor):
dotnet ef database update --project src/Quickfire.Blazor/Surefire.csproj --context ApplicationDbContext -- --migrations "Data/MigrationsRemote"
```

> Exclude the migrations folder you are *not* using from the solution (or unload it in VS) to avoid designer confusion.

## Run the Blazor Server host
```bash
cd src/Quickfire.Blazor
dotnet watch run
```
- Seeds demo data if `SUREFIRE_SEED_DEMO=true`
- Registers integrations in `Program.cs` (DocuSign, RingCentral, ePayPolicy, Graph, Agent stack)

## Desktop Publish & MAUI shell (optional)
Run the publish + MAUI target when testing the desktop SKU:
```bash
# Publish Blazor host with desktop flavor
dotnet publish src/Quickfire.Blazor/Surefire.csproj -c Debug -p:SurefireMigrationsFlavor=Local
# Package into MAUI assets
cd src/Quickfire.Desktop
dotnet msbuild Quickfire.csproj /t:PrepareSurefireHostPackage /p:TargetFramework=net9.0-windows10.0.19041.0 /p:Configuration=Debug
# Launch MAUI shell
dotnet run --project Quickfire.csproj -f net9.0-windows10.0.19041.0 -c Debug
```
More detail: [[guides/Desktop-Builds]].

## Integration Secrets at Runtime
1. Navigate to **Profile → System Settings** (`src/Quickfire.Blazor/Domain/Profile/Components/SystemSettings.razor`)
2. Enter plugin lockbox values (OpenAI override, Syncfusion, local storage path, toggles)
3. Use the dedicated settings tabs for RingCentral, ePayPolicy, DocuSign, Graph, etc. Settings are stored in `IntegrationKeyValues` via `IntegrationKeyValueService`.

## Troubleshooting Checklist
- `OPENAI` missing → `OpenAIAgent` and `OpenAIPro` log "Missing API key" and show disabled UI
- Syncfusion license missing → `SyncfusionLicenseProvider.RegisterLicense` throws and UI components fail to render
- Desktop publish missing → Quickfire.Desktop warns that `Build/Desktop/Surefire` is empty
- Gateway/tray offline → webhooks (SMS, phone, tray commands) show warnings in `StateService.UpdateStatus`

With the basics in place, continue through the feature pages or jump into [[System-Architecture]] to understand how services wire together.
