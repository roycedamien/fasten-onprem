# Fasten-onprem Compilation Guide

This guide will help you compile the Fasten On-Premise application from source.

## Prerequisites

Before you begin, ensure you have the following tools installed:

### Required Software

- **Go** `v1.21+` (tested with v1.24.11)
- **Node.js** `v18.9.0+` (tested with v20.19.6)
- **Yarn** `1.22.19` or higher
- **Angular CLI** `v14.1.3`
- **Google Chrome** (for frontend tests)

### Install Prerequisites

**On macOS:**
```bash
brew install node
npm install -g @angular/cli@14.1.3
npm install -g yarn
brew install go
brew install docker
brew install --cask google-chrome
```

**On Linux (Ubuntu/Debian):**
```bash
# Install Node.js (v18 or later)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install Yarn
npm install -g yarn

# Install Go (v1.21+)
wget https://go.dev/dl/go1.24.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.24.0-linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin

# Install Chrome (for frontend tests)
# Follow instructions at https://www.google.com/chrome/
```

## Quick Start: Complete Compilation

To build the entire application from scratch:

```bash
# 1. Install dependencies
make dep-backend
make dep-frontend

# 2. Build backend
go build -o fasten backend/cmd/fasten/fasten.go

# 3. Build frontend
make build-frontend-prod
```

The backend binary will be created as `fasten` in the root directory, and the frontend will be built into the `dist/` directory.

## Backend Compilation

### Install Backend Dependencies
```bash
make dep-backend
```
This will:
- Run `go mod tidy`
- Run `go mod vendor`
- Generate necessary files

### Build the Backend Binary
```bash
go build -o fasten backend/cmd/fasten/fasten.go
```
This creates a binary named `fasten` in the root directory (~112 MB).

### Verify the Build
```bash
./fasten version
```
Expected output:
```
Starting fasten-onprem
Loading configuration file: config.yaml

  o888o                       o8                          
o888oo ooooooo    oooooooo8 o888oo ooooooooo8 oo oooooo   
 888   ooooo888  888ooooooo  888  888oooooo8   888   888  
 888 888    888          888 888  888          888   888  
o888o 88ooo88 8o 88oooooo88   888o  88oooo888 o888o o888o
github.com/fastenhealth/fasten-onprem         .-1.1.3

1.1.3
Finished fasten-onprem
```

## Frontend Compilation

The frontend is an Angular 14 application built with TypeScript.

### Install Frontend Dependencies
```bash
make dep-frontend
```
Or manually:
```bash
cd frontend && yarn install --frozen-lockfile --network-timeout 1000000
```

### Build for Production
```bash
make build-frontend-prod
```

This will create a `dist/` directory with the compiled frontend assets (~31 MB).

### Build for Sandbox (Testing with Synthetic Data)
```bash
make build-frontend-sandbox
```

### Build Warnings
The frontend build may show some warnings about:
- CommonJS dependencies (dwv, moment, lodash, etc.)
- CSS syntax warnings
- Bundle size exceeding budget

These warnings are normal and don't prevent successful compilation.

## Running the Application

### Production Mode

After building both frontend and backend:

```bash
./fasten start --config ./config.dev.yaml
```

Then open your browser to `http://localhost:9090`

### Development Mode

For development with auto-reload:

```bash
# Terminal 1: Start frontend dev server
make serve-frontend

# Terminal 2: Start backend server  
make serve-backend
```

Then open your browser to `http://localhost:4200` (frontend dev server proxies API requests to backend).

## Testing

### Run All Tests
```bash
make test
```

### Run Backend Tests Only
```bash
make test-backend
```

### Run Frontend Tests Only
```bash
make test-frontend
```

## Using Docker

### Run with Docker Compose (Development)
```bash
docker compose up -d
```
Or:
```bash
make serve-docker
```

### Run with Docker Compose (Production)
```bash
docker compose -f docker-compose-prod.yml up -d
```
Or:
```bash
make serve-docker-prod
```

## Configuration

### Development Configuration

Create a `config.dev.yaml` file:

```yaml
version: 1
web:
  listen:
    port: 9090
    host: 0.0.0.0
    basepath: ''
  src:
    frontend:
      path: ./dist
database:
  location: 'fasten.db'
cache:
  location: ''
log:
  file: ''
  level: INFO
```

## Directory Structure

```
fasten-onprem/
├── backend/              # Go backend source code
│   ├── cmd/fasten/      # Main application entry point
│   └── pkg/             # Backend packages
├── frontend/            # Angular frontend source code
│   └── src/            # Frontend source files
├── dist/               # Built frontend files (after compilation)
├── fasten              # Compiled backend binary (after build)
├── config.yaml         # Production configuration
├── config.dev.yaml     # Development configuration
├── Makefile            # Build automation
└── go.mod              # Go dependencies
```

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `make dep-backend` | Install backend dependencies |
| `make dep-frontend` | Install frontend dependencies |
| `make test` | Run all tests |
| `make test-backend` | Run backend tests only |
| `make test-frontend` | Run frontend tests only |
| `make build-frontend-prod` | Build frontend for production |
| `make build-frontend-sandbox` | Build frontend for sandbox/testing |
| `make serve-frontend` | Start frontend dev server |
| `make serve-backend` | Start backend dev server |
| `make serve-docker` | Run with Docker Compose (dev) |
| `make serve-docker-prod` | Run with Docker Compose (prod) |

## Build Results

After successful compilation:

- **Backend binary**: `./fasten` (~112 MB)
- **Backend version**: 1.1.3
- **Frontend assets**: `./dist/` directory (~31 MB)
- **Main bundle**: ~9.52 MB (uncompressed)

## Troubleshooting

### Backend Issues

**Issue**: Go version too old
```bash
go version
# Should be 1.21 or higher
```

**Issue**: Missing dependencies
```bash
make dep-backend
```

### Frontend Issues

**Issue**: Node version incompatible
```bash
node --version
# Should be v18.9.0 or higher
```

**Issue**: Yarn not found
```bash
npm install -g yarn
```

**Issue**: Build fails with memory error
```bash
# Increase Node memory limit
export NODE_OPTIONS="--max-old-space-size=4096"
make build-frontend-prod
```

## Using with Codespaces/Devcontainer

This repository includes a `.devcontainer/devcontainer.json` configuration that automatically sets up all necessary tools:

- Go 1.21
- Node.js 20
- Angular CLI 14.1.3
- Chrome (for testing)

Simply open the repository in GitHub Codespaces or VS Code with the Remote Containers extension, and all tools will be pre-configured.

## Additional Resources

- [CONTRIBUTING.md](./CONTRIBUTING.md) - Detailed development guide
- [README.md](./README.md) - General project information
- [Fasten Docs](https://docs.fastenhealth.com) - Official documentation
- [Technical Documentation](https://github.com/fastenhealth/docs/tree/main/technical) - Advanced technical information

## License

See [LICENSE.md](./LICENSE.md) for license information.
