# Word Document Integration (`GetWordDocContents`)

Use the Windows tray app + Ember commands to pull the active Word document into Quickfire (Business Details editor, Epic Sync, SmartPaste helpers).

## Architecture
| Component | Location | Notes |
| --- | --- | --- |
| Tray command router | `src/Quickfire.Tray/Methods/WordControl.cs` | Handles `GetWordDocContents`, logs to `%LOCALAPPDATA%\Surefire\TrayLog.txt` |
| System tray | `Quickfire.Tray/System/SystemTray.cs` | Registers available commands, forwards responses via SignalR (EmberHub) |
| Ember hub | `src/Quickfire.Blazor/Hubs/EmberHub.cs` | Hosts SignalR endpoints for tray ↔ web app communication |
| Client components | `Domain/Clients/Components/BusinessDetailsEditor.razor.cs`, `Domain/Utilities/Components/EpicClientPolicySync.razor.cs` | Register handlers, send commands |

## Request flow
1. Web client calls `EmberService.RunEmberFunction("GetWordDocContents", new List<string>())`
2. EmberHub forwards the command to the connected tray app for that user
3. Tray executes `WordControl.GetWordDocContents`
   - Verifies Word is running and a document is active
   - Reads the entire document body as plain text
   - Handles COM errors and returns friendly error strings (`ERROR: ...`)
4. Tray sends results back via `SystemTray.SendEmberResponse`
5. Web client handler receives the text and processes it (SmartPaste, Business Details parsing, etc.)

## Client-side snippet
```csharp
protected override async Task OnInitializedAsync()
{
    EmberService.RegisterResponseHandler("GetWordDocContents", HandleWordResponse);
}

private async Task HandleWordResponse(List<string> response)
{
    var text = response.FirstOrDefault();
    if (text?.StartsWith("ERROR:") == true)
    {
        // show toast, log, etc.
        return;
    }

    await SmartPasteFromWordAsync(text);
}

private Task RequestWordAsync()
    => EmberService.RunEmberFunction("GetWordDocContents", new List<string>());
```

## Tray-side highlights
```csharp
case "GetWordDocContents":
    SystemControl.Log("[WordControl] Routing to GetWordDocContents");
    GetWordDocContents(parameters);
```
- Handles error scenarios: Word not running, no active doc, COM failures, empty doc
- Returns `ERROR:` prefixed strings so the web client can branch easily

## Troubleshooting
- Check `%LOCALAPPDATA%\Surefire\TrayLog.txt` for `[WordControl]` entries
- Ensure the tray app is connected (status icon lit) and EmberHub shows the user connected
- Word must be the active application with a document open; macros are not required for text extraction
- Firewall/SignalR issues will show errors in browser console + tray logs

## Extending commands
Follow the same pattern when adding more Word commands:
1. Create a method in `WordControl.cs`
2. Register it in the switch statement
3. Document expected parameters + responses here
4. Add client handlers in the relevant component(s)

This integration keeps Word → Quickfire flows local and fast without writing files to disk.
