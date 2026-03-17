# Log Analytics Cost Triage Playbook

## Purpose
This playbook is a portal-first operating guide for identifying which Log Analytics tables cost the most, which consumers dominate each table, how to drill into the root cause, and how to verify savings after a fix.

It is optimized for recurring triage rather than a one-off investigation:

1. Rank tables by recent billable ingestion.
2. Find the top 3 consumers inside each expensive table.
3. Drill into the table-specific dimensions that explain why the data exists.
4. Decide whether the right fix is application logging, diagnostic settings, transformations, retention, or table-plan changes.
5. Verify the change with before/after cost evidence.

## Package Contents
- `kql/00_workspace_usage_by_table.kql`: Workspace table ranking for 1d, 7d, and 31d windows.
- `kql/01_top3_consumers_per_table.kql`: Cross-table top-3-per-table consumer ranking using a short raw-data window.
- `kql/02_cross_table_resource_hotspots.kql`: Recent Azure resource hotspots across all tables.
- `kql/03_late_arriving_data_check.kql`: Ingestion-day vs event-time analysis for replay and delayed ingestion.
- `kql/04_active_table_inventory.kql`: Cheap active-table inventory from `Usage`, including billable share and likely doc lookup URLs.
- `kql/05_weekly_ingestion_anomalies_by_table.kql`: Automation-safe weekly anomaly scan based on `Usage`.
- `kql/10_functionapplogs_top_dimensions.kql`: `FunctionAppLogs` breakdown by app, function, category, and level.
- `kql/11_functionapplogs_repeated_messages.kql`: Normalized repeated message signatures for noisy Functions logs.
- `kql/12_functionapplogs_execution_outcomes.kql`: Function execution outcomes, failures, and duration.
- `kql/20_azurediagnostics_breakdown.kql`: `AzureDiagnostics` breakdown by provider, type, category, operation, and resource.
- `kql/21_event_breakdown.kql`: `Event` breakdown by event log, ID, level, and computer.
- `kql/22_syslog_breakdown.kql`: `Syslog` breakdown by computer, facility, severity, and process.
- `kql/23_applicationinsights_builtin_breakdown.kql`: Workspace-based Application Insights and OTel table breakdown.
- `kql/24_azureactivity_breakdown.kql`: `AzureActivity` breakdown by caller, provider, operation, and status.
- `kql/25_containerlogv2_breakdown.kql`: `ContainerLogV2` breakdown by namespace, pod, container, and log level.
- `kql/26_perf_breakdown.kql`: `Perf` breakdown by computer, object, counter, and instance.
- `kql/27_insightsmetrics_breakdown.kql`: `InsightsMetrics` breakdown by resource, origin, namespace, and metric name.
- `kql/30_generic_table_dimension_scan.kql`: Generic dimension scan for any single built-in table selected after table ranking.
- `kql/90_remediation_verification.kql`: Before/after comparison and savings estimate after a fix.
- `docs/microsoft-learn-reference-map.md`: Official Microsoft source map for built-in table discovery, per-table attributes, and optimization guidance.
- `docs/sre-weekly-runbook.md`: Short weekly operating runbook for SRE teams.
- `docs/triage-notes-template.md`: Investigation worksheet for one suspected issue.
- `docs/weekly-review-template.md`: Short weekly review artifact for recurring handoff.

## Official Source of Truth
This package is grounded in Microsoft Learn rather than a hard-coded opinionated list of tables.

Use the Microsoft docs in this order:

1. `Azure Monitor data reference`: the top-level catalog for logs, metrics, and sample queries.
2. `Azure Monitor Resource log / log analytics tables`: the built-in table universe and the resource-provider-to-table mapping.
3. Per-table table reference pages: the table schema and attributes such as Basic log support and ingestion-time transformation support.
4. `Tables that support transformations in Azure Monitor Logs`: whether ingestion-time filtering is available for a table.
5. `Analyze usage in a Log Analytics workspace`: the official cost-analysis workflow and the warning to use `find` sparingly.
6. `Optimize log queries in Azure Monitor`: the query efficiency rules that keep triage cheap.
7. `Azure Monitor service limits`: concurrency, query rate, and result-size limits that influence workbook or dashboard design.

See `docs/microsoft-learn-reference-map.md` for the direct links and how to use them.

## Built-In Table Inventory Model
There are two different questions and they need different sources:

### 1. What built-in tables can exist in Azure Monitor Logs?
Use Microsoft Learn:
- `docs/microsoft-learn-reference-map.md`
- the global `tables-index` page
- the top-level Azure Monitor data reference

Do not try to reverse-engineer the entire built-in universe from your workspace alone. Your workspace only shows active tables, not all possible built-in tables.

### 2. Which built-in tables are actually active and billable in this workspace?
Use `Usage`:
- `kql/04_active_table_inventory.kql`
- `kql/00_workspace_usage_by_table.kql`

This is the cheap path. Start from active tables, not the entire built-in catalog.

## Operating Model

### Step 0: Set Guardrails
- Start with `1d` or `3d` raw-data lookbacks for cross-table `find` queries.
- Use `7d` when you need a stable signal.
- Do not run broad cross-table raw-data scans over long retention windows unless you are already focused on a small set of tables.
- Treat `Usage` as the default source for workspace-wide ranking and raw table queries as the drill-down layer.
- Prefer one workspace at a time. Multi-workspace and multi-region queries are more expensive and less efficient.
- Prefer aggregated outputs over raw event exports.
- Separate automation-safe queries from manual-only queries.

### Query Efficiency Rules
These rules are directly aligned to Microsoft’s Azure Monitor query optimization guidance:

- Add `TimeGenerated` filters early and make sure every subquery has its own time filter.
- Add `where` filters before heavy parsing, regex extraction, joins, and large `summarize` steps.
- Filter on physical columns when possible. Avoid filtering on computed columns if the source column can be used directly.
- Prefer `has` over `contains` when token-style matching is sufficient.
- Project or summarize only the columns you need.
- Avoid `search *`, `union *`, and unrestricted `find` unless there is no cheaper alternative.
- Use `Usage` for workspace-wide table ranking and use raw table scans only after you know which table is worth the cost.
- Keep recurring dashboards and workbooks query-light. Large numbers of concurrent queries can hit Azure Monitor service limits.
- For repeated long-range reporting, consider summary rules rather than rerunning wide raw scans.

### Automation-Safe Versus Manual-Only
Use these in scheduled reviews, workbooks, or weekly jobs:
- `kql/04_active_table_inventory.kql`
- `kql/00_workspace_usage_by_table.kql`
- `kql/05_weekly_ingestion_anomalies_by_table.kql`

Use these only for manual triage after a hot table or hot resource is already known:
- `kql/01_top3_consumers_per_table.kql`
- `kql/02_cross_table_resource_hotspots.kql`
- `kql/03_late_arriving_data_check.kql`
- `kql/30_generic_table_dimension_scan.kql`
- the table-specific drill-down queries

### Step 1: Find Expensive Tables
Run `kql/04_active_table_inventory.kql` first, then `kql/00_workspace_usage_by_table.kql`, then `kql/05_weekly_ingestion_anomalies_by_table.kql`.

Use it to answer:
- Which active built-in tables are present in this workspace?
- Which tables dominate the workspace right now?
- Is the issue new in the last day, sustained over the last week, or simply large over the last month?
- Which tables are growing versus the prior comparable window?
- Which tables materially exceed their recent baseline and deserve manual drill-down?

For any unfamiliar table:
1. Copy the table name from `04_active_table_inventory.kql`.
2. Open the table’s Microsoft Learn reference page.
3. Confirm schema, table attributes, and whether Basic Logs or transformations are supported.
4. Only then design a table-specific drill-down.

Escalate a table to drill-down when any of the following is true:
- The table is at least 10% of workspace billable ingestion.
- The table shows at least 2x growth versus the prior comparable 7-day window.
- The weekly anomaly query flags it as `NewOrPreviouslyTinyTable`, `RecentAvgAboveBaseline`, or `LatestDaySpike`.
- The table is already known to be operationally noisy, such as `FunctionAppLogs`, `AzureDiagnostics`, or `Event`.

### Step 2: Find the Top 3 Consumers Inside Each Table
Run `kql/01_top3_consumers_per_table.kql` with a `1d` or `3d` lookback.

This is intentionally a short-window query because Microsoft explicitly warns that `find` across many tables is resource-intensive. Use it once per incident to find where to focus, not as a general dashboard query.

Do not try to turn this query into a targeted single-table view. Once you know the hot table, switch to `kql/30_generic_table_dimension_scan.kql` or a table-specific drill-down so the scan scope becomes truly narrow.

The query normalizes a best-effort `Consumer` key in this order:
1. `AppName`
2. `_ResourceId`
3. `Computer`
4. `Resource`
5. `_SubscriptionId`
6. `SourceSystem`

Interpretation rules:
- If one consumer is at least 25% of a table, triage that consumer first.
- If a table has multiple consumers with similar percentages, the issue is likely a shared platform setting, collection policy, or broad workload growth.
- If the same `_ResourceId` appears across multiple tables, pivot to `kql/02_cross_table_resource_hotspots.kql`.

### Step 3: Drill Down by Table Type

#### Azure resource logs in resource-specific tables
Do not assume all Azure resource logs land in `AzureDiagnostics`. Many services now emit to resource-specific built-in tables.

Use this sequence:
1. Use `kql/04_active_table_inventory.kql` and `kql/00_workspace_usage_by_table.kql` to identify the hot resource-specific table.
2. Open the official table doc for that table.
3. Run `kql/30_generic_table_dimension_scan.kql` if there is no dedicated query yet.
4. Review the resource's diagnostic settings and category groups before changing collection.

#### Any built-in table not already covered
Use this sequence:
1. Open the Microsoft Learn table reference page for the table.
2. Check the table columns, sample queries, and table attributes.
3. Run `kql/30_generic_table_dimension_scan.kql` against that table.
4. If the generic scan identifies a dominant dimension, build or adapt a table-specific query only for that table.

Use this path for unknown or newly encountered built-in tables so the workflow remains grounded in official schema and remains cheap.

#### FunctionAppLogs
Use this sequence:
1. `kql/10_functionapplogs_top_dimensions.kql`
2. `kql/11_functionapplogs_repeated_messages.kql`
3. `kql/12_functionapplogs_execution_outcomes.kql`

What you are looking for:
- One app or function dominating `GB`.
- A single `Category` or `Level` driving cost.
- Repeated normalized messages that suggest retry loops, poison queue churn, dependency failures, auth failures, or host/runtime misconfiguration.
- Large `Error` volume or large numbers of repeated `Information` messages with little operational value.

High-confidence `FunctionAppLogs` symptoms:
- `Error`-dominated cost with repeated messages: likely broken dependency, misconfiguration, or retry storm.
- `Information`-dominated cost under `Function.<name>.User`: likely overly chatty application logging.
- `Host` or runtime categories dominating multiple apps: likely Azure Functions host logging configuration or platform-side verbosity.

#### AzureDiagnostics
Run `kql/20_azurediagnostics_breakdown.kql`.

What you are looking for:
- One `ResourceProvider` or `Category` dominating usage.
- One operation producing very high volume.
- One resource ID responsible for most of the table.

Typical fixes:
- Remove unneeded diagnostic categories.
- Route only the categories you actually use.
- Apply supported ingestion-time transformations if the table is transformation-capable.
- Revisit whether logs are needed in Analytics versus another destination.

#### Event
Run `kql/21_event_breakdown.kql`.

What you are looking for:
- Specific `EventID` values that are unexpectedly frequent.
- One `Computer` or server class dominating the table.
- One `EventLog` and `EventLevelName` combination explaining most of the volume.

Typical fixes:
- Filter the event set at collection time if those IDs are not operationally useful.
- Fix the host or application generating the repeated events.
- Remove redundant event sources from collection rules.

#### Syslog
Run `kql/22_syslog_breakdown.kql`.

What you are looking for:
- One process or facility dominating ingestion.
- One host class sending unusually noisy messages.
- Severity levels that are too verbose for steady-state operations.

Typical fixes:
- Tighten Linux syslog collection.
- Reduce application verbosity on the emitting host.
- Remove low-value facilities or process streams from collection.

#### Workspace-based Application Insights and OpenTelemetry tables
Run `kql/23_applicationinsights_builtin_breakdown.kql`.

Common target tables:
- `AppTraces`
- `AppExceptions`
- `AppRequests`
- `AppDependencies`
- `OTelLogs`
- `OTelTraces`

What you are looking for:
- One app role or operation dominating the table.
- Trace-heavy informational logging with little operational value.
- Exception or dependency spam that points to a broken downstream dependency or retry storm.

Typical fixes:
- Reduce trace verbosity.
- Remove payload-heavy debug logging.
- Fix the dependency, retry loop, or instrumentation issue causing the volume.

#### AzureActivity
Run `kql/24_azureactivity_breakdown.kql`.

Use this when control plane activity appears unexpectedly large or bursty. Focus on the caller, provider, operation, and status to identify automation loops or misconfigured governance tooling.

#### ContainerLogV2
Run `kql/25_containerlogv2_breakdown.kql`.

Use this when AKS or container workloads dominate ingestion. Focus on namespace, pod, container, and log level to isolate chatty services or bad rollout behavior.

#### Perf
Run `kql/26_perf_breakdown.kql`.

Use this when VM or agent-driven performance collection becomes expensive. Focus on object, counter, and instance to identify oversampling or unnecessary counters.

#### InsightsMetrics
Run `kql/27_insightsmetrics_breakdown.kql`.

Use this when Azure Monitor metrics routed into logs become a cost driver. Focus on resource, origin, namespace, and metric name, then revisit whether the metric really needs to be stored in Logs.

### Step 4: Check for Replay or Late Arrival
Run `kql/03_late_arriving_data_check.kql` for any suspicious table spike.

Use this when:
- `Usage` shows a spike but raw queries by `TimeGenerated` do not explain it.
- You suspect backlog replay after outage, connectivity loss, or time skew.

Indicators:
- High `LateArrivalGB` and `LateArrivalPctOfReceived`: delayed or replayed ingestion is likely.
- Large delta between `UsageGB` and `ReceivedGB`: inspect the time window and ingestion path carefully before changing logging settings.

### Step 5: Choose the Fix

| Symptom | Likely cause | Preferred fix |
| --- | --- | --- |
| One app or function dominates `FunctionAppLogs` with repeated errors | Application or runtime misconfiguration | Fix the misconfiguration first, then lower noisy error logging if still excessive |
| One function emits large `Information`/`Debug`/`Trace` volume | Overly verbose app logging | Lower log levels, reduce payload dumps, keep only action-oriented logs |
| One Azure resource dominates `AzureDiagnostics` | Over-collected diagnostic categories | Remove unused categories, split high-volume categories, or transform/drop low-value records |
| One computer dominates `Event` | Host problem or bad event filter | Fix the source host and tighten event collection |
| One `_ResourceId` is hot across several tables | Shared workload issue | Triage resource behavior before changing workspace-wide policy |
| Spike exists in `Usage` but raw event time is old | Backfill or late arrival | Investigate replay path, buffering, connectivity, or clock skew |
| Table stays expensive after app fixes | Collection/retention mismatch | Review transformations, retention, and table plan eligibility |

## Decision Thresholds
- Investigate any table that is at least 10% of workspace ingestion.
- Investigate any consumer that is at least 25% of its table.
- Investigate any 7-day growth of at least 2x versus the prior comparable 7-day window.
- For `FunctionAppLogs`, prioritize any pattern dominated by `Error` or repeated normalized messages.

These are defaults, not laws. Lower them when the workspace is small or when cost sensitivity is high.

## Remediation Guidance

### Application logging fixes
- Remove request/response or payload dumps unless they are explicitly required.
- Lower default log levels in production.
- Keep errors, warnings, and a small number of high-signal state changes.
- In Azure Functions, tune `host.json` or equivalent app-setting overrides to reduce noisy categories while preserving enough telemetry for monitoring.

### Diagnostic settings fixes
- Collect only the log categories that support actual troubleshooting, alerting, audit, or compliance use cases.
- Avoid enabling every category by default.
- Validate whether metrics really need to land in Logs or can remain in Metrics.

### Transformation fixes
- Use ingestion-time transformations only on supported tables.
- Prefer dropping rows or columns that have low decision value.
- Be aware that heavy filtering can still incur processing charges in some cases.

### Retention and table-plan fixes
- Review whether long retention is still justified for high-volume debugging tables.
- Where supported and operationally acceptable, evaluate cheaper table plans.
- Do not use daily caps as the main cost-control mechanism. Use them only as a last-resort budget guardrail.

## Verification Workflow
After any remediation:

1. Wait for enough post-change data to accumulate.
2. Run `kql/90_remediation_verification.kql`.
3. Compare before and after windows using the same duration.
4. Record:
   - total `GB` before vs after
   - daily savings
   - estimated monthly savings
   - top consumer reductions
5. Re-run `kql/00_workspace_usage_by_table.kql` and `kql/01_top3_consumers_per_table.kql` to confirm the issue no longer dominates the workspace.

## Recommended Triage Cadence

### Daily
- Run `00_workspace_usage_by_table.kql` for 1d and 7d.
- Run `01_top3_consumers_per_table.kql` for 1d only when a table crosses threshold or after a verified ingestion anomaly.

### Weekly
- Run `04_active_table_inventory.kql`.
- Run `05_weekly_ingestion_anomalies_by_table.kql`.
- Review top tables over 31d.
- Inspect repeated offenders.
- Verify whether existing fixes stayed effective.
- Refresh your understanding of unfamiliar tables by checking the Microsoft Learn reference pages rather than guessing based on column names.
- Review Log Analytics Workspace Insights for usage anomalies and Query Audit findings.
- Review Azure Advisor recommendations for the workspace.

### After any incident or cost spike
- Complete `triage-notes-template.md`.
- Preserve the evidence query results.
- Add or refine alerts so the same issue is caught earlier next time.

## Cheap-First Query Order
Use this order unless you already know the exact failing table:

1. `04_active_table_inventory.kql`
2. `00_workspace_usage_by_table.kql`
3. `05_weekly_ingestion_anomalies_by_table.kql`
4. `01_top3_consumers_per_table.kql` with `1d`
5. `02_cross_table_resource_hotspots.kql` only if the issue appears resource-centric
6. `30_generic_table_dimension_scan.kql` or a table-specific drill-down
7. `03_late_arriving_data_check.kql` only if ingestion timing looks suspicious
8. `90_remediation_verification.kql` after a fix

This order keeps most investigations on top of `Usage` and short-window raw scans, which is the lowest-cost path supported by Microsoft’s guidance.

## Minimum Evidence Required Before Making a Change
Do not make a collection or logging change until you can point to:
- the expensive table
- the top consumer inside that table
- the dominant dimension inside that table
- a reason the data exists
- the expected operational risk of removing or reducing it

If you cannot answer all five, gather more evidence first.

## Official References
- [Azure Monitor data reference](https://learn.microsoft.com/azure/azure-monitor/reference/)
- [Azure Monitor Resource log / log analytics tables](https://learn.microsoft.com/azure/azure-monitor/reference/tables-index)
- [Analyze usage in a Log Analytics workspace](https://learn.microsoft.com/azure/azure-monitor/logs/analyze-usage)
- [Optimize log queries in Azure Monitor](https://learn.microsoft.com/azure/azure-monitor/logs/query-optimization)
- [Tables that support transformations in Azure Monitor Logs](https://learn.microsoft.com/azure/azure-monitor/logs/tables-feature-support)
- [Tables that support the Basic table plan in Azure Monitor Logs](https://learn.microsoft.com/azure/azure-monitor/logs/basic-logs-azure-tables)
- [Azure Monitor service limits](https://learn.microsoft.com/azure/azure-monitor/fundamentals/service-limits)
