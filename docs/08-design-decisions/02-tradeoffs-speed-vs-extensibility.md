# Tradeoffs: Speed vs. Extensibility

## Overview
AetherScript's architecture faces a core tension inherent in modular systems: the tradeoff between execution speed and extensibility. This document outlines our deliberate compromises and optimizations to balance these competing priorities while maintaining the toolkit’s lightweight ethos.

---

## The Core Dilemma
- **Extensibility Goals**:  
  Supporting zero-config plugins requires dynamic loading, runtime type validation, and dependency resolution—operations that inherently add latency.  
  
- **Speed Goals**:  
  Fast pipeline execution is critical for iterating on automation workflows and maintaining responsive real-time visualization. Heavy abstraction layers conflict with this objective.

Our design resolves this tension through *purposeful constraint scoping* and *runtime optimization*, prioritizing workflows aligned with AetherScript's core use cases (data transformation, middleware orchestration, and event-driven automation).

---

## Key Constraints & Optimizations

### 1. **Lazy-Loaded Plugin Activation**
Plugins initialize only when invoked in a pipeline, reducing startup overhead. Downside: First-run plugin execution incurs a one-time resolution cost (~20-50ms based on benchmark averages).

### 2. **Hybrid Runtime Model**
| Component          | Speed Priority                              | Extensibility Mechanism               |
|---------------------|--------------------------------------------|----------------------------------------|
| Core Engine         | Compiled (Go/Rust)                          | Statically linked binaries             |
| Plugins             | Interpreted (JS/Python) or WASM             | Dynamic loading via declarative manifests |

### 3. **Zero-Config ≠ Zero-Cost**  
Plugin discovery via filesystem scanning (`*.aetherplugin` glob) occurs at project load. We mitigate cost through:
- **Cached manifests** after first scan.
- **Parallel validation** during idle event-loop cycles.

### 4. Visualization Tradeoffs
Real-time workflow visualization requires continuous instrumentation. We:
- **Throttle DOM updates** to 60 FPS.
- **Offload rendering** to a Web Worker in the CLI’s TUI mode.
- Allow disabling visualization entirely via `--no-visualize` flag for performance-critical runs.

---

## Guidance for Pipeline Authors

### When to Prioritize Speed:
- **Tight-loop automation** (e.g., log processing):  
  Use native core modules over plugins when possible.  
- **CI/CD contexts**:  
  Run pipelines with `--optimize=no-plugins` to skip unneeded extensibility checks.

### When to Prioritize Extensibility:
- **Multi-stage workflows**:  
  Combine lightweight plugins (e.g., `http-request`, `json-transform`) without sacrificing maintainability.
- **Ad-hoc experimentation**:  
  Leverage zero-config plugins for rapid prototyping before optimizing with native code.

---

## Future Considerations
- **Ahead-of-Time (AOT) Plugin Compilation**:  
  Experimental support for precompiling frequent-use plugins to WASM.
- **Adaptive Concurrency**:  
  Dynamic thread-pool scaling based on pipeline complexity.