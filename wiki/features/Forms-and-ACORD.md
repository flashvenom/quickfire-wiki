# Forms & ACORD

Forms bring revision-safe PDF editing, JSON mappings, and AI-assisted prefills together so team members can get certificates, SL-2s, and custom riders out quickly.

## Broker workflow
- Launch a form from Clients, Renewals, or Files and bind it to a policy/client for field-level context
- Review existing revisions from the audit drawer and branch a new version when needed (non-destructive)
- Prefill fields using policy data mapped via JSON templates (`wwwroot/forms/_json/*.json`)
- Flatten or export back to attachments, proposals, or DocuSign packages

## Components & services
| Piece | Files |
| --- | --- |
| Editors + viewers | `src/Quickfire.Blazor/Domain/Forms/Pages/FormEditor.razor`, `CertificateEditor.razor`, `TestEditor.razor`, `Domain/Forms/Components/PolicyViewer.razor` |
| Service logic | `Domain/Forms/Services/FormService.cs` (revisioning, flattening, template merge)
| Template assets | `wwwroot/forms/*.pdf`, `wwwroot/forms/_json/*.json`
| DocuSign handoff | `Domain/Integrations/DocuSign/DocuSignService.cs` + widgets on Homepage/Renewals
| AI extraction (optional) | `Domain/Integrations/FormRecognition` + `OpenAIPro` for Proposler/form data cleanup

## Tips
- JSON templates map Surefire fields to PDF IDs; use `PolicyService` callouts to retrieve the right policy context before merging
- Revisions store metadata + file references (via `AttachmentService`) so you can revert without re-uploading the PDF
- When adding new forms, drop the PDF + JSON under `wwwroot/forms`, update `FormService` metadata, and ensure attachments point to the new IDs

Cross references: [[features/Proposals-and-Proposler]] for how form outputs move into proposals, and [[reference/Renewal-Data-Share]] for how form values sync back to renewals.
