# Release Process

This document outlines the standardized procedure for publishing releases of the AetherScript CLI toolkit. Releases follow semantic versioning (SemVer 2.0.0) and leverage our pipeline automation capabilities.

## Versioning Strategy
- `MAJOR.MINOR.PATCH` versioning scheme
- Version tags MUST be immutable once published
- Pre-release versions use `-{identifier}` suffix (e.g., `1.2.3-rc.1`)

## Branch Management
```
main     ➞ Active development branch
release/ ➞ Ephemeral branch per release version
```

## Release Checklist

### Pre-Release Steps
1. Create release candidate branch:
   ```bash
   aether branch create release/vX.Y.Z-rc.N
   ```

2. Update project version:
   ```bash
   aether version bump [major|minor|patch] --pre-release=[rc|beta] 
   ```

3. Generate changelog:
   ```bash
   aether changelog generate --from-last-release --output CHANGELOG.md
   ```

4. Execute full validation pipeline:
   ```bash
   aether pipeline run --name release-validation
   ```

   Pipeline components:
   - Unit & integration tests
   - Linting and static analysis
   - Compatibility matrix verification
   - Artifact integrity checks

### Release Artifact Creation
1. Build platform-specific binaries:
   ```bash
   aether build matrix \
     --os linux,darwin,windows \
     --arch amd64,arm64
   ```

2. Package artifacts:
   ```bash
   aether artifacts package --sign --checksum=sha256
   ```

3. Generate documentation bundle:
   ```bash
   aether docs bundle --format=html,markdown --compress
   ```

### Publishing Steps
1. Push tagged release to registry:
   ```bash
   aether registry publish \
     --version=tag \
     --access=public \
     --artifacts=dist/*.tar.gz
   ```

2. Update stable release channel:
   ```bash
   aether channel promote vX.Y.Z --channel=stable
   ```

3. Publish plugin updates:
   ```bash
   aether plugin registry sync --version=vX.Y.Z
   ```

### Post-Release Tasks
1. Merge release branch to main:
   ```bash
   aether branch merge release/vX.Y.Z --strategy=squash
   ```

2. Update development version:
   ```bash
   aether version bump minor --pre-id=dev
   ```

3. Verify downstream dependencies:
   ```bash
   aether dependency graph validate --depth=3
   ```

## Automation Hooks
Integrate these into your pipeline configuration:

```yaml
# .aether/pipelines/release.yml
hooks:
  pre_release:
    - name: dependency-audit
      cmd: security audit --level=critical
    - name: license-compliance
      cmd: legal validate --licenses=MIT,Apache-2.0

  post_release:
    - name: notify-stakeholders
      cmd: webhook send --event=release --version=$AETHER_VERSION
    - name: purge-cdn
      cmd: cdn invalidate /versions/vX.Y.Z/*
```

## Emergency Procedures

### Hotfix Release
1. Branch from affected version:
   ```bash
   aether branch create hotfix/vX.Y.Z --from=vX.Y.Z
   ```

2. Follow standard release process with `--hotfix` flag:
   ```bash
   aether release execute --hotfix --skip-docs
   ```

### Release Rollback
1. Demote version from stable channel:
   ```bash
   aether channel demote vX.Y.Z --channel=stable
   ```

2. Mark version as deprecated:
   ```bash
   aether registry deprecate vX.Y.Z --message="Critical bug detected"
   ```

3. Publish rollback notification:
   ```bash
   aether comms alert --severity=high --affected=vX.Y.Z
   ```