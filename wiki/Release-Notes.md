# Release-Notes

Release history mirrors the root `README.md` so the wiki stays current when it becomes the GitHub wiki repo. Dates use ISO format for easy linking.

## v1.0.1-alpha (2025-01-24)
- Massive merge from personal branch with .NET 9 upgrade, solution restructure, and SQLite desktop default
- **Accounting:** new invoicing/settlement screen + checklist enhancements, DocuSign + payment signals expose status everywhere
- **Global Profile:** agency-level system settings now stored in DB along with plugin toggles
- **SmartPaste:** email footer + blob pasting for Leads and Clients with AI field extraction
- **Attachments:** support Azure Blob/local/NAS, hashed storage, fast thumbnails, and Epic document sync
- **Leads:** prioritized pipeline, quick AI extraction to prefill forms
- **Desktop Outlook:** advanced search from inside Quickfire, triggered via tray helpers and Ember commands
- **RingCentral:** incoming-call toast, click-to-dial, call log feed, call recording download & transcription
- **Click Helpers:** trigger links for tel/mailto/outlook, ctrl-click copy shortcuts everywhere
- **Renewals:** submissions tab auto-prompts for incumbent submissions, request/update flows improved
- **AI Stack:** Semantic Kernel powered Enhanced AI Chat, SmartPaste, Proposler pipeline, pluginized `OpenAIPro`
- **SyncFusion 27.1.56**, improved FireSearch, logging, SmartPaste component, plugin system seed, state service overhaul

## v0.1.0 (2024-10-17)
- Leads management UI, RingCentral webhook prototype, CRUD hardening across core entities
- Forms tab renamed + expanded with PDF library + JSON save, credential vault for carrier logins
- Homepage layout polish, saved filters for renewals and task ordering improvements

## v0.0.4 (2024-09-08)
- Base URL/env var config cleanup, UI/UX polishing, contact avatar uploads, primary contact logic fixes

## v0.0.3 (2024-09-05)
- Renewal filter state persistence, enhanced renewal tasks, keyboard-driven fast search, file org cleanup

## v0.0.2 (2024-08-28)
- Client browsing improvements, GL/WC/Auto coverage detail screens, certificate editor, endorsement attachments

## v0.0.1 (2024-08-20)
- Initial release: Clients, Carriers, Contacts, Addresses, Policies, Renewal Center, Task Templates, Identity auth

Next milestones + backlog live on [[agents/Roadmap|Roadmap]].
