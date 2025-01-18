# Common Gotchas in AetherScript

This document highlights subtle behaviors, non-intuitive patterns, and potential pitfalls in AetherScript. Understanding these caveats will help you write robust automation pipelines and avoid runtime surprises.

---

## 1. Plugin Name Ambiguity

**Issue**:  
Zero-config plugin resolution prioritizes lexical similarity when multiple plugins match a partial name.

**Example**:  
Installing both `@aether/cloud-storage` and `aether-storage-aws` may cause `use storage;` to resolve unpredictably.

**Resolution**:  
Always specify fully qualified plugin names in critical pipelines:
```aetherscript
use @aether/cloud-storage as cloud;  // Explicit namespace
```

---

## 2. Dynamic Module Import Precedence

**Issue**:  
Local files override core modules when using identical names.

**Example**:  
Creating `utils.js` in your working directory shadows the core `utils` module.

**Resolution**:  
Use path prefixes for local modules:
```aetherscript
import ./utils.js as projectUtils;  // Local import
import utils from core;             // Explicit core reference
```

---

## 3. Concurrent Task Idempotency

**Issue**:  
Parallel tasks executing non-idempotent operations may create race conditions.

**Example**:
```aetherscript
taskA || taskB >> finalize  // Both modify shared-resource.json
```

**Resolution**:  
1. Design tasks to be idempotent
2. Use file locks:
```aetherscript
use file-lock;

taskA:
  lock("shared-resource.json") {
    // Critical section
  }
```

---

## 4. Error Propagation in Pipelines

**Issue**:  
AetherScript halts pipelines on first unhandled error by default.

**Example**:  
```aetherscript
taskA >> taskB >> taskC  // Fails entirely if taskB errors
```

**Resolution**:  
Explicitly handle partial failures:
```aetherscript
taskB? >> taskC  // ? suppresses taskB failure
```

---

## 5. Environment Variable Evaluation

**Issue**:  
Environment variables are evaluated at script compilation, not runtime.

**Example**:  
```aetherscript
echo $TIMESTAMP  # Uses value when script started
```

**Resolution**:  
Access real-time values via runtime API:
```aetherscript
use runtime;

echo runtime.env('TIMESTAMP');  // Current value
```

---

## 6. Workflow Visualization Limitations

**Issue**:  
The real-time visualizer truncates deep (>7 levels) nested workflows.

**Example**:  
Complex recursive pipelines may show incomplete dependency trees.

**Resolution**:  
1. Flatten deep hierarchies using sub-pipelines
2. Use the `--depth` flag to adjust visibility:
```bash
aether visualize --depth=10
```

---

## 7. State Isolation Between Tasks

**Issue**:  
Tasks run in isolated contexts unless explicitly sharing state.

**Example**:  
Modifying `config.timeout` in TaskA doesn't affect TaskB's copy.

**Resolution**:  
Use the global context store:
```aetherscript
taskA:
  global.set("timeout", 300);

taskB:
  timeout = global.get("timeout");
```

---

**Best Practice Reminder**:  
Always test pipelines with the `--dry-run` flag before production execution:
```bash
aether run pipeline.yml --dry-run --validate
```