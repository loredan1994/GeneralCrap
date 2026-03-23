# Event Table Drill-Down Guide

## Goal
Use the classic `Event` table to identify which Windows events are driving ingestion cost, which hosts or sources are noisy, whether the same signature keeps repeating, and which collection changes are worth considering.

## Sequence
1. Start with `kql/guest-os/21_event_breakdown.kql`.
   This tells you which `EventLog`, `EventID`, `EventLevelName`, and `Computer` combinations are costing the most.
2. Run `kql/guest-os/40_event_log_level_mix.kql`.
   This tells you whether the table is dominated by `Error` and `Warning`, or by `Information` and `Verbose`.
3. Run `kql/guest-os/38_event_hosts_by_volume.kql`.
   This tells you which hosts or VM resources dominate the table.
4. Run `kql/guest-os/39_event_id_source_matrix.kql` or `kql/guest-os/35_event_source_breakdown.kql`.
   This tells you which `Source`, `EventID`, `EventCategory`, `UserName`, or host dimension is really behind the noise.
5. Run `kql/guest-os/37_event_trend_by_id.kql` or `kql/guest-os/44_event_spikes_by_signature_vs_baseline.kql`.
   This tells you whether the issue is bursty, recent, or steady-state.
6. Run `kql/guest-os/36_event_repeated_descriptions.kql` only after you have narrowed the scope.
   This is the message-normalization step. Keep it short-window and filtered on larger workspaces.
7. Run `kql/guest-os/41_event_payload_outliers.kql` if record size looks suspicious.
   This tells you whether the cost is row-count driven or inflated by large descriptions, raw event data, or parameter XML.
8. Run `kql/guest-os/46_event_security_log_breakdown.kql` if the hot path is the classic Security log inside `Event`.
9. Run `kql/guest-os/42_event_low_severity_tuning_candidates.kql` when preparing DCR or XPath filtering discussions.

## What To Record
- `EventLog`
- `EventID`
- `Source`
- `Computer`
- whether the issue is bursty or steady-state
- whether the same normalized signature repeats
- whether the cost is row-count driven or payload-size driven

## Likely Fixes
- Fix the emitting workload if one source is producing repetitive errors or warnings.
- Reduce collection scope by log, severity, source, or event ID when the events are low value.
- Tighten Windows event selection rules when needed instead of relying only on downstream retention changes.

## Event Family Note
- Use `Event` for the classic Windows event path.
- Use `WindowsEvent` only if that table exists and is populated in the workspace.
- Use `SecurityEvent` only if the workspace uses the security-specific collection path.
