# KQL Library Map

For repo navigation, start at [README.md](/Users/loredan/Downloads/GeneralCrap/README.md).
For doc-only navigation, use [docs/README.md](/Users/loredan/Downloads/GeneralCrap/docs/README.md).

## Folders
- `core/`: workspace-wide ranking, anomaly detection, cross-table triage, and before/after verification.
- `generic/`: reusable built-in-table probe and drill-down templates for unfamiliar tables.
- `app/`: application and runtime telemetry such as Azure Functions and workspace-based Application Insights.
- `platform/`: Azure platform, control plane, container, and metrics/perf-related tables.
- `guest-os/`: Windows and Linux guest operating system log collection.
- `network/`: network analytics and flow-oriented queries.
- `resource-graph/`: Azure Resource Graph queries for current-state, governance, and recent-change questions.
- `security/`: Microsoft Entra and Windows security audit tables.

## Recommended Starting Points
- Weekly review: `core/04_active_table_inventory.kql`, `core/00_workspace_usage_by_table.kql`, `core/05_weekly_ingestion_anomalies_by_table.kql`
- Fast operator pack in Log Analytics: `core/06_usage_billable_volume_spike_by_table.kql`, `platform/40_activity_failed_control_plane_ops.kql`
- Fast operator pack in Azure Resource Graph: `resource-graph/60_arg_missing_required_tags.kql`
- Cross-table outlier hunt: `core/07_top5_largest_records_per_table.kql`
- 90d/30d hot-table cost view: `core/08_selected_tables_90d_ingestion_and_30d_footprint.kql`
- Unknown built-in table: `generic/31_builtin_table_shape_probe.kql`, then `generic/30_generic_table_dimension_scan.kql`
- Azure Functions: `app/10_functionapplogs_top_dimensions.kql`
- AppTraces and AppExceptions: `app/13_apptraces_top_dimensions.kql`, `app/14_apptraces_repeated_messages.kql`, `app/15_appexceptions_top_dimensions.kql`, `app/16_appexceptions_problem_patterns.kql`
- StorageBlobLogs: `platform/28_storagebloblogs_top_dimensions.kql`, `platform/29_storagebloblogs_callers_by_cost.kql`, `platform/30_storagebloblogs_requesters_by_cost.kql`
- Windows VM events: `guest-os/21_event_breakdown.kql`, or `guest-os/34_windowsevent_breakdown.kql` only if `WindowsEvent` exists
- Event deep dive: `guest-os/40_event_log_level_mix.kql`, `guest-os/38_event_hosts_by_volume.kql`, `guest-os/39_event_id_source_matrix.kql`, `guest-os/35_event_source_breakdown.kql`, `guest-os/36_event_repeated_descriptions.kql`, `guest-os/37_event_trend_by_id.kql`, `guest-os/41_event_payload_outliers.kql`, `guest-os/42_event_low_severity_tuning_candidates.kql`, `guest-os/44_event_spikes_by_signature_vs_baseline.kql`, `guest-os/46_event_security_log_breakdown.kql`
- Windows event path check: `guest-os/43_windows_event_path_check.kql`
- Identity and security: `security/28_signinlogs_breakdown.kql`, `security/29_auditlogs_breakdown.kql`, `security/33_securityevent_breakdown.kql` only if `SecurityEvent` exists

## Query Surfaces
- `core/`, `generic/`, `app/`, `platform/`, `guest-os/`, `security/`, and `network/` run in Log Analytics.
- `resource-graph/` runs in Azure Resource Graph, not in Log Analytics.
- If `Resources`, `PolicyResources`, `AdvisorResources`, or `resourcechanges` fails to resolve, you ran an ARG query in the wrong surface.

## Event Quick Map
- What costs the most in `Event`: `guest-os/21_event_breakdown.kql`
- Is the table mostly high or low severity: `guest-os/40_event_log_level_mix.kql`
- Which host or VM is noisy: `guest-os/38_event_hosts_by_volume.kql`
- Which `EventID` and `Source` combination is noisy: `guest-os/39_event_id_source_matrix.kql`
- Which host, user, or source needs deeper inspection: `guest-os/35_event_source_breakdown.kql`
- Is the same message repeating: `guest-os/36_event_repeated_descriptions.kql`
- Is it bursty or recently spiking: `guest-os/37_event_trend_by_id.kql` or `guest-os/44_event_spikes_by_signature_vs_baseline.kql`
- Are a few records unusually large: `guest-os/41_event_payload_outliers.kql`
- Are Security log events landing in `Event`: `guest-os/46_event_security_log_breakdown.kql`
- What should we consider filtering: `guest-os/42_event_low_severity_tuning_candidates.kql`
- If the workspace uses `WindowsEvent` instead, switch to `guest-os/34_windowsevent_breakdown.kql`
- If you are unsure which Windows-event table is active, run `guest-os/43_windows_event_path_check.kql`

## Naming
- `00-05`: cheap-first workspace and weekly triage flow
- `06-09`: focused cost and footprint helpers
- `10-19`: app and runtime drill-downs
- `20-29`: platform, guest OS, and identity drill-downs
- `30-37`: generic built-in-table helpers
- `38-49`: extended `Event` and guest OS drill-downs
- `50-69`: network and Azure Resource Graph operator-pack queries
- `90+`: remediation verification

## First Wave To Validate Next
- `core/06_usage_billable_volume_spike_by_table.kql`
- `platform/40_activity_failed_control_plane_ops.kql`
- `resource-graph/60_arg_missing_required_tags.kql`
- `resource-graph/62_arg_policy_noncompliance_by_assignment.kql`
- `resource-graph/65_arg_recent_changes_by_actor.kql`
