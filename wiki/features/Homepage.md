# Homepage

The Homepage is the shared cockpit for CSRs, producers, and ops leads. It uses Fluent UI sections backed by the global `StateService` to show what to do next, who is waiting on you, and which automations are running.

## What users see
- **Task Deck** – drag-prioritize daily items, auto-seeded by renewals/task masters, filterable per user
- **Proposal Pipeline** – track Draft → Review → Sent statuses on renewals with CTA buttons (`Domain/Home/Components/ProposalPipeline.razor`)
- **Certificate Requests** – queue of open certs/forms waiting for processing, including doc links and contact shortcuts
- **DocuSign + Payments** – tiles showing recent envelopes (`Domain/Home/Components/DocuSignEnvelopeList.razor`) and ePayPolicy webhooks via `RecentPaymentsState`
- **Leads & Smart Signals** – inbound web leads, call shortcuts, and AI-flagged items (SmartPaste, pending AI responses)

## Key implementation points
| Concern | Where it lives |
| --- | --- |
| Layout + sections | `src/Quickfire.Blazor/Domain/Home/Pages/Home.razor` |
| User layout prefs | `Domain/Shared/Models/UserPreferences.cs` and `UserPreferences.HomepageLayout` JSON |
| Data loader | `Domain/Home/Services/HomeService.cs` for tasks/certs + `StateService` for shared caches |
| Widgets | `Domain/Home/Components/*.razor` partials (DocuSign, ProposalPipeline, RecentPayments, CertificateQueue) |
| Background refresh | `StateService.BeginBackgroundDataLoad()` kicks recurring refresh for payments, DocuSign, unread chat |

## Configuration tips
- Toggle Easy Mode per user (Profile → Preferences) to collapse advanced widgets for new hires
- Payments + DocuSign tiles depend on integration keys stored in `IntegrationKeyValues` → update via Profile → Integrations
- Desktop runtime disables heavy background workers; Homepage falls back to polling intervals set in `appsettings.Desktop.json`

## Extending the Homepage
1. Create a new component under `Domain/Home/Components`. Export parameters for data, skeleton states, and row actions.
2. Register data fetch logic in `HomeService` or a purpose-built state helper (similar to `RecentPaymentsState`).
3. Update `Home.razor` layout JSON reader/writer to include the widget (users can reorder/hide it).
4. Reference the component inside the proper layout cell; respect EasyMode by checking `UserPreferences.IsEasyMode`.

Cross-reference: [[features/Proposals-and-Proposler]] for pipeline details and [[features/Integrations]] for how payments/documents stay in sync.
