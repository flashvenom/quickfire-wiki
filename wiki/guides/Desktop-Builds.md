# Desktop Builds (MAUI + Local Host)

Quickfire ships with a MAUI WebView shell (`src/Quickfire.Desktop`) plus a Windows Tray helper (`src/Quickfire.Tray`). The MAUI shell bootstraps a self-hosted publish of the Blazor app so producers can run Quickfire locally without IIS.

## Components
| Piece | Location | Purpose |
| --- | --- | --- |
| Blazor host publish | `src/Quickfire.Blazor/Surefire.csproj` → `Build/Desktop/Surefire/` | Self-contained Kestrel host the MAUI app launches |
| Publish target | `Surefire.csproj` target `PublishDesktopRuntime` | Copies files into `$(DesktopWebPublishDir)` when `SurefireMigrationsFlavor=Local` |
| MAUI project | `src/Quickfire.Desktop/Quickfire.csproj` | WebView UI, setup wizard, Surefire host bootstrapper |
| Host service | `src/Quickfire.Desktop/Services/SurefireHostService.cs` | Extracts publish to `%LOCALAPPDATA%\Surefire`, hashes manifests, starts host, waits for readiness |
| Manifest target | `Quickfire.csproj` target `PrepareSurefireHostPackage` | Harvests published files + builds `manifest.txt` |
| Tray app | `src/Quickfire.Tray` | Handles Word/Outlook/Windows automation + notifications |

## Build / run flow
1. **Publish the host**
   ```bash
   dotnet publish src/Quickfire.Blazor/Surefire.csproj \
       -c Debug \
       -p:SurefireMigrationsFlavor=Local
   ```
   - Outputs to `Build/Desktop/Surefire/`
   - Uses SQLite migrations (`Data/MigrationsLocal`)

2. **Prepare MAUI assets**
   ```bash
   cd src/Quickfire.Desktop
   dotnet msbuild Quickfire.csproj \
       /t:PrepareSurefireHostPackage \
       /p:TargetFramework=net9.0-windows10.0.19041.0 \
       /p:Configuration=Debug
   ```
   - Copies publish folder + manifest into `obj/.../SurefireHost`

3. **Run MAUI shell**
   ```bash
   dotnet run --project Quickfire.csproj \
       -f net9.0-windows10.0.19041.0 \
       -c Debug
   ```
   - `SurefireHostService` unpacks host → starts `Surefire.exe` → polls `http://127.0.0.1:<port>/healthz`
   - WebView navigates to BaseUrl once healthy

4. **Start Tray app** (optional but recommended)
   ```bash
   dotnet run --project ../Quickfire.Tray/Surefire.Tray.csproj
   ```

## Visual Studio tips
- Install the **.NET Multi-platform App UI** workload
- Configure the solution to use `Local` build flavor (sets `SurefireMigrationsFlavor=Local`)
- Set **Quickfire.Desktop** as the startup project to run publish + package + MAUI launch automatically
- If MAUI launches before the publish finishes, rerun the publish command manually then restart MAUI

## Troubleshooting
| Symptom | Fix |
| --- | --- |
| Blank WebView | Check `%LOCALAPPDATA%\Surefire\logs` and ensure `Surefire.exe` is running; verify publish folder exists |
| Host not found | Confirm `Build/Desktop/Surefire/` is populated; rerun publish target |
| Ports blocked | Update `appsettings.maui.json` (`SurefireHostOptions`) to pick a different port |
| Tray commands not working | Ensure Tray app is running and connected (see icon). Check `TrayLog.txt` for `SendEmberResponse` entries |
| Desktop mode features disabled | Desktop runtime auto-disables Blastmail, Gateway, TaskAgents (see `Program.cs` feature flags). Override via env vars only if you know what you''re doing |

## Customization
- Change publish destination via `-p:DesktopWebPublishDir="C:\SomePath"`
- Update manifest logic inside `PrepareSurefireHostPackage` if you bundle extra folders
- Adjust MAUI ready-check path/timeout inside `SurefireHostService`
- Desktop-specific settings (data directory, logging) live in `appsettings.maui.json`

Use this doc any time you tweak the desktop bootstrapper so the GitHub wiki and installer docs stay current.
