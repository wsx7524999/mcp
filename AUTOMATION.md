# Automation & Developer Workflows

This document provides comprehensive information about automation, build systems, and developer workflows in the Microsoft MCP repository.

## Table of Contents

- [Overview](#overview)
- [NPM Scripts](#npm-scripts)
- [Build System](#build-system)
- [Testing Automation](#testing-automation)
- [CI/CD Pipelines](#cicd-pipelines)
- [Code Quality](#code-quality)
- [AI-Assisted Development](#ai-assisted-development)
- [Troubleshooting](#troubleshooting)

## Overview

The Microsoft MCP repository uses a sophisticated automation system that combines:

- **NPM Scripts**: High-level commands for common development tasks
- **PowerShell Scripts**: Cross-platform automation in `eng/scripts/`
- **Azure Pipelines**: Enterprise CI/CD workflows
- **GitHub Actions**: Event-driven automation
- **AI Integration**: GitHub Copilot optimizations

## NPM Scripts

The root `package.json` provides convenient npm scripts that wrap PowerShell automation:

### Build Commands

```bash
# Full build with verification (recommended)
npm run build

# Quick build (faster, skips verification)
npm run build:quick

# Build specific code components
npm run build:code

# Build Docker images
npm run build:docker

# Clean all build artifacts
npm run clean
```

### Testing Commands

```bash
# Run all tests
npm run test

# Run only unit tests (no Azure resources needed)
npm run test:unit

# Run live tests (requires Azure credentials)
npm run test:live
```

### Code Quality Commands

```bash
# Run spelling checker
npm run lint
npm run lint:spelling

# Format code using dotnet format
npm run format

# Analyze AOT (Ahead-of-Time) compatibility
npm run analyze:aot

# General code analysis
npm run analyze:code
```

### Version Management

```bash
# Get current version information
npm run version:get

# Update version numbers
npm run version:update
```

### Development Tools

```bash
# Install Git hooks for pre-commit checks
npm run hooks:install

# Deploy test resources to Azure
npm run deploy:test-resources

# Compile changelog from individual entries
npm run changelog:compile
```

## Build System

### Architecture

The build system is organized in layers:

```
Root package.json (npm scripts)
    ↓
eng/scripts/*.ps1 (PowerShell automation)
    ↓
dotnet build / docker build (native tooling)
```

### Key Build Scripts

#### Build-Local.ps1

Primary script for local development builds.

```powershell
# Basic build
./eng/scripts/Build-Local.ps1

# Build with path resolution and npm verification
./eng/scripts/Build-Local.ps1 -UsePaths -VerifyNpx

# Build for all platforms (Linux, Windows, macOS)
./eng/scripts/Build-Local.ps1 -AllPlatforms

# Release build (optimized)
./eng/scripts/Build-Local.ps1 -ReleaseBuild

# Build specific server
./eng/scripts/Build-Local.ps1 -ServerName Azure.Mcp.Server
```

**Parameters:**
- `-ServerName`: Build a specific server (e.g., "Azure.Mcp.Server")
- `-NoTrimmed`: Disable IL trimming
- `-NoSelfContained`: Create framework-dependent builds
- `-AllPlatforms`: Build for all OS/architecture combinations
- `-VerifyNpx`: Test npm packages with npx
- `-ReleaseBuild`: Build with release configuration
- `-IncludeNative`: Include native AOT compilation

#### Build-Code.ps1

Lower-level build script for compilation.

```powershell
# Build specific server
./eng/scripts/Build-Code.ps1 -ServerName Azure.Mcp.Server

# Build with specific configurations
./eng/scripts/Build-Code.ps1 -SelfContained -Trimmed -ReleaseBuild

# Build for specific platforms
./eng/scripts/Build-Code.ps1 -OperatingSystems @('linux', 'windows') -Architectures @('x64')

# Native AOT build
./eng/scripts/Build-Code.ps1 -Native
```

#### Build-Docker.ps1

Build Docker container images.

```powershell
# Build Docker images
./eng/scripts/Build-Docker.ps1

# Build for specific server
./eng/scripts/Build-Docker.ps1 -ServerName Azure.Mcp.Server

# Build with custom tag
./eng/scripts/Build-Docker.ps1 -Tag custom-tag
```

### Build Outputs

Build artifacts are placed in:
- `.work/build/` - Compiled binaries
- `.work/packages_npm/` - NPM packages
- `.work/packages_nuget/` - NuGet packages
- `.work/vsix/` - VS Code extensions

## Testing Automation

### Test Categories

1. **Unit Tests**: Fast, isolated tests with no external dependencies
2. **Live Tests**: Integration tests requiring Azure resources
3. **Smoke Tests**: Basic functionality validation of packaged builds
4. **End-to-End Tests**: Complete workflow validation

### Running Tests

```bash
# All tests via npm
npm run test

# Unit tests only
npm run test:unit

# Live tests only (requires authentication)
npm run test:live

# Using dotnet directly
dotnet test --filter Category=UnitTest
dotnet test --filter Category=LiveTest

# Run tests for specific project
dotnet test ./core/Azure.Mcp.Core/tests/Azure.Mcp.Tests/
```

### Test Resource Deployment

Live tests require Azure resources:

```bash
# Deploy test resources
npm run deploy:test-resources

# Or use PowerShell directly
./eng/scripts/Deploy-TestResources.ps1 -ResourceGroup my-test-rg
```

Test resources are defined in `test-resources.bicep` files in each tool's test directory.

### Recorded Tests

The repository supports recorded HTTP interactions for deterministic testing. See [docs/recorded-tests.md](docs/recorded-tests.md) for details.

## CI/CD Pipelines

### GitHub Actions

Located in `.github/workflows/`:

#### Event Processor (`event-processor.yml`)

Handles GitHub events:
- Issue creation, labeling, editing
- Pull request events
- Comment processing
- Automated issue labeling using Azure AI

#### Scheduled Event Processor (`scheduled-event-processor.yml`)

Runs periodic tasks:
- Stale issue management
- Automated maintenance
- Scheduled cleanups

#### Post API View (`post-apiview.yml`)

API review automation:
- Generates API surface documentation
- Posts to Azure API review system

#### Auto Milestone (`auto-milestone-bugbash.yml`)

Bug bash automation:
- Assigns issues to milestones
- Tracks bug bash progress

### Azure Pipelines

Located in `eng/pipelines/`:

#### Pull Request Validation (`pullrequest.yml`)

Comprehensive PR checks:
1. **Code Compilation**: Multi-platform builds (Linux, Windows, macOS)
2. **Unit Tests**: Fast tests without Azure dependencies
3. **AOT Analysis**: Ahead-of-time compilation compatibility
4. **Spelling**: Automated spell checking
5. **Integration Tests**: End-to-end workflow validation
6. **Live Tests**: Tests with real Azure resources (internal only)
7. **Security**: CodeQL scanning for vulnerabilities
8. **Docker Builds**: Container image validation

#### Release Pipeline

Automated releases:
- Version tagging
- NPM package publishing
- Docker image publishing
- GitHub release creation
- VS Code extension publishing

### Pipeline Configuration

Pipelines use templates from `eng/pipelines/templates/`:
- `jobs/integration.yml` - Integration test jobs
- `jobs/live-test.yml` - Live test execution
- `jobs/docker/` - Docker build jobs
- `steps/` - Reusable pipeline steps

## Code Quality

### Spell Checking

Using cspell for automated spell checking:

```bash
# Via npm
npm run lint

# Via PowerShell
./eng/common/spelling/Invoke-Cspell.ps1
```

Configuration: `.vscode/cspell.json`

Custom words: `./eng/common/spelling/spell-check-dictionary.txt`

### AOT Compatibility

Ensures code works with Native AOT compilation:

```bash
# Run AOT analysis
npm run analyze:aot

# Or via PowerShell
./eng/scripts/Analyze-AOT-Compact.ps1
```

### Code Formatting

Using dotnet format:

```bash
# Format all code
npm run format

# Or via dotnet
dotnet format
```

### Git Hooks

Pre-commit hooks ensure code quality:

```bash
# Install hooks
npm run hooks:install

# Or via PowerShell
./eng/scripts/Install-GitHooks.ps1
```

Hooks check:
- ✅ Spelling
- ✅ AOT compatibility
- ✅ Code formatting
- ✅ Build success

## AI-Assisted Development

### GitHub Copilot Integration

The repository is optimized for AI-assisted development:

1. **Copilot Instructions**: Custom instructions in `.github/copilot-instructions.md`
   - C# coding standards (primary constructors, AOT safety)
   - Build and test commands
   - PR guidelines

2. **Agent Configurations**: See `AGENTS.md` for:
   - Custom agents for specific tasks
   - Tool selection patterns
   - Context optimization

3. **MCP Protocol**: The repository itself demonstrates MCP principles:
   - Standardized tool interfaces
   - Context-aware operations
   - LLM-friendly APIs

### Using Copilot Effectively

**For code changes:**
- Ask Copilot to follow `.github/copilot-instructions.md`
- Request AOT-safe code generation
- Validate with `npm run analyze:aot`

**For build issues:**
- Reference specific build scripts
- Ask for parameter explanations
- Debug with verbose PowerShell output

**For testing:**
- Generate test fixtures
- Create mock data
- Write recorded test scenarios

## Troubleshooting

### Build Failures

**Issue: Build fails with AOT warnings**

```bash
# Run detailed AOT analysis
./eng/scripts/Analyze-AOT-Compact.ps1 -Verbose

# Check for non-AOT-compatible code
# Common issues: reflection, dynamic code, unsupported APIs
```

**Issue: Docker build fails**

```bash
# Build with verbose output
./eng/scripts/Build-Docker.ps1 -Verbose

# Check Docker daemon
docker version

# Verify artifacts exist
ls .work/build/
```

**Issue: NPM verification fails**

```bash
# Clear npm cache
npx -y clear-npx-cache

# Rebuild with verification
npm run build
```

### Test Failures

**Issue: Live tests fail with authentication error**

```bash
# Check Azure CLI login
az account show

# Login if needed
az login

# Set subscription
az account set --subscription <subscription-id>

# Deploy test resources
npm run deploy:test-resources
```

**Issue: Recorded tests out of sync**

```bash
# Update recorded sessions
# See docs/recorded-tests.md for details

# Re-record tests
dotnet test --filter Category=Recorded -- RecordMode=Record
```

### CI/CD Issues

**Issue: Pipeline fails in PR**

1. Check pipeline logs in Azure DevOps
2. Run same checks locally:
   ```bash
   npm run build
   npm run test:unit
   npm run lint
   npm run analyze:aot
   ```
3. Fix issues and push again

**Issue: GitHub Action fails**

1. Check workflow logs in GitHub Actions tab
2. Test event handling locally if possible
3. Verify permissions and secrets

### General Debugging

**Enable verbose output:**

```powershell
# PowerShell scripts
./eng/scripts/Build-Local.ps1 -Verbose

# Or set preference
$VerbosePreference = 'Continue'
./eng/scripts/Build-Local.ps1
```

**Check environment:**

```bash
# Verify tools
node --version    # Should be >=20
npm --version     # Should be >=10
pwsh --version    # Should be >=7
dotnet --version  # Should be 10.0.100

# Verify Git status
git status
git --no-pager log --oneline -5
```

**Clean and rebuild:**

```bash
# Nuclear option - clean everything
npm run clean
git clean -fdx .work/
rm -rf ~/.nuget/packages/Azure.Mcp.*
rm -rf ~/.nuget/packages/Microsoft.Mcp.*

# Rebuild from scratch
npm run build
```

## Additional Resources

- [CONTRIBUTING.md](CONTRIBUTING.md) - Contribution guidelines
- [README.md](README.md) - Project overview
- [docs/](docs/) - Detailed documentation
  - [Authentication.md](docs/Authentication.md) - Authentication guides
  - [aot-compatibility.md](docs/aot-compatibility.md) - AOT details
  - [recorded-tests.md](docs/recorded-tests.md) - Testing guide
  - [changelog-entries.md](docs/changelog-entries.md) - Changelog process

## Support

- **Issues**: https://github.com/microsoft/mcp/issues
- **Discussions**: https://github.com/microsoft/mcp/discussions
- **Documentation**: https://learn.microsoft.com/azure/developer/azure-mcp-server/

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
