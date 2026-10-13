---
title: Repositories
linkTitle: Repositories
weight: 90
description: >
  The repositories behind Fortuna, their releases, every specification document each code repository holds, and where to follow the work.
---

Fortuna lives in two code repositories, the API and the UI, and this
documentation site has a repository of its own. Both code repositories keep
their context documents in `docs/initial/` and their normative specification in
`docs/requirements/`, and both track their work on GitHub project boards and
milestones.

## The repositories

| Repository | What it holds | Default branch | Latest release |
| --- | --- | --- | --- |
| [fortuna-api](https://github.com/artur-rios/fortuna-api) | The API, built with ASP.NET Core on .NET 10, and the native core in Rust (`native/`), with the committed OpenAPI document, the specification documents and the tests. | `develop` | [v2.1.0](https://github.com/artur-rios/fortuna-api/releases/tag/v2.1.0) |
| [fortuna-ui](https://github.com/artur-rios/fortuna-ui) | The Flutter application for the web, Windows, Linux and Android that reads and writes through the API, with its specification documents. | `develop` | [v1.0.0](https://github.com/artur-rios/fortuna-ui/releases/tag/v1.0.0) |
| [fortuna-docs](https://github.com/artur-rios/fortuna-docs) | This site: documentation only, no software and no package. The repository is created on GitHub when the site is first published. | `main`, the branch its Pages workflow publishes from | None |

Both code repositories mark those releases **Latest** on GitHub. The API's
earlier release is
[v2.0.0](https://github.com/artur-rios/fortuna-api/releases/tag/v2.0.0), and
v1.0.0 of the API has a tag but no GitHub release. Each repository's releases
page ([API](https://github.com/artur-rios/fortuna-api/releases),
[UI](https://github.com/artur-rios/fortuna-ui/releases)) lists every version,
and each repository's `CHANGELOG.md` is the curated record of what changed:
[API](https://github.com/artur-rios/fortuna-api/blob/develop/CHANGELOG.md),
[UI](https://github.com/artur-rios/fortuna-ui/blob/develop/CHANGELOG.md).

{{% alert title="GitHub releases carry no files" color="warning" %}}
The releases of both code repositories have no attached files, and no installer
or package exists yet. What you can build or take from CI instead is in
[Getting a release]({{< relref "/docs/installation#getting-a-release" >}}).
{{% /alert %}}

The documentation of each code repository is on its default branch, `develop`,
and the links below use it. Related projects are
[Heimdall](https://artur-rios.github.io/heimdall-docs/), the identity provider
Fortuna signs in through (its repositories are
[heimdall-api](https://github.com/artur-rios/heimdall-api) and
[heimdall-ui](https://github.com/artur-rios/heimdall-ui)), and
[yggdrasil](https://github.com/artur-rios/yggdrasil), the deployment platform
described under [Operations]({{< relref "/docs/operations#environments" >}}).

## fortuna-api

The API's specification is 12 documents: five in
[`docs/initial/`](https://github.com/artur-rios/fortuna-api/tree/develop/docs/initial)
and seven in
[`docs/requirements/`](https://github.com/artur-rios/fortuna-api/tree/develop/docs/requirements).
The API's OpenAPI document,
[`docs/openapi/fortuna.json`](https://github.com/artur-rios/fortuna-api/blob/develop/docs/openapi/fortuna.json),
is generated from the code and committed; the
[API reference]({{< relref "/docs/api" >}}) page maps it, and the
[explorer]({{< relref "/docs/api/explorer" >}}) renders it. The native core
has its own
[README](https://github.com/artur-rios/fortuna-api/blob/develop/native/README.md).

### Initial documents

Context and the first form of the rules, written before the formal set.

| Document | What it is |
| --- | --- |
| [Brainstorm](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Brainstorm.md) | The original free-form notes the project grew from. |
| [Project Overview](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Project%20Overview.md) | What the project is, who it is for, and how success is measured. |
| [Technology Stack](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Technology%20Stack.md) | The informal stack decisions. The exact versions are pinned once, in the Technology Stack Document. |
| [Workflow](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Workflow.md) | How one use case is delivered, step by step: the operational form of the Development Workflow Document. |
| [Business Rules](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md) | The domain entities, their relationships and the `BR-01` to `BR-46` rules. This site cites them as API-BR-xx. |

### Requirements documents

The normative specification.

| Document | What it is |
| --- | --- |
| [Vision Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Vision%20Document.md) | Why the API exists, who it serves, its stakeholders, and the `F-01` to `F-20` features. |
| [System Requirements Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/System%20Requirements%20Document.md) | The `FR-<AREA>-xx` and `NFR-xx` requirements, the data model, the endpoint surface, the authorization matrix and the traceability. |
| [Use Case Specification Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md) | The application use cases, their flows and their `AF-xx` alternatives. |
| [Technology Stack Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Technology%20Stack%20Document.md) | The single source of truth for every technology and its version. |
| [Operations & Infrastructure Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Operations%20%26%20Infrastructure%20Document.md) | Layout, configuration, logging, health, UC-75, and the `IR-01` to `IR-27` platform requirements. |
| [Testing Specification Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Testing%20Specification%20Document.md) | How tests are written, named and run. |
| [Development Workflow Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Development%20Workflow%20Document.md) | The normative branch pattern, issue lifecycle, approval gates and Definition of Done. |

## fortuna-ui

The UI's specification is also 12 documents: five in
[`docs/initial/`](https://github.com/artur-rios/fortuna-ui/tree/develop/docs/initial)
and seven in
[`docs/requirements/`](https://github.com/artur-rios/fortuna-ui/tree/develop/docs/requirements).
The UI has no documentation site of its own, so these pages on GitHub are the
source. The [User guide]({{< relref "/docs/user-guide" >}}) on this site draws
on them.

### Initial documents

| Document | What it is |
| --- | --- |
| [Brainstorm](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Brainstorm.md) | The original free-form notes the project grew from. |
| [Project Overview](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Project%20Overview.md) | What the project is, who it is for, and how success is measured. |
| [Technology Stack](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Technology%20Stack.md) | The informal stack decisions. The exact versions are pinned once, in the Technology Stack Document. |
| [Workflow](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Workflow.md) | How one use case is delivered, step by step: the operational form of the Development Workflow Document. |
| [Business Rules](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md) | What the client owns, and the `BR-xx` rules. This site cites them as UI-BR-xx. |

### Requirements documents

| Document | What it is |
| --- | --- |
| [Vision Document](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Vision%20Document.md) | Why the UI exists, who it serves, its stakeholders, and the `F-xx` features. |
| [System Requirements Document](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/System%20Requirements%20Document.md) | The `FR-<AREA>-xx` and `NFR-xx` requirements, the data model, the screen surface and the traceability. |
| [Use Case Specification Document](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md) | The `UC-xx` use cases, their flows and their `AF-xx` alternatives. |
| [Technology Stack Document](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Technology%20Stack%20Document.md) | The single source of truth for every technology and its version. |
| [Operations & Infrastructure Document](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Operations%20%26%20Infrastructure%20Document.md) | Layout, configuration, the generation pipelines, CI, packaging, and the `IR-01` to `IR-22` platform requirements. |
| [Testing Specification Document](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Testing%20Specification%20Document.md) | How tests are written, named and run. |
| [Development Workflow Document](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Development%20Workflow%20Document.md) | The normative branch pattern, issue lifecycle, approval gates and Definition of Done. |

## Project boards and milestones

Work on both code repositories is tracked on public GitHub project boards,
which are the live view of what is done and what is in progress, and in
milestones grouped by theme.

| Repository | Project board | Milestones |
| --- | --- | --- |
| fortuna-api | [Fortuna API](https://github.com/users/artur-rios/projects/12) | [Milestones](https://github.com/artur-rios/fortuna-api/milestones): M-01 Foundation, M-02 Access and cross-cutting mechanisms, M-03 Holdings, M-04 Money movement, M-05 Organization and planning, M-06 Ingestion, and M-07 Insight and output |
| fortuna-ui | [Fortuna UI](https://github.com/users/artur-rios/projects/14) | [Milestones](https://github.com/artur-rios/fortuna-ui/milestones): M-01 Foundation, M-02 Access, shell and privacy, M-03 Holdings, M-04 Money movement and lifecycle, M-05 Organization and planning, M-06 Ingestion and attachments, and M-07 Insight and output |

The board and the milestone pages show the current state. The READMEs of the
repositories carry their own status lines and counts, but GitHub is the source
of truth, so this site does not repeat progress counts that would go stale. What
the interface does not cover yet is under
[Status]({{< relref "/docs/overview#status" >}}) on the Overview page.

Each issue moves from its first status through In Progress and Testing to Done, and each
use case ships on its own branch, issue and pull request, as the Development
Workflow Documents describe. The process is on the
[Development]({{< relref "/docs/development" >}}) page.
