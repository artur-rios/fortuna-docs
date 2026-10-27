# fortuna-docs

> [!NOTE]
> This is the real, open-source code of fortuna-docs. Its home is Forgejo, at
> https://git.techcraftsmen.net/artur-rios/fortuna-docs; the copy on GitHub is a read-only mirror, for viewing only, and
> its issues and pull requests are not followed. If you want to contribute, email me at
> arturdev@duck.com.

The home page and documentation portal for
[Fortuna](https://github.com/artur-rios/fortuna-api), a personal finance system
that tracks your whole financial life in one place and turns it into tables,
charts and projections: a .NET API with a native core, and the Flutter
application that drives it. This site explains what Fortuna is, how it is put
together, how to install it, how to use it, and what its API offers. The API
reference page has an explorer that reads `fortuna.json` from fortuna-api's
`main` branch.

**[docs.artur-rios.tech/fortuna-docs](https://docs.artur-rios.tech/fortuna-docs/)**

The repository lives at <https://git.techcraftsmen.net/artur-rios/fortuna-docs>; the copy on GitHub is a
read-only mirror.

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
in that release; the site image builds with 0.165.0 — the [Go toolchain](https://go.dev/dl/)
— Docsy is consumed as a Hugo Module — and Node with npm, for Docsy's PostCSS
pipeline. Python 3.11 or newer is needed only for the checks below (the checker
imports `tomllib`).

```bash
git clone https://git.techcraftsmen.net/artur-rios/fortuna-docs.git
cd fortuna-docs
npm ci
hugo server
```

The site is then at <http://localhost:1313/fortuna-docs/>.

To reproduce what the image builds:

```bash
hugo --minify
```

## Checking it

Jenkins' `build` check builds the site image on every pull request, and the
image builds the site and then checks it, so a site that fails the check fails
the pull request and is never deployed. The check is that required pages exist, internal links
and anchors resolve, Hugo printed no unexpected warnings, and no placeholder
text is left. To run the same check locally:

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
│   ├── about/                # the about page
│   └── docs/                 # the documentation
├── docs/
│   ├── fact-sheet.md         # the cited facts the pages are written from
│   └── superpowers/          # the design spec and the plan the site was built from
├── i18n/en.toml              # footer text
├── scripts/
│   ├── check_site.py         # post-build site checker
│   ├── test_check_site.py    # tests for the checker
│   └── required_pages.txt    # pages that must be built
├── LICENSE
├── .wakatime-project         # editor time-tracking project name
├── ci/build.sh               # the pull request check: builds the image
├── yggdrasil.yml             # what Jenkins checks
├── Dockerfile                # builds, checks and serves the site
├── .dockerignore
├── renovate.json             # dependency updates
└── .gitattributes
```

Content lives under `content/en/` so a `pt-BR` translation can be added
alongside it without restructuring.

## Updating the theme

`go.mod` currently pins Docsy to v0.15.0 on purpose. v0.16.0 moved the actual
theme (`layouts/`, `assets/`) into a git submodule that Go's module zip does
not include, so it resolves as a Hugo Module but builds no output.

**v0.16.0 was checked on 2026-08-24, in the sibling alexandria-docs site, and the
fault was unchanged.** It resolves, then fails at build with `template for
shortcode "blocks/cover" not found` and writes no `public/`. Later releases
(v0.18.0 at the time of writing) have not been verified here. The pin stays
until one is.

Bumping the version therefore needs a real check, not just a green `go get`:

```bash
hugo mod get -u github.com/google/docsy
hugo --minify
```

Only commit the resulting `go.mod` and `go.sum` once that build has actually
produced `public/index.html` — an empty or missing `public/` means the new
version is broken the same way v0.16.0 was, and the pin should stay.

## Deployment

The site is deployed by Jenkins, from the yggdrasil pipeline: every push to
`main` builds the `Dockerfile` and deploys the image to production, where it
is served at <https://docs.artur-rios.tech/fortuna-docs/>. The image answers
`/healthz` for the status check. The address comes from the build arguments
`PUBLIC_HOST` and `PUBLIC_PATH`, which the deploy passes, so the `baseURL` in
`hugo.toml` only matters for `hugo server`.

Every pull request is checked by Jenkins' `build` check (`yggdrasil.yml`, run by
`ci/build.sh`), which builds the same image without pushing or running it, and the site
check above runs inside that build.

To build the image locally:

```bash
docker build -t fortuna-docs \
  --build-arg PUBLIC_HOST=docs.artur-rios.tech --build-arg PUBLIC_PATH=/fortuna-docs .
docker run --rm -p 8080:8080 fortuna-docs
```

The site is then at <http://localhost:8080/fortuna-docs/>.
