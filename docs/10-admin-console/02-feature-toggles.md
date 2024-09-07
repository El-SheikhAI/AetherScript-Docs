# Feature Toggles Management

## Introduction
AetherScript's **Feature Toggles** enable granular control over experimental, beta, or environment-specific capabilities without requiring code deployments. Toggles are managed exclusively through the Admin Console or CLI for administrator-approved configuration.

---

## Toggle Categories

### 1. Beta Features (`beta.*`)
Enable upcoming features restricted to internal teams or preview environments.  
*Example:* `beta.enable_terraform` activates experimental IaC integration.

### 2. System Integrations (`integrations.*`)
Manage third-party service connectivity at runtime.  
*Example:* `integrations.newrelic.disable` halts metrics transmission without service restart.

### 3. Experimental Optimizations (`opt.*`)
Toggle performance-critical algorithms or caching layers.  
*Example:* `opt.use_zstd_compression` swaps compression libraries during pipeline executions.

---

## Managing Toggles via CLI

### View Active Toggles
```bash
aether-script admin features get
# Output: JSON formatted list with metadata
```

### Modify Toggles
```bash
aether-script admin features set beta.enable_terraform=true
aether-script admin features delete opt.use_zstd_compression
```

**Note**: Changes apply instantly to all connected nodes but require cluster version v2.4+.

---

## Admin Console GUI Guide

### Access Controls
1. Navigate to **Admin > Feature Registry**
2. Use Quick Filters:  
   - `core`: Engine-level capabilities  
   - `plugin`: Plugin-specific overrides  
   - `deprecated`: Scheduled for removal

### Edit Workflow
1. Locate toggle in table view  
2. Click value cell to toggle state  
3. Confirm via modal dialog with optional audit comment  
4. Verify propagation status in **Activity Log**

---

## Security Considerations
- **Access Control**: `features:modify` permission required (RBAC Group: **SystemStewards**)  
- **Audit Logging**: All changes trigger events tagged with `USER_ID` and `TIMESTAMP`  
- **Risk Mitigation**:  
  - Toggling `core` features requires **dual approval**  
  - Deprecated flags are automatically deactivated after 90 days  

> ⚠️ **WARNING**  
> Avoid prolonged activation of experimental features in production. Runtime instability, performance degradation, or security vulnerabilities may occur.

---

## Use Case Example
```markdown
**Scenario**: Roll out JSON schema validation to 10% staging environments  
**Steps**:
1. `aether-script admin features set beta.enable_json_schema_validation=10`
2. Monitor validation errors in **Console Dashboard > Schema Stats**
3. Scale rollout to 100% after 24-hour observation  
4. Delete toggle when merged into stable (v2.8+)
```

---

## Summary
Feature toggles provide controlled exposure of capabilities while maintaining system stability. Follow **protocol version compatibility** guidelines before activating flags and adhere to corporate security policies for toggle retirement timelines.