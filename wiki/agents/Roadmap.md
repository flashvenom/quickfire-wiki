# Roadmap

This page captures the high-level product plan so the public wiki stays aligned with internal `ref/ToDo*.md` trackers. Dates are indicative; priorities follow feedback from pilot agencies.

## Now → December 2025 (ALPHA polish)
- **Blastmail desktop mode** – ensure `/Domain/Blastmail/Pages/Outreach.razor` gracefully degrades when the background service is disabled; add inline warnings/limits
- **Login resiliency** – harden login/logout components so stale cookies are cleared automatically (`Domain/Account/*`)
- **Attachments** – streamline Epic JSON storage, simplify metadata, re-enable combined Surefire + Epic attachment browser
- **Accounting UX** – remove placeholder dashes, improve settlement checklist instructions, sticky checklist area, and add carrier payment proof upload to `SettlementScreen`
- **Renewal tasks refresh** – update Master Task templates (Request/Attach tasks for loss runs, better goal dates) and hook up "Request Update" + "Email Underwriter" agent buttons
- **AI UX** – add thumbs up/down feedback, improve streaming UX, and let AI tasks run as background operations so the UI stays responsive

## Next (Early 2026)
1. **Desktop Installer GA**
   - Finalize MAUI bootstrapper, ensure Quickfire.Tray auto-start, add config UI for data directories and logging
2. **External Renewal Portal V2**
   - Expand templates, add per-renewal API keys, expose update audit trail to insured portals
3. **Playbooks & Agents**
   - Publish more Task Agents (renewal update drafts, Epic push, follow-up emails) and embed Quick Actions in strategic screens
4. **Proposler Enhancements**
   - Upsell page injection, driver/vehicle table duplication, Named Driver warranty, better date formatting, up-sell suggestions
5. **Integrations**
   - Applied Epic upload toggles per agency, better paylink string templates, finishing the Leads/Blastmail flows

## Later
- **Multi-agency tenancy** – scoping for multi-agency hosting with isolated integrations + SSO options
- **Underwriting workbench** – manage appetite data + underwriter SLAs directly inside Renewals
- **Embedded analytics** – save AI-generated metrics/queries for dashboards and schedule them
- **Public API** – read-only API for clients/policies/renewals so agencies can connect BI tools

## References
- `ref/ToDo.md` – bug backlog and enhancement punch list
- `ref/ToDo-AI.md` – AI-specific tasks for Cursor/Copilot pairing
- `ref/plans/*.md` – deep-dive plans (Blastmail, ePayPolicy refactor, Renewal Settlements, etc.)

Please update this page whenever priorities shift so the GitHub wiki matches the internal backlog.
