# Dependencies

This document lists all system and package dependencies required to run the Happy self-hosted stack.

## System Dependencies

These must be installed on the host system:

### Required
- **Docker** - For running PostgreSQL, Redis, and MinIO infrastructure
- **Node.js 24+** - Runtime for all JavaScript/TypeScript code
- **Yarn 1.22.22+** - Package manager (specified in package.json)

### Optional
- **FFmpeg** - Required by happy-server for media processing
- **Python3** - Required by happy-server for some operations
- **psql** (PostgreSQL client) - For database migrations and debugging

## Installation Commands

### Ubuntu/Debian
```bash
# Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
# Log out and back in for group changes to take effect

# Node.js 24
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs

# Yarn
npm install -g yarn

# Optional: PostgreSQL client (for migrations)
sudo apt-get install -y postgresql-client

# Optional: FFmpeg and Python3
sudo apt-get install -y ffmpeg python3
```

### macOS
```bash
# Docker Desktop
# Download from https://www.docker.com/products/docker-desktop

# Node.js 24 (via Homebrew)
brew install node@24

# Yarn
npm install -g yarn

# Optional: PostgreSQL client
brew install postgresql

# Optional: FFmpeg and Python3
brew install ffmpeg python3
```

## Infrastructure Services (via Docker)

The following services run in Docker containers (managed by docker-compose.yaml):

| Service | Image | Port | Purpose |
|---------|-------|------|---------|
| PostgreSQL | postgres:17 | 5432 | Primary database |
| Redis | redis:7-alpine | 6379 | Caching and pub/sub |
| MinIO | minio/minio | 9000 (API), 9001 (Console) | S3-compatible object storage |

Start infrastructure:
```bash
docker compose up -d
```

Stop infrastructure:
```bash
docker compose down
```

Stop and remove data:
```bash
docker compose down -v
```

## Package Dependencies

After installing system dependencies, install package dependencies:

```bash
make install
```

This runs `yarn install` in each submodule:
- `happy-cli/` - Installs CLI dependencies including:
  - `tsx` - TypeScript executor (devDependency)
  - `shx` - Cross-platform shell commands (devDependency)
  - `pkgroll` - Package bundler (devDependency)
  - And all production dependencies

- `happy-server/` - Installs server dependencies including:
  - `tsx` - TypeScript executor (production dependency)
  - Prisma ORM and other server dependencies

- `happy/` - Installs webapp dependencies (Expo/React Native)

## Dependency Check

The `happy-launcher.sh` script automatically checks for installed package dependencies before starting services. If you see this error:

```
[ERROR] Dependencies not installed in happy-cli
```

Run:
```bash
make install
```

## CI Dependencies

The GitHub Actions CI workflow (`.github/workflows/ci.yml`) uses:
1. Node.js 24 via `setup-node` action
2. PostgreSQL 17 and Redis 7 via Docker services (same as local docker-compose)
3. MinIO server and client downloaded during workflow
4. Playwright for browser automation testing
5. All package dependencies via `yarn install --frozen-lockfile`
