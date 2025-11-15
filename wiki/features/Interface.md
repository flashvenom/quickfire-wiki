# Interface & Shell

The layout wraps Fluent UI shell components, FireSearch, Enhanced AI Chat, Chat/Chopper, and user/system settings helpers.

## FireSearch
- Top bar search hits clients, policies, carriers, contacts, addresses simultaneously
- `src/Quickfire.Blazor/Domain/Layout/_searchbar.razor` handles UI + throttle
- `Domain/Shared/Services/SearchService.cs` executes parallel EF queries and optional stored procedures (`dbo.SearchAllWithRenewals`)
- Keyboard shortcuts: `/` to focus, arrow keys to navigate, `Enter` to open, `Ctrl+Enter` to open in new tab

## Enhanced AI Chat (Aigent)
- Page: `Domain/Agents/Pages/EnhancedAIChat.razor` plus partials for chat handlers, voice, and models
- Services: `OpenAIAgent`, `NavigationAgent`, `EmbeddingLoaderService`, `TranscriptionService`
- Features: streaming status, SQL intent detection, navigation actions, SmartPaste prompts, voice dictation
- Safety: SQL validation (no DDL/DML), sanitized markdown → HTML via `ResponseFormatter`

## Chat & Messaging
- **Staff/Web/SMS chat** share SignalR hub `Domain/Chat/Hubs/MessagingHub.cs`
- `ChopperMessaging.razor` docks to the layout with unread counts and filters (staff vs SMS vs web chat)
- `ChatService` orchestrates RingCentral + internal chat + attachments

## Status + Settings
- `Domain/Layout/_statusbar.razor` listens to `StateService.UpdateStatus` for background job progress (AI, DocuSign sync, payments)
- Profile menu exposes per-user preferences (EasyMode, animation/audio toggles, homepage layout) stored on `ApplicationUser` via `UserPreferences`
- System settings UI at `Domain/Profile/Components/SystemSettings.razor` controls API keys, plugin switches, storage locations

## Accessiblity & UX standards
- Fluent UI + Syncfusion mix; follow [[reference/Binding-Events]] for binds/handlers to avoid double-updating state
- `wwwroot/css/app.css` + component-level `.razor.css` files hold theme tokens—prefer fluent variables over hard-coded colors
- Layout-level JS lives in `wwwroot/js/enhanced-ai-chat.js` for voice recording, auto-scroll, etc. Keep cross-component JS minimal

Cross-reference: [[features/Integrations]] (for Graph/RingCentral/DocuSign rendering here) and [[agents/OpenAIPro-and-Agent-Stack]] for the AI architecture powering Enhanced Chat.
