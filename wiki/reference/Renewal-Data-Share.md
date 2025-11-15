# Renewal Data Share Rules

Keep renewals, submissions, proposals, and accounting in sync by following these automation hooks. Whenever you touch the referenced files, update this doc.

## Status / Task automation
- **Renewal status dropdown** – `Domain/Renewals/Pages/Details.razor` renders the authoritative status list (`Pending`, `Renewed`, `Unneeded`, `Defected`, `Ghosted`, `Unplaceable`, `Other`). Changes log a `RenewalNote` (user = System when automated).
- **Proposal → Renewal** – in `Domain/Proposals/Components/ProposalDetails.razor` set the highest-ranked `Submission.Status` to `Proposed` when `Proposal.Status == Sent` and auto-complete the parent TrackTask named `Proposal`.
- **Submission → Renewal** – `Domain/Renewals/Components/Submissions.razor` sets `Renewal.Status = Renewed` when a submission switches to `Bound`. It also syncs carrier/wholesaler data back to the renewal header.
- **Renewed check** – if any task name contains `Renew in Epic` and is completed, force `Renewal.Status = Renewed` to keep dashboards accurate.

## Proposal data propagation
When `RunFullProposalProcess` or `SaveProposal*` execute, push these fields to the renewal + accounting layer:
- `Proposal.Premium`, `Proposal.Commission`, `Proposal.MinimumEarned`
- `Proposal.BillType`, `Proposal.IsFinanced`, `Proposal.DownPayment`, `Proposal.TotalCost`
- `Proposal.Carrier` (used for auto-selecting submissions + DocuSign templates)

## Submission updates
- `ShowPremiumEditDialog` writes `Submission.Premium` and updates mapped renewal totals
- Carrier/wholesaler edits should sync to renewal + proposal for consistent mail-merge data

## Accounting hand-off
`Domain/Shared/Components/SettlementScreen.razor` listens for AI or manual invoice uploads:
- `Settlement.Premium`, `Settlement.CommissionAmount`, `Settlement.BillType`, `Settlement.MinEarnedPercentage`
- `SettlementItem` with `ItemCode = RBF` represents broker fees – ensure SmartPaste + AI update this item when parsing invoices

## External update portal
- `Domain/Integrations/ExternalPortal` + `RenewalUpdateService` write `RenewalNote` entries, push updates into `Renewal` + `RenewalSubmission`
- Every change should set `RenewalNote.User = System` and describe the action (`External portal: Updated payroll`)

## Implementation checklist
1. Log everything via `RenewalNote` for auditing
2. Avoid duplicate writes—use `RenewalUpdateService` helpers where available
3. Keep naming consistent (`RenewalStatus`, `SubmissionStatus`, `ProposalStatus`)
4. Add unit/integration tests whenever automation toggles statuses or writes financial numbers

Following these conventions keeps dashboards, AI summaries, and accounting math accurate across the app.
