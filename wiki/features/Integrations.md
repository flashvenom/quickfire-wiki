# Integrations

Quickfire centralizes integrations through a shared lockbox pattern (`IntegrationKeyValues` + typed providers). All configuration happens inside the app, so production teams can rotate keys without redeploying.

## Plugin Lockbox & Providers
- `Domain/Integrations/Services/IntegrationKeyValueService.cs` provides encrypted CRUD APIs
- Providers (e.g., `DocuSignSettingsProvider`, `RingCentralSettingsProvider`, `EPayPolicySettingsProvider`) cache values using `IMemoryCache`
- Admin UI lives under Profile → Integrations; saving values invalidates provider caches

## Telephony – RingCentral
- **Features**: call log feed, recordings, SMS conversations, incoming-call toast with client deep links, click-to-dial
- **Server**: `Domain/Chat/Services/ChatService.RingCentral.cs`, `Quickfire.Gateway/Controllers/SmsWebhookController.cs`
- **UI**: `Domain/Chat/Components/ChopperMessaging.razor`, `Domain/Clients/Components/ClientCallTranscriptions.razor`
- **Tray**: `Quickfire.Tray/System/SystemTray.cs` shows Windows toasts + deep links

## Payments – ePayPolicy
- **Features**: recent payment widget, instant paylink builder with policy prefill, logging for audits
- **Server**: `Domain/Integrations/ePayPolicy/EPayPolicyService.cs`, `Domain/Shared/Services/StateService.RefreshRecentPaymentsAsync`
- **UI**: `Domain/Home/Components/RecentPayments.razor`, `Domain/Shared/Components/PaylinkBuilder.razor`

## DocuSign
- **Features**: envelope history on Homepage + clients, send proposals/certs with dynamic reminders
- **Server**: `Domain/Integrations/DocuSign/DocuSignService.cs` + typed provider, `DocuSignEnvelopeRefreshWorker`
- **UI**: `Domain/Home/Components/DocuSignEnvelopeList.razor`, `Domain/Renewals/Components/DocuSignSendDialog.razor`

## Microsoft Graph / Outlook Web
- Pulls recent emails per client contact, supports search + multi-mailbox scenarios
- `Domain/Integrations/Graph/Services/GraphService.cs`, components under `Domain/Integrations/Graph/Components`

## Outlook Desktop Add-in + Tray
- `Quickfire.Tray` exposes Ember commands (Outlook search, Word text extraction, Windows notifications)
- `Quickfire.Desktop` hosts the Blazor app locally and opens the tray app automatically
- Ember messaging handled via `Surefire.Hubs.EmberHub` on the Blazor host

## External Renewal Portal
- `Domain/Integrations/ExternalPortal` + `ExternalPortalService` expose secure renewal update forms for clients
- Gateway receives portal submissions and writes back to renewals via `RenewalUpdateService`

## AI Stack
- **Semantic Kernel**: `OpenAIAgent` + `NavigationAgent` use OpenAI `gpt-4.1` and Qdrant vectors
- **OpenAIPro**: strict-schema flows for SmartPaste, Proposler, and other JSON-first automation lives in `Domain/Shared/Services/OpenAIPro.cs`
- **Voice**: `TranscriptionService` + `VoiceRecordingService` handle microphone input for Enhanced AI Chat

## Azure Form Recognizer & Document Intelligence
- Wired via `Domain/Integrations/FormRecognition` to augment PDF ingestion (Proposler + SmartPaste)

## Desktop & Gateway interplay
- Desktop MAUI host ensures offline/local deployments keep running even when IIS is unavailable
- Gateway handles inbound webhooks (RingCentral SMS/calls, Blastmail tracking) and relays them via SignalR to the main app

See [[integrations/Integration-Architecture]] for deeper technical patterns and [[System-Architecture]] for how each project communicates.
