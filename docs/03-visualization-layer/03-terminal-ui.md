# Terminal User Interface (TUI) Guide

## Overview
The AetherScript Terminal UI provides a real-time visualization layer for monitoring and interacting with automation pipelines directly within your terminal. This persistent interface renders execution flows as directed graphs while maintaining native CLI interoperability. 

## Key Features
1. **Adaptive Rendering**: Resolves terminal dimensions dynamically using ANSI escape sequences
2. **Focus-Preserving Design**: Maintains execution context during pipeline updates
3. **Bi-directional Control Surface**: Accepts keyboard inputs without interrupting stdout/stderr streams
4. **Stateful Session Tracking**: Preserves 30 minutes of historical execution data
5. **Plugin-Ready Display**: Extends visualization capabilities through visualization hooks

## Launching the TUI
Activate the visualization layer by appending the `--live` flag to pipeline executions:

```bash
aether run pipeline.as --live
```

## Interface Components

### Execution Graph
```plaintext
[API Call] ➤ (Validation) ➤ [DB Write]
           ↳ [Cache Update]
```

Nodes display with type-specific glyphs:
- `[ ]` Square brackets: Action nodes
- `( )` Parentheses: Condition nodes
- `↳` Arrow: Parallel execution branches
- `◼` Solid square: Active task (pulsing animation)
- ✓̲  Green check: Completed successfully
- ×̲  Red cross: Failed node

### Status Dashboard
```plaintext
┌──────────────┬──────────────┬──────────────┐
│ Active Nodes │  Completed   │  Queue Depth │
├──────────────┼──────────────┼──────────────┤
│      3       │      12      │      5       │
└──────────────┴──────────────┴──────────────┘
```

### Diagnostic Tray
```plaintext
[13:45:42] INFO: Plugin loaded - gh_webhooks@2.1.0
[13:45:43] DEBUG: Established DB pool (5 connections)
```

## Navigation Controls
| Key Binding | Action                          |
|-------------|---------------------------------|
| `→`/`←`     | Pan horizontally               |
| `↑`/`↓`     | Pan vertically                 |
| `+`/`-`     | Zoom in/out                    |
| `f`         | Toggle full-screen mode        |
| `c`         | Clear completed nodes          |
| `i`         | Inspect selected node details  |
| `r`         | Replay execution history       |
| `/`         | Search node tree               |

## Customization
Configure TUI behavior via `aether-config.yaml`:

```yaml
visualization:
  refresh_rate: 200ms
  max_history: 500
  palette: nightfox
  unicode: true
  indicators:
    failed: "✗"
    warning: "⚠"
```

Supported palettes: `nightfox` (default), `daylight`, `matrix`, `monochrome`

## Diagnostic Mode
Enable detailed inspection with `--debug-tui`:

```bash
aether run pipeline.as --live --debug-tui
```

Outputs:
1. Rendering frame times
2. Event loop metrics
3. Memory allocation profiling
4. Plugin load diagnostics

## Platform Support
- **Full Support**: Linux/macOS terminals supporting ANSI escape codes
- **Basic Support**: Windows Terminal (version 1.12+)
- **Headless Fallback**: Automatically switches to JSON-stream mode when terminal lacks TUI capabilities

## Security Considerations
1. All input handling uses hardware-isolated keypress buffers
2. Visualization data scrubbs sensitive parameters by default
3. Session tokens automatically expire after 15 minutes of inactivity
4. Audit logs record all control surface interactions

---
**Next**: [Web Dashboard Interface](./04-web-dashboard.md)  