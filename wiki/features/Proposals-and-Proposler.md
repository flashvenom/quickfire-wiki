# Proposals & Proposler

Proposals turn messy carrier quotes into branded deliverables. The Proposler pipeline combines AI parsing, Azure Document Intelligence, and Word automation.

## Workflow
1. **Collect quotes** – drag/drop PDFs into the renewal or proposal record; Quickfire hashes/stores them
2. **Select pages + context** – `ProposalDetails.razor` lets users pick the best pages, set billing terms, add broker fee math
3. **Run Proposler** – kicks off `OpenAIPro.RefineProposalDataAsync`, `FormRecognitionService`, and `ProposalWordDocumentService`
4. **Review Word output** – MAUI or desktop clients launch Word macros (`ref/macros/pro_proposaler_updater.bas`) to finalize formatting if needed
5. **Send** – attach SL-2/state forms, export to PDF, send via DocuSign, or email directly from Quickfire

## Code map
| Layer | Files |
| --- | --- |
| UI | `src/Quickfire.Blazor/Domain/Proposals/Components/Proposals.razor`, `ProposalDetails.razor (+ .cs partials)`, `Attachments.razor` |
| Services | `Domain/Proposals/Services/ProposalService.cs`, `ProposalWordDocumentService.cs`, `ExtractorService.cs`, `PackagerService.cs`
| AI helpers | `Domain/Shared/Services/OpenAIPro.cs` (strict schema refinement), `Domain/Integrations/FormRecognition/*`
| Word automation | `Applications/Surefire.Goodies/Proposaler/*`, `Quickfire.Tray/Methods/WordControl.cs` for on-demand document data capture

## Configuration
- OpenAI + Azure keys pulled from environment variables or Integration Key Values
- Word macros stored under `/ref/macros` – keep them versioned along with docs so the desktop installer picks them up
- Proposler templates live in `/wwwroot/templates/proposals`; update them when branding changes

## Tips
- When adding new cover pages or optional content, update `proposal-strict.json` (schema) and sample prompts in `wwwroot/prompts`
- For new state forms, drop PDFs + JSON mapping under `wwwroot/forms` and reference them from `ProposalService`
- Use [[reference/Renewal-Data-Share]] to keep renewal statuses synced when proposals change status to Sent/Bound

See also: [[features/Renewals-and-Submissions]] (proposals tie directly into renewal tasks) and [[agents/OpenAIPro-and-Agent-Stack]] for the AI plumbing behind SmartPaste + Proposler.
