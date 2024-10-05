# Prompt Engine Architecture 

AetherScript's Prompt Engine orchestrates intelligent interactions between plugins and user-defined workflows through structured input processing, dynamic templating, and adaptive validation. This guide details its core mechanisms and implementation patterns.

---

## Core Components

### 1. Input Resolution  
Prompts accept explicit inputs while auto-resolving required dependencies:  
```yaml
# aether-pipeline.yaml
prompts:
  deploy_confirmation:
    message: "Deploy {{service_name}}@v{{version}} to {{env}}?"
    inputs:
      - name: env
        options: ["prod", "staging"]
        default: "staging"
      - name: version
        pluginRef: git/current-tag
        fallback: "latest"
```

### 2. Contextual Templating  
Inject runtime variables using Handlebars-style syntax:  
```hbs
Confirm deletion of {%raw%}{{resource_type}} '{{resource_id}}'?{%endraw%}  
{%raw%}[Context: {{plugin.metadata.last_modified}}]{%endraw%}
```
*_Dynamic components are populated from:_  
- Plugin outputs  
- Workflow variables  
- User inputs  

### 3. Automatic Validation
Type and schema verification occurs pre-execution:
```js
// Validation schema example
{
  "target_env": {
    "type": "string",
    "enum": ["prod", "staging"],
    "error": "Invalid deployment target"
  },
  "force_flag": {
    "type": "boolean",
    "default": false
  }
}
```
_Validation failures immediately halt execution and display actionable errors._

---

## Best Practices

1. **Idempotent Prompts**  
   Ensure repeated prompts yield consistent results via deterministic input resolution:
   ```yaml
   inputs:
     - name: commit_hash
       pluginRef: git/head-commit
       cache: 300s # Stale-after window
   ```

2. **Modular Prompt Chains**  
   Compose prompts into reusable sequences:
   ```yaml
   workflows:
     release_flow:
       steps:
         - prompt: version_confirmation
         - prompt: changelog_preview
           condition: ${inputs.release_type == 'minor'}
         - prompt: deployment_gate
   ```

3. **Plugin-Aware Error Handling**  
   Gracefully handle missing dependencies:
   ```yaml
   fallbackStrategies:
     - condition: ${!plugin.cloud/credentials}
       action: invoke credentials_prompt
     - condition: ${default}
       action: abort
   ```

---

## Advanced Examples

### CI/CD Approval Gate  
```yaml
prompts:
  production_deploy:
    template: |
      Approve deployment of {%raw%}{{app_name}} (Build #{{build_id}})?{%endraw%}
      {%raw%}[Changes: {{plugin.git/diff --format=summary}}]{%endraw%}
    inputs:
      - name: build_id
        pluginRef: ci/latest-successful
      - name: app_name
        value: ${vars.project_name}
    constraints:
      require: ["maintainer", "devops"]
      timeout: 600s
```

### Data Transformation Hook  
```yaml
transform_prompt:
  message: "Normalize column format for {{filename}}:"
  inputs:
    - name: format_type
      options: ${plugin.csv/detect-delimiters}
      default: "pipe"
    - name: overwrite
      type: boolean
  actions:
    - run: plugin.csv/normalize
      params:
        delimiter: ${inputs.format_type}
    - if: ${inputs.overwrite}
      run: plugin.fs/backup-original
```

---

**Next: [Nested Prompts & State Management →](./03-advanced-chaining.md)**  
**Previous: [Plugin Discovery Mechanisms ←](../04-plugin-intelligence/01-discovery-layer.md)**