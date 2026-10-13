---
title: API reference
linkTitle: API reference
weight: 60
description: >
  What the Fortuna API accepts and returns: how callers authenticate, who may call what, the response envelope and error statuses, paging, filtering and sorting, rate limiting, and the 31 endpoint groups.
---

This page is the map to the Fortuna API. It explains the rules every endpoint
shares, then lists the endpoint groups. The endpoints themselves, with their
parameters and schemas, are in the [API explorer]({{< relref "/docs/api/explorer" >}}),
which renders the API's OpenAPI document.

The list of endpoints follows that OpenAPI document, which is generated from the
code. The [System Requirements Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/System%20Requirements%20Document.md)
describes the intended surface and differs from the real one in places (it lists
`PUT /api/me`, which does not exist), so when the two disagree this page follows
the document and the code. The counts below were taken from the document on
fortuna-api's `main` branch on 2026-10-13, the release v2.1.0 of the API.

## Authentication

Every endpoint needs a valid bearer token, except the
[anonymous routes](#anonymous-routes) listed below. Send the token in the
`Authorization` header as `Bearer <token>`. A missing, malformed, expired or
wrongly signed token is answered with `401 Unauthorized`.

In connected mode the token is a **JWT issued by Heimdall**, the identity
provider Fortuna is registered with. The API validates it **locally**, without
calling Heimdall: it checks the signature against the shared secret, the issuer,
the audience and the expiry, with no allowance for clock skew. The secret, issuer
and audience are the `FORTUNA_AUTH_TOKEN_SECRET`, `FORTUNA_AUTH_TOKEN_ISSUER` and
`FORTUNA_AUTH_TOKEN_AUDIENCE` settings, which must match Heimdall's; see
[Connecting to Heimdall]({{< relref "/docs/installation#connecting-to-heimdall" >}}).
The token carries the user's identifier and a numeric role, using Heimdall's
values: 1 for a system administrator, 2 for a scope administrator, 3 for a user.

Callers do not talk to Heimdall to sign in. They call Fortuna's own
`/api/auth/*` routes, and Fortuna forwards each exchange to Heimdall with its
scope attached. The full sequence, including two-factor sign-in, is in
[Signing in through Heimdall]({{< relref "/docs/architecture#signing-in-through-heimdall" >}}).
Because a token is never checked against Heimdall after it is issued, a token
revoked there stays valid in Fortuna until it expires.

### Offline: the local account

A desktop installation running offline has no Heimdall to reach. It has its own
**local account**, created through `POST /api/local-accounts` (or its native
equivalent) and signed in through `POST /api/local-accounts/authenticate`. The
token it returns is local to the installation, and the account holds the
user role. The native core offers the same routes in process through a C
interface instead of HTTP, derived from the same OpenAPI document; some routes
exist there but answer `501`. How that works and what is missing is in
[the local account]({{< relref "/docs/architecture#offline-the-local-account" >}})
and [what offline mode does not support yet]({{< relref "/docs/architecture#what-offline-mode-does-not-support-yet" >}}).

### Anonymous routes

The API denies by default: an authenticated caller is required unless a route is
explicitly marked anonymous in the code. Thirteen routes are. **The OpenAPI
document does not mark them**, so this table, checked against the controllers,
is the place to find them.

| Route | Purpose | Rate limited |
| --- | --- | :---: |
| `GET /healthcheck` | Liveness: the service name and the contract version, no dependency checks. | No |
| `POST /api/auth/login` | Exchange an email and password for a token or a two-factor challenge. | Yes |
| `POST /api/auth/google` | Exchange a Google ID token for a token. | Yes |
| `POST /api/auth/2fa/verify` | Complete a two-factor challenge. | Yes |
| `POST /api/auth/2fa/challenge/resend` | Send a new code for an outstanding challenge. | Yes |
| `POST /api/auth/password-recovery` | Ask for password-recovery instructions. | Yes |
| `POST /api/auth/password-reset` | Set a new password with a reset token. | Yes |
| `POST /api/auth/register` | Register a connected identity. | Yes |
| `POST /api/auth/verify-email` | Verify an address, or confirm a sign-up. | Yes |
| `POST /api/local-accounts` | Create the desktop local account. | No |
| `POST /api/local-accounts/authenticate` | Sign in to the local account. | Yes |
| `POST /api/local-accounts/recover` | Spend a recovery code to regain access. | Yes |
| `POST /api/local-accounts/password-reset` | Always refuses: a local account has no password reset. | No |

The local-account routes answer `404` when local authentication is turned off.
The credential routes give one
`401` for any rejected credential, so a response never reveals whether an
account exists.

## Authorization

Authentication says who the caller is; the role says what they may do. The
routes fall into five groups. The count of each, taken from the code, adds up to
the 145 operations in the document.

| Who may call | Operations | Routes |
| --- | ---: | --- |
| Anyone | 13 | The [anonymous routes](#anonymous-routes). |
| Any signed-in caller | 10 | `GET /api/currencies`, `GET /api/currencies/{code}`, the `/api/auth` routes that act on the caller's own session (Google sign-out, resend verification, the two-factor status, enable, confirm and disable, and two-factor recovery-code regeneration), and `POST /api/local-accounts/recovery-codes/regenerate`. |
| The user role | 118 | Everything that touches financial data: accounts, cards, investments, transactions, transfers, categories, reports, imports, exports, attachments, the audit trail and the caller's own profile and consents. Two of these, `POST /api/me/data-export` and `GET /api/me/data-export/{jobId}`, also accept a system administrator. |
| Installation administrator | 2 | `POST /api/exchange-rates` (record a manual rate) and `POST /api/exchange-rates/sync`. A Heimdall system administrator, or the owner of an offline installation signed in with its local account. |
| System administrator | 2 | `GET /healthcheck/detailed` and `DELETE /api/users/{id}` (erase a user). |

A caller with the wrong role gets `403 Forbidden`. Three rules sit on top of
the roles, from the authorization matrix in the System Requirements Document:

- **Nobody reads another user's records.** An administrator who targets a
  financial record is refused with `403`, because administering an instance gives
  no access to what is in it. A user who names a record that belongs to someone
  else gets `404`, the same answer as for a record that does not exist, so that
  no response reveals what another user holds.
- **Managing who may sign in is Heimdall's job**, not Fortuna's. The only
  administrator route for users in Fortuna is erasure, `DELETE /api/users/{id}`.
- **A local account can do everything an account owner can**, except use a
  network source (it has no network offline), so its imports are files only.

## The response envelope

Responses from the API use one envelope. The native core uses the same one.

| Field | Type | Meaning |
| --- | --- | --- |
| `data` | any | The result of the request. |
| `errors` | list of strings | Why the request failed. |
| `messages` | list of strings | Notes about the outcome that are not errors, such as a confirmation on success. |
| `success` | boolean | Whether the request succeeded. Read only. |
| `timestamp` | date and time | When the response was made. |

For example, a refused request:

```json
{
  "data": null,
  "errors": ["PageSize must be at least 1."],
  "messages": [],
  "success": false,
  "timestamp": "2026-10-13T12:00:00Z"
}
```

A few responses are not wrapped in it: the two health routes return their own
report, and the responses that are files (a data export, an attachment, a
finished personal-data archive) return the file.

## Errors

Handlers do not throw for expected failures. They return an error message, and
the controller turns the message into a status through a map of its own, merged
over a shared one. The result is a small, consistent set of statuses.

| Status | When |
| --- | --- |
| `400 Bad Request` | A value is invalid. The message names the failing field. |
| `401 Unauthorized` | The token is missing, invalid or expired, or credentials were rejected. |
| `403 Forbidden` | The role is not allowed, a financial record was targeted by an administrator, or consent for external data processing is required. |
| `404 Not Found` | The record does not exist, **or belongs to someone else**; also a local-account route when local authentication is off. |
| `409 Conflict` | A conflict with existing data, such as hard-deleting a record that has not been soft-deleted first, or an exchange rate that is not available for a conversion. |
| `422 Unprocessable Entity` | A required confirmation was left out, as on `verify-email`. |
| `429 Too Many Requests` | The rate limit was reached, or Heimdall's was (see [Rate limiting](#rate-limiting)). |
| `503 Service Unavailable` | A dependency is down: file storage, or Heimdall. The message gives no upstream detail. |

A request that is accepted for later work, such as a registration, a data export
or an export job, is answered with `202 Accepted`. The OpenAPI document lists
only some of these statuses (most operations document only `200`), so
read the statuses above as the rules the code follows, not as a per-route
contract. The calendar dates the API accepts are `yyyy-MM-dd`, and amounts are
sent and returned as decimal strings, never as floating-point numbers.

## Pagination, filtering and sorting

List endpoints take their paging, sorting and filters as query parameters.

| Parameter | Meaning |
| --- | --- |
| `PageNumber` | Which page, starting at 1. |
| `PageSize` | Items per page, at least 1. A larger value is capped at the instance's maximum, 100 unless `FORTUNA_PAGE_SIZE_MAX` says otherwise. |
| `SortBy` | The field to order by. Each endpoint accepts its own names and has its own default (accounts sort by `Name`, transaction search by `OccurredOn`). An unsupported name is a `400` that lists the valid ones. |
| `Descending` | `true` to reverse the order. The transaction search defaults to newest first. |
| `IncludeDeleted` | Include soft-deleted records, on the endpoints that offer it (accounts, cards, investments, recurring transactions, transactions and a transaction's attachments). |

Beyond these, each list has filters of its own, listed in the explorer. For
example, accounts filter by `Name`, `Institution`, `AccountType` and
`CurrencyCode`. The transaction search filters by `From` and `To`, account, card,
category, tag, counterparty, `Direction`, a minimum and maximum amount, free
`Text` and more, and can show figures in a `DisplayCurrencyCode` at a
`FigureDate`.

A page comes back in the envelope, with the paging fields next to the usual
ones:

| Field | Meaning |
| --- | --- |
| `data` | The items of this page. |
| `pageNumber`, `pageSize` | The page that was returned. |
| `totalItems`, `totalPages` | The size of the whole result. |

The transaction search differs: its `data` is an object holding `items`,
`pageNumber`, `pageSize`, `totalItems`, `totalPages` and `totals`, the sums over
the whole matching set.

The reports have their own parameters rather than paging: an aggregate takes a
`Dimension` (period, category, account, card, counterparty or tag), a
`Granularity` and a date range, and a drill-down takes the `Key` of the group to
open; the explorer lists the rest.

## Rate limiting

Ten anonymous routes are rate limited: the eight anonymous `/api/auth` routes
and the local-account `authenticate` and `recover` routes. The limit is
**10 requests per minute per client address**, counted in a fixed one-minute
window, with no queue. The eleventh request in a window is answered with
`429 Too Many Requests`.

- **The client address is the key.** IPv6 addresses are grouped by their /64
  prefix, so rotating addresses within one prefix does not escape the limit, and
  an IPv4 address mapped into IPv6 counts as the IPv4 address. Behind a reverse
  proxy, the address is the real client's only if the proxy is configured as a
  trusted one.
- **Heimdall has its own limit**, on the requests Fortuna forwards. Unless
  `FORTUNA_HEIMDALL_CALLER_KEY` is set, every user of one Fortuna instance shares
  Heimdall's budget, so a busy instance can see `429` before an individual
  caller reaches ten. That `429` carries the message "Too many attempts. Wait a
  minute and try again."
- **Authenticated routes have no limit** in the API. Outbound calls to other
  services back off and respect those services' own limits.

The proxy and caller-key settings belong to Operations, which is not written
yet.

## Endpoint groups

The document groups its 145 operations over 109 paths into 31 tags. Where a
group has both `DELETE {id}` and `DELETE {id}/hard`, the first is a soft delete
and the second removes a record that is already soft-deleted.

| Tag | Operations | What it covers |
| --- | ---: | --- |
| Accounts | 8 | Financial accounts: list, create, read, update, soft and hard delete, restore, and the computed balance. |
| Attachments | 3 | Read (download), delete and hard delete one attachment. |
| AuditEntries | 1 | The caller's own audit trail, filterable by entity, operation, outcome and date. |
| Auth | 15 | Connected sign-in through Heimdall: login, Google, registration, email verification, password recovery and reset, and two-factor setup and verification. |
| Budgets | 6 | Budgets: list, create, read, update, delete, and consumption against the budget. |
| Categories | 8 | Categories: the usual lifecycle plus reassigning the records of one category to another. |
| Connections | 5 | Open-banking connections: list, create, read, reauthenticate, revoke. |
| ConnectionSynchronization | 1 | Start a synchronization of one connection. |
| Counterparties | 6 | Counterparties: list, create, update, delete, merge two, and a suggested category. |
| CreditCards | 8 | Credit cards: the usual lifecycle plus a card's statements. |
| Currencies | 2 | The supported currencies, and one by code. |
| DataSources | 1 | List the data sources the API offers. |
| DetailedHealthCheck | 1 | Administrator-only health of the service and each dependency. |
| ExchangeRates | 3 | Convert amounts between currencies; record a manual rate; synchronize rates. |
| Exports | 2 | Request a data-set export and retrieve its result. |
| Goals | 6 | Goals: list, create, read, update, delete, and progress. |
| HealthCheck | 1 | Anonymous liveness. |
| ImportJobs | 4 | Import jobs: list, read, read the imported records, and retry. |
| Imports | 2 | Import an Excel workbook or a PDF statement (multipart upload). |
| InstallmentPlans | 4 | Create, read, delete and restore an installment plan. |
| Investments | 10 | Investments: the usual lifecycle, movements, and valuations (list and record). |
| LocalAccounts | 5 | The desktop local account: create, authenticate, recover, regenerate recovery codes, and a password reset that always refuses. |
| Me | 7 | The caller's profile, processing consents, personal-data export, and self-erasure. |
| Projections | 2 | Cash-flow and committed-obligation projections. |
| RecurringTransactions | 6 | Recurring transactions: the usual lifecycle plus materializing due occurrences. |
| Reports | 4 | Aggregate, drill-down, net position, and a generic table query. |
| Statements | 3 | Read, close and settle a credit-card statement. |
| Tags | 4 | Tags: list, create, update, delete. |
| Transactions | 12 | Transactions: search and the usual lifecycle, attachments, tags, reconciliation, and restore. |
| Transfers | 4 | Create, read, delete and restore a transfer between accounts. |
| Users | 1 | Administrator erasure of a user. |

A few things to know when reading the explorer:

- The document describes **a contract version of `v1`** (`info.version`), which
  is independent of the API's release numbers. `GET /healthcheck` reports it.
- It declares **no servers**; use the address of the instance you run.
- Its single security scheme is a bearer JWT, "the Heimdall JWT bearer token",
  but no operation says whether it applies. See [Authorization](#authorization)
  and the [anonymous routes](#anonymous-routes) above.
