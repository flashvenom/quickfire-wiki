# Agent Prompt Library

Canonical prompts live in `src/Quickfire.Blazor/Domain/Agents/Utilities/PromptTemplates.cs`. Keep this page in sync when prompts change so AI work stays deterministic.

## Intent Classification
Used by `OpenAIAgent.ClassifyIntentAsync` before every request.
```text
Classify this user input into one of four categories for an insurance business application:
1. AgentAction ...
2. DatabaseQuery ...
3. Navigation ...
4. GeneralAI ...
Input: "{{input}}"
Respond with ONLY raw JSON:
{
  "intent": "AgentAction|DatabaseQuery|Navigation|GeneralAI",
  "confidence": 0.0-1.0,
  "reasoning": "Brief explanation"
}
```
**Tips**
- Keep examples up to date with actual workflows (loss runs, DocuSign, SmartPaste, etc.)
- Avoid markdown/code fences so Semantic Kernel can parse JSON without trimming

## SQL Generation
Executed inside `HandleDatabaseQueryAsync` to craft read-only SQL.
```text
Generate a SQL query for this question: "{{input}}"
Database Schema:
{{schema}}
Requirements:
1. Use proper JOINs...
...
13. For phone number queries ...
Respond with ONLY raw JSON:
{
  "sql": "SELECT ...",
  "explanation": "What this query does",
  "success": true
}
```
**House rules**
- Keep business rules (client name vs ID, carrier aliasing, email joins) accurate with the real schema.
- Add new constraints when we add tables to keep SQL safe and performant.

## Parameter Extraction
Used by Task Agents and SmartActions.
```text
Extract parameters from this business action request: "{{input}}"
Extract these if present:
- client_name
- action_type (loss_run, certificate, proposal, payment_link, ...)
- policy_type
- time_period
- urgency
Respond with ONLY raw JSON (no markdown).
```
Extend the bullet list as agents learn new parameters (e.g., `carrier_name`, `policy_number`).

## How to add/update prompts
1. Edit `PromptTemplates.cs` and add/adjust your template text. Keep placeholders wrapped in `{{ }}`.
2. Reference the new template in code via `PromptTemplates.GetPrompt("TemplateName", new() { { "input", text } })`.
3. Update this wiki page with the new template and guidance.
4. Store long-form prompt files (multi-hundred lines) under `wwwroot/prompts` and load them manually if they do not fit neatly inside `PromptTemplates`.

## Testing
- Use `/agents/enhanced/chat?debug=true` to watch the actual prompt text, parsed response, and intent classification.
- For SQL prompts, run the question through the `/api/debug/database-query` endpoint and inspect server logs.
- Keep sample conversations in `/ref/ToDo-AI.md` so we can regression-test prompts before releasing.

Keeping prompts centralized here prevents drift between Cursor snippets, wiki docs, and the actual source.
