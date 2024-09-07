# Environment Configuration for CI Toolchains

## Prerequisites
1. **Node.js Environment**: Ensure Node.js v16.x or later is installed on your CI workers.
2. **Docker Runtime**: Required for containerized pipeline execution (version 20.10+ recommended).
3. **AetherScript-CLI**: Pre-installed CLI (minimum v1.3.0) accessible in your PATH:
   ```bash
   npm install -g aetherscript-cli@latest
   ```

## Configuration Methodology

### Core Configuration Files
1. `.aether/env.conf`
   ```properties
   # Base environment profile
   ENVIRONMENT=ci
   LOG_LEVEL=verbose
   CACHE_TTL=3600
   ```

2. `.aether/plugins.conf`
   ```ini
   [core]
   validation-plugin=@aether/validator@2.1.0
   reporting-plugin=@aether/reporter-ci@1.0.4

   [custom]
   security-scan=@org/sast-plugin@^1.1.0
   ```

### Environment Variables
| Variable             | Required | Default     | Description                     |
|----------------------|----------|-------------|---------------------------------|
| `AETHER_ENV`         | Yes      | development | Runtime environment context    |
| `CI_BUILD_ID`        | Yes      | -           | CI system's build identifier   |
| `AETHER_CACHE_DIR`   | No       | .aether/cache | Pipeline artifact storage     |
| `AETHER_FAIL_FAST`   | No       | false       | Abort pipeline on first error  |

### Secret Management
Handle sensitive data using your CI system's secret store:
```bash
# Example: GitHub Actions
aether-cli secrets map \
  --source env \
  --target system \
  --mapping "PAT=GH_TOKEN"
```

Supported secret providers:
1. Encrypted CI variables (GitHub Actions/GitLab CI)
2. HashiCorp Vault
3. AWS Parameter Store
4. GCP Secret Manager

## Plugin Configuration
Zero-config plugins auto-register when detected in `.aether/plugins/`. For custom setup:

```javascript
// pipeline.aether.js
export const config = {
  plugins: [
    {
      name: "cloud-deploy",
      version: "3.2.0",
      config: {
        targetEnv: "staging",
        rollback: true
      }
    }
  ]
};
```

## Visualization Setup
Enable real-time workflow visualization:
1. Add visualization agent to pipeline
2. Expose port 8080 on CI worker
3. Access through CI system's web interface

```yaml
# Example CI configuration (GitLab CI)
aether_visualization:
  image: aether-viz:3.0.1
  ports:
    - 8080:8080
  script: aether-cli viz --port 8080
```

## Dynamic Configuration
Generate environment-specific configurations at runtime:
```bash
aether-cli generate-config \
  --template config/env-template.hbs \
  --output .aether/env.ci \
  --vars "buildId=$CI_BUILD_NUMBER"
```

## Best Practices
1. **Immutable Configurations**: Freeze configuration after pipeline initialization
2. **Environment Isolation**: Use separate config sets per stage (dev/staging/prod)
3. **Configuration Auditing**: Enable `audit_trail: true` in env.conf
4. **Ephemeral Secrets**: Rotate credentials per pipeline execution

## Troubleshooting
Common configuration issues:
```bash
# Verify environment setup
aether-cli doctor --environment ci

# Validate configuration syntax
aether-cli validate-config --file .aether/env.conf

# Debug plugin loading
AETHER_DEBUG=plugins aether-cli pipeline run
```