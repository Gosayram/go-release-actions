# Go Release Actions

A collection of reusable GitHub Actions workflows for Go projects, designed to streamline CI/CD processes and ensure consistent quality across repositories.

## Overview

This repository provides pre-configured, reusable workflows that can be easily integrated into any Go project. The workflows follow industry best practices and include comprehensive configuration options for different project requirements.

## Available Workflows

### 1. Build and Test (`build-and-test.yml`)

Comprehensive build and testing workflow with coverage reporting and multi-platform support.

**Features:**
- Cross-platform builds with race detection
- Code coverage analysis with configurable thresholds
- Dependency caching for faster builds
- Artifact upload for coverage reports
- Configurable timeouts and Go versions

**Basic Usage:**
```yaml
name: Build and Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-test:
    uses: Gosayram/go-release-actions/.github/workflows/build-and-test.yml@main
    with:
      go-version: "1.21"
      coverage-threshold: 80
```

**Advanced Usage:**
```yaml
jobs:
  build-test:
    uses: Gosayram/go-release-actions/.github/workflows/build-and-test.yml@main
    with:
      go-version: "1.21"
      working-directory: "./api"
      test-timeout: "15m"
      enable-coverage: true
      coverage-threshold: 85
      cache-key-suffix: "api-service"
```

### 2. Release (`release.yml`)

Automated release workflow with cross-platform binary builds and GitHub releases.

**Features:**
- Multi-platform binary compilation
- Automated changelog generation
- SHA256 checksums for all artifacts
- GitHub release creation with assets
- Support for pre-releases and drafts

**Basic Usage:**
```yaml
name: Release

on:
  push:
    tags: [ 'v*' ]

jobs:
  release:
    uses: Gosayram/go-release-actions/.github/workflows/release.yml@main
    with:
      app-name: "my-application"
    secrets:
      release-token: ${{ secrets.GITHUB_TOKEN }}
```

**Advanced Usage:**
```yaml
jobs:
  release:
    uses: Gosayram/go-release-actions/.github/workflows/release.yml@main
    with:
      go-version: "1.21"
      app-name: "my-app"
      build-platforms: '["linux/amd64", "darwin/amd64", "windows/amd64"]'
      enable-checksums: true
      create-release: true
      prerelease: false
      draft: false
    secrets:
      release-token: ${{ secrets.GITHUB_TOKEN }}
```

### 3. Lint (`lint.yml`)

Comprehensive code quality and linting workflow.

**Features:**
- Go formatting verification
- Go vet static analysis
- Module tidiness check
- Staticcheck analysis
- GolangCI-Lint with customizable rules
- Configurable severity levels

**Basic Usage:**
```yaml
name: Lint

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  lint:
    uses: Gosayram/go-release-actions/.github/workflows/lint.yml@main
```

**Advanced Usage:**
```yaml
jobs:
  lint:
    uses: Gosayram/go-release-actions/.github/workflows/lint.yml@main
    with:
      go-version: "1.21"
      golangci-lint-version: "v1.55.2"
      enable-fmt-check: true
      enable-vet-check: true
      enable-mod-check: true
      enable-staticcheck: true
      golangci-config-file: ".golangci.yml"
      fail-on-warnings: false
```

### 4. Security Scan (`security-scan.yml`)

Multi-tool security scanning workflow for vulnerability detection.

**Features:**
- Gosec security scanner
- Nancy vulnerability scanner for dependencies
- Trivy filesystem vulnerability scanning
- CodeQL security analysis
- Dependency review for pull requests
- Configurable severity thresholds

**Basic Usage:**
```yaml
name: Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security:
    uses: Gosayram/go-release-actions/.github/workflows/security-scan.yml@main
```

**Advanced Usage:**
```yaml
jobs:
  security:
    uses: Gosayram/go-release-actions/.github/workflows/security-scan.yml@main
    with:
      go-version: "1.21"
      enable-gosec: true
      enable-nancy: true
      enable-trivy: true
      enable-codeql: true
      gosec-severity: "medium"
      trivy-severity: "HIGH,CRITICAL"
      fail-on-vulnerability: true
```

### 5. Auto Tag (`auto-tag.yml`)

Automatic tagging workflow that creates tags when version files change.

**Features:**
- Monitors version file changes
- Creates semantic version tags automatically
- Generates changelog from commits or CHANGELOG.md
- Creates GitHub releases automatically
- Configurable tag prefixes and branches

**Basic Usage:**
```yaml
name: Auto Tag

on:
  push:
    branches: [ main ]

jobs:
  auto-tag:
    uses: Gosayram/go-release-actions/.github/workflows/auto-tag.yml@main
    with:
      version-file: ".release-version"
      tag-prefix: "v"
```

**Advanced Usage:**
```yaml
jobs:
  auto-tag:
    uses: Gosayram/go-release-actions/.github/workflows/auto-tag.yml@main
    with:
      version-file: "VERSION"
      tag-prefix: "v"
      branch: "main"
      create-release: true
      release-draft: false
      release-prerelease: false
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 6. Matrix Test (`matrix-test.yml`)

Comprehensive matrix testing across multiple Go versions and platforms.

**Features:**
- Multi-version Go testing
- Cross-platform compatibility testing
- Cross-compilation verification
- Race detection support
- Benchmark testing
- Compatibility checks across Go versions

**Basic Usage:**
```yaml
name: Matrix Test

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  matrix-test:
    uses: Gosayram/go-release-actions/.github/workflows/matrix-test.yml@main
```

**Advanced Usage:**
```yaml
jobs:
  matrix-test:
    uses: Gosayram/go-release-actions/.github/workflows/matrix-test.yml@main
    with:
      go-versions: '["1.20", "1.21", "1.22"]'
      operating-systems: '["ubuntu-latest", "macos-latest", "windows-latest"]'
      enable-race-detection: true
      enable-benchmarks: true
      benchmark-time: "5s"
      extra-test-args: "-count=1"
```

## Complete CI/CD Pipeline Example

Here's a complete example combining all workflows:

```yaml
name: Complete CI/CD

on:
  push:
    branches: [ main, develop ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]

jobs:
  lint:
    uses: Gosayram/go-release-actions/.github/workflows/lint.yml@main
    with:
      fail-on-warnings: false

  security:
    uses: Gosayram/go-release-actions/.github/workflows/security-scan.yml@main
    with:
      fail-on-vulnerability: false

  matrix-test:
    uses: Gosayram/go-release-actions/.github/workflows/matrix-test.yml@main
    with:
      go-versions: '["1.20", "1.21", "1.22"]'
      enable-race-detection: true

  build-test:
    needs: [lint, security]
    uses: Gosayram/go-release-actions/.github/workflows/build-and-test.yml@main
    with:
      go-version: "1.21"
      coverage-threshold: 80

  auto-tag:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    needs: [build-test, matrix-test]
    uses: Gosayram/go-release-actions/.github/workflows/auto-tag.yml@main
    with:
      version-file: ".release-version"
      tag-prefix: "v"
      create-release: false

  release:
    if: startsWith(github.ref, 'refs/tags/')
    needs: [build-test, matrix-test]
    uses: Gosayram/go-release-actions/.github/workflows/release.yml@main
    with:
      app-name: "my-application"
    secrets:
      release-token: ${{ secrets.GITHUB_TOKEN }}
```

## Configuration Files

### GolangCI-Lint Configuration

Create a `.golangci.yml` file in your repository root for custom linting rules:

```yaml
run:
  timeout: 5m
  modules-download-mode: readonly

linters:
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - typecheck
    - unused
    - gocritic
    - gocyclo
    - gosec
    - misspell

linters-settings:
  gocyclo:
    min-complexity: 15
  
  gosec:
    severity: medium
    confidence: medium

issues:
  exclude-use-default: false
  max-issues-per-linter: 0
  max-same-issues: 0
```

### Changelog Format

For automatic changelog generation in releases, maintain a `CHANGELOG.md` file following [Keep a Changelog](https://keepachangelog.com/) format:

```markdown
# Changelog

## [v1.2.0] - 2024-01-15

### Added
- New feature implementation
- Enhanced error handling

### Fixed
- Critical bug fix in authentication
- Memory leak in processing loop

### Changed
- Updated dependency versions
- Improved performance metrics
```

## Workflow Outputs

Each workflow provides outputs that can be used in subsequent jobs:

### Build and Test Outputs
- `coverage-percentage`: Code coverage percentage
- `build-status`: Build status (success/failure)

### Release Outputs
- `release-url`: URL of the created GitHub release
- `release-tag`: Tag of the created release

### Lint Outputs
- `lint-status`: Overall lint status (success/failure)
- `issues-count`: Total number of issues found

### Security Scan Outputs
- `security-status`: Overall security scan status (success/failure)
- `vulnerabilities-count`: Total number of vulnerabilities found

### Auto Tag Outputs
- `tag-created`: Whether a new tag was created (true/false)
- `new-tag`: The new tag that was created
- `previous-tag`: The previous tag

### Matrix Test Outputs
- `matrix-status`: Overall matrix test status (success/failure)
- `failed-combinations`: Number of failed test combinations

## Requirements

### Repository Setup
- Go modules (`go.mod` and `go.sum` files)
- Proper Go project structure
- GitHub repository with Actions enabled

### Permissions
For release workflows, ensure the GitHub token has appropriate permissions:
```yaml
permissions:
  contents: write
  actions: read
  security-events: write
```

### Dependencies
All workflows automatically handle tool installation. No pre-installed dependencies required.

## Best Practices

### Version Pinning
Always pin workflows to specific versions or tags:
```yaml
uses: Gosayram/go-release-actions/.github/workflows/build-and-test.yml@v1.0.0
```

### Environment Separation
Use different configurations for different environments:
```yaml
jobs:
  test-dev:
    if: github.ref == 'refs/heads/develop'
    uses: Gosayram/go-release-actions/.github/workflows/build-and-test.yml@main
    with:
      coverage-threshold: 70

  test-prod:
    if: github.ref == 'refs/heads/main'
    uses: Gosayram/go-release-actions/.github/workflows/build-and-test.yml@main
    with:
      coverage-threshold: 85
```

### Conditional Execution
Use conditions to control workflow execution:
```yaml
jobs:
  security:
    if: github.event_name != 'pull_request' || github.event.pull_request.head.repo.full_name == github.repository
    uses: Gosayram/go-release-actions/.github/workflows/security-scan.yml@main
```

## Troubleshooting

### Common Issues

**Coverage threshold not met:**
- Adjust `coverage-threshold` parameter
- Add more comprehensive tests
- Exclude test files from coverage calculations

**Linting failures:**
- Review and fix reported issues
- Customize `.golangci.yml` configuration
- Set `fail-on-warnings: false` for non-critical issues

**Release failures:**
- Verify `GITHUB_TOKEN` permissions
- Check repository settings for releases
- Ensure proper tag format (e.g., `v1.0.0`)

**Security scan issues:**
- Update vulnerable dependencies
- Set `fail-on-vulnerability: false` for non-critical issues
- Configure appropriate severity thresholds

### Performance Optimization

**Caching:**
- Workflows automatically cache Go modules and build artifacts
- Use `cache-key-suffix` for multiple concurrent workflows

**Parallel Execution:**
- Security scans run in parallel with builds
- Lint jobs execute independently

**Resource Management:**
- Configure appropriate timeouts
- Use matrix builds for multiple Go versions

## Contributing

When contributing to this repository:

1. Follow Go coding standards and conventions
2. Maintain backward compatibility
3. Update documentation for any changes
4. Test workflows thoroughly before merging
5. Use semantic versioning for releases

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Support

For issues and questions:
- Open an issue in this repository
- Review existing workflows for examples
- Check GitHub Actions documentation for advanced configurations 