---
title: Documentation
linkTitle: Documentation
menu:
  main:
    weight: 10
weight: 20
description: >
  What Fortuna is, how it is put together, how to install it, how to use it, and its API.
---

Fortuna is a personal finance system made of two repositories: the Fortuna API,
which owns the domain, the data and the integrations, and Fortuna UI, a Flutter
application that reads and writes everything through that API. The API also
carries a native core, written in Rust, that the desktop app can run in process
for its offline mode. This documentation covers all of it. It explains what
Fortuna is, the concepts you need to read it, how it is built, how to run it,
how to use the interface, and how to call the API.

| Section | What it covers |
| --- | --- |
| [Overview]({{< relref "/docs/overview" >}}) | What Fortuna does and does not do, who it is for, and where the project stands. |
| [Concepts]({{< relref "/docs/concepts" >}}) | The ideas behind the system: holdings, money movement, installments, recurring commitments, reconciliation, currencies, imports, exports and data rights, with the business rules that govern them. |
| [Architecture]({{< relref "/docs/architecture" >}}) | How the API, the native core and the UI are built, and how they talk to each other. |
| [Installation]({{< relref "/docs/installation" >}}) | What you need and the commands to build and run the API and the UI. |
| [User guide]({{< relref "/docs/user-guide" >}}) | How to use the interface, use case by use case. |
| [API]({{< relref "/docs/api" >}}) | A map of the API's endpoints and a pointer to its specification. |
| [Operations]({{< relref "/docs/operations" >}}) | Environments, configuration in operation, health checks, logging, metrics, and the platform requirements. |
| [Development]({{< relref "/docs/development" >}}) | How the project is built, tested and delivered. |
| [Repositories]({{< relref "/docs/repositories" >}}) | The repositories, their releases and their specification documents. |

If you are new, read the [Overview]({{< relref "/docs/overview" >}}) first, then
[Concepts]({{< relref "/docs/concepts" >}}). The [About]({{< relref "/about" >}})
page says why the project exists.
