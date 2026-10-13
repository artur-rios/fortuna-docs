---
title: Installation
linkTitle: Installation
weight: 40
description: >
  What exists to download today (nothing yet), and how to run the Fortuna API, build the native core, run the UI on each target, and connect Fortuna to Heimdall.
---

Fortuna has three runnable parts: the **API** (ASP.NET Core on .NET 10), the
**native core** (a Rust library) and the **UI** (one Flutter application). What
each one is, and the two ways to deploy them, is on the
[Architecture]({{< relref "/docs/architecture" >}}) page. This page is about
getting them running.

The commands are copied from the repositories' own documentation
([fortuna-api](https://github.com/artur-rios/fortuna-api) and
[fortuna-ui](https://github.com/artur-rios/fortuna-ui)) and were checked on
2026-10-13 against their `develop` branches. Where a list changes often, such as
every API setting, this page names the settings and links to the source for the
values.

{{% alert title="What runs end to end today" color="info" %}}
In this version, signing in to the UI works only in connected (or self-hosted)
mode, against a Fortuna API that is connected to a Heimdall reachable over HTTPS
([Connecting to Heimdall](#connecting-to-heimdall)). Offline mode starts but
cannot be signed into yet: see [Offline mode](#offline-mode-windows-and-linux).
{{% /alert %}}

## Getting a release

Read this section before looking for a download, because **there is nothing to
download yet**.

At the time of writing the latest GitHub releases are API
[v2.1.0](https://github.com/artur-rios/fortuna-api/releases/tag/v2.1.0)
(then [v2.0.0](https://github.com/artur-rios/fortuna-api/releases/tag/v2.0.0);
v1.0.0 has a tag but no GitHub release) and UI
[v1.0.0](https://github.com/artur-rios/fortuna-ui/releases/tag/v1.0.0). Each
release is a tag with generated notes and **no attached files**. No installer,
no portable archive, no package and no container image on a registry exists for
any part of Fortuna.

The UI's Operations & Infrastructure Document describes a Windows installer and
portable archive, and a Linux installer, that would carry the core library and
the SQLite file. The UI's `build.yml` workflow does not build them: its header
says the packaging waits on the core library existing. So the desktop packages
are planned, not delivered.

What you can get today:

| What | Where it comes from |
| --- | --- |
| API | Clone [fortuna-api](https://github.com/artur-rios/fortuna-api) and follow [Running the API](#running-the-api). |
| API container image | Built from the repository's `Dockerfile`, which `docker compose` does for you. The `docker` job of the `tests.yml` workflow builds it with `push: false`, only to check that it builds. On the project's own servers, Jenkins builds it per environment. It is not pushed to any public registry. |
| Native core library | Built with Cargo ([Building the native core](#building-the-native-core)), or taken from a CI run of the `Native Core` workflow, below. |
| UI | Clone [fortuna-ui](https://github.com/artur-rios/fortuna-ui) and follow [Running the UI](#running-the-ui). |
| UI web container image | Built from the repository's `Dockerfile` ([Web container image](#web-container-image)). Not pushed to a registry either. |
| UI builds for the web, Windows, Linux and Android | Raw build outputs from a CI run of the `Build` workflow, below. Not confirmed to be usable as they are, see the notes below the table. |

### What CI keeps

CI keeps its build outputs as **workflow artifacts** of a run, not as release
files.

| Workflow (file) | Runs on | Artifact name | Contents |
| --- | --- | --- | --- |
| [Native Core](https://github.com/artur-rios/fortuna-api/actions/workflows/native-core.yml) (`native-core.yml`, fortuna-api) | A push or pull request to `main` or `develop` that changes `native/**`, `docs/openapi/fortuna.json` or the workflow file | `fortuna-native-linux-x64` | `libfortuna_core.so` and the C header `fortuna_core.h` |
| [Native Core](https://github.com/artur-rios/fortuna-api/actions/workflows/native-core.yml) (`native-core.yml`, fortuna-api) | Same as above | `fortuna-native-windows-x64` | `fortuna_core.dll` and the C header `fortuna_core.h` |
| [Build](https://github.com/artur-rios/fortuna-ui/actions/workflows/build.yml) (`build.yml`, fortuna-ui) | A `v*` tag, or started by hand | `fortuna-ui-web` | `build/web` |
| [Build](https://github.com/artur-rios/fortuna-ui/actions/workflows/build.yml) (`build.yml`, fortuna-ui) | Same as above | `fortuna-ui-android` | `app-release.apk` |
| [Build](https://github.com/artur-rios/fortuna-ui/actions/workflows/build.yml) (`build.yml`, fortuna-ui) | Same as above | `fortuna-ui-windows` | `build/windows/x64/runner/Release` |
| [Build](https://github.com/artur-rios/fortuna-ui/actions/workflows/build.yml) (`build.yml`, fortuna-ui) | Same as above | `fortuna-ui-linux` | `build/linux/x64/release/bundle` |

Things to know before you use one:

- **They are not permanent downloads.** GitHub keeps workflow artifacts only for
  its own retention period, and downloading one needs a GitHub sign-in. Open the
  workflow's run list to see which runs still have them.
- **They are raw folders.** The Windows and Linux outputs are the build folders,
  with no installer, and they do not contain the core library. The workflow says
  so: it builds the client only. To run offline mode today, build from source as
  described in [Offline mode](#offline-mode-windows-and-linux).
- **They name no API.** The `Build` workflow passes no `--dart-define`, so the
  UI starts at its setup screen, where you type the address of an API. A browser
  build can only call an API on its own origin, as explained under
  [Web](#web).
- **The Android APK is signed with the debug key**, which the project's Gradle
  file says it does "for now", so that `flutter run --release` works. It has not
  been confirmed that this CI release APK connects to an API. The UI README
  documents running on Android with `flutter run -d <device>`, as under
  [Android](#android).

## Running the API

### Prerequisites

| Requirement | Used for |
| --- | --- |
| .NET 10 SDK | Building and running the API from source. The `Dockerfile` builds on `mcr.microsoft.com/dotnet/sdk:10.0` and runs on `mcr.microsoft.com/dotnet/aspnet:10.0`. |
| PostgreSQL | The database of a shared instance. The repository states no minimum version; its functional tests start `postgres:17-alpine` and `postgres:18-alpine` containers. |
| Docker | The container workflow below, and the functional tests (Testcontainers). |
| Python 3 | The helper scripts under `scripts/`, such as the migrations menu. |
| Rust, stable channel | Only for the native core. |

For a **desktop offline** installation the API is not involved at all: the UI
calls the native core in process, and no PostgreSQL is needed. Go to
[Building the native core](#building-the-native-core). The API can also run on
SQLite instead of PostgreSQL ([below](#option-b-from-source-on-sqlite)), which is
a different thing: that is still the HTTP API.

```bash
git clone https://github.com/artur-rios/fortuna-api.git
cd fortuna-api
```

### Configuration

The API reads its settings from **environment variables**, mostly prefixed
`FORTUNA_` (plus the standard `ASPNETCORE_*` ones). It does not read an `.env` file on its own: with
Docker Compose the variables come from the env file you pass to Compose, and
from source they must be in the environment of the process. No secret is ever
committed; the example files under `docker/` carry placeholders.

If a required value is missing or malformed, the API logs every problem it found
(`Invalid configuration: …`) and stops, rather than failing later.

These have no default and must be set:

| Variable | What it is |
| --- | --- |
| `FORTUNA_DATA_CONNECTIONSTRING` | The database. A PostgreSQL connection string, or for SQLite a file path or a `Data Source=…` string. |
| `FORTUNA_DATA_DATABASETYPE` | `PostgreSql` or `SQLite`. |
| `FORTUNA_STORAGE_PROVIDER` | Where attachments go: `Filesystem` or `S3`. With `Filesystem`, `FORTUNA_STORAGE_PATH` is also required. With `S3`, `FORTUNA_STORAGE_S3_ENDPOINT`, `FORTUNA_STORAGE_S3_BUCKET`, `FORTUNA_STORAGE_S3_ACCESS_KEY` and `FORTUNA_STORAGE_S3_SECRET_KEY` are. |
| `FORTUNA_LOG_DIRECTORY` | Where the daily JSON log files are written. |
| `FORTUNA_AUTH_TOKEN_SECRET` | The key the API validates Heimdall's tokens with. At least 32 ASCII characters. It must equal your Heimdall's signing secret ([Connecting to Heimdall](#connecting-to-heimdall)). |
| `FORTUNA_AUTH_TOKEN_ISSUER`, `FORTUNA_AUTH_TOKEN_AUDIENCE` | The issuer and audience the tokens must carry. |
| `FORTUNA_HEIMDALL_BASE_URL` | Heimdall's address. It must be an `https` URL. |
| `FORTUNA_HEIMDALL_SCOPE_ID` | The public identifier (a GUID) of Fortuna's scope in Heimdall. |
| `FORTUNA_LOCALE` | A specific locale, for example `pt-BR`. |

The Heimdall settings are required even when you sign in with the local account
instead, so you must give them well-formed values. The local example file uses
placeholders.

Everything else is optional and has a default. By name, the optional settings
cover:

- **Migrations**: `FORTUNA_RUN_MIGRATIONS`. The API's own default is `false`; the
  Compose file and the example files set it to `true`.
- **Tokens**: `FORTUNA_AUTH_TOKEN_SECRET_PREVIOUS` (only while rotating the
  secret) and `FORTUNA_AUTH_TOKEN_EXPIRATION_IN_SECONDS` (default 3600).
- **Heimdall**: `FORTUNA_HEIMDALL_CALLER_KEY`.
- **Profiles and sign-in**: `FORTUNA_DEFAULT_DISPLAY_CURRENCY`,
  `FORTUNA_LOCAL_AUTH_ENABLED` and `FORTUNA_LOCAL_AUTH_RECOVERY_CODE_COUNT`
  (default 10), `FORTUNA_CONSENT_EXTERNAL_PROCESSING_VERSION`.
- **Integrations**: `FORTUNA_PLUGGY_CLIENT_ID`, `FORTUNA_PLUGGY_CLIENT_SECRET`,
  `FORTUNA_PLUGGY_BASE_URL`, and `FORTUNA_RATES_SOURCE_BASE_URL`,
  `FORTUNA_RATES_SYNC_CRON`, `FORTUNA_RATES_CURRENCIES` (when the rates source
  is set, the cron expression and at least two currencies are required too).
- **Limits and bounds**: `FORTUNA_UPLOAD_MAX_BYTES`,
  `FORTUNA_UPLOAD_ALLOWED_CONTENT_TYPES`, `FORTUNA_EXCEL_IMPORT_MAX_BYTES`,
  `FORTUNA_PDF_IMPORT_MAX_BYTES`, `FORTUNA_PAGE_SIZE_MAX`,
  `FORTUNA_REPORT_MAX_RANGE_DAYS`, `FORTUNA_REPORT_KEY_TTL_MINUTES`,
  `FORTUNA_PROJECTION_MAX_HORIZON_DAYS`, `FORTUNA_TRANSACTION_MAX_TAGS`,
  `FORTUNA_RECONCILIATION_AMOUNT_TOLERANCE`,
  `FORTUNA_RECONCILIATION_DATE_TOLERANCE_DAYS`.
- **Jobs, exports and health**: `FORTUNA_JOB_QUEUE_CAPACITY`,
  `FORTUNA_EXPORT_SYNC_THRESHOLD_ROWS`, `FORTUNA_EXPORT_RETENTION_HOURS`,
  `FORTUNA_HEALTH_JOB_MAX_PENDING_SECONDS`.
- **Metrics and reverse proxy**: `FORTUNA_METRICS_PORT` (default 9464, `0`
  disables), `FORTUNA_FORWARDED_KNOWN_NETWORKS`, `FORTUNA_FORWARDED_KNOWN_PROXIES`.
- **Compose only**: `API_IMAGE`, `API_IMAGE_TAG`, `API_PORT`, and the pieces of
  the connection string, `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`,
  `DB_PASSWORD`, `DB_SEARCH_PATH`, `DB_CONNECTION_EXTRA`.

The meaning of each is in the
[Operations & Infrastructure Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Operations%20%26%20Infrastructure%20Document.md)
(section 3), and the defaults are the values in
[`docker/local.env.example`](https://github.com/artur-rios/fortuna-api/blob/develop/docker/local.env.example).

{{% alert title="There is no CORS setting" color="warning" %}}
That document also lists `FORTUNA_CORS_ALLOWED_ORIGINS` and
`FORTUNA_LOG_LEVEL`. The API does not read either one, and the Compose file does
not forward them, so setting them changes nothing. **The API has no CORS
support**, per both READMEs and the code, so a browser can only call it from the
page's own origin. See [Web](#web) for what that means for the UI.
{{% /alert %}}

### Option A: Docker Compose

This is the repository's documented way to run the API. One Compose file serves
every environment; only the env file differs. PostgreSQL is **not** a service in
it: the API container reaches a PostgreSQL already running on your machine at
`host.docker.internal`, which Docker Desktop defines and which the Compose file
maps for the Docker engine on Linux.

1. Have a PostgreSQL database and a login for it. Neither the Compose file nor
   the repository creates them. The first migration creates the `fortuna`
   schema inside the database.
2. Copy the local example and edit it.

   ```bash
   cp docker/local.env.example docker/local.env
   ```

   `docker/local.env` is git-ignored. The values to change are:

   | Setting | Why |
   | --- | --- |
   | `DB_NAME`, `DB_USER`, `DB_PASSWORD` (or one complete `FORTUNA_DATA_CONNECTIONSTRING`) | Your database. The example uses `fortuna_local` for both name and user, and `replace-me` for the password. If `FORTUNA_DATA_CONNECTIONSTRING` is set, the `DB_*` settings are ignored. |
   | `FORTUNA_AUTH_TOKEN_SECRET` | At least 32 characters. Replace the placeholder. |
   | `FORTUNA_HEIMDALL_BASE_URL`, `FORTUNA_HEIMDALL_SCOPE_ID`, `FORTUNA_HEIMDALL_CALLER_KEY` | Only to sign in through Heimdall. See [Connecting to Heimdall](#connecting-to-heimdall). |
   | `FORTUNA_LOCAL_AUTH_ENABLED` | The example sets `true`: the local environment uses Fortuna's own local account, because a local Heimdall is plain HTTP. |
   | `API_PORT` | The example sets `8083`, because the local Heimdall API keeps `8080`. The Compose file's own default is `8080`. |

3. Build and start it.

   ```bash
   docker compose --env-file docker/local.env up -d --build
   ```

   The image is built from the `Dockerfile`, which also builds an EF Core
   migrations bundle. On start, the entrypoint runs the bundle when
   `FORTUNA_RUN_MIGRATIONS` is `true`, against `FORTUNA_DATA_CONNECTIONSTRING`,
   then starts the API.

4. Check it. The API is at `http://localhost:8083`, and the public liveness
   check answers without a token:

   ```bash
   curl http://localhost:8083/healthcheck
   ```

   ```json
   {"contractVersion":"v1","service":"Fortuna API"}
   ```

{{% alert title="This local setup is not a way to sign in to the UI" color="warning" %}}
The local account serves the API's own local-account endpoints. The web,
Android and desktop UI, when connected to an API, sign in through Fortuna's
`/api/auth/*`, which signs in with Heimdall; the UI has no local sign-in for a
connected instance. With the placeholder Heimdall settings of the local example, the API
runs and answers, but a connected UI cannot sign in to it. To sign in, follow
[Connecting to Heimdall](#connecting-to-heimdall). [Offline mode](#offline-mode-windows-and-linux)
needs no API, but it starts and cannot be signed into in this version (see
[Create a Desktop Local Account]({{< relref "/docs/user-guide/offline-mode#uc-06-create-a-desktop-local-account" >}})).
{{% /alert %}}

The container serves on port `8080` inside (`API_PORT` is the host side) and
exposes the private metrics port `9464`, which Compose does not publish. The
example file publishes `API_PORT` on every network interface, so that a phone on
the same network can reach it; write `127.0.0.1:8083` to keep it on this
machine. The Compose healthcheck calls `/healthcheck` every 15 seconds.

The `development`, `homologation` and `production` env examples
(`docker/<environment>.env.example`) are for the project's own servers, which
run this same Compose file under yggdrasil with a Traefik overlay. That setup is
described under [Environments]({{< relref "/docs/operations#environments" >}}) on
the [Operations]({{< relref "/docs/operations" >}}) page.

### Option B: from source, on SQLite

Running the API outside Docker needs the same settings, set as environment
variables in your shell, and a built solution. SQLite avoids needing PostgreSQL,
and the managed host applies its own EF migration set to it. Compose cannot do
this: it fixes `FORTUNA_DATA_DATABASETYPE` to `PostgreSql`.

```bash
dotnet tool restore
dotnet restore src/ArturRios.Fortuna.sln
dotnet build src/ArturRios.Fortuna.sln -m:1
```

Then set the variables and run the Web API project. This is a complete set for a
throwaway instance that signs in with the local account (bash syntax; in
PowerShell use `$env:NAME = "value"`):

```bash
export ASPNETCORE_ENVIRONMENT=Development
export ASPNETCORE_HTTP_PORTS=8083
export FORTUNA_DATA_DATABASETYPE=SQLite
export FORTUNA_DATA_CONNECTIONSTRING=/absolute/path/to/fortuna.db
export FORTUNA_RUN_MIGRATIONS=true
export FORTUNA_STORAGE_PROVIDER=Filesystem
export FORTUNA_STORAGE_PATH=/absolute/path/to/attachments
export FORTUNA_LOG_DIRECTORY=/absolute/path/to/logs
export FORTUNA_AUTH_TOKEN_SECRET=replace-me-with-a-secret-of-32-chars-or-more
export FORTUNA_AUTH_TOKEN_ISSUER=heimdall
export FORTUNA_AUTH_TOKEN_AUDIENCE=fortuna
export FORTUNA_HEIMDALL_BASE_URL=https://heimdall.example.test/
export FORTUNA_HEIMDALL_SCOPE_ID=00000000-0000-0000-0000-000000000001
export FORTUNA_LOCALE=pt-BR
export FORTUNA_LOCAL_AUTH_ENABLED=true
export FORTUNA_METRICS_PORT=0
dotnet run --project src/Presentation/ArturRios.Fortuna.WebApi --no-launch-profile
```

This instance signs in with the local account, which serves the API's own
local-account endpoints. The UI, connected to an API, signs in through Fortuna's
`/api/auth/*`, which signs in with Heimdall, instead (see the note under [Option A](#option-a-docker-compose)), so this
instance is good for the API and Swagger UI. [Offline mode](#offline-mode-windows-and-linux)
needs no API either, but it starts and cannot be signed into in this version
(see [Create a Desktop Local Account]({{< relref "/docs/user-guide/offline-mode#uc-06-create-a-desktop-local-account" >}})). To sign in through Heimdall, see
[Connecting to Heimdall](#connecting-to-heimdall).

- `ASPNETCORE_HTTP_PORTS` sets the port, and `--no-launch-profile` keeps the
  project's launch profiles out of the way (they use `http://localhost:0`).
- `FORTUNA_METRICS_PORT=0` turns the metrics exporter off, which is simplest
  outside a container. The container's entrypoint is what opens the metrics port
  there.
- `/absolute/path/to/fortuna.db` is created if it does not exist. The managed
  SQLite database and the native core's SQLite file are separate schemas and are
  not interchangeable.
- A first start can log a `no such table: data_protection_keys` error before the
  migrations have run. If the log then reports `Now listening on`, the API is
  up. Check it with `curl http://localhost:8083/healthcheck`.
- In any ASP.NET environment except `Production`, the API also serves Swagger UI,
  at `/swagger`. `Production` does not.

For PostgreSQL from source, set `FORTUNA_DATA_DATABASETYPE=PostgreSql` and a
connection string such as
`Host=localhost;Port=5432;Database=fortuna;Username=…;Password=…;Search Path=fortuna`,
as in the template at
`src/Presentation/ArturRios.Fortuna.WebApi/Environments/.env`. That file is a
template only: `scripts/migrations.py` reads copies of it, and the API does not.

### Migrations

With `FORTUNA_RUN_MIGRATIONS=true`, pending migrations are applied at start
(by the container entrypoint in Docker, and by the API itself when run from
source). To list, create or apply them by hand there is an interactive menu. It
needs the pinned EF tool, so run `dotnet tool restore` once first:

```bash
python3 scripts/migrations.py
```

## Building the native core

The native core is a Rust library inside the API repository, in
`native/fortuna-core`. You need it for desktop offline mode and nowhere else. It
needs the stable Rust toolchain; the declared minimum is Rust `1.85`.

```bash
cargo test  --manifest-path native/fortuna-core/Cargo.toml
cargo build --manifest-path native/fortuna-core/Cargo.toml --release
```

The build produces `libfortuna_core.so` on Linux and `fortuna_core.dll` on
Windows, in `native/fortuna-core/target/release/`. It also regenerates the C
header with `cbindgen`. The header is committed, and CI fails if the committed
copy differs from the generated one. You can run the same check:

```bash
git diff --exit-code -- native/fortuna-core/include/fortuna_core.h
```

The library and the header are what a client needs. How the library is laid
out, how it hands strings back and which operations it does not implement yet
are in the
[native core README](https://github.com/artur-rios/fortuna-api/blob/develop/native/README.md),
and the architecture is in [The native core]({{< relref "/docs/architecture#the-native-core" >}}).

## Running the UI

The UI is one Flutter application. It takes its settings **when it is built**,
not when it starts.

### Prerequisites

| Requirement | Used for |
| --- | --- |
| Flutter SDK, stable channel, latest release | Everything. The repository pins no number. `pubspec.yaml` needs Dart `^3.13.2`, which first ships in Flutter 3.47.2, and the web `Dockerfile` pins Flutter 3.47.4. |
| The desktop toolchain of the platform you build for | Windows or Linux desktop builds. |
| On Linux: `clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev libstdc++-12-dev libsecret-1-dev` | What the CI Linux job installs before `flutter build linux`. |
| For Android: a Java 17 JDK | What the CI Android job uses (Temurin 17). |
| Docker | The web container image only. |

```bash
git clone https://github.com/artur-rios/fortuna-ui.git
cd fortuna-ui
flutter pub get
```

A clone is on `develop`, which is ahead of the v1.0.0 release (it already has
sign-up, UI UC-47). Check out the `v1.0.0` tag if you want the released UI.

### Build-time settings

Everything the UI is configured with arrives as a `--dart-define`. These are
compiled into the build, so they are **public**: never put a secret in one.

| Define | Meaning |
| --- | --- |
| `FORTUNA_API_BASE_URL` | The default API. Empty or absent, the app starts at its setup screen and asks for an address. The address a user types there is stored on the device and overrides this one. |
| `FORTUNA_GOOGLE_CLIENT_ID` | The Google OAuth client id. Empty, Google sign-in is hidden. |
| `FORTUNA_TRANSPORT_FFI` | `true` selects desktop offline mode (the core in process). Absent or `false`, the UI uses HTTP. |
| `FORTUNA_DB_PATH` | Offline mode only: where the SQLite database file lives. |

A connected UI signs in through Fortuna's `/api/auth/*`, which signs in with
Heimdall, so a connected run needs an API that
is connected to a Heimdall over HTTPS ([Connecting to
Heimdall](#connecting-to-heimdall)). An API run with the local example's
placeholder Heimdall settings and the local account will start and answer, but
the UI cannot sign in to it. [Offline mode](#offline-mode-windows-and-linux)
runs without an API, but it starts and cannot be signed into in this version
(see [Create a Desktop Local Account]({{< relref "/docs/user-guide/offline-mode#uc-06-create-a-desktop-local-account" >}})).

For a connected run, copy the example config and pass it to every run. The
example points at the API of [Option A](#option-a-docker-compose), on
`http://localhost:8083`:

```bash
cp config/local.json.example config/local.json
```

```json
{
  "FORTUNA_API_BASE_URL": "http://localhost:8083",
  "FORTUNA_GOOGLE_CLIENT_ID": ""
}
```

`config/local.json` is git-ignored. **If your API is on another port or on
another machine, edit
`FORTUNA_API_BASE_URL` here**, or leave it empty and type the address at the
setup screen.

### Windows and Linux desktop

```bash
flutter run -d windows --dart-define-from-file=config/local.json
```

Replace `windows` with `linux`. A desktop app calls the API directly, so it
needs nothing else.

### Web

```bash
flutter run -d chrome --dart-define-from-file=config/local.json --web-browser-flag=--disable-web-security
```

The flag is needed because **the API has no CORS support**: a browser reaches it
only on the page's own origin, and a local `flutter run` is served from another
port. The flag runs Chrome, in a temporary profile, with the browser's
same-origin checks off. Use it for development only, or work on a desktop target
instead.

A deployed web UI works differently, and you have to arrange it. The UI's
Operations & Infrastructure Document (section 6.3) describes how the project
does it. A deployed web build names **its own origin** as the API address
(`https://<ui host>`), and the reverse proxy in front, Traefik on the project's
servers, routes `https://<ui host>/api/…` to the API, whose routes all begin with
`/api/`. The browser therefore makes only same-origin calls. The API keeps its
own host for the desktop and Android builds. If you host the web UI yourself,
your proxy has to do the same. The UI container's `nginx.conf` serves static
files and `/healthz` only, and has no rule for `/api/`.

### Android

```bash
flutter run -d <your-device> --dart-define-from-file=config/local.json
```

An Android emulator reaches the machine running the API at `http://10.0.2.2:8083`,
so for an emulator set `FORTUNA_API_BASE_URL` to that. A physical device needs
your machine's address on the network, and an API published on that interface.
Android runs only in connected mode, so it also needs a Heimdall-connected API.

### Offline mode (Windows and Linux)

Offline mode needs no API, no PostgreSQL and no Heimdall. It needs the native
core library next to the UI.

1. Build the core: [Building the native core](#building-the-native-core).
2. Put the library where the app looks for it. It looks first **beside the
   executable**, then at a relative path in the UI repository:
   `native/windows/fortuna_core.dll` on Windows, or
   `native/linux/libfortuna_core.so` on Linux. Those folders do not exist in a
   clone, so create the one you need and copy the library into it. The UI keeps
   its own copy of the header at `native/include/fortuna_core.h`, which CI
   compares with the API's; use a core built from a matching API checkout.
3. Run with the transport switched and a database path:

   ```bash
   flutter run -d windows --dart-define=FORTUNA_TRANSPORT_FFI=true --dart-define=FORTUNA_DB_PATH=/absolute/path/to/fortuna.db
   ```

   Replace `windows` with `linux`. **Always pass `FORTUNA_DB_PATH`.** The
   UI's Operations & Infrastructure Document says it defaults to beside the
   executable, but the code passes the core an empty path when it is unset, and
   the core answers `400` to an empty path.
4. The core creates the database file and its folder when it first starts.

Offline mode is offered only where the library loads. If the library is missing
or will not load and no `FORTUNA_API_BASE_URL` is built in, the app falls back to
the setup screen, where it reports the failure and offers to connect to an
instance instead. With a `FORTUNA_API_BASE_URL` built in, the app starts
connected at that address instead. Offline, the UI is meant to sign in
with a local account that Fortuna owns instead of Heimdall: see
[Offline: the local account]({{< relref "/docs/architecture#offline-the-local-account" >}}).

**In this version the app stops at the sign-in screen.** No route opens the
local-account screens, and the sign-in screen it shows is the connected form,
whose request the core does not serve ("This is not available offline: POST
/api/auth/login."). See [Create a Desktop Local Account]({{< relref "/docs/user-guide/offline-mode#uc-06-create-a-desktop-local-account" >}}).

Some features are not available offline yet; the list is under
[What offline mode does not support yet]({{< relref "/docs/architecture#what-offline-mode-does-not-support-yet" >}}).

### Web container image

The repository's `Dockerfile` builds the web target into an image: a pinned
Flutter SDK (3.47.4) compiles `build/web`, and `nginxinc/nginx-unprivileged:alpine`
serves it as a non-root user on port `8080`, with a fallback to `index.html` for
deep links. It expects TLS to be terminated in front of it.

| Build argument | Required | Meaning |
| --- | --- | --- |
| `FORTUNA_API_BASE_URL` | No | The default API. Empty starts the app at the setup screen. |
| `FORTUNA_GOOGLE_CLIENT_ID` | No | Empty hides Google sign-in. |
| `FLUTTER_VERSION`, `FLUTTER_SHA256` | No | Override the SDK. Pass both together. |

```bash
docker build -t fortuna-ui:web \
  --build-arg FORTUNA_API_BASE_URL=https://fortuna.example.com \
  --build-arg FORTUNA_GOOGLE_CLIENT_ID=1234567890-abc.apps.googleusercontent.com \
  .
docker run --rm -p 8080:8080 fortuna-ui:web
```

Use your own address for `FORTUNA_API_BASE_URL`: with a proxy that routes
`/api/` as described under [Web](#web), it is the web UI's own origin. The image
answers `/healthz` with `ok`. The build always uses the HTTP transport, since
offline mode cannot run in a browser.

{{% alert title="Build arguments are public" color="warning" %}}
They become `--dart-define` values compiled into the JavaScript bundle, which
anyone who loads the page can read. Never pass a secret as a build argument.
Because the values are baked in, each environment needs its own image build.
{{% /alert %}}

If you already run Heimdall on `8080`, map the container to another host port,
for example `-p 8082:8080`. The project's own local setup serves the web UI
container at `http://127.0.0.1:8082`.

## Connecting to Heimdall

In connected mode Heimdall owns identity, and Fortuna is one **scope** in it. Only
the API talks to Heimdall: the UI signs in through Fortuna's `/api/auth/*`
routes and holds no Heimdall setting of its own, only the API's address. How that
works is in
[Signing in through Heimdall]({{< relref "/docs/architecture#signing-in-through-heimdall" >}}).
What a client system needs from Heimdall, and what Heimdall expects of it, is on
Heimdall's own pages:
[Integrating with Heimdall](https://artur-rios.github.io/heimdall-docs/docs/integrating/)
and
[Heimdall installation](https://artur-rios.github.io/heimdall-docs/docs/installation/).

To connect a Fortuna API to a Heimdall, carry these through in order.

1. **Run Heimdall over HTTPS.** `FORTUNA_HEIMDALL_BASE_URL` must be an `https`
   URL, so Fortuna cannot be pointed at a Heimdall on plain `http://localhost:8080`.
   This is why the local example uses the local account instead, and why each
   environment of the project reaches its Heimdall through that Heimdall's public
   host.
2. **Create Fortuna's scope in Heimdall** and note its public identifier. Heimdall's
   Integrating page describes scopes.
3. **Set the API's Heimdall settings** in the env file (Compose) or the
   environment (source):

   | Fortuna setting | Value |
   | --- | --- |
   | `FORTUNA_HEIMDALL_BASE_URL` | Heimdall's HTTPS address. |
   | `FORTUNA_HEIMDALL_SCOPE_ID` | Fortuna's scope identifier, from step 2. Not a secret. |
   | `FORTUNA_AUTH_TOKEN_SECRET` | The same value as Heimdall's `HEIMDALL_AUTH_TOKEN_SECRET`. A secret. |
   | `FORTUNA_AUTH_TOKEN_ISSUER` | Heimdall's `HEIMDALL_AUTH_TOKEN_ISSUER`. The local example uses `heimdall`. |
   | `FORTUNA_AUTH_TOKEN_AUDIENCE` | Heimdall's `HEIMDALL_AUTH_TOKEN_AUDIENCE`. The local example uses `fortuna`. |
   | `FORTUNA_AUTH_TOKEN_SECRET_PREVIOUS` | Only while rotating the secret. |
   | `FORTUNA_HEIMDALL_CALLER_KEY` | Optional. One of the keys in Heimdall's `HEIMDALL_TRUSTED_CALLER_KEYS` (at least 32 visible ASCII characters, no spaces, or start-up fails). Set, every request to Heimdall names the end user's address, so Heimdall rate-limits per user and not once per instance. If you set it behind a reverse proxy, also list that proxy in `FORTUNA_FORWARDED_KNOWN_PROXIES` or `FORTUNA_FORWARDED_KNOWN_NETWORKS`. A secret. |

   The Heimdall API setting names are the ones on Heimdall's Integrating page;
   check them against your Heimdall.
4. **Check Heimdall's version.** The caller key and
   self-registration came to Heimdall in API 1.4.0, according to Heimdall's
   Integrating page. Self-registration (API UC-81, new in Fortuna API 2.1.0) also
   needs the scope to have opened self-registration in Heimdall; otherwise
   Fortuna answers `403`.
5. **Point the UI at the API.** In the UI, set `FORTUNA_API_BASE_URL` (or type the
   address at the setup screen), as under [Build-time settings](#build-time-settings).
   For Google sign-in, also pass `FORTUNA_GOOGLE_CLIENT_ID` when you build, and
   turn Google on in Heimdall for the scope, which Heimdall's pages describe.
6. **Restart the API.** The settings are read once, at start.

A local setup that signs in through Heimdall therefore needs a Heimdall on
HTTPS. In the local example, `FORTUNA_HEIMDALL_BASE_URL` points at the development
environment's Heimdall as a placeholder, and the comments in the file say to use
that Heimdall's signing secret if you sign in through it.

## Where to go next

- The [Architecture]({{< relref "/docs/architecture" >}}) page shows how the parts
  fit together, including both deployment modes.
- The [User guide]({{< relref "/docs/user-guide" >}}) describes the interface,
  use case by use case, and the [API reference]({{< relref "/docs/api" >}}) maps
  the endpoints.
- [Operations]({{< relref "/docs/operations" >}}) covers the environments,
  health checks, logging and metrics of a running instance, and
  [Development]({{< relref "/docs/development" >}}) how the project is built and
  tested.
- The repositories hold the source documents: the
  [fortuna-api README](https://github.com/artur-rios/fortuna-api#readme),
  the [fortuna-ui README](https://github.com/artur-rios/fortuna-ui#readme) and
  the Operations & Infrastructure Documents
  ([API](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Operations%20%26%20Infrastructure%20Document.md),
  [UI](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Operations%20%26%20Infrastructure%20Document.md)).
  The [Repositories]({{< relref "/docs/repositories" >}}) page lists
  every one.
