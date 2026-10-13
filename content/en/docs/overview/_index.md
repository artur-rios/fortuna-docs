---
title: Overview
linkTitle: Overview
weight: 10
description: >
  What Fortuna does and does not do, who it is for, and where the project stands.
---

## What Fortuna is

Fortuna is a personal finance system. It tracks a person's whole financial life
(bank accounts, credit cards, investments, expenses and earnings) in one place,
and turns that history into tabular views, drillable charts and forward
projections.

The problem it addresses is scattered data: one bank's app, another bank's
statement PDF, a broker's portal, a credit card invoice, and a hand-made
spreadsheet to make sense of the rest. Fortuna is built to answer two questions
without an afternoon of manual work: where did the money go, and where will I be
in six months? It is self-hostable, so the data stays on a machine or server you
control.

It is made of two repositories.

- **Fortuna API** is the back end: an HTTP API, built with ASP.NET Core on
  .NET 10, that owns the domain, the data and the integrations. It also contains
  a native core written in Rust, which exposes the same operations through a C
  interface so that a desktop app can work with no server at all.
- **Fortuna UI** is the front end: a single Flutter application for the web,
  Windows, Linux and Android that reads and writes everything through the API.
  The API owns the domain, the money and the integrations; the UI owns the
  experience.

## What it does

The list below is what the two READMEs describe. Where the interface does not
cover a feature yet, the [Status](#status) section says so.

- **Tracks holdings.** Bank accounts, credit cards (with billing cycles and
  statements) and investments, each with its own currency and history.
- **Records money movement.** Expenses, earnings and transfers, plus
  installment purchases and recurring commitments.
- **Brings data in from several sources.** Manual entry, the Pluggy open-banking
  service, Excel workbooks and supported PDF statement layouts, all behind one
  pluggable source contract. Today the supported PDF layout is the Nubank credit
  card invoice.
- **Handles multiple currencies.** Exact decimal arithmetic, and dated,
  attributable exchange rates from the Banco Central do Brasil's PTAX service.
- **Presents data as a table and as charts.** A spreadsheet-shaped table, and
  chart aggregations that drill down to the individual transactions behind a
  figure.
- **Projects the future.** From recurring commitments, unbilled installments and
  statements due.
- **Exports and attaches.** CSV, Excel and PDF exports, and attachments filed
  against transactions.
- **Honors data rights.** Consent before any third-party processing, a complete
  export of your personal data, and erasure of your account.
- **Runs in more than one shape.** Multi-user on a shared instance, or offline
  on a Windows or Linux desktop through the native core.

## What it does not do

These are deliberate exclusions, not features that are still to come.

- **It does not move money.** No payments, no transfers executed at a bank, no
  trading. Fortuna reads and records; it never writes to a financial
  institution.
- **It does not give financial advice.** A projection is arithmetic over your own
  data.
- **It does not do tax filing or accounting compliance.**
- **It does not store bank credentials.** Open-banking access lives with Pluggy
  and is referenced by token.
- **It does not manage users.** Identity belongs to
  [Heimdall](https://artur-rios.github.io/heimdall-docs/). The one exception is
  the desktop offline account, which Fortuna owns because there is no network to
  reach Heimdall over.
- **The API does not render a user interface.** It serves the numbers and the
  Flutter client draws them.

The UI adds exclusions of its own.

- **It does not compute money.** Every balance, total, aggregation and
  projection is asked of the API. The client formats figures; it never derives
  them.
- **It does not own data.** There is no client-side database, and nothing is
  authoritative until the API has accepted it.
- **It does not parse or render files.** An import is uploaded whole, and an
  export is produced by the API and only saved by the client.
- **It does not talk to external services.** Everything is reached through the
  API. The one exception is Google, which issues a sign-in token that the API
  exchanges.
- **It does not run on iOS or macOS.**
- **It does not work offline outside desktop offline mode.** A lost connection
  is reported, never disguised by a cache.

## Who it is for

Fortuna is for individuals and households who want a single, trustworthy ledger
of their own money and are not willing to hand that data to a service they do
not control. The Project Overview documents of both repositories name these
roles.

| Role | What they do |
| --- | --- |
| **Account owner** | The everyday user. Records and reviews their own financial data, imports statements, connects banks, reads charts and projections. Sees only their own data. |
| **Instance administrator** | Runs a shared instance: manages the instance and watches its operational health. Has no access to anybody's financial records, and the UI shows an administrator no financial data anywhere. |
| **Desktop offline user** | An account owner on a Windows or Linux installation with no network and no identity provider reachable. Signs in with a local account and works against a local database. The local-account screens are not reachable in the interface yet; see [What the interface does not cover yet](#what-the-interface-does-not-cover-yet). |

The business rules behind the first two rows are
[API-BR-01 and API-BR-02]({{< relref "/docs/concepts#ownership-and-isolation" >}})
for the API and
[UI-BR-21]({{< relref "/docs/concepts#ownership-and-isolation" >}}) for the
interface.

### Three ways to run it

The UI runs in three shapes, and the interface is meant to be the same in all
three.

- **Connected.** Signed in through the Fortuna API against the shared identity
  provider, talking to a remote instance. This is the normal path. Connected or
  self-hosted are the only modes on the web and Android.
- **Self-hosted.** The same experience, pointed at an instance you run yourself.
- **Desktop offline.** Windows and Linux only, and only where the installation
  provides the Fortuna core library. The app calls the core in process, with no
  network and no server. In this version the app starts but cannot be signed
  into yet.

The API supports two deployment shapes: a single self-hosted owner, or a shared
instance with several registered users, each strictly isolated from the others.

## Status

This page was checked on 2026-10-13. The released versions below come from
GitHub; the READMEs' own status lines and backlog counts are not repeated here
because they go stale.

| Part | Latest GitHub release | Earlier |
| --- | --- | --- |
| Fortuna API | [v2.1.0](https://github.com/artur-rios/fortuna-api/releases/tag/v2.1.0), 2026-10-13 | [v2.0.0](https://github.com/artur-rios/fortuna-api/releases/tag/v2.0.0), 2026-10-12 |
| Fortuna UI | [v1.0.0](https://github.com/artur-rios/fortuna-ui/releases/tag/v1.0.0), 2026-10-12 | none |

None of these releases has downloadable files attached, so there is nothing to
download from a release page yet. The project boards are the live view of the
work: [Fortuna API](https://github.com/users/artur-rios/projects/12) and
[Fortuna UI](https://github.com/users/artur-rios/projects/14).

The API README still names version 1.0.0 as the released version, which GitHub
no longer matches. This site follows GitHub.

### What the interface does not cover yet

The UI README describes its own state as specification complete with
implementation under way, and names UC-04, UC-23 and UC-35 as the use cases
that remain. GitHub agrees: those three issues are open. Here is what that
means in practice.

- **UC-04, completing a two-factor challenge at sign-in, is unfinished.** The
  route exists, but it shows a placeholder screen. See issue
  [#5](https://github.com/artur-rios/fortuna-ui/issues/5).
- **UC-23, managing recurring commitments, has no screen.** No route for it
  exists yet. See issue [#24](https://github.com/artur-rios/fortuna-ui/issues/24).
- **UC-35, managing a transaction's attachments, is open.** The UI has no
  attachment screen yet. See issue
  [#36](https://github.com/artur-rios/fortuna-ui/issues/36).
- **The home screen is a placeholder.** The route `/` shows a screen titled
  Overview that is marked as implemented by UC-38; it is not a dashboard.
- **There is no navigation menu in the code yet.** Most screens have no link
  from any other screen, so on the web they are opened by typing their address,
  and the code has no control that opens them on Windows, Linux or Android. See
  [There is no menu]({{< relref "/docs/user-guide#there-is-no-menu" >}}) in the
  [User guide]({{< relref "/docs/user-guide" >}}).
- **The desktop local-account screens and two-factor management have no route.**
  The screens for creating a local account, signing in to it and recovering it
  (UC-06, UC-07 and UC-08), and for managing two-factor authentication (UC-10),
  are written and tested, but no route opens them and no screen links to them.
  See
  [Offline mode]({{< relref "/docs/user-guide/offline-mode#uc-06-create-a-desktop-local-account" >}})
  and
  [Settings]({{< relref "/docs/user-guide/settings#uc-10-manage-two-factor-authentication" >}})
  in the [User guide]({{< relref "/docs/user-guide" >}}).
- **The file-import screen sends only the file.** The API also requires a target
  and, for a workbook, a column mapping. See
  [Imports and connections]({{< relref "/docs/user-guide/imports#uc-32-import-a-file" >}})
  (UC-32) in the [User guide]({{< relref "/docs/user-guide" >}}).
- **Sign-up (UC-47) is on the `develop` branch but not in a release.** The UI
  changelog lists it under Unreleased.

### Desktop offline mode is partial

The native core does not yet implement everything the API does. In offline mode
the UI shows these as "Not available offline", with the core's reason, instead
of offering them and then refusing: file imports and import retry, data-set
export, charts, net position and projections, transfers, installment purchases,
card statements and closing or settling them, budget consumption, goal progress,
and reconciliation. The exact list of operations, including the personal-data
archive the core does provide, is in the
[native core README](https://github.com/artur-rios/fortuna-api/blob/develop/native/README.md),
and the API's open issues
[#200](https://github.com/artur-rios/fortuna-api/issues/200) and
[#201](https://github.com/artur-rios/fortuna-api/issues/201) track the native
core's remaining work.

## Sources

This page is written from the READMEs, the Project Overview documents and the
Vision Documents of
[fortuna-api](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Project%20Overview.md)
and
[fortuna-ui](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Project%20Overview.md),
checked against the code and the GitHub releases and issues where they differ.
