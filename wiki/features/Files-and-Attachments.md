# Files & Attachments

Quickfire treats attachments as first-class data so producers can drag quotes/forms into proposals, link Epic docs, and open anything on disk instantly.

## User capabilities
- Drag/drop anywhere you see `DropzoneContainer` (renewals, proposals, clients) and the app auto-hashes, stores, and thumbnails the file
- Pin quotes/forms to proposals with quick stacks in `Domain/Proposals/Components/Attachments.razor`
- Flip between local, network, or Azure Blob storage without code changes (handled via `AttachmentService` + system settings)
- Launch Explorer/Word/Browser directly using `StringHelper.BuildWindowsPath` (Ctrl+Click copies paths your team shares)
- Import/export Epic attachments via `AttachmentEpicListGrid` and `EpicAttachmentImportService`

## Implementation details
| Concern | Files |
| --- | --- |
| UI grids + quick stacks | `src/Quickfire.Blazor/Domain/Attachments/Components/AttachmentListGrid.razor`, `AttachmentIcons.razor`, `AttachmentPreview.razor` |
| Drop zones | `Domain/Attachments/Components/DropzoneContainer.razor` + CSS for focus states |
| Service layer | `Domain/Attachments/Services/AttachmentService.cs`, `AttachmentUploaderApi.cs`, `EpicAttachmentImportService.cs` |
| Helpers | `Domain/Shared/Helpers/StringHelpers.cs` (hashing, paths, thumbnails), `Domain/Utilities/Services/PolicyXmlImportService.cs` |
| Storage selection | Profile → System Settings toggles (local path vs. Azure) feed `StateService.GetSystemSettingsAsync()` |

## Notes on storage
- Files live under `wwwroot/<client>/<YYYY>/<hash>` for local/dev and can be redirected to UNC paths for production
- Hashing (`GenerateFiveCharacterHash`) keeps filenames short but `StringHelper.RemoveHashedFileName` reconstructs friendly names in grids
- Azure Blob support hooks into `AttachmentService.UploadToAzureAsync` once credentials exist in IntegrationKeyValues

## Best practices
1. Reuse the `AttachmentGroupId` associated with renewals/policies so attachments show up everywhere automatically
2. When adding new UI around attachments, use the helper components (icons/list grid) so keyboard + drag/drop behavior stays consistent
3. Reference [[reference/Binding-Events]] for the proper `@bind`/`ValueChanged` combos on Fluent/Sf controls used inside attachment dialogs

Related docs: [[features/Proposals-and-Proposler]] for how attachments feed Proposler, and [[features/Renewals-and-Submissions]] for renewal-specific attachment handling.
