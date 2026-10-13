# fortuna-docs

The home page and documentation portal for
[Fortuna](https://github.com/artur-rios/fortuna-api), a personal finance system
that tracks your whole financial life in one place and turns it into tables,
charts and projections: a .NET API with a native core, and the Flutter
application that drives it. This site explains what Fortuna is, how it is put
together, how to install it, how to use it, and what its API offers. The API
reference page has an explorer that reads `fortuna.json` from fortuna-api's
`main` branch.

**[artur-rios.github.io/fortuna-docs](https://artur-rios.github.io/fortuna-docs/)**

This repository contains documentation only. It builds no software and ships no
package.

## The project

| Repository | What it holds |
|---|---|
| [fortuna-api](https://github.com/artur-rios/fortuna-api) | The .NET API and the native core. |
| [fortuna-ui](https://github.com/artur-rios/fortuna-ui) | The Flutter application for web, Windows, Linux and Android. |
| [fortuna-docs](https://github.com/artur-rios/fortuna-docs) | This site. |

## Running it locally

**Requirements:** [Hugo extended](https://gohugo.io/installation/) 0.158 or
newer — the configuration uses `locale` and `languages.*.label`, which arrived
in that release; CI builds with 0.165.0 — the [Go toolchain](https://go.dev/dl/)
— Docsy is consumed as a Hugo Module — and Node with npm, for Docsy's PostCSS
pipeline. Python 3 is needed only for the checks below.

```bash
git clone https://github.com/artur-rios/fortuna-docs.git
cd fortuna-docs
npm ci
hugo server
```

The site is then at <http://localhost:1313/>.

To reproduce what CI builds:

```bash
hugo --minify
```

## Checking it

CI builds the site and then checks it: required pages exist, internal links and
anchors resolve, Hugo printed no unexpected warnings, and no placeholder text
is left. To run the same check locally:

```bash
hugo --minify 2>&1 | tee hugo.log && python scripts/check_site.py --public public --build-log hugo.log
```

The pages that must exist are listed in `scripts/required_pages.txt`. The
checker has its own tests:

```bash
python -m unittest discover -s scripts -p "test_*.py"
```

## Layout

```txt
fortuna-docs/
├── hugo.toml                 # site configuration
├── go.mod                    # the Docsy theme, as a Hugo Module
├── package.json              # autoprefixer + postcss-cli
├── assets/scss/
│   └── _styles_project.scss  # light/dark code-block styles, diagrams, tables
├── content/en/
│   ├── _index.md             # landing page
│   └── docs/                 # the documentation
├── docs/
│   └── fact-sheet.md         # the cited facts the pages are written from
├── i18n/en.toml              # footer text
├── scripts/
│   ├── check_site.py         # post-build site checker
│   ├── test_check_site.py    # tests for the checker
│   └── required_pages.txt    # pages that must be built
└── .github/workflows/deploy.yml
```

Content lives under `content/en/` so a `pt-BR` translation can be added
alongside it without restructuring.

## Updating the theme

`go.mod` currently pins Docsy to v0.15.0 on purpose. v0.16.0 moved the actual
theme (`layouts/`, `assets/`) into a git submodule that Go's module zip does
not include, so it resolves as a Hugo Module but builds no output.

**Re-checked 2026-08-24 against v0.16.0, still the latest release: the fault is
unchanged.** It resolves, then fails at build with `template for shortcode
"blocks/cover" not found` and writes no `public/`. The pin stays.

Bumping the version therefore needs a real check, not just a green `go get`:

```bash
hugo mod get -u github.com/google/docsy
hugo --minify
```

Only commit the resulting `go.mod` and `go.sum` once that build has actually
produced `public/index.html` — an empty or missing `public/` means the new
version is broken the same way v0.16.0 was, and the pin should stay.

## Deployment

Every push to `main` builds the site and publishes it to GitHub Pages. Pages
must be configured once, in the repository's Settings → Pages, with **Source**
set to **GitHub Actions**.
