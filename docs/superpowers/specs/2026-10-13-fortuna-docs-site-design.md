# Fortuna documentation site — design

- **Date:** 2026-10-13
- **Repository:** `fortuna-docs`
- **Published at:** GitHub Pages — <https://artur-rios.github.io/fortuna-docs/>
- **Status:** approved

## 1. Problem

Fortuna is a personal finance system spread across two code repositories.
`fortuna-api` holds the .NET HTTP API and the native core used for desktop
offline mode; `fortuna-ui` holds the Flutter application for the web, Windows,
Linux and Android. Each has a long README and full `initial/` and
`requirements/` documents, and the API ships an OpenAPI document — but there is
no site at all. Nothing introduces Fortuna as a whole, explains how the pieces
relate, tells a person how to install and use it, or gives a developer one
place to read the API.

`fortuna-docs` becomes that home page and the project's complete
documentation source, for users and developers alike.

## 2. Scope

A Hugo site using the Docsy theme, **published on GitHub Pages** at
`https://artur-rios.github.io/fortuna-docs/`, covering the whole project:

- a landing page that serves as Fortuna's home page,
- an overview, and the domain concepts a user must understand,
- the architecture, with Mermaid diagrams,
- installation for end users and building from source,
- a user guide by area, including desktop offline mode,
- an API reference in prose and an interactive API explorer,
- operations,
- development: workflow, testing and contributing,
- the project's repositories and their canonical documents.

### 2.1 Explicitly out of scope

- The site builds no software and publishes no artifact. It documents the
  artifacts the code repositories' own workflows produce.
- Fortuna's identity is Heimdall's. The site explains how Fortuna uses Heimdall
  and links to the Heimdall portal
  (<https://artur-rios.github.io/heimdall-docs/>) for Heimdall itself.
- No change to `fortuna-api` or `fortuna-ui`.

## 3. Site structure

```txt
fortuna-docs/
├── go.mod, go.sum              # Docsy, as a Hugo Module
├── hugo.toml                   # baseURL, Mermaid, menus, repository links
├── package.json                # autoprefixer + postcss-cli for Docsy's SCSS
├── assets/scss/                # code-block, project, and Swagger UI styles
├── i18n/en.toml                # footer override
├── content/en/
│   ├── _index.md               # landing page
│   ├── about/_index.md
│   └── docs/
│       ├── _index.md
│       ├── overview/
│       ├── concepts/
│       ├── architecture/
│       ├── installation/
│       ├── user-guide/         # one page per area
│       ├── api/                # reference + explorer
│       ├── operations/
│       ├── development/
│       └── repositories/
└── .github/workflows/deploy.yml
```

Content lives under `content/en/` so a `pt-BR` tree can be added later without
restructuring. The site ships English only.

## 4. Content plan

### 4.1 Landing page

A Docsy cover with the one-line pitch, "Get started" and "Source" buttons, a
lead paragraph, and feature blocks: every account in one place, many ways to
bring data in, a spreadsheet and drillable charts, projections, multiple
currencies, desktop offline mode, and what Fortuna never does (move money,
store bank credentials).

### 4.2 Overview

What Fortuna does and does not do, who it is for, and its status — taken from
the READMEs and the Vision Document.

### 4.3 Concepts

Holdings (bank accounts, credit cards with billing cycles and statements,
investments and positions), money movement (expenses, earnings, transfers,
installment purchases, recurring commitments), categories, budgets and goals,
reconciliation, currencies and dated PTAX exchange rates, imports, exports,
attachments, consent and data rights — with the business rules (`BR-xx`) a
user must understand, each cited.

### 4.4 Architecture

Mermaid diagrams:

1. **Component graph** — the UI, the API, the managed and native cores, the
   data stores, and the integrations (Pluggy, Banco Central do Brasil PTAX,
   Heimdall).
2. **Deployment modes** — multi-user on a shared instance, contrasted with
   desktop offline mode calling the native core in process.
3. **Import pipeline** — sources behind the one pluggable source contract, to
   per-row outcomes.
4. **Signing in through Heimdall** — a sequence diagram, plus the offline
   account exception.
5. **Chart drill-down** — from an aggregation down to the transactions behind
   it.

Plus what offline mode does not yet support, and why.

### 4.5 Installation

End-user path first: where released builds come from, for each platform.
Then building from source: the API (prerequisites, configuration, PostgreSQL or
the offline store, migrations, Docker Compose), the native core, and the UI for
each target — with the prerequisites the repositories actually state.

### 4.6 User guide

Task-oriented pages, from the UI's use cases: signing in and signing up,
holdings, recording transactions, installments and recurring commitments,
imports and synchronizations, the spreadsheet view, charts and drill-down,
projections and net position, exports, data rights and consent, and desktop
offline mode.

### 4.7 API reference and explorer

- **Reference** — endpoint groups, authentication and the authorization matrix,
  the error model, pagination, filtering and sorting conventions, rate
  limiting — summarised from the System Requirements Document and the OpenAPI
  document, with links to both.
- **Explorer** — a read-only Swagger UI. It loads
  `https://raw.githubusercontent.com/artur-rios/fortuna-api/main/docs/openapi/fortuna.json`
  at view time (served with `Access-Control-Allow-Origin: *`), so the explorer
  always shows the API's released document and the site keeps no copy to fall
  out of date. "Try it out" and "Authorize" are disabled: there is no server
  behind the page. It follows the explorer `heimdall-api` already publishes.

### 4.8 Operations

Configuration and environment variables (names only), environments, health
checks, logging, and the platform requirements `IR-01` … `IR-20`, summarised
with links.

### 4.9 Development

How a use case is delivered (the Development Workflow Document), how tests are
written and run, and contributing — summarised, linking to the canonical
documents and each repository's `CONTRIBUTING.md`.

### 4.10 Repositories

The three repositories, what each holds, its default branch, its released
versions, its project board, and deep links to every document under each code
repository's `docs/initial/` and `docs/requirements/`.

### 4.11 About

Why Fortuna exists, the stack in one line per repository, and the licence.

## 5. Build and deployment

- Docsy v0.15.0 as a Hugo Module, pinned for the same reason as
  `alexandria-docs`: v0.16.0 resolves but builds no output. Requires the Go
  toolchain locally and in CI.
- Hugo extended, minimum 0.158.0; CI builds with 0.165.0.
- `.github/workflows/deploy.yml`: on pull request, build only; on push to
  `main` and on manual dispatch, build with the Pages base URL and deploy
  through `actions/upload-pages-artifact` and `actions/deploy-pages`. Every
  build asserts that `public/index.html` was produced.
- **GitHub Pages**, with Settings → Pages → Source set to **GitHub Actions**.
  `baseURL` is `https://artur-rios.github.io/fortuna-docs/`.

## 6. Correctness rule

Every factual claim on the site is sourced from the two code repositories'
READMEs, changelogs, `initial/` and `requirements/` documents, the OpenAPI
document, workflows and published releases. No feature, version, endpoint,
environment variable or command is invented. Where a detail cannot be verified
from those sources, the page links to them instead of asserting it.

## 7. Verification

- `hugo --minify` builds with no errors and no warnings, and writes
  `public/index.html`.
- No broken internal links in the built output; external links to the
  repositories and the Heimdall portal resolve.
- Every page renders legibly in light and dark mode at desktop and phone
  widths, every Mermaid diagram renders, and the explorer loads the OpenAPI
  document.
- After the first push to `main`, the Deploy workflow succeeds and
  `https://artur-rios.github.io/fortuna-docs/` serves the site.

## 8. Assumptions

1. The GitHub repository is `artur-rios/fortuna-docs`, public, default branch
   `main`, created as part of this work from the existing local repository.
2. The initial site is pushed directly to `main`; the repository is new and
   nothing protects it yet.
3. The site is English-only at launch.
