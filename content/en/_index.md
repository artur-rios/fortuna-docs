---
title: Fortuna
linkTitle: Fortuna
---

{{< blocks/cover title="Fortuna" image_anchor="top" height="med" color="dark" >}}
<div class="mx-auto">
  <p class="lead mt-5">Your whole financial life in one place, turned into tables, charts and projections.</p>
  <a class="btn btn-lg btn-primary me-3 mb-4" href="{{< relref "/docs" >}}">
    Get started <i class="fas fa-arrow-alt-circle-right ms-2"></i>
  </a>
  <a class="btn btn-lg btn-secondary mb-4" href="https://github.com/artur-rios/fortuna-api">
    Source <i class="fab fa-github ms-2"></i>
  </a>
</div>
{{< /blocks/cover >}}

{{% blocks/lead color="primary" %}}
Fortuna is a personal finance system. It tracks bank accounts, credit cards,
investments, expenses and earnings in one place, and answers two questions that
a scattering of bank apps, statement PDFs and hand-made spreadsheets cannot:
where did the money go, and where will I be in six months?

It has two parts: the Fortuna API, built on .NET 10 with an optional Rust native
core, and Fortuna UI, a Flutter application for the web, Windows, Linux and
Android. You can run it on your own machine or server. The interface is still
being completed, and the [Overview]({{< relref "/docs/overview" >}}) says which
parts are not finished.
{{% /blocks/lead %}}

{{% blocks/section color="dark" type="row" %}}

{{% blocks/feature icon="fa-solid fa-wallet" title="Every account in one place" %}}
Bank accounts, credit cards with their billing cycles and statements, and
investments with their positions. Each one keeps its own currency and its own
history.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-file-import" title="Many ways in" %}}
Enter records by hand, synchronize through the Pluggy open-banking service, or
import an Excel workbook or a Nubank credit card invoice PDF. Every source goes
through the same contract, and duplicates are detected rather than imported
twice.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-chart-pie" title="Tables and drill-down charts" %}}
Read your records as a filterable, sortable spreadsheet, or as chart
aggregations. Tap a chart element to see the breakdown behind it, down to the
individual transactions.
{{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="light" type="row" %}}

{{% blocks/feature icon="fa-solid fa-chart-line" title="Projections" %}}
Look ahead using your recurring commitments, unbilled installments and
statements due. A projected figure is always marked as projected, never passed
off as history.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-coins" title="Multiple currencies, with PTAX" %}}
Every amount carries its currency, and money is exact decimal arithmetic.
Conversions use dated exchange rates from the Banco Central do Brasil's PTAX
service, and each conversion records the rate and the date it used.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-laptop" title="Desktop offline mode" %}}
On Windows and Linux the app can run against a local core with no network and no
server. In this version the app starts but cannot be signed into yet. The core
does not yet implement everything the API does, and the app labels what is
unavailable offline instead of offering it.
{{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="dark" %}}
## What Fortuna never does

These are deliberate exclusions, not features that are still to come.
{{% /blocks/section %}}

{{% blocks/section color="dark" type="row" %}}

{{% blocks/feature icon="fa-solid fa-ban" title="It never moves money" %}}
No payments, no transfers executed at a bank, no trading. Fortuna reads and
records; it never writes to a financial institution.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-lock" title="It never stores bank credentials" %}}
Open-banking access lives with Pluggy and is referenced by token. Fortuna never
holds a bank username, password or authentication factor.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-circle-info" title="It never gives advice" %}}
A projection is arithmetic over your own data, not a recommendation. Fortuna
also does no tax filing or accounting compliance.
{{% /blocks/feature %}}

{{% /blocks/section %}}
