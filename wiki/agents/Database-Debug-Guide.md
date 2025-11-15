# Database Query Debug Guide

Enhanced AI Chat answers data questions by routing through `OpenAIAgent.HandleDatabaseQueryAsync`. Use this playbook whenever a question like "How many renewals are pending for Acme?" fails.

## Turn on verbose logs
Update `src/Quickfire.Blazor/appsettings.Development.json` so the agent logs everything:
```json
{
  "Logging": {
    "LogLevel": {
      "Surefire.Domain.Agents.Services.OpenAIAgent": "Debug",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  }
}
```
Then restart `dotnet watch run`. All SQL-agent logs use the `[DBQuery]` prefix.

## Request pipeline
1. **Semantic search** – `PerformSemanticSearchAsync` queries Qdrant (`OpenAIEmbeddingService` + `QdrantVectorStore`). Logs: `[DBQuery] Starting semantic search...`, `[DBQuery] Enhanced input with ...`.
2. **Prompt creation** – `CreateSqlGenerationPrompt` merges the user question, semantic matches, and schema docs loaded from `wwwroot/schema/*.sql` (`LoadDatabaseSchema`).
3. **LLM call** – Semantic Kernel `IChatCompletionService` (`gpt-4.1`) returns JSON (SQL, parameters, explanation). Logged as `[DBQuery] Raw SQL Generation Response`.
4. **Validation** – `ValidateSQL` blocks dangerous commands, enforces upper limits, and logs `[DBQuery] SQL Validation - IsValid ...`.
5. **Execution** – The validated SQL runs via EF Core on an open connection. Timing + row count logged as `[DBQuery] Query executed successfully with {ResultCount} rows in {Elapsed}`.
6. **Formatting** – `ResponseFormatter` renders narrative + table HTML. Suggestions list is auto-generated for follow-up questions.

## Debugging checklist
| Problem | What to inspect |
| --- | --- |
| Semantic search misses entity | Ensure embeddings are up to date by running `EmbeddingLoaderService` (Profile → System Settings) and confirm Qdrant is reachable (`QDRANT_BASE_URL`)
| SQL generation fails | Look at `[DBQuery] Raw SQL Generation Response` – often the JSON is malformed when schema docs are stale. Check `wwwroot/schema` contents and rerun the extractor script if needed.
| Validation rejects query | The log includes exact errors (keywords, multiple statements, missing FROM). Refine prompts or teach the model to auto-add `TOP (25)` to heavy queries.
| Execution exception | EF logs include the SQL + parameters. Verify column names (maybe a migration renamed a column), or update indexes if timeouts occur.
| No results | Inspect the generated SQL and consider adding more filters or a different semantic match. `[DBQuery] Enhanced input...` shows which entities were injected.

## Helpful tools
- `/agents/enhanced/chat?debug=true` shows inline intent badges and the final SQL snippet rendered in the UI.
- `wwwroot/schema` holds `.sql` files with `LLM_SearchHints` extended properties; keep them updated to teach the model which fields matter.
- `/ref/schema/extractor.json` documents the canonical schema. Update it when you add tables or rename columns.

## Instrumentation markers
- `[DBQuery] Starting semantic search for: ...`
- `[DBQuery] Enhanced input with ... semantic matches`
- `[DBQuery] Raw SQL Generation Response: ...`
- `[DBQuery] Parsed SQL Success: True/False`
- `[DBQuery] SQL Validation - IsValid: ...`
- `[DBQuery] Query executed successfully with X rows in Y ms`

Following these steps will show you exactly where a database question fell down—entity resolution, prompt, validation, or SQL execution—and what to fix next.
