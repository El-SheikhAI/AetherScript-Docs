# Pipeline Schema Definition

The AetherScript pipeline engine operates on a declarative schema that defines the structure, execution rules, and environment configuration for automation workflows. This document specifies the schema syntax, properties, and validation constraints.

## Schema Structure
```yaml
version: 2.1
metadata:
  name: "Production Deployment"
  description: "Deploy containerized application to Kubernetes"
  env: [ "prod-us-east", "prod-eu-west" ]
  
pipeline:
  steps:
    - id: build-image
      type: docker/build
      args:
        image: "myapp:${GIT_COMMIT}"
        dockerfile: "./Dockerfile.prod"
        
    - id: deploy-frontend
      type: kubernetes/apply
      dependsOn: [ "build-image" ]
      parallel: true
      args:
        manifest: "./k8s/frontend.yaml"
        namespace: "production"

hooks:
  onSuccess:
    - type: slack/notify
      args:
        channel: "#deployments"
        message: "Deployment ${METADATA.NAME} succeeded"
        
  onFailure:
    - type: email/alert
      args:
        recipient: "devops@company.com"

connectors:
  kubernetes:
    config: "${KUBECONFIG_PATH}"
  
variables:
  GIT_COMMIT: ${CI_COMMIT_SHA}
  TIMEOUT_SECONDS: 300
```

## Schema Components

### 1. Version Declaration (Required)
```yaml
version: 2.1
```
- Enforces schema compatibility
- Current stable version: `2.1`
- Must be the first property in the document

### 2. Metadata Block (Required)
```yaml
metadata:
  name: "Unique Workflow Name"    # Required (alphanumeric + hyphens)
  description: "Workflow purpose" # Optional
  env:                            # Optional
    - "production"
    - "staging"
  labels:                         # Optional
    department: "devops"
    criticality: "high"
```

### 3. Pipeline Core Definition (Required)
```yaml
pipeline:
  steps:
    - id: step1              # Unique identifier
      type: plugin/action    # Plugin reference format
      dependsOn: [step0]     # Execution dependencies
      parallel: false        # Default: false
      timeout: 300           # Seconds (optional)
      retries: 2             # Default: 0
      args:                  # Plugin-specific arguments
        param1: "value"
        param2: ["array", "values"]
```

### 4. Execution Hooks (Optional)
```yaml
hooks:
  onSuccess:
    - type: notification/slack
      args:
        channel: "#alerts"
  
  onFailure:
    - type: alert/pagerduty
      args:
        service: "production-deployments"

  onComplete:    # Always executes
    - type: util/cleanup
```

### 5. Connector Configuration (Optional)
```yaml
connectors:
  kubernetes:
    config: "/path/to/kubeconfig"
    context: "prod-cluster"
  
  aws:
    profile: "production-account"
    region: "us-west-2"
```

### 6. Variable Declaration (Optional)
```yaml
variables:
  ENVIRONMENT: "production"   # Static value
  BUILD_NUMBER: ${CI_BUILD}   # External env reference
  CUSTOM_VAR: ${steps.step1.output}  # Step output reference
```

## Schema Validation Rules
1. **Strict Typing**:
   - `version` must be semantic version string
   - `metadata.name` supports `[a-zA-Z0-9_-]` only
   - `steps[*].id` must be unique within pipeline

2. **Execution Constraints**:
   - Circular dependencies prohibited
   - Parallel step groups cannot have shared dependencies
   - Maximum nesting depth: 5 levels

3. **Security Restrictions**:
   - No raw credential storage
   - Environment variables must use `${ENV_VAR}` syntax
   - Local file references require explicit `file://` prefix

## Interpolation Syntax
```yaml
# Environment variables
${ENV_VAR_NAME}

# Step outputs
${steps.step_id.output.property}

# System metadata
${TIMESTAMP_UTC}
${EXECUTION_ID}

# File contents
${file./path/to/file.txt}
```

## Example Schema Breakdown
```yaml
# Simple ETL Pipeline
version: 2.1
metadata:
  name: "nightly-data-sync"
  env: ["bigquery", "snowflake"]
  
pipeline:
  steps:
    - id: extract
      type: db/postgres-query
      args:
        query: "SELECT * FROM transactions"
        output: "/data/raw.csv"
        
    - id: transform
      type: python/script
      dependsOn: ["extract"]
      args:
        script: "./scripts/clean_data.py"
        input: "/data/raw.csv"
        output: "/data/processed.parquet"
      
    - id: load
      type: gcp/bigquery-load
      dependsOn: ["transform"]
      args:
        dataset: "analytics"
        table: "transactions"
        source: "/data/processed.parquet"

hooks:
  onComplete:
    - type: util/clean-temp
      args:
        patterns: ["/data/*.csv", "/data/*.parquet"]
        
variables:
  PROCESSING_DATE: ${date.YYYY-MM-DD}
```

---

Next: [03 - Execution Model](./03-execution-model.md)  
Previous: [01 - Core Concepts](./01-core-concepts.md)