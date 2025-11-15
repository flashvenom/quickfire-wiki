# Feature Map

Use this table for embeddings, navigation intents, and search hints. Keep it in sync with `Domain/*/Pages` so AI navigation stays accurate.

| Short Name | Route | Component | Notes |
| --- | --- | --- | --- |
| Home | `/` | `Domain/Home/Pages/Home.razor` | Widgets driven by `HomeService` + `StateService` |
| Clients | `/clients` | `Domain/Clients/Pages/Clients.razor` | Tabs: Summary, Policies, Renewals, Attachments, Notes |
| Client List | `/clients/list` | `Domain/Clients/Pages/Index.razor` | Grid for quick search/filter |
| Client Create | `/clients/create` | `Domain/Clients/Pages/Create.razor` | SmartPaste-enabled intake |
| Leads | `/leads` | `Domain/Leads/Pages/Leads.razor` | Pipeline view + SmartPaste toolbar |
| Renewals | `/renewals` | `Domain/Renewals/Pages/Index.razor` | Board view grouped by stage |
| Renewal Details | `/renewals/details/{id}` | `Domain/Renewals/Pages/Details.razor` | Tasks, submissions, proposals, accounting, portal |
| Proposals | `/renewals/details/{id}?tab=proposals` | `Domain/Proposals/Components/Proposals.razor` | Launch Proposler, DocuSign, attachments |
| Accounting | `/accounting/settlements/{id}` | `Domain/Shared/Components/SettlementScreen.razor` | Broker fee math, carrier payment upload |
| Integrations Settings | `/profile/integrations` | `Domain/Profile/Components/SystemSettings.razor` | Plugin lockbox + feature toggles |
| Agents (Enhanced Chat) | `/agents/enhanced/chat` | `Domain/Agents/Pages/EnhancedAIChat.razor` | Semantic Kernel chat UI |
| Task Agent Admin | `/agents/task-buttons` | `Domain/Agents/Pages/AgentButtonsSandbox.razor` | Designer + quick test harness |
| Attachments | `/attachments` | `Domain/Attachments/Components/AttachmentListGrid.razor` | File explorer for Surefire/Epic |
| Forms | `/forms` | `Domain/Forms/Pages/FormEditor.razor` | ACORD/certificate editor |
| Integrations Dashboard | `/integrations` | `Domain/Integrations/Pages/Index.razor` | Health + API logs (if enabled) |
| Blastmail | `/blastmail/outreach` | `Domain/Blastmail/Pages/Outreach.razor` | Campaign builder + send history |
| Desktop Settings | `/profile/desktop` | `Domain/Profile/Components/DesktopSettings.razor` | Data directories, tray options |

### Tips
- When you add or rename routes, update this table **and** `ref/schema/extractor.json` so AI navigation stays accurate.
- For intents, reference the `Short Name` column and include the route in the vector metadata.
