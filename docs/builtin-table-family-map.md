# Built-In Table Family Map

## Purpose
Use this map when a hot table is unfamiliar and you need to decide which drill-down path to start with.

## Families
- Azure resource logs:
  Examples: `AzureDiagnostics`, service-specific resource tables such as `AGWAccessLogs`, `AZKVAuditLogs`, `AKSControlPlane`
  Start with: `kql/generic/31_builtin_table_shape_probe.kql`, then `kql/generic/30_generic_table_dimension_scan.kql`
- Azure Functions and App Service runtime logs:
  Examples: `FunctionAppLogs`
  Start with: `kql/app/10_*`, `kql/app/11_*`, `kql/app/12_*`
- Workspace-based Application Insights and OTel:
  Examples: `AppTraces`, `AppExceptions`, `AppRequests`, `AppDependencies`, `OTelLogs`, `OTelTraces`
  Start with: `kql/app/23_applicationinsights_builtin_breakdown.kql`
- Windows event logs from MMA or legacy agent collection:
  Examples: `Event`
  Start with: `kql/guest-os/21_event_breakdown.kql`
  Use when the table has `EventLog`, `Source`, `RenderedDescription`, and other classic Windows event fields.
- Windows event logs from AMA or DCR-style collection:
  Examples: `WindowsEvent`
  Start with: `kql/guest-os/34_windowsevent_breakdown.kql`
  Use when the table has `Channel`, `Provider`, `EventLevelName`, and parsed `EventData`.
- Linux guest OS logs:
  Examples: `Syslog`
  Start with: `kql/guest-os/22_syslog_breakdown.kql`
- Container and AKS logs:
  Examples: `ContainerLogV2`, `KubeEvents`, `ContainerInventory`
  Start with: `kql/platform/25_containerlogv2_breakdown.kql` or the generic probe path
- VM and agent performance data:
  Examples: `Perf`, `InsightsMetrics`, `Heartbeat`
  Start with: `kql/platform/26_perf_breakdown.kql`, `kql/platform/27_insightsmetrics_breakdown.kql`
- Azure control plane and governance:
  Examples: `AzureActivity`
  Start with: `kql/platform/24_azureactivity_breakdown.kql`
- Identity platform sign-in logs:
  Examples: `SigninLogs`
  Start with: `kql/security/28_signinlogs_breakdown.kql`
- Identity platform audit logs:
  Examples: `AuditLogs`
  Start with: `kql/security/29_auditlogs_breakdown.kql`
- Windows security audit events:
  Examples: `SecurityEvent`
  Start with: `kql/security/33_securityevent_breakdown.kql`
  Use when the table is focused on security auditing, logons, account changes, policy changes, and security-relevant Windows events.

## Event Family Distinctions
- `Event`: classic Windows Event Log collection. Use when the workspace contains the legacy `Event` table and you need to group by `EventLog`, `EventID`, `EventLevelName`, and `Computer`.
- `WindowsEvent`: newer Windows event collection shape with `Channel`, `Provider`, and parsed event payload support. Use when the table name is explicitly `WindowsEvent`.
- `SecurityEvent`: Windows security auditing data. Use this when you care about sign-ins, privilege changes, process creation, policy changes, or account activity on Windows hosts.
- `SigninLogs`: Microsoft Entra sign-in activity. Start with app, user, result, and conditional access status.
- `AuditLogs`: Microsoft Entra audit activity. Start with activity, operation type, initiator, and result.

## Unknown Table Recipe
1. Check `LikelyTableClass` from `kql/core/04_active_table_inventory.kql`.
2. Confirm built-in status on Microsoft Learn `tables-index`.
3. Run `kql/generic/31_builtin_table_shape_probe.kql`.
4. Pick 2 to 4 populated, high-signal columns.
5. Adjust `kql/generic/32_builtin_table_drilldown_template.kql`.
6. Only widen the time range if the short query stays healthy in Query Details.
