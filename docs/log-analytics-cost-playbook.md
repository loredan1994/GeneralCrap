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
- `kql/core/00_workspace_usage_by_table.kql`: Workspace table ranking for 1d, 7d, and 31d windows.
- `kql/core/01_top3_consumers_per_table.kql`: Cross-table top-3-per-table consumer ranking using a short raw-data window.
- `kql/core/02_cross_table_resource_hotspots.kql`: Recent Azure resource hotspots across all tables.
- `kql/core/03_late_arriving_data_check.kql`: Ingestion-day vs event-time analysis for replay and delayed ingestion.
- `kql/core/04_active_table_inventory.kql`: Cheap active-table inventory from `Usage`, including billable share and likely doc lookup URLs.
- `kql/core/05_weekly_ingestion_anomalies_by_table.kql`: Automation-safe weekly anomaly scan based on `Usage`.
- `kql/core/07_top5_largest_records_per_table.kql`: Manual-only top 5 largest billable records per table for short-window outlier detection.
- `kql/app/10_functionapplogs_top_dimensions.kql`: `FunctionAppLogs` breakdown by app, function, category, and level.
- `kql/app/11_functionapplogs_repeated_messages.kql`: Normalized repeated message signatures for noisy Functions logs.
- `kql/app/12_functionapplogs_execution_outcomes.kql`: Function execution outcomes, failures, and duration.
- `kql/platform/20_azurediagnostics_breakdown.kql`: `AzureDiagnostics` breakdown by provider, type, category, operation, and resource.
- `kql/guest-os/21_event_breakdown.kql`: `Event` breakdown by event log, ID, level, and computer.
- `kql/guest-os/35_event_source_breakdown.kql`: `Event` breakdown by source, user, host, and event ID.
- `kql/guest-os/36_event_repeated_descriptions.kql`: Normalized repeated `Event` descriptions to catch chatty guest issues.
- `kql/guest-os/37_event_trend_by_id.kql`: `Event` bursts over time by event ID, source, and computer.
- `kql/guest-os/38_event_hosts_by_volume.kql`: `Event` breakdown by host and log to find noisy machines.
- `kql/guest-os/39_event_id_source_matrix.kql`: `Event` breakdown by event log, ID, and source across the estate.
- `kql/guest-os/40_event_log_level_mix.kql`: `Event` severity mix by log for collection-tuning decisions.
- `kql/guest-os/41_event_payload_outliers.kql`: `Event` payload heaviness by event log, ID, and source.
- `kql/guest-os/42_event_low_severity_tuning_candidates.kql`: High-volume `Information` or `Verbose` `Event` records that may be collection candidates.
- `kql/guest-os/43_windows_event_path_check.kql`: Cheap `Usage`-based check for whether the workspace is using `Event`, `WindowsEvent`, or `SecurityEvent`.
- `kql/guest-os/44_event_spikes_by_signature_vs_baseline.kql`: `Event` signature spikes versus recent baseline.
- `kql/guest-os/46_event_security_log_breakdown.kql`: Windows Security log events when they land in `Event`.
- `kql/guest-os/22_syslog_breakdown.kql`: `Syslog` breakdown by computer, facility, severity, and process.
- `kql/app/23_applicationinsights_builtin_breakdown.kql`: Workspace-based Application Insights and OTel table breakdown.
- `kql/platform/24_azureactivity_breakdown.kql`: `AzureActivity` breakdown by caller, provider, operation, and status.
- `kql/platform/25_containerlogv2_breakdown.kql`: `ContainerLogV2` breakdown by namespace, pod, container, and log level.
- `kql/platform/26_perf_breakdown.kql`: `Perf` breakdown by computer, object, counter, and instance.
- `kql/platform/27_insightsmetrics_breakdown.kql`: `InsightsMetrics` breakdown by resource, origin, namespace, and metric name.
- `kql/security/28_signinlogs_breakdown.kql`: `SigninLogs` breakdown by app, resource, user, result, and conditional access status.
- `kql/security/29_auditlogs_breakdown.kql`: `AuditLogs` breakdown by activity, operation type, initiator, and result.
- `kql/generic/30_generic_table_dimension_scan.kql`: Generic dimension scan for any single built-in table selected after table ranking.
- `kql/generic/31_builtin_table_shape_probe.kql`: Probe for populated common dimensions in any built-in table before building a custom drill-down.
- `kql/generic/32_builtin_table_drilldown_template.kql`: Reusable short-window template for table-specific drill-downs.
- `kql/security/33_securityevent_breakdown.kql`: `SecurityEvent` breakdown by event ID, computer, activity, and target account, only if `SecurityEvent` is active in the workspace.
- `kql/guest-os/34_windowsevent_breakdown.kql`: `WindowsEvent` breakdown by channel, provider, event ID, and computer, only if `WindowsEvent` is active in the workspace.
- `kql/core/90_remediation_verification.kql`: Before/after comparison and savings estimate after a fix.
- `docs/microsoft-learn-reference-map.md`: Official Microsoft source map for built-in table discovery, per-table attributes, and optimization guidance.
- `docs/builtin-table-family-map.md`: Quick family map for unfamiliar built-in tables.
- `docs/sre-weekly-runbook.md`: Short weekly operating runbook for SRE teams.
- `docs/triage-notes-template.md`: Investigation worksheet for one suspected issue.
- `docs/weekly-review-template.md`: Short weekly review artifact for recurring handoff.
- `docs/event-table-drilldown-guide.md`: Short operator guide for deeper `Event` investigations.
- `kql/README.md`: Folder map and quick starting points for the KQL library.
- `docs/operator-pack-first-wave.md`: Operator-grade Azure Monitor and Resource Graph query pack for platform, governance, and drift questions.

## Folder Layout
- `kql/core`: workspace-level and weekly-safe queries plus remediation verification.
- `kql/generic`: built-in-table probe and reusable drill-down templates.
- `kql/app`: app/runtime telemetry such as Functions and Application Insights.
- `kql/platform`: Azure platform, control plane, container, and metrics/perf tables.
- `kql/guest-os`: Windows and Linux guest log collection.
- `kql/network`: network-focused queries such as Traffic Analytics.
- `kql/resource-graph`: Azure Resource Graph governance, inventory, and recent-change queries.
- `kql/security`: identity and security-focused tables.

## Adjacent Operator Pack
This repo now also includes a small operator-grade pack for stable Azure Monitor and Azure Resource Graph questions that are adjacent to cost triage but not limited to ingestion cost.

See `docs/operator-pack-first-wave.md` for:
- control-plane failure and IAM change queries
- heartbeat and stale-agent checks
- `NTANetAnalytics` network hygiene
- Resource Graph governance, advisor, and recent-change queries

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
- `kql/core/04_active_table_inventory.kql`
- `kql/core/00_workspace_usage_by_table.kql`

This is the cheap path. Start from active tables, not the entire built-in catalog.
Treat `LikelyTableClass` in `kql/core/04_active_table_inventory.kql` as a heuristic only. Active billable tables can include custom tables, so confirm built-in status against Microsoft Learn before using the built-in-table path.

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
- Keep returned result sets small. Azure Monitor enforces record-count, size, concurrency, and timeout limits.

### Automation-Safe Versus Manual-Only
Use these in scheduled reviews, workbooks, or weekly jobs:
- `kql/core/04_active_table_inventory.kql`
- `kql/core/00_workspace_usage_by_table.kql`
- `kql/core/05_weekly_ingestion_anomalies_by_table.kql`

Use these only for manual triage after a hot table or hot resource is already known:
- `kql/core/01_top3_consumers_per_table.kql`
- `kql/core/02_cross_table_resource_hotspots.kql`
- `kql/core/03_late_arriving_data_check.kql`
- `kql/core/07_top5_largest_records_per_table.kql`
- `kql/generic/30_generic_table_dimension_scan.kql`
- the table-specific drill-down queries

### Step 1: Find Expensive Tables
Run `kql/core/04_active_table_inventory.kql` first, then `kql/core/00_workspace_usage_by_table.kql`, then `kql/core/05_weekly_ingestion_anomalies_by_table.kql`.

Use it to answer:
- Which active billable tables are present in this workspace, and which are likely built-in?
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
- The table is already known to be operationally noisy, such as `FunctionAppLogs`, `AzureDiagnostics`, `Event`, `WindowsEvent`, `SecurityEvent`, `SigninLogs`, or `AuditLogs`.

### Step 2: Find the Top 3 Consumers Inside Each Table
Run `kql/core/01_top3_consumers_per_table.kql` with a `1d` or `3d` lookback.

This is intentionally a short-window query because Microsoft explicitly warns that `find` across many tables is resource-intensive. Use it once per incident to find where to focus, not as a general dashboard query.

Do not try to turn this query into a targeted single-table view. Once you know the hot table, switch to `kql/generic/30_generic_table_dimension_scan.kql` or a table-specific drill-down so the scan scope becomes truly narrow.

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
- If the same `_ResourceId` appears across multiple tables, pivot to `kql/core/02_cross_table_resource_hotspots.kql`.

### When You Need Largest Individual Records
Run `kql/core/07_top5_largest_records_per_table.kql`.

Use this when:
- table totals look high but you suspect a small number of oversized events
- one app may be dumping payloads, stack traces, or verbose structured blobs
- you want a quick outlier view before building a table-specific drill-down

Keep the window short. This query is for manual investigation only.

### Step 3: Drill Down by Table Type

#### Azure resource logs in resource-specific tables
Do not assume all Azure resource logs land in `AzureDiagnostics`. Many services now emit to resource-specific built-in tables.

Use this sequence:
1. Use `kql/core/04_active_table_inventory.kql` and `kql/core/00_workspace_usage_by_table.kql` to identify the hot resource-specific table.
2. Open the official table doc for that table.
3. Run `kql/generic/30_generic_table_dimension_scan.kql` if there is no dedicated query yet.
4. Review the resource's diagnostic settings and category groups before changing collection.

#### Any built-in table not already covered
Use this sequence:
1. Open the Microsoft Learn table reference page for the table.
2. Check the table columns, sample queries, and table attributes.
3. Run `kql/generic/31_builtin_table_shape_probe.kql` to see which common dimensions are actually populated.
4. Run `kql/generic/30_generic_table_dimension_scan.kql` against that table.
5. If the generic scan identifies a dominant dimension, build or adapt a table-specific query only for that table.

Use this path for unknown or newly encountered built-in tables so the workflow remains grounded in official schema and remains cheap.

#### FunctionAppLogs
Use this sequence:
1. `kql/app/10_functionapplogs_top_dimensions.kql`
2. `kql/app/11_functionapplogs_repeated_messages.kql`
3. `kql/app/12_functionapplogs_execution_outcomes.kql`

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
Run `kql/platform/20_azurediagnostics_breakdown.kql`.

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
Run `kql/guest-os/21_event_breakdown.kql`.

Use this for the classic `Event` table, typically when Windows events are collected into the legacy event schema with `EventLog`, `Source`, and `RenderedDescription`.

What you are looking for:
- Specific `EventID` values that are unexpectedly frequent.
- One `Computer` or server class dominating the table.
- One `EventLog` and `EventLevelName` combination explaining most of the volume.

Typical fixes:
- Filter the event set at collection time if those IDs are not operationally useful.
- Fix the host or application generating the repeated events.
- Remove redundant event sources from collection rules.

For deeper Event investigations:
1. Run `kql/guest-os/40_event_log_level_mix.kql` to see whether the table is mostly `Error` and `Warning`, or mostly `Information` and `Verbose`.
2. Run `kql/guest-os/38_event_hosts_by_volume.kql` to identify the noisiest computers and whether one host dominates one log.
3. Run `kql/guest-os/39_event_id_source_matrix.kql` to identify the dominant event ID and source combinations across the estate.
4. Run `kql/guest-os/35_event_source_breakdown.kql` to see which source, user, or computer is producing the cost.
5. Run `kql/guest-os/37_event_trend_by_id.kql` or `kql/guest-os/44_event_spikes_by_signature_vs_baseline.kql` to see whether the issue is bursty, newly spiking, or steady-state.
6. Run `kql/guest-os/36_event_repeated_descriptions.kql` to identify repeated event signatures after normalizing numbers and GUIDs.
7. Run `kql/guest-os/41_event_payload_outliers.kql` when the issue looks like oversized descriptions, XML payloads, or unusually heavy records.
8. Run `kql/guest-os/46_event_security_log_breakdown.kql` if the hot path is actually the classic Security log inside `Event`.
9. Run `kql/guest-os/42_event_low_severity_tuning_candidates.kql` to surface `Information` and `Verbose` records that may be better filtered in the DCR.

If `Event` is dominated by `Information` or `Verbose` records, Microsoft’s current VM guidance is to start with `Critical`, `Error`, and `Warning` for the `System` and `Application` logs, and add `Information` only when you need it for troubleshooting or trend analysis.

#### WindowsEvent
Run `kql/guest-os/34_windowsevent_breakdown.kql`.

Use this for AMA or DCR-style Windows event collection where the table exposes `Channel`, `Provider`, and parsed event payload fields.
Do this only if `WindowsEvent` is an active table in the workspace. If the query would otherwise fail or return nothing in a workspace that only uses the classic schema, switch to `Event` first.

What you are looking for:
- One `Channel` or `Provider` dominating ingestion.
- Specific `EventID` values that are far more frequent than expected.
- One `Computer` or VM class dominating the table.

Typical fixes:
- Tighten DCR event selection by channel, provider, or event ID.
- Remove noisy providers or low-value event streams.
- Fix the emitting workload if one provider is generating repetitive errors or informational spam.

If `WindowsEvent` returns no rows even on a wide window, run `kql/guest-os/43_windows_event_path_check.kql`. In many workspaces that is normal because Windows events are landing in `Event` or `SecurityEvent` instead of `WindowsEvent`.

#### SecurityEvent
Run `kql/security/33_securityevent_breakdown.kql`.

Use this for Windows security audit events, especially logons, account management, process execution, and policy change activity.
Do this only if `SecurityEvent` is an active table in the workspace. If the workspace does not have `SecurityEvent`, check whether the same security events are routed to `Event` instead or whether the security-specific connector path is not enabled.

What you are looking for:
- One `EventID` dominating the table, such as repetitive logon success or failure events.
- One `Computer` or server tier generating disproportionate security audit volume.
- One `LogonTypeName`, `Activity`, or account dimension explaining the cost.

Typical fixes:
- Review Windows audit policy scope before collecting more categories than needed.
- Fix hosts or workloads causing repeated failed logons or security churn.
- Reduce unnecessary security event forwarding if it is not required for operations or compliance.

#### SigninLogs
Run `kql/security/28_signinlogs_breakdown.kql`.

Use this for Microsoft Entra sign-in activity.

What you are looking for:
- One `AppDisplayName` or `ResourceDisplayName` dominating ingestion.
- One `UserPrincipalName` or automation identity creating unusual volume.
- Repeated failure-heavy `ResultType` or `ResultDescription` patterns.

Typical fixes:
- Investigate misconfigured clients, broken automation, or repeated auth failures.
- Review whether all sign-in categories must stay in Analytics if the table options allow a different plan.
- Use the per-table attributes page before considering Basic Logs or ingestion-time transformations.

#### AuditLogs
Run `kql/security/29_auditlogs_breakdown.kql`.

Use this for Microsoft Entra audit activity such as user, group, app, and directory changes.

What you are looking for:
- One `ActivityDisplayName` or `AADOperationType` dominating the table.
- One initiator or automation identity driving repeated changes.
- Large bursts of success or failure events tied to one workflow.

Typical fixes:
- Fix automation loops or governance tooling that repeatedly updates objects.
- Review audit routing and retention decisions if the volume is operationally low-value but required for compliance.
- Validate transformation and table-plan support on the Microsoft Learn table page before changing ingestion strategy.

#### Syslog
Run `kql/guest-os/22_syslog_breakdown.kql`.

What you are looking for:
- One process or facility dominating ingestion.
- One host class sending unusually noisy messages.
- Severity levels that are too verbose for steady-state operations.

Typical fixes:
- Tighten Linux syslog collection.
- Reduce application verbosity on the emitting host.
- Remove low-value facilities or process streams from collection.

#### Workspace-based Application Insights and OpenTelemetry tables
Run `kql/app/23_applicationinsights_builtin_breakdown.kql`.

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
Run `kql/platform/24_azureactivity_breakdown.kql`.

Use this when control plane activity appears unexpectedly large or bursty. Focus on the caller, provider, operation, and status to identify automation loops or misconfigured governance tooling.

#### ContainerLogV2
Run `kql/platform/25_containerlogv2_breakdown.kql`.

Use this when AKS or container workloads dominate ingestion. Focus on namespace, pod, container, and log level to isolate chatty services or bad rollout behavior.

#### Perf
Run `kql/platform/26_perf_breakdown.kql`.

Use this when VM or agent-driven performance collection becomes expensive. Focus on object, counter, and instance to identify oversampling or unnecessary counters.

#### InsightsMetrics
Run `kql/platform/27_insightsmetrics_breakdown.kql`.

Use this when Azure Monitor metrics routed into logs become a cost driver. Focus on resource, origin, namespace, and metric name, then revisit whether the metric really needs to be stored in Logs.

### Step 4: Check for Replay or Late Arrival
Run `kql/core/03_late_arriving_data_check.kql` for any suspicious table spike.

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
| One provider or channel dominates `WindowsEvent` | AMA or DCR event selection too broad or one noisy provider | Tighten DCR event criteria by channel, provider, or event ID |
| One event ID or logon type dominates `SecurityEvent` | Audit policy too broad or repeated auth churn | Review audit policy scope and fix the failing or looping source |
| One app, resource, or identity dominates `SigninLogs` | Broken client, auth storm, or noisy automation | Fix the caller and review sign-in log plan and retention options |
| One activity or initiator dominates `AuditLogs` | Automation loop or governance process churn | Fix the initiator workflow and review audit data routing |
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
2. Run `kql/core/90_remediation_verification.kql`.
3. Compare before and after windows using the same duration.
4. Record:
   - total `GB` before vs after
   - daily savings
   - estimated monthly savings
   - top consumer reductions
5. Re-run `kql/core/00_workspace_usage_by_table.kql` and `kql/core/01_top3_consumers_per_table.kql` to confirm the issue no longer dominates the workspace.

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

## Warning And Limit Hygiene
- If a query shows excessive-resource warnings, reduce time range before changing the query logic.
- If Query Details shows queue time, review concurrent workbook, dashboard, or scheduled usage before assuming the query text is the only problem.
- If a query is intended for recurring use, keep it on `Usage` where possible and avoid cross-table `find`.
- If the table is Basic or Auxiliary, remember that recurring dashboard refreshes still incur query cost.
- Keep workspace scope narrow. Cross-region and multi-workspace queries are more likely to warn or throttle.
- Treat 100 seconds total CPU as the point where Microsoft considers a query resource-intensive, and 1,000 seconds as unacceptable for this package.
- Before scheduling a new query, verify it in Query Details and reject it if it repeatedly appears in Workspace Insights Query Audit as slow or resource-intensive.

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
6. `31_builtin_table_shape_probe.kql` for unfamiliar built-in tables
7. `30_generic_table_dimension_scan.kql` or `32_builtin_table_drilldown_template.kql`
8. `03_late_arriving_data_check.kql` only if ingestion timing looks suspicious
9. `90_remediation_verification.kql` after a fix

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
