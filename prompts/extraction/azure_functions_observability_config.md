---
title: "Azure Functions Application Insights Sampling Configuration"
category: "extraction"
subcategory: "azure_observability"
author: "Erik Kuris"
created_date: "2026-03-20"
last_modified: "2026-03-20"
version: "1.0.0"
status: "draft"
compliance_frameworks: []
tags: ["azure-functions", "application-insights", "sampling", "dotnet", "observability", "telemetry", "app-settings"]
difficulty_level: "intermediate"
estimated_time: "5-15 minutes"
confidence_score: 0.90
dependencies: [".NET 8 SDK", "Azure Functions Core Tools v4", "Application Insights"]
related_prompts: ["octocat_supply_development.md"]
---

# Azure Functions Application Insights Sampling Configuration

## Purpose

Guide AI-assisted development of Azure Functions apps (.NET 8 Isolated) with Application Insights adaptive sampling that is fully configurable via app settings — no redeployment required, only a restart. This prompt captures battle-tested patterns from the Quest IFA Sampling project.

### Primary Use Cases
- Configuring Application Insights adaptive sampling via app settings in Azure Functions
- Implementing the "restart-only config change" pattern for telemetry sampling
- Adding diagnostic endpoints that expose current sampling configuration
- Generating bulk telemetry for sampling validation and load testing
- Disabling `host.json` built-in sampling in favor of code-based `Program.cs` configuration

### Expected Outcomes
- Sampling toggled on/off without code redeployment (app settings + restart only)
- `host.json` sampling disabled so code-based config takes full control
- Settings read from `IConfiguration` using `Sampling__Enabled`, `Sampling__Percentage`, `Sampling__ExcludedTypes`
- Diagnostic endpoint returning current sampling state
- Clean separation: `Program.cs` for DI/config, `Functions/` for triggers, `TelemetryConfig/` for POCOs

### Target Audience
- **Platform Engineers**: Configuring observability for Azure Functions
- **Developers**: Building .NET 8 isolated worker Azure Functions with Application Insights
- **SREs/DevOps**: Adjusting telemetry sampling in production without redeployment

## Prompt Text

```
You are working on an Azure Functions app (.NET 8 Isolated Worker) with Application Insights adaptive sampling.

**Key Design Principle:**
All sampling configuration is driven by app settings / environment variables, NOT hardcoded values.
In Azure: change Configuration > Application Settings, then restart the Function App. No code redeploy needed.

**Project Structure:**
- Program.cs           — Configures Application Insights with sampling settings from IConfiguration
- host.json            — Minimal host config (sampling DISABLED here: "isEnabled": false)
- local.settings.json  — Local environment variables including sampling toggles
- Functions/           — HTTP trigger functions that generate telemetry
- TelemetryConfig/     — POCO classes for sampling configuration

**App Settings Pattern:**
- Sampling__Enabled       (bool, default: true)  — Master toggle for sampling
- Sampling__Percentage    (double, default: 100)  — Sampling percentage
- Sampling__ExcludedTypes (string, nullable)      — Semicolon-separated telemetry types to never sample

**Critical: host.json Sampling Must Be Disabled:**
```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": false
      }
    }
  }
}
```
This ensures the built-in host sampling doesn't interfere with code-based configuration.

**Program.cs Pattern:**
1. Call services.AddApplicationInsightsTelemetryWorkerService()
2. Call services.ConfigureFunctionsApplicationInsights()
3. Read sampling settings from IConfiguration (context.Configuration.GetValue)
4. If sampling is enabled, configure TelemetryConfiguration with UseAdaptiveSampling
5. Register SamplingSettings as a singleton for diagnostic endpoints

**Conventions:**
- Use .NET 8 isolated worker process model (ConfigureFunctionsWebApplication)
- Read ALL config from IConfiguration — never hardcode sampling values
- Use constructor injection for ILogger, TelemetryClient, and SamplingSettings
- Cap bulk telemetry generation (Math.Min) to prevent abuse
- Include a SamplingStatus diagnostic endpoint
- Use TelemetryClient.TrackEvent and TrackTrace for custom telemetry

When generating code for this pattern:
1. Always disable host.json built-in sampling
2. Read settings from IConfiguration with sensible defaults
3. Register SamplingSettings POCO as singleton
4. Include diagnostic endpoints that expose current config
5. Use constructor injection, never static access
6. Flush TelemetryClient after bulk operations
```

## Example Usage

### Input:
```
Add a new timer-triggered function that generates telemetry every 5 minutes and respects
the current sampling configuration.
```

### Expected Output:

A new function file following the established pattern:
- Constructor injection of `ILogger`, `TelemetryClient`, and `SamplingSettings`
- Timer trigger with `[TimerTrigger("0 */5 * * * *")]` schedule
- Custom events and traces via `TelemetryClient`
- Logging the current sampling state for diagnostics
- `_telemetryClient.Flush()` after generating telemetry

### Example 2: Add a New Telemetry Type
**Input:**
```
Add custom metric tracking for response latency that is excluded from sampling.
```

**Expected Output:**
- Update `SamplingSettings.cs` to include a new `ExcludedMetricTypes` property if needed
- Update `Program.cs` to read the new setting from `IConfiguration`
- Add `TelemetryClient.TrackMetric` calls in the function
- Update `local.settings.json` and `local.settings.example.json` with the new setting

## Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `{Sampling__Enabled}` | Master toggle for adaptive sampling | No | `true` |
| `{Sampling__Percentage}` | Percentage of telemetry to retain | No | `100` |
| `{Sampling__ExcludedTypes}` | Semicolon-separated types to never sample | No | `Request;Exception` |
| `{APPLICATIONINSIGHTS_CONNECTION_STRING}` | App Insights connection string | Yes | N/A |
| `{FUNCTIONS_WORKER_RUNTIME}` | Azure Functions runtime | No | `dotnet-isolated` |

## Guidelines

### Best Practices
1. **Disable host.json Sampling**: Always set `"isEnabled": false` in `host.json` sampling settings. Code-based config in `Program.cs` takes full control.
2. **Read from IConfiguration**: Never hardcode sampling values. Use `context.Configuration.GetValue<T>()` with sensible defaults.
3. **Register POCOs as Singletons**: Register `SamplingSettings` via DI so any function can inspect current config.
4. **Diagnostic Endpoints**: Always include a `/api/SamplingStatus` endpoint that returns the current configuration for troubleshooting.
5. **Cap Bulk Operations**: When generating bulk telemetry, always cap the count (e.g., `Math.Min(count, 1000)`) to prevent abuse.

### Quality Criteria
- **No Hardcoded Config**: Every sampling parameter must come from app settings
- **Restart-Only Changes**: Config changes must take effect with only a restart, no redeployment
- **Constructor Injection**: All dependencies via constructor injection, no static/global access
- **Flush After Bulk**: Call `TelemetryClient.Flush()` after generating batches of telemetry

### Common Pitfalls
- **Leaving host.json sampling enabled**: This causes double-sampling or unpredictable behavior when combined with code-based config
- **Hardcoding sampling values**: Defeats the purpose — the whole pattern is runtime-configurable
- **Forgetting to restart**: App settings are read at startup; changing them without restart has no effect
- **Not capping bulk generation**: Unbounded telemetry generation can overwhelm App Insights and incur cost
- **Missing connection string**: Without `APPLICATIONINSIGHTS_CONNECTION_STRING`, telemetry goes nowhere in Azure

### Performance Optimization
- Keep `Request` and `Exception` in `ExcludedTypes` by default — these are critical for debugging
- Use adaptive sampling (`UseAdaptiveSampling`) rather than fixed-rate for production workloads
- Set `maxTelemetryItemsPerSecond` appropriately (5 is a good starting point)
- For high-throughput scenarios, consider separate sampling chains for different telemetry types

## Related Prompts

- [OctoCAT Supply Development](../generation/octocat_supply_development.md) for full-stack TypeScript patterns

## Compliance Notes

- `APPLICATIONINSIGHTS_CONNECTION_STRING` is a secret — use Azure Key Vault references in production, never commit to source control
- `local.settings.json` must be in `.gitignore` (it contains connection strings); provide `local.settings.example.json` with placeholders instead
- Sampling can affect audit completeness — exclude telemetry types required for compliance (e.g., `Exception`, `Request`) via `Sampling__ExcludedTypes`
- Load test configurations (`loadtest/`) may contain endpoint URLs — review before committing to public repositories
