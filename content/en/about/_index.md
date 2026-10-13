---
title: About Fortuna
linkTitle: About
menu:
  main:
    weight: 40
---

{{< blocks/cover title="About Fortuna" height="auto" color="primary" >}}
A personal finance system for people who want one trustworthy ledger of their own money.
{{< /blocks/cover >}}

{{% blocks/lead %}}
A person's financial life is usually scattered across places that do not talk to
each other: one bank's app, another bank's statement PDF, a broker's portal, a
credit card invoice, and a spreadsheet kept by hand to make sense of the rest.
Nothing reconciles, so *where did the money go?* and *where will I be in six
months?* take an afternoon of manual work to answer.

Fortuna exists to answer them from a single ledger. It is built for individuals
and households who do not want to hand that data to a service they do not
control, so it is self-hostable: it runs on your own machine or server, stores no
bank credential, and computes money with exact decimal arithmetic rather than
floating point.

The API holds the data and the rules. On its own it answers in JSON, and nobody
reads their finances in JSON, so Fortuna UI is the application a person actually
uses. The API owns the domain, the money and the integrations, and the UI owns
the experience.
{{% /blocks/lead %}}

{{% blocks/section color="dark" type="row" %}}

{{% blocks/feature icon="fa-solid fa-server" title="API (.NET)" url="https://github.com/artur-rios/fortuna-api" %}}
ASP.NET Core on .NET 10 with C# 14, in a layered Domain, Application,
Infrastructure and Presentation structure. Data lives in PostgreSQL for shared
deployments, with SQLite as an alternative for local use. A Dockerfile builds
its container image.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-microchip" title="Native core (Rust)" url="https://github.com/artur-rios/fortuna-api/blob/develop/native/README.md" %}}
A Rust library, in the API repository, with its own SQLite storage. It exposes
the API's operations through a C interface so the desktop app can run offline,
in process. It does not implement everything the API does yet.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-mobile-screen-button" title="UI (Flutter)" url="https://github.com/artur-rios/fortuna-ui" %}}
One Flutter code base for the web, Windows, Linux and Android. It reaches the API
over HTTP, or, in desktop offline mode, the native core through FFI. The
interface is still being completed.
{{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="light" type="row" %}}

{{% blocks/feature icon="fa-solid fa-shield-halved" title="Signs in through Heimdall" url="https://artur-rios.github.io/heimdall-docs/" %}}
In connected mode, identity is not Fortuna's. Fortuna is a client system of
Heimdall, the identity service from the same author, and it forwards sign-in to
Heimdall and trusts the tokens it issues. The desktop offline account is the one
exception.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-scale-balanced" title="Licence" url="https://github.com/artur-rios/fortuna-api/blob/develop/LICENSE" %}}
The Artur Rios Attribution License: free and open source, so you may view, use,
fork, modify and extend it, crediting the author. Both repositories carry the
same licence, and the licence file has the terms.
{{% /blocks/feature %}}

{{% /blocks/section %}}
