# Configuration Manager

## Overview
The AetherScript Configuration Manager provides centralized control over system-wide settings, environment variables, and security credentials. This subsystem handles all persistent configuration through a hierarchical structure that supports both global defaults and environment-specific overrides.

![Configuration Manager Architecture](assets/config-manager-flow.png)  
*Figure 1: Configuration resolution workflow*

## Configuration Scopes

### 1. Global Configuration
```bash
aether config set --global <key> <value>
```
- Location: `~/.aether/config.yaml`
- Applies to all projects and environments
- Typical uses: Default plugins, CLI preferences, developer profiles

### 2. Project Configuration
```bash
aether config set --project <key> <value>
```
- Location: `./.aether/project.yaml`
- Scoped to current working directory
- Typical uses: Pipeline dependencies, environment mappings, team policies

### 3. Environment Configuration
```bash
aether config set --env=production <key> <value>
```
- Location: `./.aether/env/<environment>.yaml`
- Environment-specific overrides
- Typical uses: API endpoints, credentials, execution parameters

| Scope         | Precedence | Persistence Location | Edit Method       |
|---------------|------------|----------------------|-------------------|
| Environment   | Highest    | Project directory    | CLI or manual     |
| Project       | Medium     | Project directory    | CLI or manual     |
| Global        | Lowest     | User home            | CLI only          |

## Common Operations

### List Current Configuration
```bash
aether config list [--scope=global|project|env]
```
Displays merged configuration with scope indicators:
```yaml
plugins:
  - aws-lambda@1.2.0 (global)
  - slack-notify@0.9.3 (project)
api_endpoint: "https://prod.example.com" (env:production)
```

### Set Configuration Values
```bash
aether config set <key.path> <value> [--scope] [--env]
```
Examples:
```bash
# Set global default timeout
aether config set --global execution.timeout 300

# Configure project-specific Slack channel
aether config set --project notifications.slack.channel "#alerts"

# Override API endpoint for staging
aether config set --env=staging api.endpoint "https://stage.example.com"
```

### Unset Configuration
```bash
aether config unset <key.path> [--scope] [--env]
```
Removes specified key from chosen scope while preserving hierarchy:
```bash
aether config unset --env=test credentials.aws
```

### Configuration Verification
```bash
aether config validate
```
Performs:
1. Syntax checking
2. Value type validation
3. Cross-reference resolution
4. Secret existence verification

## Security Practices

### Credential Management
```bash
aether config secure-set <key.path> <value>
```
- Stores secrets in OS secure storage (Keychain/Windows Credential Manager/Linux Secret Service)
- References secrets in configuration via:
  ```yaml
  database:
    password: !secret db_prod_password
  ```

### Encryption Options
| Method          | Command                          | Use Case                     |
|-----------------|----------------------------------|------------------------------|
| CLI Encryption  | `aether config encrypt <file>`   | Config files with secrets    |
| Runtime Masking | Automatic                        | Console output redaction     |
| Audit Trail     | `aether config audit`            | Secret access history        |

## Advanced Configuration

### Environment Variable Overrides
Temporarily override any configuration value:
```bash
export AETHER_CONFIG_execution_timeout=120
aether run pipeline
```

### Configuration Precedence
1. Command-line arguments
2. Environment variables (`AETHER_CONFIG_*`)
3. Environment-specific configs
4. Project configuration
5. Global configuration
6. Built-in defaults