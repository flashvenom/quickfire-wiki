# Clients

Clients is the 360° view for an account. It brings AI summaries, contact + policy context, attachments, call/email history, and SmartPaste-driven onboarding into a single hub.

## Daily workflows
- **AI summaries** – Client toolbar buttons call `Clients.AI.cs` helpers backed by `OpenAIAgent.ProcessRequestAsync` for "Client Info" and "Recent Activity" digests
- **SmartPaste intake** – SmartPaste buttons on create/edit (`Domain/Clients/Pages/Create.razor`) use OpenAIPro to parse email blobs into client/contact data
- **Call + email activity** – `Domain/Chat/Services/ChatService.RingCentral.cs` feeds `ClientCallTranscriptions.razor`; Microsoft Graph integration renders `RecentClientEmails.razor`
- **Policy + renewal tabs** – policies, submissions, invoices, and attachments are stacked per tab for quick drill-down without leaving the page
- **Attachments & forms** – drop zones and thumbnails leverage `AttachmentService`, hashed storage, and `AttachmentIcons.razor`

## Technical map
| Area | Files |
| --- | --- |
| Page + layout | `src/Quickfire.Blazor/Domain/Clients/Pages/Clients.razor (+ .cs, .AI.cs, .razor.css)` |
| Toolbar + SmartPaste | `Domain/Clients/Components/_toolbar.razor`, `BusinessDetail.razor`, SmartPaste button wiring to `OpenAIPro` |
| AI + integration context | `Domain/Clients/Pages/Clients.AI.cs`, `Domain/Agents/Services/OpenAIAgent.cs`, `Domain/Agents/TaskAgents` |
| Telephony + transcripts | `Domain/Chat/Services/ChatService.RingCentral.cs`, `Domain/Clients/Components/ClientCallTranscriptions.razor` |
| Email feed | `Domain/Integrations/Graph/Services/GraphService.cs`, `Domain/Integrations/Graph/Components/RecentClientEmails.razor` |
| Attachments | `Domain/Attachments/Components/*`, `Domain/Attachments/Services/AttachmentService.cs` |

## Configuration
- RingCentral + Graph credentials live in `IntegrationKeyValues` and can be updated in Profile → Integrations without redeploys
- AI features require `OPENAI` in env vars; Semantic Kernel + Qdrant settings configured in `Program.cs`
- Telephony recording download/transcription toggles live in `appsettings.*` under `RingCentral`

## Implementation tips
1. Extend AI prompts by editing `Domain/Clients/Pages/Clients.AI.cs` or by adding agent commands in `Domain/Agents/Services/OpenAIAgent.cs`
2. When adding new client widgets, wire `@bind` + callbacks per [[reference/Binding-Events]] so data stays in sync
3. Shared utilities (formatting, hashed file names, attachments) are under `Domain/Shared/Helpers/StringHelpers.cs` – use those helpers to keep client screens consistent

Also see: [[features/Renewals-and-Submissions]] for renewal data that surfaces on client tabs and [[reference/Word-Doc-Integration]] for capturing Word data straight into Business Details.
