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

This demo is built for [Self-Host Pro](https://selfhostpro.com) and its registry at **shpcr.io**. To run the workflows against *your* Self-Host Pro account (e.g. for testing or your own product), fork the repo and configure three values in GitHub.

**You will need:**

| Type    | Name            | Where to set | Description |
|---------|-----------------|--------------|-------------|
| Variable | `IMAGE_NAME`    | **Settings → Secrets and variables → Actions → Variables** | Image name without tag, e.g. `shpcr.io/your-org/your-app` |
| Secret   | `SHPCR_USERNAME` | **Settings → Secrets and variables → Actions → Secrets** | Self-Host Pro registry username (see below) |
| Secret   | `SHPCR_PASSWORD` | **Settings → Secrets and variables → Actions → Secrets** | Self-Host Pro registry password (see below) |

**Step 1 — Self-Host Pro account and token**

1. Create an account (if needed): [Register at Self-Host Pro](https://app.selfhostpro.com/register).
2. Create an access token for the registry: [Access tokens](https://app.selfhostpro.com/profile/access-tokens). This token is used to authenticate with **shpcr.io**.
3. In your fork, go to **Settings → Secrets and variables → Actions**.
   - **Secrets:** Add `SHPCR_USERNAME` and `SHPCR_PASSWORD`. For Self-Host Pro, use your **access token as both the username and the password** when logging in to the registry.
   - **Variables:** Add `IMAGE_NAME` with your image name (no tag), e.g. `shpcr.io/your-org/your-app`.

**Step 2 — Run the workflows**

After the variable and secrets are set, the existing workflows (stable release, prerelease, edge) will build and push to your image on shpcr.io. Trigger them by pushing to `main`, creating a pre-release, or publishing a release.

**Defaults when not forking**

If you run the repo without setting `IMAGE_NAME`, builds still run and push to the default `shpcr.io/selfhostpro/demo` (which will likely fail because you don't have permission to push to that image). The workflow always uses the `SHPCR_*` secrets for registry login when present, so set those only in your fork if you want to publish to your own account.

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
