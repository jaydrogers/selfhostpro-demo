# Self-Host Pro Demo App

A reference implementation demonstrating how to build, version, and distribute self-hosted software using [Self-Host Pro](https://selfhostpro.com).

**While this demo uses Laravel, Self-Host Pro works with any containerized application** — Node.js, Python, Go, Ruby, or any stack that runs in Docker.

## What This Demonstrates

This app showcases the complete workflow for selling self-hosted software:

- **Database Persistence** — SQLite data survives container restarts
- **Version Embedding** — App version injected at build time via `composer.json`
- **Automated Builds** — GitHub Actions builds and pushes to Self-Host Pro's registry
- **Release Strategies** — Edge, pre-release, and stable release workflows

## GitHub Actions Workflows

The CI/CD setup uses a single reusable workflow called by three trigger files. Check out [`.github/workflows/`](.github/workflows/) to see:

| File | Trigger | Tags Produced |
|------|---------|---------------|
| `action_stable-release.yml` | GitHub Release | `latest`, `1.2.0`, `1` |
| `action_prerelease.yml` | GitHub Pre-release | `prerelease`, `prerelease-v1.0.0-beta` |
| `action_edge.yml` | Push to `main` | `edge-main` |
| `service_docker-build-and-publish.yml` | Called by above | (reusable build logic) |

Each trigger workflow is ~20 lines — they just pass different tags to the shared build pipeline. Copy a trigger file to add new release channels like `nightly` or `canary`.

### Forking and Publishing to Your Own Self-Host Pro Account

This demo pushes to [Self-Host Pro](https://selfhostpro.com)’s registry (**shpcr.io**). To have GitHub Actions build and push to *your* account instead:

1. **Fork** this repo.
2. **Get a Self-Host Pro access token:** [Register](https://app.selfhostpro.com/register) (if needed), then [create an access token](https://app.selfhostpro.com/profile/access-tokens). The token is used as both **username and password** when logging in to the registry.
3. **Configure your fork** in GitHub (see below). Use **Variables** for image/registry names and **Secrets** only for the token.

---

#### Where to configure: Settings → Secrets and variables → Actions

In your fork, open **Settings** → **Secrets and variables** → **Actions**. The first tab is **Secrets**, then **Variables**. Configure in that order:

**Secrets tab** (first) — for credentials. Add:

| Name | Value |
|------|--------|
| `SHPCR_USERNAME` | Your Self-Host Pro access token |
| `SHPCR_PASSWORD` | Same access token (use as both username and password) |

**Variables tab** (second) — for names (not sensitive). Add:

| Name | Value |
|------|--------|
| `IMAGE_NAME` | Image path only, e.g. `your-org/your-app` (no registry, no tag). This is the only variable you need for Self-Host Pro. |

**Optional:** `REGISTRY` — Only set this if you’re using a different registry host (e.g. a dev/staging registry). Everyone else can leave it unset; the workflow defaults to `shpcr.io`.

---

**Triggering builds:** After the Variables and Secrets above are set, push to `main` (edge), create a pre-release, or publish a release. The workflows will build and push to your image. If you don’t set anything, the workflow still runs but pushes to `shpcr.io/selfhostpro/demo`, which will fail unless you have push access to that image.

## Local Development with Spin

This project uses [Spin](https://serversideup.net/open-source/spin/) for local Docker development. Spin provides a consistent environment that mirrors production.

### Prerequisites

- Docker & Docker Compose
- [Spin](https://serversideup.net/open-source/spin/)

### Getting Started

```bash
# Clone the repo
git clone https://github.com/selfhostpro/demo-app.git
cd demo-app

# Copy environment file
cp .env.example .env

# Install dependencies
spin run php composer install
spin run node yarn install

# Generate app key and run migrations
spin run php php artisan key:generate
spin run php php artisan migrate

# Seed demo data
spin run php php artisan initialize --force

# Start application servers with Vite
spin up
```

Visit `https://laravel.dev.test` (add to `127.0.0.1 laravel.dev.test` in `/etc/hosts` if needed).

## Docker Setup

The project includes two Dockerfiles:

- **`Dockerfile.php`** — Production PHP image with FrankenPHP
- **`Dockerfile.node`** — Node.js for building frontend assets

### Building for Production

The GitHub Actions workflow handles production builds, but you can build manually:

```bash
# Install dependencies first
spin run php composer install --optimize-autoloader --no-dev
spin run node yarn install && yarn build

# Build the image
docker build -t shpcr.io/yourname/app:latest -f Dockerfile.php .

# Push to Self-Host Pro registry
docker login shpcr.io
docker push shpcr.io/yourname/app:latest
```

## Project Structure

```
├── .github/workflows/       # CI/CD workflows (start here!)
│   ├── action_stable-release.yml
│   ├── action_prerelease.yml
│   ├── action_edge.yml
│   └── service_docker-build-and-publish.yml
├── app/
│   ├── Console/Commands/    # Artisan commands
│   └── Models/              # Eloquent models
├── resources/
│   ├── css/                 # Tailwind CSS
│   ├── js/                  # Globe animation & app JS
│   └── views/               # Blade templates
├── docker-compose.yml       # Base compose config
├── docker-compose.dev.yml   # Development overrides
├── docker-compose.ci.yml    # CI build config
├── Dockerfile.php           # Production PHP image
└── Dockerfile.node          # Node build image
```

## Versioning

Set the version in `composer.json` or via environment variable:

```json
{
  "version": "1.0.0"
}
```

```bash
APP_VERSION=1.0.0
```

The GitHub Actions workflow automatically updates `composer.json` with the release tag version before building.

## Artisan Commands

```bash
# Seed space mission demo data
php artisan initialize [--force]

# Check app status
php artisan status

# Reset to clean state
php artisan reset [--force]
```

## Learn More

- [Self-Host Pro Documentation](https://selfhostpro.com/docs)
- [Spin Documentation](https://serversideup.net/open-source/spin/docs)
- [Server Side Up Discord](https://serversideup.net/discord)
