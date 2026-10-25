# Contributing to TorrentDrive

Thank you for your interest in contributing to TorrentDrive. This document provides guidelines for contributing to the deployment repository and the overall project.

## Repository Structure

TorrentDrive is split across four repositories:

| Repository | Purpose |
|------------|---------|
| [TorrentDrive/backend](https://github.com/TorrentDrive/backend) | .NET 9.0 API and background workers |
| [TorrentDrive/frontend](https://github.com/TorrentDrive/frontend) | Next.js 16 web application |
| [TorrentDrive/deploy](https://github.com/TorrentDrive/deploy) | Dockerfile, CI/CD, deployment scripts (this repo) |
| [TorrentDrive/website](https://github.com/TorrentDrive/website) | Documentation site (`tc.gitnasr.com/docs`) |

## How to Contribute

### Reporting Issues

- Use the **Issues** tab on the relevant repository
- Include steps to reproduce, expected behavior, and actual behavior
- For Docker/deployment issues, include `docker logs torrentdrive` output

### Pull Requests

1. Fork the relevant repository
2. Create a feature branch from `main`
   ```bash
   git checkout -b feat/your-feature-name
   ```
3. Make your changes
4. Test locally
5. Submit a pull request to `main` using a Conventional Commit title (`feat: ...`, `fix: ...`, `docs: ...`)

### Branch Naming

| Prefix | Purpose |
|--------|---------|
| `feat/` | New functionality |
| `fix/` | Bug fixes |
| `docs/` | Documentation changes |
| `refactor/` | Code restructuring |

## Development Setup

### Backend

```bash
git clone https://github.com/TorrentDrive/backend.git
cd backend
cp .env.example .env
docker-compose up -d
```

Requires: .NET 9.0 SDK, Docker

### Frontend

```bash
git clone https://github.com/TorrentDrive/frontend.git
cd frontend
cp .env.example .env.local
npm ci
npm run dev
```

Requires: Node.js 22+, npm

### Deploy Repo

Changes to the Dockerfile, entrypoint, or supervisord config should be tested by building the image locally:

```bash
git clone https://github.com/TorrentDrive/deploy.git
cd deploy

# Clone source repos for build context
git clone https://github.com/TorrentDrive/backend.git
git clone https://github.com/TorrentDrive/frontend.git

# Build locally
docker build -t torrentdrive-test .
```

## Build & Release Process

Every repository releases automatically when changes land on `main`:

1. Merging to `main` in `backend` or `frontend` runs CI, computes the next SemVer version from Conventional Commits, publishes a GitHub Release, builds and pushes component Docker images, and sends a `repository_dispatch` (`build-combined`) to `deploy`.
2. `deploy` runs the smoke test and multi-arch build (`amd64` + `arm64`), publishes the combined manifest (`ghcr.io/torrentdrive/torrentdrive`), updates `components.lock.json` and `VERSION` directly on `main`, and creates a matching GitHub Release.
3. Details are on the [Architecture](https://tc.gitnasr.com/docs) documentation page.

Use pull requests rather than pushing to `main` directly.

## Code Style

### Backend (.NET)
- Follow existing Clean Architecture patterns
- Use async/await consistently
- Configuration a user should be able to change belongs in `SystemSettings` or
  `UserSettings`, not in an environment variable. The environment is for things the app
  cannot discover itself: connection strings, and overrides for unusual deployments.

### Frontend (Next.js)
- TypeScript strict mode
- Tailwind CSS for styling
- Zod for runtime validation
- React Query for server state

### Deploy Repo
- Shell scripts must be POSIX-compatible where possible
- Dockerfile instructions should be ordered for optimal layer caching
- Every environment variable must have a default in `entrypoint.sh`. `docker run` with
  no `--env-file` and no `-e` flags has to produce a working install — that is the
  property the installer depends on, so a change that breaks it is a bug.

## Questions?

Open an issue or reach out to the maintainers.
