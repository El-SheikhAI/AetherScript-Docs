# Plugin Integration in Continuous Integration Pipelines

## Overview  
AetherScript integrates seamlessly with Continuous Integration (CI) systems to enable automated testing, validation, and deployment of plugin-powered workflows. This guide details best practices for configuring plugins in CI environments and optimizing pipeline execution.

---

## Prerequisites  
1. AetherScript CLI installed in the CI environment  
2. Access credentials configured for your CI platform (e.g., GitHub Actions, GitLab CI)  
3. Existing knowledge of [plugin development](03-plugin-development/01-core-concepts.md)  

---

## Configuring Plugins in CI  

### 1. Plugin Declaration  
Create `.aether/plugins.yaml` in your repository:  

```yaml
# Standard plugin configuration
timezone_utils:
  version: 1.2.0
  source: registry.aetherscript.io/official

# Custom private plugin (CI-aware configuration)
deployment_tools:
  version: 0.9.3
  source: ${{ secrets.PLUGIN_REGISTRY_URL }}
  auth_token: ${{ secrets.REGISTRY_API_KEY }}
```

### 2. CI Pipeline Configuration (GitHub Actions Example)  
```yaml
name: AetherScript CI Pipeline

jobs:
  aether-execution:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
        
      - name: Install AetherScript CLI
        run: |
          curl -fsSL https://get.aetherscript.io | bash
          aether --version
          
      - name: Execute Main Pipeline
        run: aether run main.aether --ci-mode --visualize=ci-report.html
        
      - name: Upload Visualization
        uses: actions/upload-artifact@v3
        with:
          name: workflow-visualization
          path: ci-report.html
```

---

## Best Practices  

### 1. Caching Strategies  
```yaml
# GitHub Actions cache example
- name: Cache Plugins
  uses: actions/cache@v3
  with:
    path: ~/.aether/plugins
    key: ${{ runner.os }}-aether-plugins-${{ hashFiles('.aether/plugins.yaml') }}
```

### 2. Security Recommendations  
- Always store plugin registry credentials as secrets  
- Use official plugins where possible (validated checksums)  
- Restrict plugin execution scope:  
  ```shell
  aether run --sandbox=strict --allowed-actions=core,network
  ```

### 3. Performance Optimization  
- Pre-cache plugins during image builds  
- Use `--no-realtime-visualization` in production CI runs  
- Enable parallel execution where supported:  
  ```shell
  aether run --parallel-worker=4
  ```

---

## CI Environment Setup  

| Platform          | Setup Command                               |  
|-------------------|---------------------------------------------|  
| GitHub Actions    | `curl -fsSL https://get.aetherscript.io | bash` |  
| GitLab CI         | `apk add --no-cache curl bash && install-aetherscript` |  
| CircleCI          | `- run: curl -fsSL https://get.aetherscript.io > install.sh` |  
| Jenkins           | `sh 'bash -c "$(curl -fsSL https://get.aetherscript.io)"'` |  

---

## Workflow Anatomy  
Example CI-optimized workflow (`main.aether`):  
```javascript
pipeline("CI Validation") {
  requires: ["git_tools@2.1", "security_scanner@1.7"]
  
  stage("Static Analysis") {
    parallel {
      task("Lint Sources") {
        uses: "linter/eslint_runner"
        params: { config: "ci.eslintrc" }
      }
      task("Security Scan") {
        uses: "security_scanner/owasp_check"
      }
    }
  }
  
  stage("Build Artifacts", if: env.CI_BRANCH == "main") {
    task("Package Release") {
      uses: "build_tools/webpack_optimizer"
      env: { NODE_ENV: "production" }
    }
  }
}
```