# Fortuna — fact sheet

The cited facts this site is written from, compiled 2026-10-13 from
`fortuna-api` and `fortuna-ui`. When a page and its source disagree, the
source wins: fix the page, then this file.

Compiled 2026-10-13 from the local checkouts and GitHub. **Read-only research; nothing invented.**
Every bullet ends with its source.

**Citation shorthand**
- `A:` = `D:\Repositories\fortuna-api`, `U:` = `D:\Repositories\fortuna-ui`.
- `A:SRD` = `docs/requirements/System Requirements Document.md`; `A:UCS` = Use Case Specification
  Document; `A:OPS` = Operations & Infrastructure Document; `A:TSD` = Technology Stack Document;
  `A:VIS` = Vision Document; `A:TST` = Testing Specification Document; `A:DWD` = Development Workflow
  Document; `A:BR` = `docs/initial/Business Rules.md`; `A:PO` = `docs/initial/Project Overview.md`.
  The same abbreviations with `U:` apply to fortuna-ui.
- `A:OAS` = `A:docs/openapi/fortuna.json` (counts were computed by script over the file).

**Which snapshot was read**
- fortuna-api: the working tree was on `release/2.1.0` when the research finished. Its content matches
  `origin/develop` (`git diff develop origin/release/2.1.0` is empty). Head commit `27844c2` "Merge pull
  request #210 from artur-rios/feature/changelog-2.1.0". The branch was switched while this research ran
  (the first `git branch --show-current` returned `develop`). Docs read from the tree match develop.
- fortuna-ui: `develop`, head `3d8e31c` "Merge pull request #109 from artur-rios/feature/self-service-sign-up".
- Both repos have default branch **`develop`** (`gh repo view … --json defaultBranchRef`).

---

## 1. Pitch, what it does, what it doesn't

### Pitch (shared wording in both READMEs)
- "Fortuna is a personal finance system that tracks a person's whole financial life — bank accounts,
  credit cards, investments, expenses and earnings — in one place, and turns that history into tabular
  views, drillable charts and forward projections." (A:README.md intro; U:README.md intro uses "charts
  and forward-looking projections").
- fortuna-api is "the back end only: an HTTP API that owns the domain, the data and the integrations"
  (A:README intro). fortuna-ui is "the front end only: a single Flutter application that runs on the
  web, Windows, Linux and Android from one code base" (U:README intro).
- "The API owns the domain, the money and the integrations; this application owns the experience." (U:README intro)
- Problem statement: data scattered across bank apps, statement PDFs, broker portals, card invoices and
  a hand-made spreadsheet; the two questions are *where did the money go?* and *where will I be in six
  months?* (A:VIS §2; A:PO "The Problem").
- Positioning: self-hostable; "Runs on the owner's own machine or server, stores no bank credential at
  any point, computes money with exact decimal arithmetic rather than floating point, and treats every
  ingestion source as a plug-in" (A:VIS §3 Product Position Statement).
- Users/roles: Account owner, Instance administrator (no access to financial records), Desktop offline
  user (U:PO "Who It's For"; A:PO "Who It's For").
- Deployment shapes: single self-hosted owner, shared instance (A:PO); UI modes: **Connected**,
  **Self-hosted**, **Desktop offline** (U:PO "Who It's For"; U:BR BR-24).

### What it does (A:README "What it does"; U:README "What it does")
- Tracks bank accounts, credit cards (billing cycles, statements) and investments (positions), each with
  its own currency and history.
- Records expenses, earnings, transfers, installment purchases, recurring commitments.
- Ingests from manual entry, Pluggy open banking, Excel workbooks and supported PDF statement layouts
  (currently Nubank credit-card invoice) behind one pluggable source contract.
- Multi-currency with exact decimals and dated, attributable BCB PTAX rates.
- Spreadsheet-shaped tables; chart aggregations that drill down to the transactions.
- Projections from recurring commitments, unbilled installments and statements due.
- Exports CSV, Excel, PDF; attachments on transactions.
- Data rights: consent before third-party processing, complete personal-data export (ZIP), account erasure.
- Desktop offline (Windows/Linux) via an in-process native core; or multi-user shared instance.
- UI: sign in by email/password or Google with 2FA, or desktop local account; sign-up (UC-47).

### What it doesn't do
- API (A:README "What it doesn't do"; A:VIS §1.2 Out of scope): does not move money (no payments, no
  bank transfers, no trading; never writes to a financial institution); no financial advice; no tax
  filing/accounting compliance; does not store bank credentials; does not manage users (Heimdall does;
  desktop offline account is the exception); does not render a UI.
- UI (U:README "What it doesn't do"; U:PO): does not compute money; does not own data (no client-side
  database); does not parse/render files; does not talk to external services except Google ID token;
  no iOS/macOS; no offline outside desktop offline mode. Inherits API exclusions.

---

## 2. Domain concepts and the business rules users must understand

> BR numbering collides between repos: fortuna-api has `BR-01…BR-46` (domain), fortuna-ui has
> `BR-01…BR-42` (client). Prefix them (e.g. `API-BR-05`, `UI-BR-06`) on the docs site.

### Entities (A:BR "Domain Entities"; A:VIS §1.3)
User, LocalAccount, RecoveryCode, Currency, ExchangeRate, FinancialAccount, CreditCard,
CreditCardStatement, Investment, InvestmentTransaction (movement), InvestmentValuation, Transaction,
Transfer, InstallmentPlan, RecurringTransaction, Category, Tag, Counterparty, Attachment, Budget, Goal,
DataSource, Connection, ImportJob, ImportedRecord, Projection, AuditLog. Two IDs per entity: internal
`bigint Id` never exposed, public GUID `PublicId` everywhere else (A:SRD §4.0; NFR-12).

### Enumerations published in OpenAPI (A:OAS `x-enum-varnames`, required by A:SRD NFR-37)
`FinancialAccountType` Checking/Savings/Cash/Other · `InvestmentType` FixedIncome/Equity/Fund/Other ·
`InvestmentMovementType` Contribution/Withdrawal/Yield/Fee · `TransactionDirection` Expense/Earning ·
`TransactionSourceType` Manual/Pluggy/Excel/Pdf · `CreditCardStatementStatus` Open/Closed/Settled ·
`RecurrenceFrequency` Weekly/Monthly/Quarterly/Yearly · `BudgetPeriodType` Monthly/Quarterly/Yearly ·
`GoalResourceType` Account/Investment · `ConnectionStatus` Active/RequiresReauthentication/Revoked ·
`ImportJobStatus` & `DataExportStatus` Pending/Running/Completed/Failed · `ImportedRecordOutcome`
Imported/Duplicate/Rejected · `ImportTargetType` Account/CreditCard · `DataSourceKind` Network/File ·
`DataExportFormat` Csv/Excel/Pdf/Zip · `DataExportDelivery` Direct/Queued · `ExchangeRateSource`
Published/Manual · `CashFlowFigureKind` Recorded/Projected/Committed/Estimated · `CashFlowPeriodicity`
Daily/Weekly/Monthly · `CommittedObligationKind` Installment/Statement · `LocalAccountStorageMode`
InMemory/OperatingSystem · `AuditOutcome` Succeeded/Refused · `TableColumnType` Uuid/Text/Integer/Decimal/Boolean/Date/Timestamp/Enumeration.

### Ownership and isolation
- BR-01 every record belongs to exactly one user; BR-02 administrator has no access to financial
  records; BR-03 owner fixed at creation; BR-04 another user's record is refused "as if the record did
  not exist" → `404` (A:BR "Ownership and isolation"; A:UCS §1.1 "Isolation").
- Admin role reading financial data is ❌ in every column (A:SRD §7 Authorization Matrix).

### Money and currencies
- BR-05 exact decimal + currency, never binary floating point; JSON publishes decimals as invariant
  strings (A:BR). Wire pattern `^-?(?:0|[1-9][0-9]*)(?:\.[0-9]+)?$`, `type: string`, `format: decimal`
  (A:TSD §4.2). Change shipped by issue #162 (A:README backlog M-01; U:README "Formerly blocked": "All
  167 monetary fields were published as JSON numbers with `format: double` … `8017.61` arrived as
  `8017.60999999999967`").
- BR-06 no cross-currency sum/compare without explicit conversion; BR-07 every conversion records rate
  and date; BR-08 one immutable currency per account/card/investment; BR-09 transaction in its
  account's currency, original amount/currency retained; BR-10 rounding only at presentation or
  conversion, to minor-unit precision; BR-11 balances always derived, never stored (A:BR).
- Rounding rule: target minor unit, **half away from zero** (A:SRD FR-CU-10).
- Storage: PostgreSQL `numeric(19,4)` money, `numeric(19,8)` rates; SQLite `TEXT`, never `REAL`
  (A:SRD NFR-07; A:TSD §4.2; A:OPS IR-06).
- Currencies: ISO 4217 reference set seeded idempotently at startup (A:OPS IR-07; A:SRD FR-CU-01).
- **PTAX**: Banco Central do Brasil's official rates via "the free, key-less Olinda OData service",
  publishing *cotação* (currency↔BRL) and *paridade* (currency↔USD) so cross rates are derivable
  (A:TSD §6.1). Synced on schedule as a job; cross-rate derivation; weekend/holiday uses most recent
  prior publication attributed to its own date; manual rate overrides published for that pair/date;
  when the source is unreachable conversions continue on the most recent known rate, marked with its
  date (A:SRD FR-CU-05…11; A:UCS UC-08, UC-09, UC-10).
- Code fact: recording a manual rate and triggering sync require "a system administrator or the local
  installation owner" — `403` otherwise (A:`src/Application/ArturRios.Fortuna.Shared/Messages/ManualExchangeRateMessages.cs`,
  `ExchangeRateSyncMessages.cs`; `RecordManualExchangeRateCommandHandler.cs` comment "Manual rates are
  global: they take precedence for every user"). Admin = Heimdall `SystemAdmin` role or local installation
  (A:`src/Application/ArturRios.Fortuna.Shared/Security/InstallationAdministration.cs`).
- Display currency (UC-10): group by own currency, convert each group total once with the rate on the
  figure's date, report rate+date per source currency; if no rate ever stored, return split by currency,
  unconverted, with the reason (A:UCS UC-10).
- Profile default display currency from `FORTUNA_DEFAULT_DISPLAY_CURRENCY`, else inferred from required
  `FORTUNA_LOCALE` and the client must ask the user to confirm (A:OPS §3).

### Financial accounts
- Name, institution, type, currency, opening balance (may be zero/negative); balance = opening balance
  + live transactions; no "set balance" operation; name unique per user per kind (A:SRD FR-AC-01…08;
  A:BR Validation Constraints).
- Net position across accounts, investments and cards, in a display currency (A:SRD FR-AC-12, FR-RP-10).

### Credit cards, billing cycles and statements
- Card: issuer, limit > 0, currency, closing day and due day 1–31 (A:SRD FR-CC-01/02).
- BR-12 a card is not a bank account; charges accumulate into a statement settled by a payment from a
  financial account; the settlement is a **transfer**, not an expense (A:BR; A:SRD FR-CC-07).
- BR-13 charge falls in the statement whose billing cycle contains its date, by closing day; statement
  opened automatically (A:SRD FR-CC-03/04).
- BR-14 settled statement is frozen; later charges go to next open statement, flagged late-arriving
  (A:SRD FR-CC-08/09).
- Statement states: `Open` → `Closed` (total fixed) → `Settled`; never goes back (A:BR "Lifecycle").
- Used/available limit reported (A:SRD FR-CC-10).
- 2.0.0 fixes: installments land in successive cycles incl. purchases on 29th–31st; budget periods
  anchored 29–31 contiguous (A:CHANGELOG [2.0.0] Fixed).

### Investments and positions
- BR-17 Fortuna records valuations; never prices an instrument (A:BR). Position = movements + most
  recent valuation, no market data (A:SRD FR-IV-04). Movements: contribution, withdrawal, yield, fee
  (A:SRD FR-IV-02). Valuation history over a period (FR-IV-06).

### Transactions, transfers, installments, recurring
- BR-18 date, amount, direction, owning account, category; BR-19 amount positive, direction carries the
  sign (A:BR). Date more than one day in the future rejected (A:BR Validation; A:SRD FR-TX-04).
  Optional description, counterparty, tags; max live tags per transaction `FORTUNA_TRANSACTION_MAX_TAGS`
  default 50 (A:SRD FR-TX-08; A:OPS §3).
- Transfer: one record producing paired legs, atomic; origin ≠ destination; excluded from income and
  expense; deleting either leg deletes both (BR-15, BR-16; A:SRD FR-TX-13…17).
- Installment plan: ≥ 2 installments summing **exactly** to the total; remainder on the **first**
  installment (BR-20; A:SRD FR-TX-18/19; example 100.00/3 → 33.34, 33.33, 33.33 in A:TST §5). Each
  installment in successive billing cycles (FR-TX-20).
- Recurring transaction = rule, not a movement; only materialized occurrences are real (BR-21); edits
  affect future only (BR-22). Materialization is idempotent via a last-materialized marker; a
  materialized occurrence that duplicates an imported transaction is created and flagged as possible
  duplicate (A:UCS UC-38). Frequencies Weekly/Monthly/Quarterly/Yearly (A:OAS enum).
- Transaction lifecycle: `Recorded` → `Reconciled` → `SoftDeleted` → `HardDeleted` (A:BR "Lifecycle").

### Reconciliation
- Mark a transaction reconciled against an imported record, storing which record (A:SRD FR-TX-25).
  Record already linked → `409`; differences beyond tolerance accepted but flagged; already reconciled →
  `409`; unreconcile removes the link (A:UCS UC-40). Tolerances `FORTUNA_RECONCILIATION_AMOUNT_TOLERANCE`
  (default 0.01) and `FORTUNA_RECONCILIATION_DATE_TOLERANCE_DAYS` (default 1) (A:OPS §3).

### Categories, tags, counterparties
- Category tree, at most one parent, no cycles (BR-23); name unique among live siblings; reassign all
  transactions between categories in one operation; hard delete refused while live transactions
  reference it (A:SRD FR-CT-01…07).
- Tags unique per user; attach/detach any number (FR-CT-08/09).
- Counterparties matched by normalized name before creation; category suggested from the categories most
  recently used for the same counterparty (FR-CT-10/11). Merge endpoint `POST /api/counterparties/{id}/merge` (A:OAS).

### Budgets and goals
- Budget over one or more categories, amount > 0, period (Monthly/Quarterly/Yearly); consumption =
  spent vs amount, exceeded flag and amount; excludes transfers and soft-deleted (A:SRD FR-PL-01…04).
- Goal: target amount > 0, target date in the future at creation, linked accounts (Account/Investment);
  progress from current balances of linked accounts (FR-PL-05/06).

### Imports and ingestion
- Sources (A:`src/Infrastructure/ArturRios.Fortuna.Integration/Ingestion/*`): `pluggy` "Pluggy open
  banking" (Network); `excel` "Excel workbook" (File, `.xlsx`, inputs "Workbook file", "Column mapping",
  layout "Caller-mapped worksheet"); `nubank-pdf` "Nubank credit card invoice" (File, `.pdf`). Pluggy is
  listed unavailable when unconfigured or network unavailable.
- **Supported PDF layouts: one — the Nubank credit card invoice** (A:SRD §3.9.1; A:README M-06 "Excel
  and Nubank PDF imports"). Layout detected from content; unrecognized layout → rejected naming
  supported layouts, nothing imported (FR-IM-18/20).
- Nubank rules (A:SRD FR-IM-24…38; A:UCS UC-60): read due/issue date and billing period; summary
  (previous balance, payments, purchases, international transaction tax, amount due) must reconcile or
  nothing is imported; Brazilian number format; ASCII hyphen and Unicode minus `U+2212`; `Parcela n/N`
  markers; foreign purchase original amount/currency/rate; IOF tax lines and reversals; credit
  adjustments; payments section = settlement of previous statement; year inferred across year boundary;
  no OCR (no text layer → fails).
- Excel (UC-59): multipart fields `File`, `TargetType` (Account/CreditCard), `TargetId`, `DateColumn`,
  `AmountColumn`, `DescriptionColumn`, `DirectionColumn`, `CategoryColumn`, `ExternalIdColumn`,
  `CreateMissingCategories` (A:OAS `POST /api/imports/excel`). PDF: `File`, `CreditCardId` (A:OAS).
- Rules: BR-24 imported raw records immutable; BR-25 provenance on every transaction; BR-26 duplicate =
  same account, date, amount and source id where supplied; BR-27 per-row outcome imported/duplicate/
  rejected, one bad row never aborts others; BR-28 bank credentials never stored/logged/transmitted;
  BR-29 revoking keeps imported data; BR-30 new sources additive (A:BR).
- 2.0.0 behavior: "A failed Excel or Pluggy import commits none of its rows" (A:CHANGELOG [2.0.0] Fixed).
- Jobs: `Pending` → `Running` → `Completed`/`Failed`; re-queued on restart; retry of a Failed job never
  duplicates (A:SRD FR-IM-04, 21, 22). Size limits `FORTUNA_EXCEL_IMPORT_MAX_BYTES` (example 10485760),
  `FORTUNA_PDF_IMPORT_MAX_BYTES` (example 20971520) (A:`docker/local.env.example`).
- Pluggy connection: requires current `external-data-processing` consent (else `403`), stores only
  reference + access token encrypted at rest (`AccessTokenCipher`), states Active →
  RequiresReauthentication → Active, or → Revoked (terminal) (A:UCS UC-55…58; A:SRD §4.11; A:BR Lifecycle).
  Sync creates accounts/cards it does not yet track; rate limit → back off (UC-56 AF-03, AF-06).

### Exports
- CSV, Excel, PDF of a tabular query; job when rows exceed `FORTUNA_EXPORT_SYNC_THRESHOLD_ROWS`
  (example 1000), file retained `FORTUNA_EXPORT_RETENTION_HOURS` (example 24); owner-only; soft-deleted
  excluded; amounts exact decimal with currency, formatted for the requested locale (A:SRD FR-EX-01…08;
  A:`docker/local.env.example`). Request body: `recordSet`, `columns`, `filters`, `sorts`, `format`,
  `locale`, `displayCurrencyCode`; response `delivery` Direct/Queued (A:OAS).
- Personal-data archive (UC-79): async job, ZIP with manifest + versioned JSON data + JSON Schema per
  entity kind, attachments as files, decimals as strings, excludes passwords/recovery codes/hashes/
  connection tokens, expiring, owner-only (A:SRD FR-EX-09…15). Available natively too (A:native/README).

### Attachments
- Filed against a transaction; size and content-type limits (`FORTUNA_UPLOAD_MAX_BYTES` example
  10485760; `FORTUNA_UPLOAD_ALLOWED_CONTENT_TYPES` example `application/pdf,image/jpeg,image/png`);
  storage abstraction with Filesystem and S3-compatible (MEGA S4 via AWSSDK.S3); object kept on soft
  delete, removed on hard delete (A:SRD FR-AT-01…11; A:TSD §4.3; A:`docker/local.env.example`).

### Deletion, audit, consent, erasure
- BR-38 two-stage: soft delete, then hard delete only from soft-deleted; BR-39 soft-deleted excluded
  from all figures but restorable; BR-40 hard delete refused while referenced; BR-41 audit append-only;
  BR-42 audit subject is an opaque reference, mapping destroyed on erasure (A:BR).
- Cascades: account/card → transactions; card → statements; investment → movements/valuations;
  transaction → attachments and other transfer leg. Restore reverses exactly that cascade (A:SRD §8).
- One audit entry per attempted write, succeeded or refused (A:SRD FR-RL-06; NFR-17).
- Consent: BR-43 explicit, recognized purpose (`external-data-processing`) and current disclosure version
  (`FORTUNA_CONSENT_EXTERNAL_PROCESSING_VERSION`, default `1.0`); BR-44 version-bound; BR-45 withdrawal
  revokes dependent Pluggy connections immediately, keeps history; local/file ingestion never gated
  (A:BR; A:SRD FR-ID-38…43; A:OPS §3).
- Erasure (UC-78): exact confirmation (`EraseUserCommand.confirmation`), dependency-ordered single
  transaction, removes profile, local credentials, recovery hashes, imported records, etc.; audit keeps
  only opaque reference; admin route `DELETE /api/users/{id}` returns counts only (A:SRD FR-ID-36/37,
  FR-RL-12/13, §8; A:OAS).
- BR-46: hosted API logs every request start with client IP (personal data under GDPR/LGPD), tagged
  `client.address` (A:BR).

### Identity (connected vs local)
- BR-35 Fortuna never stores a Heimdall password; may only forward in transit (A:BR).
- BR-36 desktop local account: no password reset, no e-mail recovery; recovery codes issued once at
  creation, single-use, hashed, unrecoverable (A:BR). Default count 10
  (`FORTUNA_LOCAL_AUTH_RECOVERY_CODE_COUNT`) (A:OPS §3). `POST /api/local-accounts/password-reset`
  exists only to refuse ("PasswordResetUnavailable") (A:`src/Presentation/.../Controllers/LocalAccountsController.cs`).
- BR-37 local data never synced to a shared instance implicitly (A:BR).

### UI-side rules worth surfacing to users (U:BR)
- BR-01…05: API is sole authority; client never computes figures; client validation is feedback only;
  nothing shown as saved before confirmation; API's refusal reason shown verbatim.
- BR-11 projected figures always visually distinct; BR-12 deleted records only where asked, marked;
  BR-13/14 every aggregate drillable and reversible, each level requested from API.
- BR-16 token only in platform secure storage; BR-17 no silent refresh/replay on expiry; BR-18 sign-out
  clears everything; BR-19 a 2FA challenge grants nothing.
- BR-25 desktop offline only on Windows/Linux and only when a local core is present; BR-26 lost
  connection reported, never cached.
- BR-28 import file uploaded whole; BR-29 export rendered by API only; BR-31 per-row outcomes shown.
- BR-39 no analytics, crash reporting or telemetry; BR-40 consent before external processing;
  BR-41 export and erasure reachable without contacting anyone; BR-42 erasure irreversible, not a
  two-stage delete.

---

## 3. Architecture

### API layers and projects (A:TSD §2–3; A:OPS §2.1–2.3; A:`src/**/*.csproj`)
| Layer | Project | Holds |
|---|---|---|
| Domain | `ArturRios.Fortuna.Domain` | entities, money, billing cycles, invariants |
| Application | `ArturRios.Fortuna.Command` / `ArturRios.Fortuna.Query` / `ArturRios.Fortuna.Shared` | write handlers+validators / read handlers+read models / conversion, ownership checks |
| Infrastructure | `ArturRios.Fortuna.Data` | EF Core `AppDbContext`, entity maps, PostgreSQL migrations, seeding |
| Infrastructure | `ArturRios.Fortuna.Data.Sqlite.Migrations` | EF SQLite migration set |
| Infrastructure | `ArturRios.Fortuna.Integration` | `Ingestion/` (Excel parser, Nubank PDF parser, Pluggy gateways, source registry), `Rates/PtaxRateClient`, `Exports/DataExportRenderer`, `Storage/` (Filesystem, S3), `Security/HeimdallAuthGateway` |
| Presentation | `ArturRios.Fortuna.WebApi` | ASP.NET Core host, 31 controllers (one per OpenAPI tag) plus base `FortunaController` and `FortunaStatusMap`, security, OpenAPI, observability |
- Solution `src/ArturRios.Fortuna.sln`. `Integration` deliberately does not reference `Data` (A:OPS §2.2, IR-01).
- Handlers dispatched by `ArturRios.Mediator`, return `DataOutput<T>`; `ToActionResult` maps message→status
  (A:TSD §3.1, §6). Shared message-to-status map `FortunaStatusMap.cs` (A:`src/Presentation/.../Controllers/FortunaStatusMap.cs`).
- Background jobs: `BackgroundService` + bounded `System.Threading.Channels` queue
  (`FORTUNA_JOB_QUEUE_CAPACITY`, compose default 256) + durable job table; re-queue on start (A:TSD §6; A:OPS IR-10/11; A:docker-compose.yml).
- Same skeleton as Heimdall API by design (A:initial/Technology Stack.md intro).

### Managed core vs native core
- **Managed core** = the .NET API above, PostgreSQL for shared deployments; can also run locally on
  SQLite via EF (`FORTUNA_DATA_DATABASETYPE=SQLite`) — "an alternative local deployment" (A:README
  "Running"; A:OPS §6 last paragraph). Managed and native SQLite schemas "are not interchangeable files" (A:OPS §6).
- **Native core**: a **Rust** `cdylib` (crate `fortuna-native-core` v`0.1.0`, lib name `fortuna_core`,
  edition 2024, `rust-version = "1.85"`) with its own SQLite persistence (`rusqlite` bundled, tables
  prefixed `native_`, idempotent migrations at init). Chosen "after the .NET NativeAOT feasibility probe
  showed that EF Core 10 could not produce this library reliably"; it does not link the .NET assemblies
  (A:native/README.md intro, "Persistence boundary"; A:native/fortuna-core/Cargo.toml; A:TSD §4.1).
- Outputs `fortuna_core.dll` (Windows x64) / `libfortuna_core.so` (Linux x64) + committed header
  `native/fortuna-core/include/fortuna_core.h` generated by `cbindgen` 0.29.4 (A:native/README; A:OPS IR-21).
- `build.rs` generates the operation table and C header **from `docs/openapi/fortuna.json`** — the
  contract is a build input (A:.github/workflows/native-core.yml header comment; A:CONTRIBUTING "OpenAPI document").
- **C ABI** (A:native/README "ABI contract"): every function except `fortuna_string_free` returns an
  HTTP-compatible status and writes UTF-8 JSON to `char **response_json`; envelope `data, messages,
  errors, timestamp, success`; caller frees every non-null response with `fortuna_string_free` once; no
  panic unwinds; thread-safe, one SQLite connection per operation.
  - Lifecycle exports: `fortuna_initialize` (`{"databasePath":…,"localAuthEnabled":true,"tokenLifetimeSeconds":3600}`),
    `fortuna_shutdown`, `fortuna_version`, `fortuna_health`, `fortuna_capabilities`, `fortuna_string_free`.
  - Route request shape `{"token":…,"route":{…},"query":{…},"body":{…}}`; names `fortuna_<path segments>_<method>`,
    route params `by_<name>`, e.g. `fortuna_api_transactions_by_id_put` (A:native/README; A:SRD §5).
  - **115 generated route exports** (A:README; A:SRD §5). The header has 117 `fortuna_api_*` symbols:
    the 115 plus `fortuna_api_local_accounts_authenticate` and `fortuna_api_accounts_get_by_id`, listed
    separately in the README table (counted from `fortuna_core.h`).
  - Status macros: `FORTUNA_STATUS_OK 200`, `CREATED 201`, `ACCEPTED 202`, `BAD_REQUEST 400`,
    `UNAUTHORIZED 401`, `NOT_FOUND 404`, `CONFLICT 409`, `INTERNAL_ERROR 500`, `NOT_IMPLEMENTED 501`,
    `NOT_INITIALIZED 503` (A:native/fortuna-core/include/fortuna_core.h). Call before init → `503` (A:OPS IR-23).
  - `fortuna_capabilities` → `operations` (implemented), `notImplemented` (exported, answers `501`, with
    `reason`; 26 routes), `unavailable` (not exported) (A:native/README; A:CHANGELOG [2.0.0] Changed "Breaking (native C ABI)").
  - Local tokens opaque, stored as SHA-256 hashes, invalidated at shutdown; Argon2id secrets; zeroize (A:native/README; A:TSD §6).
- OpenAPI operations **not exported natively (30)**: all 15 `/api/auth/*`, 6 connections incl. sync,
  `GET /api/data-sources`, `POST /api/exchange-rates/sync`, `POST /api/local-accounts/password-reset`,
  3 `/api/me/consents*`, `DELETE /api/users/{id}`, `GET /healthcheck`, `GET /healthcheck/detailed`
  (computed: OAS 145 ops − header symbols; matches A:SRD §5 intro list).

### What is unavailable offline and why (A:native/README "Exported but not implemented offline"; A:UCS §1.4)
| Routes (answer `501`) | Reason (core's) |
|---|---|
| `POST /api/imports/excel`, `POST /api/imports/pdf`, `POST /api/import-jobs/{id}/retry` | No native workbook/PDF parser |
| `POST /api/exports` | No native data-set export renderer (personal archive is available) |
| `/api/reports/*` (4), `/api/projections/*` (2) | Reports and projections not computed natively |
| `/api/transfers…` (4) | Paired legs, conversion and cascade not implemented |
| `/api/installment-plans…` (4) | Splitting into billing cycles not implemented |
| `GET /api/statements/{id}`, `POST …/close`, `POST …/settle`, `GET /api/credit-cards/{id}/statements` | Charges not assigned to billing cycles natively |
| `POST /api/recurring-transactions/materialize` | Occurrences not materialized natively |
| `GET /api/budgets/{id}/consumption`, `GET /api/goals/{id}/progress` | Not computed natively |
| `POST /api/transactions/{id}/reconcile` | Needs imported records |
- Deliberately absent (network/host): connected identity, admin user erasure, Pluggy, remote rate sync,
  HTTP health routes, hosted consent (A:native/README; A:SRD §5).
- UI wording: shown as **"Not available offline"** with the core's reason; capabilities read **once** when
  offline mode starts; a `501` that still arrives is `FailureKind.unavailableOffline`, no retry
  (U:README "What it does"; U:native/README; U:CHANGELOG [1.0.0] Changed; U:`lib/core/network/offline_capabilities.dart`).
- Tracking: open issue A#200 "Native core: implement the operations that now return 501 offline"; A#201
  "Native core: generic records are stored without validation" (e.g. accepts `{"amount":"abc"}`,
  search ignores filters) (gh issue view).

### Data stores
- Shared: PostgreSQL, schema `fortuna`, `Search Path` pinned, snake_case singular naming; not a compose
  service — each machine's own PostgreSQL via `host.docker.internal` (A:OPS IR-04/05, §6; A:docker-compose.yml).
- Offline desktop: native core's own SQLite file (path from `fortuna_initialize`; UI `FORTUNA_DB_PATH`,
  default beside the executable) (A:native/README; U:OPS §3).
- Alternative local HTTP: EF SQLite (A:README "Running").
- Binary: Filesystem (`FORTUNA_STORAGE_PATH`, compose `/app/attachments` volume) or S3 (MEGA S4) (A:TSD §4.3; A:docker-compose.yml).
- UI stores only the token (flutter_secure_storage: Keystore/DPAPI/libsecret/WebCrypto) and preferences
  (shared_preferences); no financial data (U:TSD §4).

### Integrations (A:TSD §6.1; A:VIS §5)
- **Heimdall API**: Fortuna is a Heimdall **scope**; typed `HttpClient` proxies `api/auth/*` exchanges
  attaching `FORTUNA_HEIMDALL_SCOPE_ID`; tokens (HS256 JWT) validated **locally** with
  `FORTUNA_AUTH_TOKEN_SECRET` (= Heimdall signing secret, ≥ 32 chars), issuer, audience; previous secret
  for rotation. Optional `FORTUNA_HEIMDALL_CALLER_KEY` sends `X-Heimdall-Caller-Key` and
  `X-Heimdall-Client-Address` (A:OPS §3, §4.1; A:`docker/local.env.example`).
  Heimdall paths called: `api/auth/login`, `google`, `google/sign-out`, `2fa`, `2fa/verify`,
  `2fa/challenge/resend`, `2fa/enable`, `2fa/confirm`, `2fa/disable`, `2fa/recovery-codes/regenerate`,
  `password-recovery`, `password-reset`, `register`, `resend-verification`, `verify-email`
  (A:`src/Infrastructure/ArturRios.Fortuna.Integration/Security/HeimdallAuthGateway.cs`).
- **Pluggy**: typed HTTP client behind the source abstraction (`PluggyApiKeyClient`, `PluggyConnectionGateway`,
  `PluggySynchronizationGateway`); env `FORTUNA_PLUGGY_CLIENT_ID/_SECRET/_BASE_URL`.
- **BCB PTAX**: `PtaxRateClient`, `HttpRetryPolicy`; env `FORTUNA_RATES_SOURCE_BASE_URL`, `_SYNC_CRON`, `_CURRENCIES`.
- **MEGA S4**: S3-compatible via `AWSSDK.S3`.
- **Google**: UI obtains a Google ID token (`google_sign_in`), API exchanges it via Heimdall (U:BR BR-20; A:SRD FR-ID-18).
- **Email**: no email integration in Fortuna; verification/recovery emails are Heimdall's (Fortuna only
  forwards `password-recovery`, `resend-verification`, `verify-email`) (A:SRD FR-ID-25…27; HeimdallAuthGateway).

### How the UI talks to the API vs the native core (U:SRD §2; U:OPS §2.2; U:TSD §5)
- Features → repository interfaces → one implementation per transport; **HTTP**: `dio` + generated
  retrofit client (`packages/fortuna_api_client`, from `api/fortuna.json` via `swagger_parser`);
  **FFI**: `dart:ffi` bindings generated by `ffigen` from vendored `native/include/fortuna_core.h` into
  `lib/core/bindings/fortuna_bindings.dart` + generated route table `lib/core/bindings/fortuna_routes.dart`.
  Transport chosen at build time by `FORTUNA_TRANSPORT_FFI` (U:`lib/core/config/app_config.dart`).
- FFI calls run on a worker isolate (`IsolateCoreDispatcher`, U:`lib/core/bindings/core_dispatcher_native.dart`);
  `dart:ffi` confined to bindings layer, enforced by `tool/check_boundaries.dart` (U:OPS IR-13; U:ci.yml).
- Library lookup: beside the executable first, then `native/windows|linux/` in the repo; loading (not
  file existence) proves availability; failure → offline not offered (U:`lib/core/bindings/core_library_native.dart`).
- FFI payloads: same JSON bodies and envelope as HTTP, so generated models are reused (U:TSD §5.1).
- Contract version check: client constant `ApiContract.version = 'v1'` compared with `/healthcheck`
  `contractVersion`; mismatch or absence → incompatible (U:`lib/core/config/api_contract.dart`; UC-01 AF-05).

### Diagram material
1. **Component graph**: U:SRD §2 mermaid (Targets → App shell → Features → Repositories → HTTP transport
   / FFI bindings → Fortuna API / Fortuna core library; Core → Google). A:SRD §2 mermaid (Identity and
   Access, domain modules, Ingestion, Reporting, Export, Attachments, Job runner → PostgreSQL/SQLite,
   Filesystem/S3, Heimdall, Pluggy, BCB PTAX). A:OPS §2.2 mermaid (projects + Native fortuna_core + SQLite `native_` tables).
2. **Deployment modes**: Shared = browser/desktop/Android → Traefik (`<ui host>/api/` and API host) →
   fortuna-api container (8080; metrics 9464 private) → host PostgreSQL, attachments volume/S3, Heimdall
   (HTTPS), Pluggy, PTAX; web UI = nginx-unprivileged container on 8080 (A:OPS §6; U:OPS §6.3; U:Dockerfile).
   Desktop offline = Flutter app + `fortuna_core.dll`/`.so` + SQLite file in-process, no socket, no
   Heimdall/Pluggy/PTAX (A:OPS §6 "A desktop installation…"; A:VIS §5 "A desktop installation can be an island").
3. **Import pipeline**: upload (multipart) → validate → job persisted, `202`/job id → queue → Job Runner →
   source parser (Excel/Nubank PDF/Pluggy) → ImportedRecord stored raw → duplicate check (account, date,
   amount, external id) → Transaction created linked to record → statement assignment (cards) →
   counts; UI polls import-jobs, shows per-row records (A:UCS UC-56, UC-59, UC-60, UC-61, UC-63; A:SRD FR-IM-03…11).
4. **Sign-in via Heimdall**: UI `POST /api/auth/login` {email,password} → Fortuna adds `scopeId` →
   Heimdall `api/auth/login` → token or 2FA challenge → (challenge) UI `POST /api/auth/2fa/verify` →
   token → UI stores in secure storage → later requests `Authorization: Bearer` → Fortuna validates
   signature/issuer/audience/expiry locally, resolves subject, provisions profile on first access (UC-02)
   (A:UCS UC-01, UC-02, UC-76; U:BR BR-16, BR-20). Google: UI gets ID token → `POST /api/auth/google`.
   Anonymous auth limited to 10/min per source address (IPv6 /64) (A:UCS UC-76; A:OPS §4.1).
5. **Chart drill-down**: UI `GET /api/reports/aggregate` (Dimension, Granularity, From, To,
   RollupCategories, filters, DisplayCurrencyCode, paging) → buckets with label, total, share,
   drill-down key → user taps element (value read from data, not coordinate) → `GET
   /api/reports/drill-down?Key=…&Dimension=…` → finer buckets … → transactions at finest level; UI keeps a
   DrillPath for stepping back; key TTL `FORTUNA_REPORT_KEY_TTL_MINUTES` (example 15); range max
   `FORTUNA_REPORT_MAX_RANGE_DAYS` (example 366) (A:UCS UC-68, UC-69; A:OAS; U:UCS UC-36, UC-37; U:BR BR-10, BR-13, BR-14).
- Ready-made mermaid already in repos: A:VIS §5 C4Context, §7 erDiagram, §8 roles; A:SRD §8 deletion
  flowchart; A:UCS §4 state diagrams (record, statement, transaction, import job, connection, recovery
  code); A:OPS §5.4 detailed-health sequence; U:VIS §5 C4Context; U:SRD §8 session flowchart; U:UCS §4
  (session, view state, job handle, drill path); A:DWD §2 workflow flowchart.

---

## 4. Install and run

### Prerequisites (exact where pinned)
- API: **.NET 10 SDK** (`net10.0`, C# 14); PostgreSQL (shared) or SQLite (offline); **Rust stable**
  (MSRV `1.85`, edition 2024) for the native core; Docker for container workflow and functional tests
  (Testcontainers); Python 3 for `scripts/` (A:README "Installation"; A:CONTRIBUTING "Prerequisites"; A:TSD §2).
- Local dotnet tools pinned in `.config/dotnet-tools.json`: `dotnet-ef` 10.0.12,
  `dotnet-reportgenerator-globaltool` 5.5.11, `swashbuckle.aspnetcore.cli` 10.3.0 (A:.config/dotnet-tools.json).
- UI: **Flutter SDK stable, latest** (no pin in docs, U:TSD §1.1); `pubspec.yaml` `sdk: ^3.13.2`;
  `pubspec.lock` sdks `dart: ">=3.13.2 <4.0.0"`, `flutter: ">=3.44.0"`; Docker image pins Flutter
  **3.47.4** ("the first 3.47 release is 3.47.2, the earliest Flutter whose Dart satisfies `sdk: ^3.13.2`")
  (U:README "Container image (web)"; U:Dockerfile). Linux desktop toolchain: `clang cmake ninja-build
  pkg-config libgtk-3-dev liblzma-dev libstdc++-12-dev libsecret-1-dev`; Android build uses Temurin Java 17 (U:.github/workflows/build.yml).

### API configuration (names only) — all `FORTUNA_` env vars (A:docker-compose.yml; A:OPS §3; A:docker/*.env.example)
- Runtime: `ASPNETCORE_ENVIRONMENT`, `ASPNETCORE_HTTP_PORTS` (entrypoint-derived).
- Database: `FORTUNA_DATA_CONNECTIONSTRING`, `FORTUNA_DATA_DATABASETYPE` (`PostgreSql`|`SQLite`),
  `FORTUNA_RUN_MIGRATIONS`; compose fallbacks `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`,
  `DB_SEARCH_PATH`, `DB_CONNECTION_EXTRA`.
- Token: `FORTUNA_AUTH_TOKEN_SECRET`, `FORTUNA_AUTH_TOKEN_SECRET_PREVIOUS`, `FORTUNA_AUTH_TOKEN_ISSUER`,
  `FORTUNA_AUTH_TOKEN_AUDIENCE`, `FORTUNA_AUTH_TOKEN_EXPIRATION_IN_SECONDS` (default 3600).
- Heimdall: `FORTUNA_HEIMDALL_BASE_URL` (must be HTTPS), `FORTUNA_HEIMDALL_SCOPE_ID`, `FORTUNA_HEIMDALL_CALLER_KEY`.
- Profile: `FORTUNA_DEFAULT_DISPLAY_CURRENCY`, `FORTUNA_LOCALE` (required).
- Local auth: `FORTUNA_LOCAL_AUTH_ENABLED` (compose default false), `FORTUNA_LOCAL_AUTH_RECOVERY_CODE_COUNT` (10).
- Consent: `FORTUNA_CONSENT_EXTERNAL_PROCESSING_VERSION`.
- Pluggy: `FORTUNA_PLUGGY_CLIENT_ID`, `FORTUNA_PLUGGY_CLIENT_SECRET`, `FORTUNA_PLUGGY_BASE_URL`.
- Rates: `FORTUNA_RATES_SOURCE_BASE_URL`, `FORTUNA_RATES_SYNC_CRON`, `FORTUNA_RATES_CURRENCIES`.
- Storage: `FORTUNA_STORAGE_PROVIDER` (`Filesystem`|`S3`), `FORTUNA_STORAGE_PATH`,
  `FORTUNA_STORAGE_S3_ENDPOINT`, `FORTUNA_STORAGE_S3_BUCKET`, `FORTUNA_STORAGE_S3_ACCESS_KEY`, `FORTUNA_STORAGE_S3_SECRET_KEY`.
- Limits: `FORTUNA_UPLOAD_MAX_BYTES`, `FORTUNA_UPLOAD_ALLOWED_CONTENT_TYPES`, `FORTUNA_EXCEL_IMPORT_MAX_BYTES`,
  `FORTUNA_PDF_IMPORT_MAX_BYTES`, `FORTUNA_PAGE_SIZE_MAX`, `FORTUNA_REPORT_MAX_RANGE_DAYS`,
  `FORTUNA_REPORT_KEY_TTL_MINUTES`, `FORTUNA_PROJECTION_MAX_HORIZON_DAYS`, `FORTUNA_TRANSACTION_MAX_TAGS`.
- Jobs/exports/health: `FORTUNA_JOB_QUEUE_CAPACITY`, `FORTUNA_EXPORT_SYNC_THRESHOLD_ROWS`,
  `FORTUNA_EXPORT_RETENTION_HOURS`, `FORTUNA_HEALTH_JOB_MAX_PENDING_SECONDS` (default 300).
- Reconciliation: `FORTUNA_RECONCILIATION_AMOUNT_TOLERANCE`, `FORTUNA_RECONCILIATION_DATE_TOLERANCE_DAYS`.
- Observability/proxy: `FORTUNA_METRICS_PORT` (default 9464; 0 disables), `FORTUNA_LOG_DIRECTORY`,
  `FORTUNA_FORWARDED_KNOWN_NETWORKS`, `FORTUNA_FORWARDED_KNOWN_PROXIES`.
- Compose/deploy only: `API_IMAGE`, `API_IMAGE_TAG`, `API_PORT`; templates also carry `PUBLIC_HOST`, `UI_HOST`.
- Listed in A:OPS §3 but **not** forwarded by compose and not found in WebApi code: `FORTUNA_CORS_ALLOWED_ORIGINS`, `FORTUNA_LOG_LEVEL` (see §10).

### API: clone, build, run (A:README; A:CONTRIBUTING)
```bash
git clone https://github.com/artur-rios/fortuna-api.git
dotnet tool restore
dotnet restore src/ArturRios.Fortuna.sln
dotnet build src/ArturRios.Fortuna.sln -m:1
cp docker/local.env.example docker/local.env
docker compose --env-file docker/local.env up -d --build      # API on http://localhost:8083
```
- Local signs in with Fortuna's own local account (`FORTUNA_LOCAL_AUTH_ENABLED=true`) because the local
  heimdall-api `http://localhost:8080` is plain HTTP (A:README "Running"; A:docker/local.env.example).
- Compose: one service `api`, image `${API_IMAGE:-fortuna-api}:${API_IMAGE_TAG:-latest}`, port
  `${API_PORT:-8080}:8080`, volumes `logs`, `attachments`, healthcheck `curl --fail --silent
  http://localhost:8080/healthcheck` every 15s, `extra_hosts host.docker.internal:host-gateway`;
  PostgreSQL is not a service (A:docker-compose.yml).
- Dockerfile: `mcr.microsoft.com/dotnet/sdk:10.0` (digest-pinned) build → `dotnet publish` → `dotnet ef
  migrations bundle` to `/app/fortuna-migrate`; runtime `mcr.microsoft.com/dotnet/aspnet:10.0` (digest-pinned),
  runs as `$APP_UID`, `EXPOSE 8080 9464`, entrypoint `docker/entrypoint.sh` (A:Dockerfile).
- **Migrations**: entrypoint runs `/app/fortuna-migrate` when `FORTUNA_RUN_MIGRATIONS` is `true`
  (default) using `FORTUNA_DATA_CONNECTIONSTRING`; developer menu `python3 scripts/migrations.py`
  (lists/creates/applies EF migrations, reads env files under the Web API's `Environments` folder)
  (A:docker/entrypoint.sh; A:CONTRIBUTING "Migrations").
- Desktop/SQLite via managed host: `FORTUNA_DATA_DATABASETYPE=SQLite`,
  `FORTUNA_DATA_CONNECTIONSTRING=/absolute/path/to/fortuna.db`, `FORTUNA_RUN_MIGRATIONS=true` (A:README "Running").
- Tests: `dotnet test src/ArturRios.Fortuna.sln -m:1` (+ `--filter "Category=Unit"` / `"Category=Functional"`),
  `cargo test --manifest-path native/fortuna-core/Cargo.toml --locked`, coverage `python3 scripts/coverage.py`
  (90% merged line floor) (A:CONTRIBUTING "Testing"; A:SRD NFR-25).
- Other checks: `python3 -m unittest discover -s scripts -p "test_*.py"`, `python3 scripts/check_blank_lines.py`,
  `python3 scripts/vulnerabilities.py`, `python3 scripts/openapi.py --check`; regenerate OpenAPI `python3 scripts/openapi.py` (A:CONTRIBUTING).

### Native core build (A:native/README "Build and test")
```bash
cargo test  --manifest-path native/fortuna-core/Cargo.toml
cargo build --manifest-path native/fortuna-core/Cargo.toml --release
git diff --exit-code -- native/fortuna-core/include/fortuna_core.h   # header drift check
```
- Crates: argon2 0.6.0, base64 0.23.1, chrono 0.4.45, rusqlite 0.40.2 (bundled), serde 1.0.228,
  serde_json 1.0.151 (arbitrary_precision), sha2 0.11.0, rust_decimal 1.40.0, uuid 1.26.0, zeroize 1.9.0,
  zip 7.2.0; build: cbindgen 0.29.4 (A:native/fortuna-core/Cargo.toml). (A:TSD §3.3 says serde `1.0.229` — see §10.)

### UI: run per target (U:README "Installation", "Running"; U:CONTRIBUTING)
```bash
git clone https://github.com/artur-rios/fortuna-ui.git
cd fortuna-ui
flutter pub get
cp config/local.json.example config/local.json   # {"FORTUNA_API_BASE_URL":"http://localhost:8083","FORTUNA_GOOGLE_CLIENT_ID":""}
flutter run -d windows --dart-define-from-file=config/local.json
flutter run -d chrome --dart-define-from-file=config/local.json --web-browser-flag=--disable-web-security
```
- Replace `-d windows` with `linux`, `chrome` or an Android device; emulator reaches host at `http://10.0.2.2:8083`.
- No `FORTUNA_API_BASE_URL` → app starts at setup screen. Offline: `--dart-define=FORTUNA_TRANSPORT_FFI=true`,
  optional `FORTUNA_DB_PATH`; requires the core library beside the executable or in `native/<os>/` (U:README; U:OPS §3).
- Build-time defines (all public, compiled into the artifact): `FORTUNA_API_BASE_URL`,
  `FORTUNA_GOOGLE_CLIENT_ID`, `FORTUNA_TRANSPORT_FFI`, `FORTUNA_DB_PATH` (U:`lib/core/config/app_config.dart`; U:OPS §3).
- Generation: `dart run tool/generate_api_client.dart` (after copying `docs/openapi/fortuna.json` →
  `api/fortuna.json`); `dart run tool/generate_bindings.dart` (needs vendored header) (U:CONTRIBUTING "Generated code").
- Tests: `flutter analyze`, `flutter test`, `dart format --output=none --set-exit-if-changed lib test tool`,
  `dart run tool/check_boundaries.dart`; integration `flutter test integration_test -d windows` (U:CONTRIBUTING "Testing").
- Web image: `docker build -t fortuna-ui:web --build-arg FORTUNA_API_BASE_URL=… --build-arg
  FORTUNA_GOOGLE_CLIENT_ID=… .` then `docker run --rm -p 8080:8080 fortuna-ui:web`; build args
  `FORTUNA_API_BASE_URL`, `FORTUNA_GOOGLE_CLIENT_ID`, `FLUTTER_VERSION`/`FLUTTER_SHA256`; built with
  `--no-web-resources-cdn` and `FORTUNA_TRANSPORT_FFI=false`; runtime `nginxinc/nginx-unprivileged:alpine`
  on 8080, `/healthz` returns `ok`, SPA fallback `try_files $uri $uri/ /index.html` (U:README; U:Dockerfile; U:docker/nginx.conf).
  Warning: build args "are public. Never pass a secret" (U:README).

### Release artifacts — what is actually produced
| Artifact | Produced by | Where it goes |
|---|---|---|
| API Docker image | `tests.yml` job `docker` builds with `push: false` (CI only); deploy image built by yggdrasil Jenkins, tagged `x.y.z-<short sha>` | Not pushed to any public registry from these workflows (A:tests.yml; A:CONTRIBUTING "Versioning") |
| Native libraries | `native-core.yml` matrix → workflow artifacts `fortuna-native-linux-x64` (`libfortuna_core.so` + header) and `fortuna-native-windows-x64` (`fortuna_core.dll` + header); runs only when `native/**` or the OpenAPI doc change | GitHub Actions artifacts, not releases (A:.github/workflows/native-core.yml) |
| Coverage/test results | `tests.yml` → `coverage-report`, `test-results` | Actions artifacts |
| UI web | `build.yml` (on `v*` tags + manual) → `fortuna-ui-web` (`build/web`) | Actions artifact; deployed image built by yggdrasil per environment (U:build.yml; U:Jenkinsfile) |
| UI Android | `build.yml` → `fortuna-ui-android` (`app-release.apk`) | Actions artifact |
| UI Windows | `build.yml` → `fortuna-ui-windows` (`build/windows/x64/runner/Release`) | Actions artifact — raw folder, no installer |
| UI Linux | `build.yml` → `fortuna-ui-linux` (`build/linux/x64/release/bundle`) | Actions artifact — raw bundle, no installer |
| GitHub releases | Jenkins creates tag + GitHub release on release merge | `fortuna-api` v2.0.0 and `fortuna-ui` v1.0.0 releases have **no assets** (`gh release view --json assets` → `[]`) |
- build.yml header: "The desktop jobs build the client only. The packaging of IR-21 — the installer and
  the portable archive, which also carry the Fortuna core library and the SQLite database file — waits
  on that library existing (artur-rios/fortuna-api#156)." (U:.github/workflows/build.yml). #156 is closed.

---

## 5. Usage (UI)

### Use cases (U:UCS headings and Description rows; status from U:README Backlog + `gh issue list`)
**Foundation** — UC-01 Configure the Instance and Mode (address/mode, offline only if core present,
contract-version check); UC-02 Reach the Fortuna Core Over the Configured Transport (mechanism); UC-46
Guard a Route by Session and Role (mechanism).
**Session and identity** — UC-03 Sign In with Credentials; UC-04 Complete a Two-Factor Challenge
(**open, #5**; `/sign-in/two-factor` is a `PlaceholderScreen` in `lib/app/router.dart`); UC-05 Sign In with
Google (first sign-in creates the account); UC-47 Sign Up (Google or name/email/password; account exists
only after emailed link + re-entered password); UC-06 Create a Desktop Local Account (codes shown once);
UC-07 Sign In to a Desktop Local Account; UC-08 Recover a Desktop Local Account (code spent, new
secret, offer regeneration); UC-09 Recover a Password and Verify an Address; UC-10 Manage Two-Factor
Authentication (app, email or both; QR + text secret); UC-11 Restore a Session at Start; UC-12 End a
Session; UC-13 Choose Theme, Locale and Display Currency (locales `en-US`, `en-GB`, `en-150`, `pt-BR`; light/dark/system).
**Holdings** — UC-14 Manage Financial Accounts; UC-15 Manage Credit Cards; UC-16 Review and Settle a
Credit Card Statement (close is irreversible; settlement shown as transfer; late-arriving marked);
UC-17 Manage Investments; UC-18 Record an Investment Movement or Valuation.
**Money movement** — UC-19 Record a Transaction; UC-20 Update or Delete a Transaction (shows source and
imported record); UC-21 Record a Transfer; UC-22 Record an Installment Purchase; UC-23 Manage Recurring
Commitments (**open, #24**; no route exists in `lib/app/routes.dart`); UC-24 Reconcile a Transaction;
UC-25 Explore Records as a Spreadsheet.
**Organization and planning** — UC-26 Manage the Category Tree; UC-27 Manage Tags and Counterparties;
UC-28 Manage Budgets; UC-29 Manage Goals.
**Ingestion** — UC-30 Connect an Institution (consent first); UC-31 Synchronize, Reauthenticate or
Revoke a Connection; UC-32 Import a File (type/size checked before upload, uploaded whole); UC-33
Monitor an Import Job; UC-34 Review Imported Records; UC-35 Manage a Transaction's Attachments (**open, #36**).
**Insight and output** — UC-36 Read an Aggregation as a Chart (by period, category, account or
counterparty; remainder group drillable); UC-37 Drill Into a Chart Aggregation; UC-38 View the Net
Position, Projections and Obligations; UC-39 Export a Data Set.
**Lifecycle, privacy, administration** — UC-40 Restore or Permanently Remove a Deleted Record; UC-41
Read the Audit Trail; UC-42 Review, Give and Withdraw Consents; UC-43 Export All Personal Data; UC-44
Erase the Account; UC-45 View Instance Health (admin only).

### Screens / routes as implemented (U:`lib/app/routes.dart`, `lib/app/router.dart`)
- Anonymous: `/setup` (SetupScreen), `/sign-in` (SignInScreen), `/sign-up` (SignUpScreen),
  `/sign-in/two-factor` (placeholder, UC-04), `/password-recovery`, `/password-reset?token=`,
  `/verify-email?token=`. Destination kept in query param `from` (UC-46 AF-02).
- Owner: `/` Overview (**PlaceholderScreen, "implementedBy: UC-38"**), `/accounts`, `/credit-cards`,
  `/credit-cards/:id/statements`, `/credit-cards/:id/statements/:statementId`, `/investments`,
  `/investments/:id`, `/transactions` (spreadsheet), `/transactions/new`, `/transactions/:id`,
  `/transfers/new`, `/installments/new`, `/budgets`, `/goals`, `/categories`, `/labels` (tags and
  counterparties), `/insight` (charts), `/insight/projections`, `/deleted`, `/audit`, `/sources`,
  `/connections`, `/imports`, `/imports/new`, `/imports/:id/records`.
- Both roles: `/settings`, `/privacy`. Administrator: `/admin` (InstanceHealthScreen).
- Local-account screens exist as widgets (`create_local_account_screen.dart`, `local_sign_in_screen.dart`,
  `local_recovery_screen.dart` in `lib/features/auth/ui/`) but have no own routes (spec routes
  `/sign-in/local*` not in routes.dart).
- `lib/features/`: administration, audit, auth, categories, holdings, ingestion, insight, labels,
  lifecycle, planning, preferences, privacy, session, setup, transactions; `lib/core/`: bindings, config,
  format (money, money_formatter, money_parser, supported_locales), network (api_client, ffi_transport,
  offline_capabilities, date_only_fields), result, session, storage; `lib/shared/widgets/`: `money_text.dart`,
  `offline_unavailable.dart`; 160 Dart files under `lib/` (directory listing).
- No navigation rail/drawer/bar found in `lib/` (grep); in-app navigation is via screen buttons; the
  home route is a placeholder (see §10).

### Offline mode user experience
- Offered only on Windows/Linux when the core library loads; never on web/Android (U:BR BR-25; U:SRD FR-CF-04/05).
- Sign-in paths: only local account offline; only identity-provider paths online (U:SRD FR-CF-07). No
  sign-up offline: "Offline, the local account has its own creation path" (U:`lib/features/auth/ui/sign_in_screen.dart` comment line 179).
- Local account: create (codes shown once, warned they cannot be shown again), sign in (same message for
  unknown name and wrong secret), recover with a code (states code spent and how many remain), no
  password reset — explains why (U:UCS UC-06…08).
- Unavailable features labeled "Not available offline" with the core's reason, read once at startup:
  file imports and import retry, data-set export, charts, net position and projections, transfers,
  installment purchases, card statements and closing/settling them, budget consumption, goal progress,
  reconciliation (U:CHANGELOG [1.0.0] Changed; U:UCS UC-02 AF-08). Older cores without
  `notImplemented` still work (U:CHANGELOG [1.0.0]).
- A stored session the core cannot verify offline is discarded with the reason (U:CHANGELOG [1.0.0] Changed).
- The database is beside the executable by default so a portable copy keeps its data (U:OPS §3, §6.2).
- Personal data export works offline (native archive) (A:native/README).

---

## 6. API reference summary

### OpenAPI document (A:OAS)
- `openapi: 3.0.4`; `info.title: "Fortuna API"`; `info.version: "v1"` (the "contract version", independent
  of release numbers — A:CONTRIBUTING "Versioning"). No `servers`. 109 paths, **145 operations**, 356
  component schemas, 31 tags.
- Security scheme `Bearer`: `type: http`, `scheme: Bearer`, `bearerFormat: JWT`, description "Enter the
  Heimdall JWT bearer token." Top-level `security: [{}]` and **no per-operation `security`** — the
  document does not mark which operations are anonymous (see §10).
- Generated from code with Swashbuckle (`python3 scripts/openapi.py`), CI-checked by `check-openapi.yml` (A:CONTRIBUTING).
- Copy in UI: `U:api/fortuna.json` is identical in paths/ops but lags: lacks the `429` responses on
  `register` and `verify-email` added on develop (diff of the two files).

### Endpoint groups (tag → op count) (A:OAS)
| Tag | Ops | Operations |
|---|---|---|
| Accounts | 8 | `GET/POST /api/accounts`, `GET/PUT/DELETE /api/accounts/{id}`, `GET …/balance`, `DELETE …/hard`, `POST …/restore` |
| Attachments | 3 | `GET/DELETE /api/attachments/{id}`, `DELETE …/hard` |
| AuditEntries | 1 | `GET /api/audit-entries` |
| Auth | 15 | `GET /api/auth/2fa`; `POST /api/auth/` `2fa/challenge/resend`, `2fa/confirm`, `2fa/disable`, `2fa/enable`, `2fa/recovery-codes/regenerate`, `2fa/verify`, `google`, `google/sign-out`, `login`, `password-recovery`, `password-reset`, `register`, `resend-verification`, `verify-email` |
| Budgets | 6 | CRUD + `GET /api/budgets/{id}/consumption` |
| Categories | 8 | CRUD + `DELETE …/hard`, `POST …/reassign`, `POST …/restore` |
| Connections | 5 | `GET/POST /api/connections`, `GET …/{id}`, `POST …/reauthenticate`, `POST …/revoke` |
| ConnectionSynchronization | 1 | `POST /api/connections/{id}/sync` |
| Counterparties | 6 | `GET/POST`, `PUT/DELETE {id}`, `POST …/merge`, `GET …/suggested-category` |
| CreditCards | 8 | CRUD + `DELETE …/hard`, `POST …/restore`, `GET …/statements` |
| Currencies | 2 | `GET /api/currencies`, `GET /api/currencies/{code}` |
| DataSources | 1 | `GET /api/data-sources` |
| DetailedHealthCheck | 1 | `GET /healthcheck/detailed` |
| ExchangeRates | 3 | `POST /api/exchange-rates`, `POST …/convert`, `POST …/sync` |
| Exports | 2 | `POST /api/exports`, `GET /api/exports/{id}` |
| Goals | 6 | CRUD + `GET …/progress` |
| HealthCheck | 1 | `GET /healthcheck` |
| ImportJobs | 4 | `GET /api/import-jobs`, `GET …/{id}`, `GET …/{id}/records`, `POST …/{id}/retry` |
| Imports | 2 | `POST /api/imports/excel`, `POST /api/imports/pdf` (multipart) |
| InstallmentPlans | 4 | `POST`, `GET/DELETE {id}`, `POST …/restore` |
| Investments | 10 | CRUD + `DELETE …/hard`, `POST …/restore`, `POST …/movements`, `GET/POST …/valuations` |
| LocalAccounts | 5 | `POST /api/local-accounts`, `…/authenticate`, `…/password-reset`, `…/recover`, `…/recovery-codes/regenerate` |
| Me | 7 | `GET /api/me`, `GET/POST /api/me/consents`, `DELETE …/consents/{purpose}`, `POST /api/me/data-export`, `GET …/data-export/{jobId}`, `POST /api/me/erasure` |
| Projections | 2 | `GET /api/projections/cash-flow`, `GET /api/projections/commitments` |
| RecurringTransactions | 6 | CRUD + `POST /api/recurring-transactions/materialize` |
| Reports | 4 | `GET …/aggregate`, `GET …/drill-down`, `GET …/net-position`, `POST /api/reports/table` |
| Statements | 3 | `GET /api/statements/{id}`, `POST …/close`, `POST …/settle` |
| Tags | 4 | `GET/POST`, `PUT/DELETE {id}` |
| Transactions | 12 | CRUD + `GET/POST …/attachments`, `DELETE …/hard`, `POST …/reconcile`, `POST …/restore`, `POST/DELETE …/tags/{tagId}` |
| Transfers | 4 | `POST`, `GET/DELETE {id}`, `POST …/restore` |
| Users | 1 | `DELETE /api/users/{id}` (admin erasure) |
- SRD endpoint surface by area: A:SRD §5.1 Identity, §5.2 Currency, §5.3 Accounts, §5.4 Cards and
  Statements, §5.5 Investments, §5.6 Transactions (incl. transfers, installments, recurring), §5.7
  Classification and Planning, §5.8 Ingestion, §5.9 Attachments/Reporting/Projection/Export/Audit, §5.10
  Health — each row traced to `FR-*`.
- Documented non-200 responses: `register` 202/400/403/429/503; `verify-email` 400/401/422/429/503;
  `exports` 202; `me/data-export` 202; `healthcheck/detailed` 503 (A:OAS). Other statuses come from the
  message-to-status maps and UC alternative flows (A:UCS).

### Authentication and authorization
- Global fallback policy `RequireAuthenticatedUser()`; anonymous endpoints marked `[AllowAnonymous]`
  (A:`src/Presentation/.../Extensions/FortunaWebApiServiceCollectionExtensions.cs`; A:SRD NFR-10).
- Anonymous: `/healthcheck`, local-account create/authenticate/recover (desktop mode only — `404` when
  local auth disabled), and the anonymous `api/auth` exchanges (login, google, 2fa/verify,
  2fa/challenge/resend, password-recovery, password-reset, register, verify-email) (A:SRD §5.1, FR-ID-15; A:UCS §1.1).
- Authorization matrix (A:SRD §7): Account Owner and Local Account: full CRUD on own records, reports,
  exports, attachments, audit; Local Account: imports only from files — no network source offline;
  Instance Administrator: own profile, instance health and job counts, configuration; user access via
  Heimdall; nobody reads another user's records. Admin targeting a financial record → `403` (A:UCS UC-01 AF-05).
- Admin-only: `GET /healthcheck/detailed` (403 otherwise), `DELETE /api/users/{id}`; manual rates and rate
  sync need admin or local owner (A:OPS §5.1; A:SRD §5.1; code messages above).
- UI matrix: A:SRD-equivalent at U:SRD §7.

### Error model
- Every response is the `DataOutput` envelope: `data`, `errors` (string[]), `messages` (string[]),
  `success` (read-only bool), `timestamp` (date-time) (A:OAS schema `ObjectDataOutput`; A:native/README "ABI contract").
- Handlers return errors rather than throw; message→status map per controller merged over shared map
  (e.g. storage unavailable → `503`; exchange rate unavailable → `409`; consent required → `403`)
  (A:TSD §6; A:`Controllers/FortunaStatusMap.cs`).
- Conventions: `401` missing/invalid token; `404` for missing **or foreign** record; validation `400`
  naming the field; `409` conflicts; anonymous credential failures collapsed to one non-enumerating
  `401`; Heimdall down → sanitized `503`; Heimdall `429` → `429` "Too many attempts. Wait a minute and
  try again." (A:UCS §1.1; A:SRD FR-ID-22/23/49, NFR-11, NFR-16; A:CHANGELOG [2.1.0] Changed).
- UI shows the API's reason from `errors` (U:CHANGELOG [1.0.0] Fixed; U:BR BR-05).
- Calendar dates are sent as `yyyy-MM-dd` — "the only form it accepts" (U:CHANGELOG [1.0.0] Fixed); OAS `format: date`.

### Pagination, filtering, sorting
- Query params `PageNumber`, `PageSize`, `SortBy`, `Descending`, `IncludeDeleted`; transaction search
  filters `From`, `To`, `FinancialAccountId`, `CreditCardId`, `CategoryId`, `TagId`, `CounterpartyId`,
  `Direction`, `MinimumAmount`, `MaximumAmount` (decimal strings), `Text`, `DisplayCurrencyCode`,
  `FigureDate` (A:OAS `GET /api/transactions`).
- Paged output: `items`, `pageNumber`, `pageSize`, `totalItems`, `totalPages`, plus `totals` for
  transaction search (A:OAS `TransactionSearchOutput`).
- Max page size `FORTUNA_PAGE_SIZE_MAX` (example 100) (A:docker/local.env.example).
- Aggregate: `Dimension`, `Granularity`, `From`, `To`, `RollupCategories` + filters; drill-down: `Key`,
  `Dimension`; cash flow: `HorizonDays`, `DisplayCurrencyCode`, `IncludeEstimate`, `Periodicity` (A:OAS).
- Granularities day/week/month/quarter/year; dimensions period, category, account, card, counterparty,
  tag (A:SRD FR-RP-02…04; A:UCS UC-68). `IncludeDeleted` on card and investment lists added by #178 (A:README).

### Rate limiting
- Policy on anonymous auth routes (8 in `AuthController`, 2 in `LocalAccountsController`): fixed window,
  `PermitLimit = 10`, `Window = 1 minute`, `QueueLimit = 0`, partitioned by client IP (IPv6 by /64),
  rejection `429` (A:`Extensions/FortunaWebApiServiceCollectionExtensions.cs`; grep count of
  `EnableRateLimiting`; A:OPS §4.1; A:CHANGELOG [2.1.0]).
- Heimdall's own anonymous budget is shared per Fortuna instance unless `FORTUNA_HEIMDALL_CALLER_KEY` is set (A:OPS §4.1).
- Outbound: external calls respect rate limits and back off (A:SRD NFR-23; UC-08 AF-05; UC-56 AF-06).

### Performance targets (A:SRD NFR-02…05; U:SRD NFR-01…03)
API p95: single read/page < 200 ms; write < 500 ms; aggregation/drill-down over a year < 1 s; jobs exempt.
UI: 60 fps, no frame > 16 ms; paint < 100 ms after response; cold start < 3 s desktop, < 5 s web.

---

## 7. Operations

### Environments (A:README "Running"; A:OPS §6; U:README "Environments"; U:OPS §5)
| Env | Where | Deployed by | API address | Web UI | DB | ASP.NET env |
|---|---|---|---|---|---|---|
| local | developer's Windows machine, Docker Desktop | by hand or yggdrasil `scripts/deploy.sh local …` | `http://localhost:8083` | `http://127.0.0.1:8082` | `fortuna_local` | Development |
| development | VPS, on demand | Jenkins on push to `develop` | `https://fortuna-api-dev.example.com` | `https://fortuna-dev.example.com` | `fortuna_development` | Development |
| homologation | VPS, on demand | Jenkins on push of `release/x.y.z` | `https://fortuna-api-hml.example.com` | `https://fortuna-hml.example.com` | `fortuna_homologation` | Production |
| production | VPS, always on | Jenkins on green `release/x.y.z → main` PR | `https://fortuna-api.example.com` | `https://fortuna.example.com` | `fortuna` | Production |
- `example.com` stands for the real domain. Traefik serves the API also under the UI host at `/api/`
  so the browser makes same-origin calls; desktop/Android use the API host. Docker networks
  `172.16.0.0/12` are the trusted proxies on the VPS. Env files at `/etc/yggdrasil/<environment>/fortuna-api.env`
  and `…/fortuna-ui.env`. Start/stop on demand: `scripts/ygg.sh env start|stop homologation` (A:OPS §6; A:CONTRIBUTING "Releasing").
- Heimdall per env: `https://heimdall-api-dev.example.com/`, `-hml`, `https://heimdall-api.example.com/` (A:CHANGELOG [2.0.0] Changed).
- Deployment platform: yggdrasil (`catalog.yaml` entries `fortuna-api`, `fortuna-ui`), Jenkinsfile
  `yggdrasilPipeline(stack: 'fortuna-api')` / `'fortuna-ui'`; console at `https://yggdrasil.<domain>`
  (A:Jenkinsfile; U:Jenkinsfile; A/U:CONTRIBUTING "Where this is deployed from").
- Desktop offline is "another shape rather than another environment" (A:OPS §6).

### Health
- `GET /healthcheck` public liveness: `{"contractVersion":"v1","service":"Fortuna API"}`, never touches
  dependencies (FR-HC-01, FR-HC-12; A:OPS §5.3; OAS `LivenessOutput`).
- `GET /healthcheck/detailed` admin only: `status` + `services[]` (Database, AttachmentStorage, JobRunner
  with `queueDepth`/`oldestPendingSeconds`, Aggregator, ExchangeRateSource); Healthy / Degraded (only
  optional external down) / Unhealthy (required down); 200 for Healthy/Degraded, 503 Unhealthy (A:OPS §5.2–5.3).
- Native: `fortuna_health` (A:UCS §1.4 UC-75 row).
- UI web container: `/healthz` (U:docker/nginx.conf; U:CONTRIBUTING).
- **UC-75 Check API Health** (A:OPS §5.4): actors monitor/anonymous (liveness), Instance Administrator
  (detailed); AF-01 non-admin → 403; AF-02 no token → 401; AF-03 DB down → 503 Unhealthy; AF-04 only
  aggregator/rate source down → 200 Degraded; AF-05 unconfigured external not counted; AF-06 oldest
  pending job over `FORTUNA_HEALTH_JOB_MAX_PENDING_SECONDS` → JobRunner Unhealthy; AF-07/08 liveness stable.

### Metrics
- OpenTelemetry → Prometheus text at `GET /metrics` only on private port `FORTUNA_METRICS_PORT` (9464),
  never on 8080; service name `fortuna-api`; meters ASP.NET Core, HttpClient, `System.Runtime`,
  `Microsoft.AspNetCore.Server.Kestrel`, `Microsoft.EntityFrameworkCore`, `Npgsql`; scrape not logged;
  native core has no metrics; UI serves none (A:OPS §5.5; U:CONTRIBUTING "Where this is deployed from").

### Logging
- Serilog structured JSON to console + daily file `FORTUNA_LOG_DIRECTORY/<yyyy>/<MM>/fortuna-<yyyyMMdd>.json`;
  correlation = W3C trace id (`traceparent`) propagated into jobs; request start logged
  `Started request with TraceId {TraceId} from {ClientIp}`; never logged: amounts, account identifiers or
  names, tokens, keys, secrets, recovery codes, attachment content, import contents, connection-string
  passwords (A:OPS §4). Client IP is personal data; trusted-proxy table in A:OPS §4.1.
- UI: `developer.log` debug builds only; release builds emit no logs; no crash reporting/analytics
  (U:OPS §4). Server observability Prometheus + Loki is "the API's concern" (U:TSD §6; U:OPS §4).

### Platform requirements
- **API IR-01…IR-27** (A:OPS §2.4): IR-01 layering (Integration ↛ Data); IR-02 central package mgmt with
  transitive pinning; IR-03 EF code-first + design-time factory; IR-04 snake_case, diagnostics only
  outside Production; IR-05 schema `fortuna` + pinned Search Path; IR-06 numeric(19,4)/(19,8); IR-07
  idempotent ISO 4217 seed; IR-08 env-only `FORTUNA_` config, refuse to start when required missing;
  IR-09 env files never baked into image; IR-10 hosted job runner, bounded queue + durable table; IR-11
  re-queue on start, no duplicates; IR-12 source registry; IR-13 attachment storage abstraction; IR-14
  OpenAPI served and CI-verified; IR-15 migration script never prints password; IR-16 tests mirror src,
  functional harness on real PostgreSQL; IR-17 CI stages; IR-18 single compose for every env; IR-19
  container healthcheck; IR-20 no Heimdall on request path, desktop with no network; IR-21 publish
  `fortuna_core.dll`/`libfortuna_core.so` + header; IR-22 native init with explicit DB path, shutdown
  invalidates sessions; IR-23 pre-init call → 503; IR-24 `fortuna_string_free` ownership; IR-25 native CI
  on Windows/Linux, header drift; IR-26 native namespaced SQLite schema, TEXT money; IR-27 metrics on
  private listener only.
- **UI IR-01…IR-22** (U:OPS §2.5): layout; feature organization; one central guard; sealed result type;
  money type with no float conversion; locale formatting (4 locales); secure storage/prefs wrappers;
  one dio client; generate client (IR-09) and bindings (IR-10); header drift fails build (IR-11); CI
  regenerates both (IR-12); `dart:ffi` only in bindings (IR-13); no float money (IR-14); worker isolate +
  string release (IR-15); `--dart-define` config, no secrets (IR-16); no release logs (IR-17); no
  analytics (IR-18); CI analyze+test (IR-19); CI builds all targets (IR-20); desktop packages with core
  + SQLite (IR-21); web image static, no secret (IR-22).

### CI workflows
- API (`.github/workflows/`): `tests.yml` (helpers → blank-line check → restore → vulnerability scan →
  Release build → category check → Unit → Functional on Testcontainers PostgreSQL → coverage ≥ 90% →
  upload; separate `docker` build job), `check-openapi.yml`, `native-core.yml` (MSRV job + Linux/Windows
  build matrix, fmt, clippy `-D warnings`, tests, release build, header drift, C/C++ header smoke test,
  upload), `audit.yml` (RustSec `rustsec/audit-check` on every PR), `branch-policy.yml` (A:.github/workflows; A:OPS §7).
- UI: `ci.yml` (format, analyze, boundaries, `flutter test --coverage`), `check-generated.yml`
  (api-client-drift, header-drift against `artur-rios/fortuna-api` checkout, bindings-drift),
  `branch-policy.yml`, `build.yml` (tags `v*` + manual) (U:.github/workflows).
- Branch model (both): `feature/<name>`/`fix/<name>` → `develop` → `release/x.y.z` → `main` (tag
  `vx.y.z`); release branches carry no commits; Branch Policy rejects reused versions; use-case branches
  `feature/uc-##-use-case-name`; issue Status Todo → In Progress → Testing → Done (A/U:CONTRIBUTING; A:DWD §3, Step 1).

---

## 8. Versions and status

### Released versions
| Repo | Tags (`git tag`) | GitHub releases (`gh release list`) | Changelog |
|---|---|---|---|
| fortuna-api | `v1.0.0`, `v2.0.0` | `v2.0.0` Latest, 2026-10-12T23:08:39Z (no assets); **no GitHub release for v1.0.0** ("release not found") | [1.0.0] 2026-09-19; [2.0.0] 2026-10-12; [2.1.0] 2026-10-13 on develop / release/2.1.0 (A:CHANGELOG) |
| fortuna-ui | `v1.0.0` | `v1.0.0` Latest, 2026-10-12T22:25:36Z (no assets) | [1.0.0] 2026-10-12; [Unreleased] adds UC-47 sign-up (U:CHANGELOG) |
- fortuna-api **2.1.0 is in flight**: open PR #212 "release: v2.1.0" (`release/2.1.0` → `main`), opened
  2026-10-13; not yet tagged (`gh pr list`). 2.1.0 = UC-81 register, verify-email password, caller key,
  429 mapping, IPv6 /64 limiter (A:CHANGELOG [2.1.0]).
- 2.0.0 was **major** because of the breaking native C ABI change (`501` + `notImplemented`) (A:CHANGELOG [2.0.0]; A:CONTRIBUTING "Versioning").
- Independent version numbers: API contract `v1` (OpenAPI `info.version`), native crate `0.1.0`, UI
  `pubspec.yaml` `version: 1.0.0+1` (A:CONTRIBUTING "Versioning"; A:Cargo.toml; U:pubspec.yaml).

### Backlog status
- API README: "> **Status:** version 1.0.0 is released, and every issue in the backlog below is
  closed." — still says 1.0.0 on main, develop and release/2.1.0 (A:README line 9; `git show`).
  GitHub: 92 closed, 2 open (#200, #201 — not on a milestone); milestones M-01…M-07 all 0 open (closed
  counts 6/17/20/12/11/11/15).
- UI README: "> **Status:** specification complete; implementation under way — 44 of the 47 issues are
  closed, and UC-04, UC-23 and UC-35 remain." GitHub: 45 closed, 3 open (#5 UC-04, #24 UC-23, #36
  UC-35); milestone M-02 has 18 closed + 1 open (incl. #108 UC-47) (U:README line ~22; `gh issue list`; milestones API).
- Project boards: API "Fortuna API" https://github.com/users/artur-rios/projects/12; UI "Fortuna UI"
  https://github.com/users/artur-rios/projects/14 (both: "One use case = one branch = one issue = one
  pull request; Status follows Todo, In Progress, Testing, Done.") (`gh project view`).

### Tech stack versions
- **API pinned** (A:Directory.Packages.props): ArturRios.Data.PostgreSql 3.0.2, ArturRios.Data.Relational.Core
  5.0.0, ArturRios.Mediator 2.0.0, ArturRios.Util 2.1.0, ArturRios.Util.Test 4.0.0, ArturRios.Util.WebApi 5.1.0,
  ClosedXML 0.105.1, PdfPig 0.1.16, EFCore.NamingConventions 10.0.1, Microsoft.EntityFrameworkCore(+Abstractions,
  Analyzers, Design, Relational, Sqlite) 10.0.12, Microsoft.AspNetCore.DataProtection(+EntityFrameworkCore) 10.0.12,
  Microsoft.AspNetCore.Mvc.Testing 10.0.12, AWSSDK.S3 4.0.104.1, FluentValidation 12.1.1, Serilog 4.4.0,
  Serilog.AspNetCore 10.0.0, Serilog.Sinks.Map 2.0.0, Swashbuckle.AspNetCore 10.3.0,
  OpenTelemetry.Extensions.Hosting 1.19.1, OpenTelemetry.Instrumentation.AspNetCore/Http 1.19.0,
  OpenTelemetry.Exporter.Prometheus.AspNetCore 1.19.1-beta.1, Bogus 35.6.5, Microsoft.NET.Test.Sdk 18.10.1,
  Moq 4.21.0, SSH.NET 2026.0.0, Testcontainers.PostgreSql 4.15.0, coverlet.collector 10.1.0, xunit 2.9.3,
  xunit.runner.visualstudio 4.0.0. `TreatWarningsAsErrors` true (A:Directory.Build.props).
- API platform: .NET 10 / C# 14; native Rust 2024, MSRV 1.85 (A:TSD §2).
- **UI resolved** (U:pubspec.lock): flutter_riverpod 3.4.3, go_router 18.0.2, dio 5.11.1, retrofit 4.10.0,
  json_annotation 4.12.0, decimal 3.2.6, intl 0.20.3, flutter_secure_storage 11.2.0, shared_preferences 2.5.5,
  file_picker 13.1.0, ffi 2.2.0, google_sign_in 7.2.0, qr_flutter 4.1.0, file_saver 0.4.0, fl_chart 1.2.0
  (pubspec `^1.1.1`), flutter_lints 6.0.0, mocktail 1.0.5, build_runner 2.16.1, json_serializable 6.14.1,
  retrofit_generator 10.2.11, swagger_parser 1.45.0, ffigen 22.0.0. Material 3 (U:TSD §2).

---

## 9. Link map

Base: `https://github.com/artur-rios/<repo>/blob/develop/…` (default branch `develop` for both; all files
below confirmed present on `develop` via `gh api repos/…/contents/…?ref=develop`). Use `%20` for spaces
and `%26` for `&` (the READMEs use `%26`; the API returned a raw `&`).

**fortuna-api — initial/**
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Brainstorm.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Project%20Overview.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Technology%20Stack.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Workflow.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md

**fortuna-api — requirements/**
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Vision%20Document.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/System%20Requirements%20Document.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Development%20Workflow%20Document.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Testing%20Specification%20Document.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Technology%20Stack%20Document.md
- https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Operations%20%26%20Infrastructure%20Document.md
- OpenAPI: https://github.com/artur-rios/fortuna-api/blob/develop/docs/openapi/fortuna.json
  (raw: https://raw.githubusercontent.com/artur-rios/fortuna-api/develop/docs/openapi/fortuna.json — standard raw pattern, not separately verified)
- Native: https://github.com/artur-rios/fortuna-api/blob/develop/native/README.md ,
  header https://github.com/artur-rios/fortuna-api/blob/develop/native/fortuna-core/include/fortuna_core.h

**fortuna-ui — initial/**
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Brainstorm.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Project%20Overview.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Technology%20Stack.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Workflow.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md

**fortuna-ui — requirements/**
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Vision%20Document.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/System%20Requirements%20Document.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Development%20Workflow%20Document.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Testing%20Specification%20Document.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Technology%20Stack%20Document.md
- https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Operations%20%26%20Infrastructure%20Document.md
- Also present (not part of the spec set): `docs/superpowers/specs/2026-10-12-self-service-sign-up-design.md`
  and three plans `docs/superpowers/plans/2026-10-12-self-service-sign-up-{1-heimdall,2-fortuna-api,3-fortuna-ui}.md` (U:docs/superpowers).

**Boards, milestones, other**
- Project boards: https://github.com/users/artur-rios/projects/12 (Fortuna API), https://github.com/users/artur-rios/projects/14 (Fortuna UI).
- Milestones: https://github.com/artur-rios/fortuna-api/milestones (M-01 Foundation … M-07 Insight and
  output, `/milestone/1`…`/7`); https://github.com/artur-rios/fortuna-ui/milestones (`/milestone/1`…`/7`).
  API names: M-01 Foundation; M-02 Access and cross-cutting mechanisms; M-03 Holdings; M-04 Money movement;
  M-05 Organization and planning; M-06 Ingestion; M-07 Insight and output. UI names: M-01 Foundation;
  M-02 Access, shell and privacy; M-03 Holdings; M-04 Money movement and lifecycle; M-05 Organization and
  planning; M-06 Ingestion and attachments; M-07 Insight and output (`gh api …/milestones`).
- Releases: https://github.com/artur-rios/fortuna-api/releases/tag/v2.0.0 ; https://github.com/artur-rios/fortuna-ui/releases/tag/v1.0.0
- Related repos: https://github.com/artur-rios/heimdall-api , https://github.com/artur-rios/yggdrasil ,
  pattern source for FFI https://github.com/artur-rios/alexandria-ui (U:TSD §5.1).
- Pluggy docs link from brainstorm: https://docs.pluggy.ai/docs/quick-pluggy-introduction (A:initial/Brainstorm.md).
- License: "Artur Rios Attribution License"; credit link artur-rios.tech or github.com/artur-rios; © 2026 Artur Rios (A/U:README "Legal").

---

## 10. Open uncertainties and contradictions (flagged)

**Versions and status**
1. **API README status is stale.** It says "version 1.0.0 is released" on main, develop and
   release/2.1.0, while tags `v1.0.0` and `v2.0.0` exist, GitHub release `v2.0.0` is Latest, and 2.1.0 is
   pending in open PR #212 (A:README line 9; `git tag`; `gh release list`; `gh pr list`). The README
   roadmap also says closed counts are "as of the 1.0.0 release".
2. **API v1.0.0 has a tag but no GitHub release** (`gh release view v1.0.0` → "release not found"), while
   CHANGELOG links `[1.0.0]` to `/releases/tag/v1.0.0`.
3. **"2.1.0" question resolved:** branch `release/2.1.0` and `feature/changelog-2.1.0` exist; develop's
   CHANGELOG has `## [2.1.0] - 2026-10-13`; not yet tagged or released. Treat 2.0.0 as latest released API version.
4. **API README roadmap M-02 says 16 issues / "16 / 16 closed"**, but its own backlog table lists 17 rows
   and GitHub shows 17 closed (UC-81 #208 added) (A:README Roadmap vs Backlog; milestones API).
5. **UI README status says "44 of the 47 issues are closed"**; GitHub has 45 closed + 3 open = 48 (UC-47
   #108 added and closed). The Backlog intro also says "47 issues" and the M-02 roadmap row says 18 (U:README; `gh issue list`).
6. **API open issues #200/#201 are outside the backlog/milestones**, so "every issue in the backlog below
   is closed" is true only for the listed backlog.

**Architecture and contract**
7. **Vision vs. auth proxy.** A:VIS §5 C4 diagram has the client "Authenticates against" Heimdall
   directly and says "Fortuna never calls Heimdall on a request path"; UC-76/77/81 and A:SRD §2 route all
   credential exchanges **through** Fortuna (`/api/auth/*` → Heimdall). Both agree token *validation* is local.
   A:initial/Technology Stack.md still lists the validation choice as "*Undecided*", resolved locally in A:TSD §6.
8. **SRD endpoint surface vs OpenAPI**: SRD §5.1 lists `PUT /api/me` (not in OAS or `MeController`);
   SRD §5.2 lists `GET /api/exchange-rates` (absent) and omits `POST /api/exchange-rates/convert` and
   `GET /api/currencies/{code}` (present); SRD §5.4 omits `POST /api/statements/{id}/close` (present); SRD
   §5.1 omits `POST /api/local-accounts/password-reset` (present, always refuses).
9. **OpenAPI security not per-operation.** `components.securitySchemes.Bearer` exists, but top-level
   `security: [{}]` and no operation-level `security`; the doc cannot tell an explorer which routes are
   anonymous. Anonymous set must come from SRD §5.1/NFR-10 and `[AllowAnonymous]`. NFR-10's anonymous list
   also omits the `/api/auth/*` anonymous exchanges that §5.1 marks anonymous.
10. **CORS.** A:OPS §3 documents `FORTUNA_CORS_ALLOWED_ORIGINS` ("Empty by default, which refuses every
    cross-origin request"), but A:README and U:README/U:OPS say "The API has no CORS support"; no CORS code
    in `src/Presentation` (grep) and compose does not forward it. Same for `FORTUNA_LOG_LEVEL` (in OPS §3, not compose).
11. **Rate/sync authority.** A:UCS UC-08/UC-09 name the Account Owner as actor; code requires "a system
    administrator or the local installation owner" for manual rates and sync (403 otherwise).
12. **Committed obligations definition differs.** API: remaining installments and **statements not yet
    settled** (A:SRD FR-PJ-04; OAS `CommittedObligationKind` Installment/Statement). UI: "installments and
    **recurring commitments** not yet materialized" (U:SRD FR-PJ-03; U:UCS UC-38).
13. **Libraries named but not used.** A:TSD §3.2/§8 list QuestPDF (PDF export) and CsvHelper (CSV export);
    neither is in `Directory.Packages.props`; `DataExportRenderer.cs` hand-writes CSV/PDF and uses ClosedXML
    for Excel. A:TSD names `UglyToad.PdfPig`; the package id pinned is `PdfPig` 0.1.16. U:TSD lists
    **trina_grid** for the spreadsheet view; it is not in `pubspec.yaml`/`pubspec.lock`.
14. **serde version**: A:TSD §3.3 says serde `1.0.229`; Cargo.toml says `1.0.228`.
15. **IR range in README.** A:README describes OPS as "`IR-01` … `IR-20`"; the document defines IR-01…IR-27.
16. **Ops repository layout drift (API).** A:OPS §2.3 lists workflows `tests.yml, check-openapi.yml,
    native-core.yml` and scripts `coverage.py, migrations.py, vulnerabilities.py, openapi.py`; repo also
    has `audit.yml`, `branch-policy.yml`, `check_blank_lines.py`, `Jenkinsfile`, `Data.Sqlite.Migrations` project.
17. **UI layout drift.** U:OPS §2.3 shows `lib/l10n/`, `integration_test/`, `packaging/windows|linux`,
    `features/<f>/validation/`; none exist in the checkout (`ls`). CONTRIBUTING's `flutter test
    integration_test -d windows` therefore has nothing to run today.
18. **Desktop packaging not delivered.** U:PO, U:VIS §10, U:OPS §6.2/IR-21 promise a Windows `.exe`
    installer + portable `.zip` and a Linux installer carrying the core + SQLite file; `build.yml`
    uploads raw build folders only, and its comment says packaging "waits on" fortuna-api#156 — which is
    closed. GitHub releases carry no assets. U:OPS §6.4 says desktop artifacts "are published alongside
    the web image" — not what the workflows do. A:native/README says "CI … publishes a native library
    plus the header" — only as Actions artifacts.
19. **UI route spec vs code.** U:SRD §5 uses `/cards…`, `/recurring`, `/tags`, `/counterparties`,
    `/settings/security`, `/settings/privacy`, `/sign-in/local`, `/sign-in/local/create`,
    `/sign-in/local/recover`, `/imports/:jobId`; code uses `/credit-cards…`, `/labels`, `/privacy`,
    `/imports/:id/records`, `/imports/new`, `/connections`, `/password-reset`, and has no `/recurring`,
    `/settings/security` or local sign-in routes (U:`lib/app/routes.dart`).
20. **Home/overview is a placeholder** (`PlaceholderScreen`, "implementedBy: UC-38") even though UC-38 is
    closed; and no global navigation widget exists in `lib/` (grep for NavigationRail/Drawer/NavigationBar).
    Verify the intended navigation before writing the user guide's "screens" page.
21. **UC-04 status.** Issue #5 open and README ⬜, yet `two_factor_screen.dart`, `two_factor_controller.dart`
    and `two_factor_repository.dart` exist; the route still renders a placeholder. Partial work, not shipped.
22. **UI contract copy lags.** `U:api/fortuna.json` differs from `A:docs/openapi/fortuna.json` (missing the
    `429` responses on `register` and `verify-email` added for 2.1.0). Same paths and `info.version` `v1`.
    The vendored header `U:native/include/fortuna_core.h` is byte-identical to the API's.
23. **Offline account storage.** A:BR/A:PO describe a local account "held in memory or in the operating
    system's credential store"; A:OPS §3 says the portable host supports `InMemory` and an OS adapter must
    be supplied by a host; the Rust core stores credential hashes in SQLite (A:native/README). Which
    storage the desktop app actually uses should be confirmed before documenting.
24. **Brainstorm mentions "OS credentials"** for desktop auth (A:initial/Brainstorm.md); no use case
    implements OS-credential sign-in.
25. **Local env uses plain HTTP Heimdall?** compose requires `FORTUNA_HEIMDALL_BASE_URL` (HTTPS) even when
    local auth is used; local.env.example points it at dev Heimdall as a placeholder (A:docker/local.env.example comments).
26. **Swagger UI exposure**: served in any non-Production ASP.NET environment (local, development) and off
    in homologation/production (A:README "Running"; A:OPS §6) — the docs site API explorer should load the
    committed JSON, not a live Swagger endpoint.
27. **Local branch state.** The fortuna-api checkout switched from `develop` to `release/2.1.0` during this
    research (presumably by another process); `origin/HEAD` locally points to `origin/main` although the
    GitHub default is `develop`. Link to `develop` (default) or to tags for versioned docs.
28. **Brainstorm divergence**: the two `Brainstorm.md` files differ slightly (UI adds the front-end note
    and "One interface, four platforms"; API has "Use .Net 10 on back-end") — `diff`.
