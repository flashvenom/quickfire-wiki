# Quickfire Wiki

Quickfire is an insurance AMS built for independent P&C teams that expect desktop-grade speed, strong AI copilots, and deep integrations with the tools they already use. This wiki mirrors the GitHub Wiki format (Home + _Sidebar + section folders) so it can be pushed directly into the `quickfire` public repo.

![Surefire MVP Progress](https://files.flashvenom.com/surefireflyer.jpg)

## Why Quickfire
- **Realtime 360° book management** – clients, renewals, forms, accounting, calls, mail, and documents share the same Blazor surface and state service
- **OpenAI + Semantic Kernel copilot** – intent detection, SQL-backed insights, SmartPaste, Proposler data shaping, and navigation agents all share one stack
- **Desktop & tray support** – MAUI shell, Windows tray controller, and gateway webhooks keep phones, Outlook, and DocuSign in sync
- **Plugin lockbox** – API credentials, paylinks, carrier logins, and agent toggles live in the DB so ops can manage them without redeploys

## Jump In
- [[Getting-Started]] – prerequisites, environment variables, and the quick `dotnet` loop
- [[System-Architecture]] – solution layout, runtime modes, and how the desktop, gateway, and tray apps coordinate
- [[Release-Notes]] – what shipped in v1.0.1-alpha and prior builds (kept in lockstep with `README.md`)
- [[features/Homepage|Feature pages]] – walkthroughs for Homepage, Clients, Attachments, Forms, Integrations, Interface, Proposals, and Renewals
- [[agents/OpenAIPro-and-Agent-Stack|Agent stack]] – OpenAIAgent + OpenAIPro design, prompts, task agents, and roadmap
- [[reference/Binding-Events|Reference]] – UI binding recipes, dropdown/style guides, product types, renewal data share, and desktop build steps

## Quick Links
- Feature Deep Dives: [[features/Homepage|Homepage]], [[features/Clients|Clients]], [[features/Renewals-and-Submissions|Renewals]], [[features/Proposals-and-Proposler|Proposler]], [[features/Integrations|Integrations]]
- AI & Automations: [[agents/OpenAIPro-and-Agent-Stack|OpenAIPro]], [[agents/Agent-Prompts|Prompt library]], [[agents/Database-Debug-Guide|Database debug toolkit]], [[agents/Roadmap|Roadmap]]
- Reference & Guides: [[reference/Binding-Events|Binding events]], [[reference/Dropdowns|Dropdowns]], [[reference/Feature-Map|Feature map]], [[reference/Product-Types|Product types]], [[reference/Renewal-Data-Share|Renewal data share]], [[guides/Desktop-Builds|Desktop builds]]

## Need Help Fast?
- **Ctrl/Cmd+K** in GitHub wiki jumps between pages
- Each page cites the owning files (e.g., `src/Quickfire.Blazor/Domain/Clients/Pages/Clients.razor`) so you can open code beside the doc
- Issues/ideas? See [[agents/Roadmap|Roadmap & backlog]] before opening a ticket so we avoid re-work

_Last updated: 2025-11-14_
