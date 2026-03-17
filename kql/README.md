# KQL Library Map

## Folders
- `core/`: workspace-wide ranking, anomaly detection, cross-table triage, and before/after verification.
- `generic/`: reusable built-in-table probe and drill-down templates for unfamiliar tables.
- `app/`: application and runtime telemetry such as Azure Functions and workspace-based Application Insights.
- `platform/`: Azure platform, control plane, container, and metrics/perf-related tables.
- `guest-os/`: Windows and Linux guest operating system log collection.
- `security/`: Microsoft Entra and Windows security audit tables.

## Recommended Starting Points
- Weekly review: `core/04_active_table_inventory.kql`, `core/00_workspace_usage_by_table.kql`, `core/05_weekly_ingestion_anomalies_by_table.kql`
- Unknown built-in table: `generic/31_builtin_table_shape_probe.kql`, then `generic/30_generic_table_dimension_scan.kql`
- Azure Functions: `app/10_functionapplogs_top_dimensions.kql`
- Windows VM events: `guest-os/21_event_breakdown.kql` or `guest-os/34_windowsevent_breakdown.kql`
- Identity and security: `security/28_signinlogs_breakdown.kql`, `security/29_auditlogs_breakdown.kql`, `security/33_securityevent_breakdown.kql`

## Naming
- `00-05`: cheap-first workspace and weekly triage flow
- `10-19`: app and runtime drill-downs
- `20-29`: platform, guest OS, and identity drill-downs
- `30-39`: generic built-in-table helpers
- `90+`: remediation verification
