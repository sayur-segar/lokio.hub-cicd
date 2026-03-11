# lokio.hub-cicd

Collection of reusable CD pipeline templates for [Lokio](https://lokio.dev). Generate production-ready GitHub Actions workflows for any stack in seconds.

## Quick Start

```bash
lokio g cd-go          # Go service
lokio g cd-bun         # Bun/Node service
lokio g cd-rust        # Rust service
lokio g cd-python      # Python service
lokio g cd-php         # PHP service
lokio g cd-java        # Java service
lokio g cd-kotlin      # Kotlin service
lokio g cd-ruby        # Ruby service
lokio g cd-dotnet      # .NET service
lokio g cd-elixir      # Elixir service
lokio g cd-fe          # Frontend app
lokio g cd-flutter     # Flutter mobile
lokio g cd-react-native  # React Native mobile
lokio g cd-kmp         # Kotlin Multiplatform
```

Output: `.github/workflows/cd-<name>.yml`

---

## Templates

### Backend

| Template | Language | Frameworks | Test |
|----------|----------|------------|------|
| `cd-go` | Go | — | `go test ./... -race -coverprofile` |
| `cd-bun` | Bun / Node.js | — | `bun test --coverage` |
| `cd-rust` | Rust | — | `cargo test` + clippy + `cargo fmt` |
| `cd-python` | Python | FastAPI · Django · Flask | `pytest --cov` + ruff + mypy |
| `cd-php` | PHP | Laravel · Symfony | `php artisan test` / phpunit |
| `cd-java` | Java | Spring Boot | `./gradlew test` / `./mvnw test` + JaCoCo |
| `cd-kotlin` | Kotlin | Spring Boot · Ktor | `./gradlew test` + ktlint / detekt |
| `cd-ruby` | Ruby | Rails · Sinatra | `bundle exec rspec` + rubocop |
| `cd-dotnet` | .NET C# | ASP.NET Core | `dotnet test --collect:"XPlat Code Coverage"` |
| `cd-elixir` | Elixir | Phoenix | `mix test` + credo + dialyzer |

### Frontend

| Template | Frameworks | Deploy Targets |
|----------|-----------|----------------|
| `cd-fe` | Next.js · React · Vue · Nuxt · SvelteKit · Angular · Astro · Remix | Vercel · Netlify · S3+CloudFront · Kubernetes |

### Mobile

| Template | Platform | Deploy Targets |
|----------|---------|----------------|
| `cd-flutter` | Android + iOS | Firebase App Distribution · Play Store · TestFlight |
| `cd-react-native` | Android + iOS | Firebase · Play Store · TestFlight · Expo EAS |
| `cd-kmp` | Android · iOS · JVM Desktop · Kotlin/JS | Firebase · Play Store · TestFlight |

---

## Parameters

Each template is interactive — Lokio will prompt for each option. All parameters have sensible defaults.

### Shared (backend Docker templates)

| Parameter | Type | Options | Default |
|-----------|------|---------|---------|
| `branch` | string | — | `main` |
| `registry` | options | `ghcr` · `dockerhub` · `ecr` | `ghcr` |
| `deploy_type` | options | `ssh` · `k8s` · `railway` · `fly` · `none` | `ssh` |
| `has_tests` | boolean | — | `true` |
| `has_migration` | boolean | — | `false` |
| `port` | number | — | varies |
| `dockerfile_path` | string | — | `Dockerfile` |

### Per-language extras

**`cd-go`** — `go_version` (default: `1.22`), `test_command`

**`cd-bun`** — `bun_version` (default: `latest`), `test_command`

**`cd-rust`** — `rust_version`: `stable` · `beta` · `nightly`

**`cd-python`** — `framework`: `fastapi` · `django` · `flask`, `package_manager`: `uv` · `poetry` · `pip` · `pipenv`, `python_version`

**`cd-php`** — `framework`: `laravel` · `symfony`, `php_version`, `deploy_type` also includes `forge`

**`cd-java`** — `java_version`: `21` · `17` · `11`, `build_tool`: `gradle` · `maven`

**`cd-kotlin`** — `framework`: `springboot` · `ktor`, `java_version`

**`cd-ruby`** — `framework`: `rails` · `sinatra`, `ruby_version`

**`cd-dotnet`** — `dotnet_version`: `9.0` · `8.0` · `6.0`, `deploy_type` also includes `azure`

**`cd-elixir`** — `elixir_version`, `otp_version`

**`cd-fe`** — `framework`: `nextjs` · `react` · `vue` · `nuxt` · `sveltekit` · `angular` · `astro` · `remix`, `package_manager`: `bun` · `pnpm` · `npm` · `yarn`, `deploy_type`: `vercel` · `netlify` · `s3` · `k8s`, `build_dir`

**`cd-flutter`** — `flutter_version`, `flutter_channel`, `platform`: `both` · `android` · `ios`, `android_deploy`: `firebase` · `play_store`, `ios_deploy`: `firebase` · `testflight`, `android_flavor`

**`cd-react-native`** — `package_manager`, `platform`, `android_deploy`: `firebase` · `play_store` · `expo`, `ios_deploy`: `firebase` · `testflight` · `expo`

**`cd-kmp`** — `targets`: `all` · `both` · `android` · `ios` · `jvm` · `js`, `java_version`, `android_deploy`, `ios_deploy`

---

## What Each Pipeline Includes

### Backend Pipelines

```
push → test → build & push Docker image → deploy → notify Slack
```

- **Test job** — language-specific linting, type checking, unit tests, coverage upload to Codecov
- **Build & Push** — multi-platform Docker image (`linux/amd64` + `linux/arm64`), GHA layer cache, Docker metadata with SHA tags
- **Deploy** — choice of target:
  - `ssh` — pull image, run migrations, zero-downtime container swap with health check
  - `k8s` — `kubectl set image` + rollout status wait
  - `railway` — Railway CLI deploy
  - `fly` — `flyctl deploy --remote-only`
- **Notify** — Slack notification on success or failure

### Frontend Pipelines

```
push → test → build → deploy → notify Slack
```

- `vercel` — `amondnet/vercel-action`
- `netlify` — `nwtgck/actions-netlify`
- `s3` — sync to S3 bucket with cache headers + CloudFront invalidation
- `k8s` — Docker build & push + Kubernetes rolling update

### Mobile Pipelines

```
push → test (ubuntu) → build-android (ubuntu) + build-ios (macos) → deploy → notify Slack
```

- Android builds produce signed `.aab` (App Bundle)
- iOS builds produce signed `.ipa` via Xcode archive
- Parallel Android + iOS jobs (runs concurrently)
- Artifacts uploaded for manual distribution

---

## Required Secrets

### All backend pipelines

| Secret | Description |
|--------|-------------|
| `SLACK_WEBHOOK` | Slack Incoming Webhook URL |

### Registry secrets

| Registry | Secrets |
|----------|---------|
| `ghcr` | `GITHUB_TOKEN` (automatic) |
| `dockerhub` | `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN` |
| `ecr` | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` |

### Deploy secrets

| Target | Secrets |
|--------|---------|
| `ssh` | `SSH_HOST`, `SSH_USER`, `SSH_PRIVATE_KEY`, `SSH_PORT` (optional) |
| `k8s` | `KUBE_CONFIG`, `KUBE_NAMESPACE` |
| `railway` | `RAILWAY_TOKEN` |
| `fly` | `FLY_API_TOKEN` |
| `azure` (.NET) | `AZURE_CREDENTIALS`, `AZURE_APP_NAME` |
| `forge` (PHP) | `FORGE_DEPLOY_URL` |

### Frontend secrets

| Target | Secrets |
|--------|---------|
| `vercel` | `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID` |
| `netlify` | `NETLIFY_AUTH_TOKEN`, `NETLIFY_SITE_ID` |
| `s3` | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `S3_BUCKET`, `CLOUDFRONT_DISTRIBUTION_ID` |

### Mobile secrets

| Secret | Description |
|--------|-------------|
| `ANDROID_KEYSTORE_BASE64` | Base64-encoded `.jks` keystore file |
| `ANDROID_KEYSTORE_PASSWORD` | Keystore password |
| `ANDROID_KEY_ALIAS` | Key alias |
| `ANDROID_KEY_PASSWORD` | Key password |
| `ANDROID_PACKAGE_NAME` | App package name (Play Store) |
| `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` | Google Play service account JSON |
| `FIREBASE_ANDROID_APP_ID` | Firebase Android App ID |
| `FIREBASE_IOS_APP_ID` | Firebase iOS App ID |
| `FIREBASE_SERVICE_ACCOUNT` | Firebase service account JSON |
| `IOS_CERTIFICATE_BASE64` | Base64-encoded `.p12` certificate |
| `IOS_CERTIFICATE_PASSWORD` | Certificate password |
| `IOS_BUNDLE_ID` | iOS Bundle ID |
| `APPSTORE_ISSUER_ID` | App Store Connect Issuer ID |
| `APPSTORE_KEY_ID` | App Store Connect Key ID |
| `APPSTORE_PRIVATE_KEY` | App Store Connect private key |
| `EXPO_TOKEN` | Expo access token (React Native + Expo only) |

---

## Migration Commands

Each language uses the appropriate migration tool automatically:

| Language | Command |
|----------|---------|
| Go | `./app migrate` |
| Bun | `bun run migrate` |
| Rust | `sqlx migrate run` |
| PHP (Laravel) | `php artisan migrate --force` |
| PHP (Symfony) | `php bin/console doctrine:migrations:migrate --no-interaction` |
| Python (FastAPI) | `alembic upgrade head` |
| Python (Django) | `python manage.py migrate --noinput` |
| Python (Flask) | `flask db upgrade` |
| Ruby (Rails) | `bundle exec rails db:migrate` |
| Elixir (Phoenix) | `bin/<name> eval "App.Release.migrate()"` |
| Kotlin | `java -jar app.jar --spring.flyway.enabled=true` |

Migrations run **before** the new container starts (SSH) or as a Kubernetes Job (k8s).

---

## Concurrency & Caching

All pipelines include:

- **Concurrency control** — cancels in-progress runs on new push to the same branch
- **Docker layer cache** — via GitHub Actions cache (`type=gha`)
- **Language-level cache** — Go modules, Cargo registry, Maven/Gradle, Composer, pip/uv/poetry, Ruby gems, NuGet, Mix deps
- **Multi-platform builds** — `linux/amd64` + `linux/arm64` (Apple Silicon compatible)

---

## Version

`v2.0.16` — built with [Lokio](https://lokio.dev)
