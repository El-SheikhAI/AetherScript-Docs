# State Management in AetherScript

## Overview
AetherScript employs a centralized state management system to track and coordinate workflow execution across all pipeline nodes. This system ensures deterministic behavior while providing extensibility points for plugins and real-time visualization tools.

---

## Core State Objects

### 1. Pipeline State (Immutable)
```json
{
  "id": "pipeline_01H3XYZ9",
  "version": "2.1.0",
  "start_timestamp": 1689876543.212,
  "config_hash": "e3b0c44298fc1cd"
}
```

### 2. Node State (Mutable)
```json
{
  "node_id": "transform_csv_01H3XZAB",
  "status": "executing",
  "input_payload": {"path": "/data/raw.csv"},
  "output_payload": null,
  "error": null,
  "retry_count": 0,
  "execution_time": 0.87
}
```

### 3. Execution Context (Dynamic)
```json
{
  "environment_vars": {"DEBUG": "true"},
  "secret_store": "vault://aether-prod",
  "active_plugins": [
    "DataValidator@1.4.0",
    "SlackNotifier@3.2.1"
  ],
  "artifacts": {
    "raw_data": "/tmp/aether/raw_data.bin"
  }
}
```

---

## State Manager Lifecycle
### Four-Phase Operation

1. **Initialization Phase**
   - Validates pipeline configuration
   - Resolves plugin dependencies
   - Creates initial state snapshot

2. **Execution Phase**
   - Manages node transitions:
     ```plaintext
     Pending → Executing → (Succeeded|Failed|Retrying)
     ```
   - Enforces execution order constraints
   - Handles error propagation

3. **Mutation Phase**
   - Processes state transitions via CRDT (Conflict-Free Replicated Data Type) model
   - Coordinates plugins' write operations using MVCC (Multiversion Concurrency Control)

4. **Finalization Phase**
   - Generates final execution report
   - Triggers cleanup operations
   - Writes state history to persistent storage (if enabled)

---

## State Persistence
Enable persistent state tracking with CLI flag:
```bash
aetherscript run --state-history=./executions/
```

### Storage Format
```bash
executions/
└── 2023-07-20_133452_pipeline_01H3XYZ9/
    ├── state_init.json
    ├── state_node_transform_csv_01H7XZAB_attempt1.json
    ├── state_node_transform_csv_01H7XZAB_final.json
    └── state_final.json
```

---

## Visualization Integration
State changes broadcast through:
```javascript
// WebSocket message structure
{
  "event_type": "state_update",
  "update_mask": ["nodes.transform_csv.status", "context.artifacts"],
  "timestamp": 1689876543.215,
  "payload": {/* Partial state update */}
}
```

### Real-Time Constraints
- Max latency: 200ms
- Update granularity: Field-level diffs
- Broadcast frequency: 4Hz (configurable via `--vis-update-rate`)

---

## Hooks & Events

### State Transition Events
1. `pre_state_initialize`
2. `post_node_execute`
3. `pre_state_mutation`
4. `post_state_finalize`

### Example Hook Implementation
```python
# plugins/state_logger.py
from aethersdk import StateHook

class StateLogger(StateHook):
    def post_node_execute(self, node_id, previous_state, current_state):
        if previous_state.status != current_state.status:
            self.logger.info(
                f"Node {node_id} transitioned: "
                f"{previous_state.status} → {current_state.status}"
            )
```

---

## Best Practices
1. **Immutable State Access**: Plugins should treat pipeline state as immutable
2. **State Size Management**: Keep single node state < 16KB
3. **Concurrency Control**: Use `context.artifacts` for shared resource handling
4. **Error Handling**: Preserve error contexts in mutable node state
5. **State Encryption**: Enable `--state-encryption-key` for sensitive data