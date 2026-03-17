# Microsoft Learn Reference Map For Log Analytics Cost Triage

This file is the official-source companion to the KQL package. It explains which Microsoft Learn pages answer which operational questions.

## Core References

### Azure Monitor data reference
- URL: [Azure Monitor data reference](https://learn.microsoft.com/azure/azure-monitor/reference/)
- Use for: the top-level catalog of logs, metrics, and sample queries.
- Why it matters: this is the official entry point for understanding what Azure Monitor can emit and where to find per-table documentation.

### Azure Monitor Resource log / log analytics tables
- URL: [Azure Monitor Resource log / log analytics tables](https://learn.microsoft.com/azure/azure-monitor/reference/tables-index)
- Use for: the built-in table universe and the mapping between Azure resource providers and Log Analytics tables.
- Why it matters: this is the correct source for “which built-in tables can exist,” not the contents of one workspace.

### Tables that support transformations in Azure Monitor Logs
- URL: [Tables that support transformations in Azure Monitor Logs](https://learn.microsoft.com/azure/azure-monitor/logs/tables-feature-support)
- Use for: deciding whether ingestion-time transformations are available for a target table.
- Why it matters: not every table supports transformations, and some tables have partial support.

### Tables that support the Basic table plan in Azure Monitor Logs
- URL: [Tables that support the Basic table plan in Azure Monitor Logs](https://learn.microsoft.com/azure/azure-monitor/logs/basic-logs-azure-tables)
- Use for: deciding whether a high-volume table can move to Basic Logs.
- Why it matters: this is a direct cost lever, but only for tables Microsoft lists as eligible.

### Analyze usage in a Log Analytics workspace
- URL: [Analyze usage in a Log Analytics workspace](https://learn.microsoft.com/azure/azure-monitor/logs/analyze-usage)
- Use for: the official ingestion-cost workflow, `Usage`-first analysis, and the caution that `find` should be used sparingly.
- Why it matters: this is the most important operational reference for cost triage.

### Optimize log queries in Azure Monitor
- URL: [Optimize log queries in Azure Monitor](https://learn.microsoft.com/azure/azure-monitor/logs/query-optimization)
- Use for: performance and cost-aware query construction.
- Why it matters: this is the official source for early filtering, reducing retrieved columns, avoiding broad scans, and other efficiency rules.

### Log Analytics Workspace Insights
- URL: [Log Analytics Workspace Insights](https://learn.microsoft.com/azure/azure-monitor/logs/log-analytics-workspace-insights-overview)
- Use for: regular review of top tables, ingestion anomalies, workspace usage, and query audit findings.
- Why it matters: this is the official workspace-level operational view that SRE teams should check weekly alongside the custom KQL package.

### Audit queries in Azure Monitor Logs
- URL: [Audit queries in Azure Monitor Logs](https://learn.microsoft.com/azure/azure-monitor/logs/query-audit)
- Use for: enabling query audit telemetry and reviewing expensive recurring queries.
- Why it matters: weekly cost governance should include the queries that operators and dashboards run, not just the data being ingested.

### Monitor operational issues in your Azure Monitor Log Analytics workspace
- URL: [Monitor operational issues in your Azure Monitor Log Analytics workspace](https://learn.microsoft.com/azure/azure-monitor/logs/monitor-workspace)
- Use for: `_LogOperation`, workspace warnings, and workspace-level alerts.
- Why it matters: ingestion and query problems can distort cost analysis or hide backfill and throttling behavior.

### Aggregate data in a Log Analytics workspace by using summary rules
- URL: [Aggregate data in a Log Analytics workspace by using summary rules](https://learn.microsoft.com/azure/azure-monitor/logs/summary-rules)
- Use for: long-range reporting and repeated analysis on very high-volume data streams.
- Why it matters: summary rules are the official way to reduce repeated long-range raw scans for recurring reporting.

### Diagnostic settings in Azure Monitor
- URL: [Diagnostic settings in Azure Monitor](https://learn.microsoft.com/azure/azure-monitor/platform/diagnostic-settings)
- Use for: resource-log category selection, category groups, routing destinations, and cost controls.
- Why it matters: most Azure resource-log cost issues start with diagnostic settings, not with Log Analytics itself.

### Azure Monitor service limits
- URL: [Azure Monitor service limits](https://learn.microsoft.com/azure/azure-monitor/fundamentals/service-limits)
- Use for: concurrency, rate, timeout, and result-size constraints.
- Why it matters: workbook and dashboard design should respect these limits to avoid self-inflicted throttling.

### Azure Resource Graph query language
- URL: [Understanding the Azure Resource Graph query language](https://learn.microsoft.com/azure/governance/resource-graph/concepts/query-language)
- Use for: supported tables, joins, and engine behavior in Resource Graph Explorer.
- Why it matters: Resource Graph queries look similar to KQL but they run in a different engine and table universe than Log Analytics.

### Azure Resource Graph table reference
- URL: [Azure Resource Graph table and resource type reference](https://learn.microsoft.com/azure/governance/resource-graph/reference/supported-tables-resources)
- Use for: confirming supported tables such as `Resources`, `PolicyResources`, and `AdvisorResources`.
- Why it matters: this is the source of truth for current-state governance and inventory queries.

### Change Analysis in Resource Graph
- URL: [Get resource changes](https://learn.microsoft.com/azure/governance/resource-graph/changes/get-resource-changes)
- Use for: `resourcechanges` schema, query examples, and recent-change analysis.
- Why it matters: Microsoft documents that change analysis queries are for recent windows and should be treated as a short-retention drift tool.

## How To Use These References In Practice

### If you only know that costs are high
1. Use `Usage`-based queries first.
2. Only after you identify hot tables should you open the official table docs.
3. Do not start with raw scans across every table.

### If you find an unfamiliar built-in table
1. Search the table name on the `tables-index` page.
2. Open the table reference page.
3. Inspect:
   - columns
   - category or solution
   - Basic log support
   - ingestion-time transformation support
4. Then run `kql/generic/30_generic_table_dimension_scan.kql` for that table.

### If the hot table is a Windows or identity/security table
- `Event`: use the Microsoft Learn `Event` table page and its sample queries.
- `WindowsEvent`: use the `WindowsEvent` table page and confirm AMA or DCR-style fields such as `Channel` and `Provider`.
- `SecurityEvent`: use the `SecurityEvent` table page and sample queries for common high-volume event IDs.
- `SigninLogs`: use the `SigninLogs` table page and sample queries to confirm app, resource, user, and result fields.
- `AuditLogs`: use the `AuditLogs` table page to confirm activity, operation type, initiator, and target resource fields.

### If the query is Resource Graph instead of Log Analytics
1. Confirm the table on the Azure Resource Graph table reference page.
2. Run the query in Resource Graph Explorer, not Log Analytics.
3. For `resourcechanges`, keep the investigation window recent because Change Analysis is not a long-retention source.
4. When adding joins, stay on subscription or resource ID keys and keep the projected columns tight.

### If you are deciding on a remediation
Use the official table page plus the transformations and Basic Logs reference pages before you decide:
- drop at source
- prune diagnostic categories
- use a transformation
- change table plan
- change retention

## Best-Practice Rules From Microsoft That This Package Follows
- Use `Usage` before raw event scans for workspace-wide ranking.
- Use `find` sparingly because it is resource-intensive.
- Add `TimeGenerated` filters early and in every subquery.
- Filter on physical columns where possible rather than computed ones.
- Reduce the number of columns retrieved.
- Avoid `search *` and `union *` unless absolutely necessary.
- Prefer shorter time windows for high-volume raw scans.
- Be cautious with joins and high-cardinality summarize keys.
- Review Workspace Insights and Query Audit regularly for recurring-query overhead.
- Review Azure Advisor anomaly and Basic Logs recommendations alongside raw ingestion analysis.
- Keep recurring query sets within Azure Monitor limits for concurrency, queue time, result size, and timeout.

## What This Means For “All Built-In Tables”
You should think in two layers:

### Layer 1: Built-in table universe
Microsoft Learn `tables-index` is the official answer to “what built-in tables can exist.”

### Layer 2: Active tables in this workspace
`Usage` is the official cheap answer to “which built-in tables are actually active and billable here.”

Trying to do layer 1 purely from workspace data is incorrect. Trying to do layer 2 purely from docs is inefficient.

Use both.
