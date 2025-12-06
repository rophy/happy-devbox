# Dependencies Installed

This document tracks all dependencies installed during the self-hosted setup process.

**Note**: Infrastructure services (PostgreSQL, Redis, MinIO) now run via Docker Compose. See the main `DEPENDENCIES.md` in the project root for installation instructions.

## System Packages

### Docker
- **Package**: `docker-ce` or `docker.io`
- **Purpose**: Runs infrastructure services (PostgreSQL, Redis, MinIO)
- **Status**: Required - all infrastructure runs in containers

### Node.js
- **Package**: `nodejs` (v24+)
- **Installed via**: nodesource repository
- **Purpose**: Runtime for happy-server, happy-cli, and webapp

### Yarn
- **Package**: `yarn` (v1.22.22+)
- **Installed via**: `npm install -g yarn`
- **Purpose**: Package manager for all JavaScript projects

## Infrastructure Services (via Docker Compose)

All infrastructure services are defined in `docker-compose.yaml` and managed via:
```bash
docker compose up -d      # Start
docker compose down       # Stop
docker compose down -v    # Stop and remove data
```

### PostgreSQL
- **Image**: `postgres:17`
- **Port**: 5432
- **Database**: handy (created automatically)
- **Credentials**: postgres/postgres

### Redis
- **Image**: `redis:7-alpine`
- **Port**: 6379

### MinIO (S3-compatible storage)
- **Image**: `minio/minio`
- **Ports**: 9000 (API), 9001 (Console)
- **Credentials**: minioadmin/minioadmin
- **Bucket**: `happy` (created by minio-init container)

## Node.js Dependencies

### happy-server
- Installed via `yarn install` in `/happy-server/`
- Includes: Fastify, Prisma, Socket.io, Redis client, MinIO SDK, etc.
- See `/happy-server/package.json` for full list

### happy-cli
- Installed via `yarn install` in `/happy-cli/`
- Includes: Claude Code SDK, Socket.io client, TweetNaCl for encryption, etc.
- See `/happy-cli/package.json` for full list

### happy webapp
- Installed via `yarn install` in `/happy/`
- Includes: Expo, React Native, etc.
- See `/happy/package.json` for full list

## Testing Scripts

### setup-test-credentials.mjs
- **Location**: `/scripts/setup-test-credentials.mjs`
- **Purpose**: Automates authentication flow for headless e2e testing
- **Dependencies**: tweetnacl, axios (via symlink to happy-cli/node_modules)
- **Creates**: Test credentials in `~/.happy-dev-test/`
- **Usage**: `node scripts/setup-test-credentials.mjs`

### e2e-demo.sh
- **Location**: `/e2e-demo.sh`
- **Purpose**: Complete e2e demo script that shows the full self-hosted flow
- **Dependencies**: happy-launcher.sh, setup-test-credentials.mjs
- **Usage**: `./e2e-demo.sh`

## Environment Variables

### For Testing
- `HAPPY_HOME_DIR=/root/.happy-dev-test` - Test credentials directory (separate from prod)
- `HAPPY_SERVER_URL=http://localhost:3005` - Local server URL

## Directory Structure

The `/scripts` directory contains e2e testing scripts and uses a symlink to access happy-cli dependencies:
```
scripts/
├── setup-test-credentials.mjs
├── auto-auth.mjs
└── node_modules -> ../happy-cli/node_modules  (symlink)
```

This approach keeps the system-under-test repos (happy-cli, happy-server) clean while allowing test scripts to access necessary dependencies.

## Browser Automation (Playwright)

### Installation

Playwright with Chromium is installed for headless browser testing of the webapp.

```bash
# Install Playwright globally
npm install -g playwright

# Install Chromium browser binaries
npx playwright install chromium

# Install system dependencies (fonts, xvfb, etc.)
npx playwright install-deps chromium
```

### Browser Test Scripts

Located in `/scripts/browser/`:

- **`inspect-webapp.mjs`** - Basic webapp inspection and screenshot tool
- **`test-webapp-e2e.mjs`** - Full E2E test with login flow

### Usage

```bash
cd scripts/browser

# Basic inspection with screenshot
node inspect-webapp.mjs --screenshot --console

# Full E2E test with login
node test-webapp-e2e.mjs "YOUR-SECRET-KEY"
```

### Environment Variables

- `WEBAPP_URL` - Override webapp URL (default: `http://localhost:8081`)
- `SCREENSHOT_DIR` - Directory for screenshots (default: `/tmp`)
