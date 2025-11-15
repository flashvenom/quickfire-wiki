# Renewals & Submissions

Renewals orchestrate everything from renewal tasks to external client updates, submissions, and accounting. They are the glue between policies, proposals, DocuSign, and attachments.

## Overview
- **Board view** identifies every expiring policy with status, bill type, expiration, premium, and assigned owners
- **Task Masters** seed task lists with offsets relative to expiration dates; teams can tweak tasks per renewal
- **Submissions grid** groups quotes by wholesaler/carrier, tracks status/premiums, and stores underwriter contacts
- **External portal** lets insureds update info securely; updates write back via `RenewalUpdateService`
- **Accounting & settlements** integrate with the settlement screen, paylink builder, and DocuSign to keep billing in sync

## Code map
| Area | Files |
| --- | --- |
| Pages | `src/Quickfire.Blazor/Domain/Renewals/Pages/Index.razor`, `Details.razor`, `Edit.razor` |
| Submissions | `Domain/Renewals/Components/Submissions.razor (+ code-behind)` + `SubmissionService.cs`
| Tasks | `Domain/Renewals/Components/TaskBoard.razor`, `TaskService.cs`, Master Task admin pages
| External updates | `Domain/Integrations/ExternalPortal/*`, `RenewalUpdateService.cs`, `Quickfire.Gateway` receivers
| Status sync | `Reference/Renewal-Data-Share` rules implemented in `ProposalDetails`, `Submissions`, and `StateService`
| Accounting hand-off | `Domain/Shared/Components/SettlementScreen.razor`, `Domain/Accounting/Services/AccountingService.cs`

## Automation highlights
- **Auto-create renewals** from policies via `RenewalService.CreateRenewalFromPolicy`
- **Goal dates** and `TrackTask` checklist items auto-adjust when expiration dates shift
- **Proposal sent** automatically marks the leading submission as Proposed and checks "Proposal" tasks off (see [[reference/Renewal-Data-Share]])
- **Submission bound** updates renewal status + accounting tasks
- **External portal** writes `RenewalNote` entries with "System" user attribution for auditable changes

## Configuration tips
- Task templates live in DB; adjust via the master task admin page
- External portal uses API keys + short-lived tokens defined in `ExternalPortalService`
- Accounting integration toggles live in system settings (agency wants Epic upload first? toggle the flag in Profile → Agency)

Related docs: [[features/Proposals-and-Proposler]] for how proposals drive submissions, [[reference/Renewal-Data-Share]] for automation rules, and [[features/Files-and-Attachments]] for dealing with renewal documents.
