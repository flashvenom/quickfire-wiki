# Binding & Change Events

These conventions keep Fluent UI + Syncfusion components in sync with EF models inside the .NET 9 Blazor host. Follow them to avoid double updates, event floods, or validation regressions.

## Universal rules
1. **Bind + callback** – always pair an `@bind-*` directive with a method (`@bind-Value:after`, `OnChange`, etc.) when the UI writes to the database.
2. **Async first** – callbacks that touch services or EF must return `Task` and be awaited. Avoid `async void`.
3. **PascalCase attributes** – Fluent/Syncfusion components use PascalCase (`OnChange`). DOM events stay lowercase (`@oninput`).
4. **Small lambdas** – keep inline lambdas to one statement (e.g., `@(args => SaveProposal(true))`). Move logic into a method when it grows.
5. **`Immediate="true"`** – prefer built-in Immediate bindings instead of manual `@oninput` when you need per-keystroke updates.

## Common patterns
| Control | Binding | Change hook | Example |
| --- | --- | --- | --- |
| `FluentTextField` | `@bind-Value` | `@bind-Value:after="ApplyFilters"` or `Immediate="true"` | `Domain/Attachments/Components/AttachmentListGrid.razor` |
| `FluentCheckbox` | `@bind-Value` | `OnChange="OnCheckedChanged"` | `Domain/Proposals/Components/ProposalDetails.razor` |
| `FluentRadioGroup` | `@bind-Value` | `OnChange="BillTypeChanged"` | `Domain/Proposals/Components/ProposalDetails.razor` |
| `SfDropDownList` | `@bind-Value` | `@bind-Value:after="OnSelectionChange"` + `<DropDownListEvents ValueChange="OnSelectionChange" />` | `Domain/Accounting/Components/SettlementScreen.razor` |
| `SfNumericTextBox` | `@bind-Value` | `OnChange="@(args => SaveProposal(true))"` | `Domain/Proposals/Components/ProposalDetails.razor` |
| `SfDatePicker` | `@bind-Value` | `OnChange="@(args => OnDateChanged())"` (lambda required) | `Domain/Renewals/Components/Submissions.razor` |
| `SfCheckBox` | `@bind-Checked` | `@onchange="SaveProposalSimple"` (explicit closing tag) | `Domain/Proposals/Components/ProposalDetails.razor` |

## Fluent UI specifics
- **TextField/TextArea** – use `Immediate="true"` for filters; for expensive operations use `@bind-Value:after` to debounce inside your method.
- **NumberField** – `ValueChanged` + `ValueExpression` are required when used inside `EditForm`. Example: `Domain/Accounting/Components/SettlementScreen.razor`.
- **Checkbox** – `@bind-Value` works for boolean states; avoid `@bind-Checked` with Fluent components.
- **Combobox** – prefer Fluent Combobox for short lists, Syncfusion for large datasets. When using Combobox, handle `ValueChanged` and `TValue` explicitly.

## Syncfusion specifics
### Imports
```razor
@using Syncfusion.Blazor
@using Syncfusion.Blazor.DropDowns
@using Syncfusion.Blazor.Inputs
```
### DropDowns
```razor
<SfDropDownList TItem="RelationshipTypeItem" TValue="RelationshipType"
                DataSource="@RelationshipTypes"
                @bind-Value="SelectedRelationship"
                @bind-Value:after="OnRelationshipChanged">
    <DropDownListFieldSettings Text="DisplayName" Value="Value" />
    <DropDownListEvents TValue="RelationshipType" TItem="RelationshipTypeItem"
                        ValueChange="OnRelationshipChanged" />
</SfDropDownList>
```
- Generics (`TItem`, `TValue`) must match `DataSource` and bound property.
- Use `<DropDownListEvents>` for `ValueChange`; do **not** put `ValueChange` on the root element.
- Filtering? Set `AllowFiltering="true"` and supply a `Filter` handler or `FilterType`, but not both.

### DatePicker
```razor
<SfDatePicker TValue="DateTime?"
              @bind-Value="Proposal.SendDate"
              OnChange="@(args => OnSendDateChanged())" />
```
- Use lambda to ignore event args; direct method references fail because Syncfusion expects a parameter.

### NumericTextBox
```razor
<SfNumericTextBox TValue="decimal?"
                  @bind-Value="Proposal.BrokerFee"
                  Format="C2"
                  OnChange="@(args => SaveProposal(true))" />
```
- Avoid `ValueChange` (wrong event) – use `OnChange` or `@bind-Value:after`.

## Validation helpers
- Use `<EditForm>` + `<DataAnnotationsValidator>` where possible; `ValueExpression="() => Model.Property"` is required for Syncfusion + validation.
- `ValidationMessage` pairs nicely with Fluent/Syncfusion controls inside forms.

## Anti-patterns to avoid
- Mixing DOM events (`@onchange`) with component events on the same control
- Self-closing `<SfCheckBox />` tags (Syncfusion needs explicit closing tags)
- `async void` callbacks – always `async Task`
- Passing `null` `DataSource` to `SfDropDownList` – pre-initialize with `Array.Empty<T>()`
- Using `@bind-Value` **and** manual `Value`/`ValueChanged` simultaneously

## References
- See `Domain/Proposals/Components/ProposalDetails.razor` for nearly every control pattern in one file
- `Domain/Attachments/Components/AttachmentListGrid.razor` shows Fluent filters + `Immediate`
- `Domain/Accounting/Components/SettlementScreen.razor` demonstrates numeric/date/dropdown combos with data annotations

Use this doc as the authoritative source before nudging Cursor/Copilot—copy/paste the snippets above so generated code compiles the first time.
