# Built-In Table Family Map

## Purpose
Use this map when a hot table is unfamiliar and you need to decide which drill-down path to start with.

## Families
- Azure resource logs:
  Examples: `AzureDiagnostics`, service-specific resource tables such as `AGWAccessLogs`, `AZKVAuditLogs`, `AKSControlPlane`
  Start with: `kql/31_builtin_table_shape_probe.kql`, then `kql/30_generic_table_dimension_scan.kql`
- Azure Functions and App Service runtime logs:
  Examples: `FunctionAppLogs`
  Start with: `kql/10_*`, `kql/11_*`, `kql/12_*`
- Workspace-based Application Insights and OTel:
  Examples: `AppTraces`, `AppExceptions`, `AppRequests`, `AppDependencies`, `OTelLogs`, `OTelTraces`
  Start with: `kql/23_applicationinsights_builtin_breakdown.kql`
- Windows guest OS logs:
  Examples: `Event`, `WindowsEvent`
  Start with: `kql/21_event_breakdown.kql`
- Linux guest OS logs:
  Examples: `Syslog`
  Start with: `kql/22_syslog_breakdown.kql`
- Container and AKS logs:
  Examples: `ContainerLogV2`, `KubeEvents`, `ContainerInventory`
  Start with: `kql/25_containerlogv2_breakdown.kql` or the generic probe path
- VM and agent performance data:
  Examples: `Perf`, `InsightsMetrics`, `Heartbeat`
  Start with: `kql/26_perf_breakdown.kql`, `kql/27_insightsmetrics_breakdown.kql`
- Azure control plane and governance:
  Examples: `AzureActivity`
  Start with: `kql/24_azureactivity_breakdown.kql`
- Identity and security platform logs:
  Examples: `SigninLogs`, `AuditLogs`, `SecurityEvent`
  Start with: `kql/31_builtin_table_shape_probe.kql`, then a one-dimension summary via `kql/32_builtin_table_drilldown_template.kql`

## Unknown Table Recipe
1. Check `LikelyTableClass` from `kql/04_active_table_inventory.kql`.
2. Confirm built-in status on Microsoft Learn `tables-index`.
3. Run `kql/31_builtin_table_shape_probe.kql`.
4. Pick 2 to 4 populated, high-signal columns.
5. Adjust `kql/32_builtin_table_drilldown_template.kql`.
6. Only widen the time range if the short query stays healthy in Query Details.
