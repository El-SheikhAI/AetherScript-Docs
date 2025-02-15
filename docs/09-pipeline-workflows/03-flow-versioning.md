# Workflow Versioning

AetherScript employs semantic versioning (SemVer) to manage changes in pipeline definitions while maintaining compatibility across environments. This system ensures predictable updates and dependency management throughout your automation lifecycle.

## Version Schema

Workflow versions follow `MAJOR.MINOR.PATCH` format specified in pipeline manifests:

```yaml
# aether-pipeline.yaml
api_version: "2.3.0"  # Explicit version declaration
workflow:
  name: data-processing
  version: "1.2.0"    # Workflow-specific version
```

### Version Components
- **MAJOR**: Breaking changes (requires migration)
- **MINOR**: Backward-compatible feature additions
- **PATCH**: Backward-compatible bug fixes

## Workflow Compatibility

### Forward Compatibility
- Workflows with same MAJOR version automatically accept MINOR/PATCH updates
- CLI caches previous workflow versions for rollback capability

```bash
aetherscript versions list my-workflow
# Output:
# - 1.2.0 (active)
# - 1.1.3
# - 1.1.2
```

### Backward Compatibility
Major version changes require explicit migration:

```bash
aetherscript migrate my-workflow --from 1.2.0 --to 2.0.0
```
The CLI performs automated schema validation during migration, generating compatibility reports in `migrations/` directory.

## Plugin Version Constraints

Define plugin compatibility ranges in your workflow manifest:

```yaml
plugins:
  - name: aws-connector
    version: "^1.3.0"  # Compatible with 1.3.0+, <2.0.0
  - name: data-validator
    version: "~2.1.4"  # Compatible with 2.1.4+, <2.2.0
```

## Best Practices

1. **Incremental Versioning**
   ```bash
   aetherscript version bump minor my-workflow
   ```

2. **Change Documentation**
   ```yaml
   changelog:
     - version: "1.2.0"
       changes:
         - "Added parallel execution nodes"
         - "Deprecated legacy CSV processor"
   ```

3. **Environment Locking**
   ```bash
   aetherscript lock generate --env production
   # Creates aether-lock.production.yaml
   ```

## Migrating Workflows

### Automated Migration Steps
1. Backup current workflow state
2. Validate plugin compatibility matrices
3. Convert deprecated node syntax
4. Preserve environment variables
5. Output migration report

### Manual Intervention Points
- Schema changes requiring data transformation
- Deprecated plugin replacements
- Security policy updates
- Resource allocation modifications

## Version Visualization

Generate version history diagrams:

```bash
aetherscript graph versions --workflow my-workflow --output version-history.svg
```

![Workflow Version History](../assets/workflow-version-history.png)

> **Important**: Always test version migrations in staging environments using `aetherscript replay --version <target_version>` before production deployment.