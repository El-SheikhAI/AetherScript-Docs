# AetherScript Workflow Renderer  

## Introduction  
The Workflow Renderer is a core visualization component of AetherScript that dynamically generates human-readable representations of automation pipelines. It operates in two modes:  
1. **Real-Time Visualization** (Active Pipeline Execution)  
2. **Static Rendering** (Pipeline Blueprint Analysis)  

## Key Features  
- **Multi-Format Output**: Renders workflows as SVG, PNG, or ASCII diagrams  
- **Zero-Config Plugin Detection**: Automatically incorporates visual elements from connected plugins  
- **Interactive Debugging**: Clickable node inspection during pipeline execution  
- **Dependency Mapping**: Visualizes resource dependencies and execution order  

## Syntax Schema  
Define visual representations using declarative comments in your `.aether` scripts:  

```aether
#!visual title="Data Processing Pipeline"  
#!visual group="Ingestion" color=#4A90E2  
fetch_data --source=api <<< #!visual icon="cloud-download"  
transform --filter=invalid <<< #!visual icon="gear"  

#!visual group="Analysis" color=#50BFA9  
run_ml_model <<< #!visual icon="neural-network"  
generate_report --format=html <<< #!visual icon="document"  
```  

## CLI Integration  
### Live Monitoring  
```bash  
aether run pipeline.aether --visualize=live  
```  
![Live Rendering Interface](/assets/live-visual.png)  

### Static Export  
```bash  
aether render pipeline.aether --format=svg --output=docs/pipeline.svg  
```  

## Plugin Visualization  
Third-party plugins automatically contribute to visualization through:  
1. Standardized icon libraries  
2. Color-coded namespace tagging  
3. Custom metadata annotations  

```aether  
# Installed "aether-db" plugin automatically registers its visual schema  
query_db --table=users <<< #!visual db.plugin:icon="database"  
```  

## Configuration  
Customize rendering behavior via `aether.config.yaml`:  

```yaml  
visual:  
  theme: "dark"  
  layout: "vertical"  
  node:  
    width: 180  
    height: 80  
  edge:  
    style: "ortho"  
    color: "#606060"  
  plugins:  
    custom-icons:  
      neural-network: "/assets/icons/ai-chip.svg"  
```  

## Examples  
### Horizontal Workflow  
```bash  
aether render workflow.aether --layout=horizontal  
```  
![Horizontal Layout](/assets/horizontal-example.png)  

### Compact ASCII Output  
```bash  
aether render workflow.aether --format=ascii  
```  
```  
[Fetch] → [Transform] → [Analyze]  
  ↓             ↑  
[Validate] ← [Cache]  
```  

## Troubleshooting  
**Missing Visual Elements**:  
1. Verify plugin installation with `aether plugins list`  
2. Check for conflicting theme configurations  
3. Ensure renderer version matches core CLI (`aether --version`)  

**Performance Issues**:  
- Limit real-time rendering complexity with `--visualize=simple` flag  
- Reduce node density using `#!visual group` aggregations  

---  
Next: [02-realtime-debugger.md](./02-realtime-debugger.md)  
Previous: [../02-execution-engine/05-error-handling.md](../02-execution-engine/05-error-handling.md)