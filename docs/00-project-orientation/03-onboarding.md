# Project Onboarding Guide

Welcome to AetherScript, a toolkit designed for rapid implementation of automation workflows. Follow this guide to configure your environment and execute your first pipeline.

## Prerequisites
- **Node.js v18+** (with npm v9+)
- **Python 3.10+** (for plugin interoperability)
- **Terminal** with bash/zsh equivalent
- **5GB Disk Space** (minimum for dependency caching)

## Installation
### Recommended Method (Global Install)
```bash
npm install -g @aetherscript/cli
```

### Alternative (Homebrew)
```bash
brew tap aetherscript/tools
brew install aetherscript
```

### Verify Installation
```bash
aether --version
# Expected output pattern: vX.Y.Z (Node vXX.YY.ZZ)
```

## Initial Setup
1. **Create Project Directory**
   ```bash
   mkdir aetherscript-project && cd $_
   ```

2. **Initialize Configuration**
   ```bash
   aether init
   ```
   Generates:
   - `aether.config.yaml` (Core configuration)
   - `.aether/plugins/` (Plugin storage)
   - `.aether/cache/` (Pipeline artifacts)

## First Workflow Execution
1. **Create Hello World Pipeline**
   ```yaml
   # hello.yaml
   name: GreetingWorkflow
   steps:
     - action: core/echo
       params:
         message: "Hello, AetherScript!"
   ```

2. **Execute Pipeline**
   ```bash
   aether run hello.yaml
   ```

3. **Expected Output**
   ```
   [VISUAL] Launching workflow inspector at http://localhost:8080
   [EXEC] Starting GreetingWorkflow (ID: 3FA7)
   [OUT] Hello, AetherScript!
   [TIME] Completed in 142ms
   ```

## Real-Time Monitoring
All executions automatically launch our visualization engine:

1. Open `http://localhost:8080` during execution
2. View:
   - Step dependency graph
   - Live resource utilization
   - Error propagation paths
   - Historical performance metrics

## Plugin Ecosystem
Example: Integrate HTTP requests plugin

1. **Autoload Plugin**
   ```yaml
   # aether.config.yaml
   plugins:
     - "@aether/http-request@^2.1.0"
   ```

2. **Use in Pipeline**
   ```yaml
   steps:
     - action: http-request/get
       params:
         url: "https://api.example.com/data"
         timeout: 5000
   ```

## Next Steps
1. Explore CLI commands:
   ```bash
   aether --help
   ```
2. Study our pipeline specification:
   ```bash
   aether docs spec
   ```
3. Visit `./docs/04-plugin-development` to create custom modules

> **Support Tip**: Run `aether diagnostics` if you encounter environment issues. This generates troubleshooting reports in `.aether/diagnostics/`.