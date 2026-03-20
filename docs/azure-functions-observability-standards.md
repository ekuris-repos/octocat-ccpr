# Azure Functions Observability Standards

Standards extracted from the Quest IFA Sampling project — a .NET 8 Isolated Azure Functions app demonstrating runtime-configurable Application Insights adaptive sampling.

## Source Project

- **Repository**: Quest IFA Sampling
- **Stack**: .NET 8, Azure Functions v4 (Isolated Worker), Application Insights
- **Pattern**: App-settings-driven observability configuration (restart-only, no redeploy)

## Standard: Runtime-Configurable Sampling

### Principle

All telemetry sampling configuration MUST be driven by app settings / environment variables, never hardcoded. Changes take effect on restart — no code redeployment required.

### Required App Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `Sampling__Enabled` | `bool` | `true` | Master toggle for adaptive sampling |
| `Sampling__Percentage` | `double` | `100` | Sampling retention percentage |
| `Sampling__ExcludedTypes` | `string?` | `Request;Exception` | Semicolon-separated types to never sample |
| `APPLICATIONINSIGHTS_CONNECTION_STRING` | `string` | (required) | App Insights connection string |

### Excluded Telemetry Types Reference

| Type | Description | Default Excluded? |
|------|-------------|-------------------|
| `Request` | HTTP request telemetry | Yes |
| `Exception` | Exception telemetry | Yes |
| `Dependency` | Dependency calls | No |
| `Event` | Custom events | No |
| `Trace` | Trace/log messages | No |

## Standard: host.json Configuration

`host.json` sampling MUST be disabled when using code-based sampling in `Program.cs`:

```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": false
      }
    },
    "logLevel": {
      "default": "Information",
      "Microsoft": "Warning"
    }
  }
}
```

**Rationale**: Leaving host.json sampling enabled causes double-sampling or unpredictable behavior.

## Standard: Program.cs Configuration Pattern

```csharp
var host = new HostBuilder()
    .ConfigureFunctionsWebApplication()
    .ConfigureServices((context, services) =>
    {
        services.AddApplicationInsightsTelemetryWorkerService();
        services.ConfigureFunctionsApplicationInsights();

        var samplingEnabled = context.Configuration.GetValue("Sampling:Enabled", true);
        var samplingPercentage = context.Configuration.GetValue("Sampling:Percentage", 100.0);
        var excludedTypes = context.Configuration.GetValue<string>("Sampling:ExcludedTypes");

        if (samplingEnabled)
        {
            services.Configure<TelemetryConfiguration>(config =>
            {
                var builder = config.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
                builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond: 5,
                    excludedTypes: excludedTypes);
                builder.Build();
            });
        }

        services.AddSingleton(new SamplingSettings
        {
            Enabled = samplingEnabled,
            Percentage = samplingPercentage,
            ExcludedTypes = excludedTypes
        });
    })
    .Build();
```

## Standard: Settings POCO

A dedicated POCO class registered as a singleton for diagnostic injection:

```csharp
public class SamplingSettings
{
    public bool Enabled { get; set; } = true;
    public double Percentage { get; set; } = 100.0;
    public string? ExcludedTypes { get; set; }
}
```

## Standard: Diagnostic Endpoints

Every Functions app with configurable sampling MUST include a `/api/SamplingStatus` endpoint:

```csharp
[Function("SamplingStatus")]
public IActionResult GetSamplingStatus(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get")] HttpRequest req)
{
    return new OkObjectResult(new
    {
        SamplingEnabled = _samplingSettings.Enabled,
        SamplingPercentage = _samplingSettings.Percentage,
        ExcludedTypes = _samplingSettings.ExcludedTypes,
        Note = "Change Sampling__Enabled, Sampling__Percentage, or Sampling__ExcludedTypes in app settings, then restart."
    });
}
```

## Standard: Bulk Telemetry Safety

When generating bulk telemetry (e.g., for testing), always cap the count:

```csharp
var count = int.TryParse(req.Query["count"], out var c) ? c : 10;
count = Math.Min(count, 1000); // cap to prevent abuse
```

Always call `_telemetryClient.Flush()` after bulk operations.

## Standard: File Organization

```
├── Program.cs                    # DI + App Insights + sampling config from IConfiguration
├── host.json                     # Minimal config (sampling DISABLED here)
├── local.settings.json           # Local app settings (gitignored)
├── local.settings.example.json   # Template with placeholders (committed)
├── Functions/
│   └── *.cs                      # HTTP/timer trigger functions
├── TelemetryConfig/
│   └── SamplingSettings.cs       # POCO for sampling config
└── loadtest/
    ├── config.yaml               # Azure Load Testing config
    └── *.jmx                     # JMeter test plans
```

## Standard: Secret Management

| File | Contains Secrets? | Committed? |
|------|-------------------|------------|
| `local.settings.json` | Yes (connection strings) | NO (`.gitignore`) |
| `local.settings.example.json` | No (placeholders only) | Yes |
| App Settings in Azure | Yes | Managed via Portal/CLI/Key Vault |

## Standard: Dependency Injection

- Use constructor injection for `ILogger<T>`, `TelemetryClient`, and `SamplingSettings`
- Never use static access or service locator patterns
- Register `SamplingSettings` as singleton (read once at startup)

## Standard: NuGet Dependencies

| Package | Purpose |
|---------|---------|
| `Microsoft.Azure.Functions.Worker` | Isolated worker model |
| `Microsoft.Azure.Functions.Worker.Extensions.Http.AspNetCore` | HTTP trigger support |
| `Microsoft.Azure.Functions.Worker.ApplicationInsights` | App Insights integration for workers |
| `Microsoft.ApplicationInsights.WorkerService` | Telemetry pipeline and sampling |
