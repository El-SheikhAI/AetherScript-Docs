# Metrics Dashboard Guide

## Overview
The AetherScript Metrics Dashboard provides centralized operational visibility into pipeline execution and system health. This browser-based interface displays real-time and historical telemetry, enabling performance optimization and proactive incident management.

## Accessing the Dashboard
1. **CLI Launch**  
   Start the dashboard service:
   ```bash
   aether metrics start --port 9300
   ```

2. **Web Interface**  
   Navigate to:  
   `http://localhost:9300` (default)  
   For production deployments:  
   `https://<your-domain>/aether-metrics`

SSL configuration requires adding certificates to your `aether.config.yaml`:
```yaml
metrics:
  ssl:
    cert: /path/to/fullchain.pem
    key: /path/to/privkey.pem
```

## Key Metrics

### Pipeline Performance
- **Execution Duration**: Per-task/pipeline timing distributions
- **Success Rate**: Percentage of successful completions (per pipeline/plugin)
- **Throughput**: Workflows processed per minute (15-minute moving avg)
- **Queue Depth**: Pending tasks in execution buffer
- **Error Frequency**: Categorized error occurrences (last 24h)

### System Health
- **Resource Utilization**
  - CPU Load (1/5/15-min averages)
  - Memory Allocation
  - Garbage Collection Frequency
- **I/O Metrics**
  - Disk Throughput (Read/Write ops)
  - Network Bandwidth (Inbound/Outbound)
- **Plugin Health**
  - Latency Percentiles (P50/P95/P99)
  - Memory Leak Indicators
  - Thread Activity

## Visualization Features
![Dashboard Layout](https://github.com/aetherscript/docs/blob/main/media/dashboard-schematic.png?raw=true)

- **Real-Time Monitoring**
  - Live streaming of active pipeline events
  - Execution progress overlays
  - Instant anomaly detection markers

- **Time-Series Analysis**
  - Configurable time windows (30m, 24h, 7d, custom)
  - Comparative period overlays
  - Exportable CSV data

- **Histogram Displays**
  - Task duration distributions
  - Memory usage frequency
  - Error rate bucketing

- **Interaction Controls**
  ```python
  # Sample dashboard filter
  filters = {
    "time_range": "last_6_hours",
    "pipeline": ["data_ingress", "ml_transform"],
    "min_error_rate": 0.5  # Percentage threshold
  }
  ```
  - Dynamic metric grouping
  - Context-aware drill-downs
  - Custom threshold markers

## Alerting & Troubleshooting

### Threshold Configuration
Set triggers in `alert_rules.yaml`:
```yaml
memory_alert:
  metric: system.memory_usage
  condition: > 85% for 5m
  severity: CRITICAL
  notify:
    - slack#ops-channel
    - email#admin-team

pipeline_timeout:
  metric: pipelines.execution_duration
  condition: p99 > 120s
  severity: WARNING
```

### Diagnostic Procedures
1. **High Resource Utilization**
   - Check `System Health > Memory Allocation > Top Consumers`
   - Analyze flame graphs for thread contention
   - Scale horizontally using `aether cluster scale --workers +3`

2. **Pipeline Timeouts**
   - Inspect `Pipeline Analysis > Slowest Tasks`
   - Profile with `aether trace --pipeline <name>`
   - Enable verbose logging: `aether run --debug-level 3`

3. **Plugin Failures**
   - Review `Plugin Health > Error Origins`
   - Validate dependencies: `aether deps verify --plugin <name>`
   - Roll back versions: `aether plugins revert <name>@2.1.4`

---

> **Production Advisory**  
> For enterprise deployments, configure dashboard retention policies via:
> ```bash
> aether metrics configure --retention 30d --sampling-rate 5s
> ```
> Monitor dashboard resource consumption using the built-in `_self` instrumentation group.