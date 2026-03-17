# Weekly SRE Runbook For Log Analytics Cost

## Goal
Use a cheap weekly process to detect ingestion-cost regressions early, then escalate only the tables that need manual triage.

## Weekly Loop
1. Run `kql/04_active_table_inventory.kql`.
2. Run `kql/00_workspace_usage_by_table.kql`.
3. Run `kql/05_weekly_ingestion_anomalies_by_table.kql`.
4. Check Log Analytics Workspace Insights:
   - Usage tab for top tables and anomalies.
   - Query Audit for slow or inefficient recurring queries.
5. Check Azure Advisor recommendations for the workspace:
   - data ingestion anomaly
   - Basic Logs eligibility
   - pricing tier change
   - restored table cleanup
6. Record the results in `docs/weekly-review-template.md`.
7. Escalate only the tables that cross threshold or show a new anomaly.

## Safe Query Set
Use these for scheduled runs, workbooks, or a weekly checklist:
- `kql/04_active_table_inventory.kql`
- `kql/00_workspace_usage_by_table.kql`
- `kql/05_weekly_ingestion_anomalies_by_table.kql`

## Manual Investigation Set
Use these only after a hot table or resource has already been identified:
- `kql/01_top3_consumers_per_table.kql`
- `kql/02_cross_table_resource_hotspots.kql`
- `kql/03_late_arriving_data_check.kql`
- `kql/30_generic_table_dimension_scan.kql`
- all table-specific drill-down files under `kql/10+`

## Escalation Thresholds
- Table is at least 10% of workspace ingestion.
- Recent 7-day average is at least 2x the previous 28-day baseline.
- One consumer is at least 25% of a hot table.
- Error-heavy or repeated-message patterns dominate the table.

## Which Query To Use By Table Family
- Azure Functions and App Service runtime logs: `kql/10_*`, `kql/11_*`, `kql/12_*`
- Azure resource logs: `kql/20_azurediagnostics_breakdown.kql` or `kql/30_generic_table_dimension_scan.kql` for resource-specific tables
- Windows guest OS logs: `kql/21_event_breakdown.kql`
- Linux guest OS logs: `kql/22_syslog_breakdown.kql`
- Workspace-based Application Insights and OTel tables: `kql/23_applicationinsights_builtin_breakdown.kql`
- Azure control plane logs: `kql/24_azureactivity_breakdown.kql`
- AKS or container logs: `kql/25_containerlogv2_breakdown.kql`
- VM performance counters: `kql/26_perf_breakdown.kql`
- Azure Monitor metrics stored in logs: `kql/27_insightsmetrics_breakdown.kql`

## If Azure Monitor Shows Query Warnings
- Reduce the time range first. Start at `1d` or `3d`, not `7d` or `31d`, for raw table scans.
- Move from `kql/01_top3_consumers_per_table.kql` or `kql/02_cross_table_resource_hotspots.kql` to a single-table drill-down as soon as the hot table or resource is known.
- Use the Query Details pane and Query Audit to identify whether the query is CPU-heavy, queued, or returning too much data.
- Break weekly automation into a small safe set. Do not put cross-table `find` queries on a frequent refresh.
- For unknown built-in tables, run `kql/31_builtin_table_shape_probe.kql` before building a custom drill-down.

## Guardrails
- `00` and `04` use last full-day buckets. They are for accounting and weekly review, not real-time incident response.
- `01` and `02` use cross-table `find`. Keep them short-window and manual.
- Most targeted raw table drill-downs now default to `3d`. Expand only when the short window is insufficient.
- Prefer fixing noisy collection at source before using retention or plan changes.
- Use Microsoft Learn table docs before changing Basic Logs, transformations, or diagnostic categories.

## Acceptance Gate Before Automation
- Query Details total CPU should stay comfortably below the Microsoft “excessive resources” threshold of 100 seconds.
- Treat any query approaching 1,000 seconds total CPU as unacceptable for this package.
- Treat non-trivial Service Queue Time as a concurrency warning.
- Respect the main Azure Monitor limits that matter here:
  - 5 concurrent Analytics queries
  - 2 concurrent Basic or Auxiliary queries
  - 3 minutes queue wait
  - 200 queries per 30 seconds
  - 500,000 returned rows
  - about 100 MB returned data
  - 10 minute query timeout

## Expected Weekly Output
- top hot tables
- new anomalies
- owner for each candidate issue
- chosen next action
- deferred risks
- verification date
