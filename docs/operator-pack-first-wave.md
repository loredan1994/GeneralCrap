# Operator Pack First Wave

## Purpose
This pack answers a small set of recurring platform questions fast without turning the repo into a random snippet dump.

It is intentionally biased toward stable, broadly useful tables and queries:
- `Usage`
- `AzureActivity`
- `Heartbeat`
- `NTANetAnalytics`
- Azure Resource Graph tables such as `Resources`, `PolicyResources`, `AdvisorResources`, and `resourcechanges`

## Execution Targets
- Run `kql/core`, `kql/platform`, and `kql/network` queries in Log Analytics.
- Run `kql/resource-graph` queries in Resource Graph Explorer, `az graph query`, or `Search-AzGraph`.
- Do not paste Resource Graph queries into Log Analytics and expect them to work.

## Recommended First Order
1. `kql/core/06_usage_billable_volume_spike_by_table.kql`
2. `kql/platform/40_activity_failed_control_plane_ops.kql`
3. `kql/resource-graph/60_arg_missing_required_tags.kql`
4. `kql/resource-graph/62_arg_policy_noncompliance_by_assignment.kql`
5. `kql/resource-graph/65_arg_recent_changes_by_actor.kql`

## Query Map

### Azure Monitor
- `kql/core/06_usage_billable_volume_spike_by_table.kql`
  Use for: per-table ingestion spikes above recent baseline.
- `kql/platform/40_activity_failed_control_plane_ops.kql`
  Use for: failing control-plane churn by caller and operation.
- `kql/platform/41_activity_sensitive_iam_policy_lock_changes.kql`
  Use for: RBAC, policy assignment, and lock changes.
- `kql/platform/42_activity_deployment_failure_rate_by_caller.kql`
  Use for: chronic deployment and pipeline failure patterns.
- `kql/platform/43_heartbeat_stale_resources.kql`
  Use for: stale or silently disconnected agents and VMs.

### Network
- `kql/network/50_nta_denied_public_flows.kql`
  Use for: denied public or malicious traffic patterns by rule, target, and port.

### Azure Resource Graph
- `kql/resource-graph/60_arg_missing_required_tags.kql`
  Use for: missing owner, environment, and cost center tags.
- `kql/resource-graph/61_arg_unattached_public_ips.kql`
  Use for: orphaned public IPs.
- `kql/resource-graph/62_arg_policy_noncompliance_by_assignment.kql`
  Use for: policy noncompliance by assignment and location.
- `kql/resource-graph/63_arg_policy_exemptions_expiring_90d.kql`
  Use for: exemptions about to expire.
- `kql/resource-graph/64_arg_advisor_cost_savings_summary.kql`
  Use for: quick cost-savings summaries from Azure Advisor.
- `kql/resource-graph/65_arg_recent_changes_by_actor.kql`
  Use for: recent drift and change attribution.

## Guardrails
- Keep Log Analytics time filters tight and early.
- Project only the columns needed for the question.
- Treat `AzureDiagnostics` as an exception table, not the center of the pack.
- For repeated dashboards and alerts, prefer summary rules or summarized views over repeated wide raw scans.
- Remember that `resourcechanges` is a recent-change tool, not long-term history. Microsoft documents a 14-day retention window for Change Analysis queries.

## When To Extend
Add a new query only if it answers one of these:
- what is failing repeatedly
- what stopped reporting
- what changed
- what is noncompliant
- what is wasting money

If it does not answer one of those cleanly, it probably does not belong in the first-wave pack.
