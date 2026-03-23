# Weekly SRE Runbook For Log Analytics Cost

## Goal
Use a cheap weekly process to detect ingestion-cost regressions early, then escalate only the tables that need manual triage.

## Weekly Loop
1. Run `kql/core/04_active_table_inventory.kql`.
2. Run `kql/core/00_workspace_usage_by_table.kql`.
3. Run `kql/core/05_weekly_ingestion_anomalies_by_table.kql`.
4. Check Log Analytics Workspace Insights:
   - Usage tab for top tables and anomalies.
   - Query Audit for slow or inefficient recurring queries.
5. Run `kql/platform/43_heartbeat_stale_resources.kql`.
6. Check Azure Advisor recommendations for the workspace:
   - data ingestion anomaly
   - Basic Logs eligibility
   - pricing tier change
   - restored table cleanup
7. Run the Resource Graph weekly governance checks.
8. Record the results in `docs/weekly-review-template.md`.
9. Escalate only the tables or governance findings that cross threshold or show a new anomaly.

## Safe Query Set
Use these for scheduled runs, workbooks, or a weekly checklist:
- `kql/core/04_active_table_inventory.kql`
- `kql/core/00_workspace_usage_by_table.kql`
- `kql/core/05_weekly_ingestion_anomalies_by_table.kql`
- `kql/platform/43_heartbeat_stale_resources.kql`
- `kql/core/06_usage_billable_volume_spike_by_table.kql`

## Manual Investigation Set
Use these only after a hot table or resource has already been identified:
- `kql/core/01_top3_consumers_per_table.kql`
- `kql/core/02_cross_table_resource_hotspots.kql`
- `kql/core/03_late_arriving_data_check.kql`
- `kql/core/07_top5_largest_records_per_table.kql`
- `kql/generic/30_generic_table_dimension_scan.kql`
- the table-specific drill-down files under `kql/app`, `kql/platform`, `kql/guest-os`, and `kql/security`

## Escalation Thresholds
- Table is at least 10% of workspace ingestion.
- Recent 7-day average is at least 2x the previous 28-day baseline.
- One consumer is at least 25% of a hot table.
- Error-heavy or repeated-message patterns dominate the table.

## Which Query To Use By Table Family
- Azure Functions and App Service runtime logs: `kql/app/10_*`, `kql/app/11_*`, `kql/app/12_*`
- Azure resource logs: `kql/platform/20_azurediagnostics_breakdown.kql` or `kql/generic/30_generic_table_dimension_scan.kql` for resource-specific tables
- Windows guest OS logs, classic `Event` table: `kql/guest-os/21_event_breakdown.kql`
- Event deep dive after a hot classic Event table is found: `kql/guest-os/35_event_source_breakdown.kql`, `kql/guest-os/36_event_repeated_descriptions.kql`, `kql/guest-os/37_event_trend_by_id.kql`
- Windows guest OS logs, AMA or DCR `WindowsEvent` table: `kql/guest-os/34_windowsevent_breakdown.kql`
- Windows security audit events: `kql/security/33_securityevent_breakdown.kql`
- Linux guest OS logs: `kql/guest-os/22_syslog_breakdown.kql`
- Microsoft Entra sign-in logs: `kql/security/28_signinlogs_breakdown.kql`
- Microsoft Entra audit logs: `kql/security/29_auditlogs_breakdown.kql`
- Workspace-based Application Insights and OTel tables: `kql/app/23_applicationinsights_builtin_breakdown.kql`
- Azure control plane logs: `kql/platform/24_azureactivity_breakdown.kql`
- AKS or container logs: `kql/platform/25_containerlogv2_breakdown.kql`
- VM performance counters: `kql/platform/26_perf_breakdown.kql`
- Azure Monitor metrics stored in logs: `kql/platform/27_insightsmetrics_breakdown.kql`
- If the hot VM event table is `Event`, use the classic Windows event path. If it is `WindowsEvent`, use the AMA or DCR path. Do not treat them as interchangeable because the schemas and likely collection controls are different.

## AzureActivity Operator Pack
- Failing control-plane churn: `kql/platform/40_activity_failed_control_plane_ops.kql`
- Sensitive IAM, policy, and lock changes: `kql/platform/41_activity_sensitive_iam_policy_lock_changes.kql`
- Deployment failure rate by caller: `kql/platform/42_activity_deployment_failure_rate_by_caller.kql`

## Resource Graph Weekly Governance Checks
- Missing required tags: `kql/resource-graph/60_arg_missing_required_tags.kql`
- Policy noncompliance by assignment: `kql/resource-graph/62_arg_policy_noncompliance_by_assignment.kql`
- Policy exemptions expiring in 90 days: `kql/resource-graph/63_arg_policy_exemptions_expiring_90d.kql`
- Advisor cost savings summary: `kql/resource-graph/64_arg_advisor_cost_savings_summary.kql`
- Unattached public IPs: `kql/resource-graph/61_arg_unattached_public_ips.kql`
- Recent changes by actor: `kql/resource-graph/65_arg_recent_changes_by_actor.kql`

Resource Graph queries are cheap governance and drift checks, but they are not Log Analytics ingestion queries. Treat `resourcechanges` as recent-drift analysis with a short history window, not long-term reporting.

## If Azure Monitor Shows Query Warnings
- Reduce the time range first. Start at `1d` or `3d`, not `7d` or `31d`, for raw table scans.
- Move from `kql/core/01_top3_consumers_per_table.kql` or `kql/core/02_cross_table_resource_hotspots.kql` to a single-table drill-down as soon as the hot table or resource is known.
- Use the Query Details pane and Query Audit to identify whether the query is CPU-heavy, queued, or returning too much data.
- Break weekly automation into a small safe set. Do not put cross-table `find` queries on a frequent refresh.
- For unknown built-in tables, run `kql/generic/31_builtin_table_shape_probe.kql` before building a custom drill-down.

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
