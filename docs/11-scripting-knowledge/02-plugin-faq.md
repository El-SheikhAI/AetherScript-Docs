# Plugin Development FAQ

## What is an AetherScript Plugin?
AetherScript plugins are isolated modules that extend core functionality without modifying the runtime. They adhere to a zero-config architecture—simply install a compatible package, and AetherScript automatically integrates it into your workflow. Plugins can add commands, modify execution behavior, or visualize data in real time.

## How Do I Install a Plugin?
Use your preferred package manager to install the plugin, then run `aether plugins sync` to refresh available modules. Example:
```bash
npm install @aetherscript/plugin-aws
aether plugins sync
```
AetherScript validates plugins at startup and lists active plugins in the `aether.plugins` section of the runtime configuration.

## Can I Create Custom Plugins?
Yes. Implement the mandatory `Plugin` interface in TypeScript:
```typescript
import { Plugin, ExecutionContext } from 'aetherscript-core';

export default class CustomPlugin implements Plugin {
  static meta = {
    name: 'custom-plugin',
    version: '1.0.0',
  };

  async execute(ctx: ExecutionContext) {
    // Your logic here
  }
}
```
Package your plugin as an ES module and publish it to npm or a private registry. See the [Plugin Development Guide](../07-plugin-development/01-getting-started.md) for details.

## How Are Plugin Conflicts Resolved?
AetherScript loads plugins in dependency-sorted order. For competing functionalities:
1. **Command Overrides**: Last-loaded plugin takes precedence.
2. **Event Hooks**: All registered hooks execute in load order.  
Use `aether plugins list --verbose` to inspect load sequences.

## How Do I Debug Plugin Issues?
1. Run workflows with `--debug-plugins` to isolate plugin activity:
   ```bash
   aether run pipeline.yaml --debug-plugins
   ```
2. Check `/var/log/aetherscript/plugins.log` for execution traces.
3. Test plugins independently using the `aether dev plugin-test` sandbox.

## Can Plugins Access Core Internals?
No. Plugins interact through controlled APIs:
- **ExecutionContext**: Read/write pipeline state.
- **Hook System**: Subscribe to lifecycle events (e.g., `pre-execute`, `post-failure`).
- **Visualization Gateway**: Push metrics to the real-time dashboard via `ctx.visualize()`.

## Are There Performance Limits?
Plugins undergo strict resource profiling:
- **Timeout**: 5s per synchronous operation (configurable).
- **Memory**: Heap capped at 256MB per plugin instance.
Critical paths should offload heavy processing via `ctx.spawnWorker()`.

## Where Can I Find Official Plugins?
Browse verified plugins at [plugins.aetherscript.io](https://plugins.aetherscript.io). All packages are:
- Signed with AetherScript’s artifact attestation.
- Benchmarked against performance baselines.
- Version-locked to specific runtime compatibility ranges.  

---

For unsolved issues, consult our [Community Forum](https://discuss.aetherscript.io) or file an issue in the [Plugin GitHub Repository](https://github.com/aetherscript/plugins).