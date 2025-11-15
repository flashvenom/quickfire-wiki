# Integration Architecture

Quickfire consolidates integrations (RingCentral, ePayPolicy, DocuSign, Graph, Azure Form Recognizer, Qdrant, etc.) behind a consistent lockbox pattern. Use this doc when adding a new integration or updating existing ones.

## Goals
- Store credentials + metadata securely in the database
- Provide typed settings objects for runtime code (no direct `IOptions`/`Configuration` reads)
- Allow admins to rotate keys from the UI with instant cache invalidation
- Offer light-weight state helpers instead of bloating `StateService`

## Building blocks
| Component | Location | Responsibility |
| --- | --- | --- |
| `IntegrationKeyValues` table | `Data/ApplicationDbContext.MainEntities.cs` | Persists encrypted key/value pairs with scope, rotation metadata, timestamps |
| `IntegrationKeyValueService` | `Domain/Integrations/Services/IntegrationKeyValueService.cs` | CRUD ops, encryption/decryption, rotation helpers, DTO mapping |
| Typed providers | e.g., `RingCentralSettingsProvider`, `DocuSignSettingsProvider` under `Domain/Integrations/*/Services` | Translate DB rows into strongly typed `Settings` objects, cache with `IMemoryCache`, expose `Invalidate()` |
| Integration services | e.g., `RingCentralService`, `EPayPolicyService`, `DocuSignService` | Consume providers, build HttpClient calls, retry policies, DTO mapping |
| UI/admin | Profile → Integrations pages (`Domain/Profile/Components/SystemSettings.razor`, `Payments.razor`, etc.) | Capture credentials, toggle features, call `Invalidate()` on save |
| Scoped state helpers | e.g., `RecentPaymentsState`, `DocuSignEnvelopeState` | Cache frequently displayed data for Homepage widgets without bloating `StateService` |

## Flow
1. **Seed** the table with placeholder keys (migration or admin UI).
2. **Admin** enters actual credentials via the UI → `IntegrationKeyValueService.UpsertAsync` encrypts and stores them.
3. **Provider** (`GetAsync`) pulls rows, validates required keys, applies defaults (base URLs, timeouts), caches the typed settings for a short TTL.
4. **Service** obtains settings, configures HttpClient from `IHttpClientFactory`, executes the call, logs failures, and returns domain models.
5. **UI/Workers** consume the integration service/state helper.

## Implementation checklist for new integrations
1. **Settings model** – define a record/class (Enabled, BaseUrl, ApiKey, extra toggles).
2. **Provider** – implement `IIntegrationSettingsProvider<TSettings>`; use `_cache.GetOrCreateAsync` to cache results.
3. **Service** – inject provider + `HttpClientFactory`, handle retry/backoff, log errors using `ILogger<T>`.
4. **Admin UI** – add a tab or section under Profile → Integrations with Fluent UI form controls; call `Invalidate` upon save.
5. **State/UI** – where widgets need cached data, create a small state service (similar to `RecentPaymentsState`).
6. **Health** – optional `IHostedService` to ping the integration and emit warnings if credentials fail.

## Example: ePayPolicy
- Settings provider: `Domain/Integrations/ePayPolicy/Services/EPayPolicySettingsProvider.cs`
- Service: `EPayPolicyService` fetches recent transactions + builds paylinks
- State: `RecentPaymentsState` caches transactions for Homepage widget
- Admin UI: Profile → Payments populates `IntegrationKeyValues`

## Example: RingCentral
- Provider ensures Client ID/Secret/Base URI/Extension are configured
- `ChatService.RingCentral` fetches tokens using provider values, caches them, and pushes events to SignalR + Tray
- Gateway handles incoming webhooks and relays them via SignalR hubs (`Quickfire.Gateway/Controllers/SmsWebhookController.cs`)

## Example: DocuSign
- Provider stores `IntegratorKey`, `UserId`, `AuthServer`, RSA keys
- `DocuSignService` handles JWT auth + envelope APIs
- `DocuSignEnvelopeRefreshWorker` refreshes envelope status for Homepage widget

## Debugging & observability
- Each provider logs warnings if required keys are missing or disabled
- Services log upstream failures with correlation IDs – search for `[Integration]` in logs
- UI optionally surfaces "Disconnected" badges when providers throw `IntegrationDisabledException`

## Security reminders
- `IntegrationKeyValueService` encrypts data at rest; never log raw secrets
- Only expose secrets in UI to admins
- When running desktop installs, secrets still come from the DB – configure the local DB with safe defaults before shipping

Follow this blueprint whenever new APIs (carrier portals, CRMs, payment gateways) are onboarded.
