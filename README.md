# Golang Builder & Deployer Action

A reusable GitHub Action for building, testing, and deploying Go applications to Kubernetes with multi-platform Docker support.

## Features

- 🧪 Runs Go unit tests and generates coverage reports
- 🐳 Builds multi-platform Docker images (amd64/arm64)
- 📦 Pushes images to GitHub Container Registry
- 🚀 Deploys to Kubernetes (staging and/or production)
- 🔐 Integrates with HashiCorp Vault for secrets management
- ⚙️ Configurable staging and production deployments

## Usage

### Basic Example

```yaml
name: Build & Deploy

on:
  push:
    branches: [ "*" ]
  pull_request:
    branches: [ "*" ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build and Deploy
        uses: cfindlayisme/golang-builder@v1
        with:
          vault-addr: ${{ secrets.VAULT_ADDR }}
          vault-role-id: ${{ secrets.VAULT_ROLE_ID }}
          vault-secret-id: ${{ secrets.VAULT_SECRET_ID }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{ github.repository }}
          ref: ${{ github.ref }}
          deployment-name: 'my-app'
          staging-enabled: 'true'
          production-enabled: 'true'
```

### Advanced Example with Custom Configuration

```yaml
name: Build & Deploy

on:
  push:
    branches: [ "*" ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build and Deploy
        uses: cfindlayisme/golang-builder@v1
        with:
          # Go Configuration (optional - defaults to latest stable)
          go-version: '1.24.2'
          
          # Vault Configuration
          vault-addr: ${{ secrets.VAULT_ADDR }}
          vault-role-id: ${{ secrets.VAULT_ROLE_ID }}
          vault-secret-id: ${{ secrets.VAULT_SECRET_ID }}
          vault-kubeconfig-path: 'kv/data/pipeline/k3s'
          
          # GitHub Configuration
          github-token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{ github.repository }}
          ref: ${{ github.ref }}
          
          # Deployment Configuration
          deployment-name: 'wmb'
          staging-enabled: 'false'
          production-enabled: 'true'
          staging-namespace: 'staging'
          production-namespace: 'production'
          
          # Docker Configuration
          docker-platforms: 'linux/amd64,linux/arm64'
          dockerfile-path: './Dockerfile'
          docker-context: '.'
```

### Disable Staging Deployment

```yaml
- name: Build and Deploy (Production Only)
  uses: cfindlayisme/golang-builder@v1
  with:
    vault-addr: ${{ secrets.VAULT_ADDR }}
    vault-role-id: ${{ secrets.VAULT_ROLE_ID }}
    vault-secret-id: ${{ secrets.VAULT_SECRET_ID }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    ref: ${{ github.ref }}
    deployment-name: 'my-app'
    staging-enabled: 'false'
    production-enabled: 'true'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `go-version` | Go version to use | No | `stable` (latest) |
| `vault-addr` | Vault server address | Yes | - |
| `vault-role-id` | Vault AppRole role ID | Yes | - |
| `vault-secret-id` | Vault AppRole secret ID | Yes | - |
| `vault-kubeconfig-path` | Vault path to kubeconfig secret | No | `kv/data/pipeline/k3s` |
| `github-token` | GitHub token for container registry | Yes | - |
| `repository` | GitHub repository (owner/name) | Yes | - |
| `ref` | Git reference being built | Yes | - |
| `staging-enabled` | Enable staging deployment | No | `true` |
| `production-enabled` | Enable production deployment | No | `true` |
| `staging-namespace` | Kubernetes namespace for staging | No | `staging` |
| `production-namespace` | Kubernetes namespace for production | No | `production` |
| `deployment-name` | Kubernetes deployment name | Yes | - |
| `docker-platforms` | Docker platforms to build for | No | `linux/amd64,linux/arm64` |
| `dockerfile-path` | Path to Dockerfile | No | `./Dockerfile` |
| `docker-context` | Docker build context | No | `.` |

## Outputs

| Output | Description |
|--------|-------------|
| `test-result` | Unit test result (success/failure) |
| `build-result` | Docker build result (success/failure) |
| `deploy-result` | Deployment result (success/failure) |

## Workflow Behavior

### Staging Deployment
- Runs when `staging-enabled: 'true'`
- Builds and pushes `ghcr.io/<repository>:staging` tag
- Deploys to staging namespace for all branches

### Production Deployment
- Runs when `production-enabled: 'true'` AND `ref == 'refs/heads/main'`
- Builds and pushes `ghcr.io/<repository>:latest` tag
- Only deploys to production on main branch pushes

## Prerequisites

### GitHub Secrets Required
- `VAULT_ADDR`: Your HashiCorp Vault server URL
- `VAULT_ROLE_ID`: Vault AppRole role ID
- `VAULT_SECRET_ID`: Vault AppRole secret ID
- `GITHUB_TOKEN`: Automatically provided by GitHub Actions

### Vault Configuration
The action expects a kubeconfig stored in Vault at the specified path (default: `kv/data/pipeline/k3s`).

### Kubernetes Setup
Your Kubernetes cluster should have:
- Namespaces created (default: `staging` and `production`)
- Deployment resources with the specified `deployment-name`

## Example Project Structure

```
my-go-project/
├── .github/
│   └── workflows/
│       └── build-deploy.yml
├── main.go
├── go.mod
├── go.sum
├── Dockerfile
└── *_test.go
```

## License

LGPL v3

## Author

cfindlayisme
