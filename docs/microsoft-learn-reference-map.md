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

### Azure Monitor service limits
- URL: [Azure Monitor service limits](https://learn.microsoft.com/azure/azure-monitor/fundamentals/service-limits)
- Use for: concurrency, rate, timeout, and result-size constraints.
- Why it matters: workbook and dashboard design should respect these limits to avoid self-inflicted throttling.

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
4. Then run `kql/30_generic_table_dimension_scan.kql` for that table.

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

## What This Means For “All Built-In Tables”
You should think in two layers:

### Layer 1: Built-in table universe
Microsoft Learn `tables-index` is the official answer to “what built-in tables can exist.”

### Layer 2: Active tables in this workspace
`Usage` is the official cheap answer to “which built-in tables are actually active and billable here.”

Trying to do layer 1 purely from workspace data is incorrect. Trying to do layer 2 purely from docs is inefficient.

Use both.
