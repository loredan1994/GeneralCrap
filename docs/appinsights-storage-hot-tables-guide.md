# AppInsights And Storage Hot Tables Guide

## Goal
Use this guide when `AppExceptions`, `AppTraces`, or `StorageBlobLogs` are among the largest tables and you want to understand ingestion over the last 90 days plus the current 30-day visible footprint.

## Start Here
1. Run [08_selected_tables_90d_ingestion_and_30d_footprint.kql](/Users/loredan/Downloads/GeneralCrap/kql/core/08_selected_tables_90d_ingestion_and_30d_footprint.kql).
2. Use the table-specific ladder below for the table that dominates.

The `Usage` query gives you the 30-day and 90-day accounting. The raw table drill-downs default to 7 days or less on purpose so they stay safer on large workspaces.

## What The Cost Query Shows
- `Ingested90dGB`: billable ingestion over the last 90 days from `Usage`
- `Ingested30dGB`: billable ingestion over the last 30 days from `Usage`
- `LatestCompletedDayGB`: last full day of ingestion from `Usage`
- `Retained30dGB`: current 30-day visible footprint from raw billable records
- `Retained30dRows`: current 30-day visible row count from raw billable records

Treat `Retained30dGB` as a visibility metric, not as a billing metric.
Use `Ingested90dGB`, `Ingested30dGB`, and `LatestCompletedDayGB` for ingestion-cost decisions.
Check actual table-level retention settings before you assume `AppExceptions` or `AppTraces` follow the workspace default. These tables can be configured differently from the workspace baseline.

## AppTraces
1. Run [13_apptraces_top_dimensions.kql](/Users/loredan/Downloads/GeneralCrap/kql/app/13_apptraces_top_dimensions.kql)
2. Run [14_apptraces_repeated_messages.kql](/Users/loredan/Downloads/GeneralCrap/kql/app/14_apptraces_repeated_messages.kql) after narrowing by app role or operation
3. If you need a generic Application Insights fallback, run [23_applicationinsights_builtin_breakdown.kql](/Users/loredan/Downloads/GeneralCrap/kql/app/23_applicationinsights_builtin_breakdown.kql) with `TargetTable = "AppTraces"`

## AppExceptions
1. Run [15_appexceptions_top_dimensions.kql](/Users/loredan/Downloads/GeneralCrap/kql/app/15_appexceptions_top_dimensions.kql)
2. Run [16_appexceptions_problem_patterns.kql](/Users/loredan/Downloads/GeneralCrap/kql/app/16_appexceptions_problem_patterns.kql)
3. If you need a generic Application Insights fallback, run [23_applicationinsights_builtin_breakdown.kql](/Users/loredan/Downloads/GeneralCrap/kql/app/23_applicationinsights_builtin_breakdown.kql) with `TargetTable = "AppExceptions"`

## StorageBlobLogs
1. Run [28_storagebloblogs_top_dimensions.kql](/Users/loredan/Downloads/GeneralCrap/kql/platform/28_storagebloblogs_top_dimensions.kql)
2. Run [29_storagebloblogs_callers_by_cost.kql](/Users/loredan/Downloads/GeneralCrap/kql/platform/29_storagebloblogs_callers_by_cost.kql) for IP and network-path hotspots
3. Run [30_storagebloblogs_requesters_by_cost.kql](/Users/loredan/Downloads/GeneralCrap/kql/platform/30_storagebloblogs_requesters_by_cost.kql) for workload identity hotspots

## What To Look For
- `AppTraces`: one `AppRoleName`, `OperationName`, or severity dominating the table, or repeated message signatures
- `AppExceptions`: one `ProblemId`, `ExceptionType`, or app role dominating the table, or large exception details inflating record size
- `StorageBlobLogs`: one storage account, operation, caller IP, requester identity, auth type, or status pattern dominating the table

## Cost Levers
- `AppTraces`: lower trace verbosity, remove repeated low-value traces, or consider Basic Logs and ingestion-time transformations if the access pattern fits
- `AppExceptions`: fix exception floods first; this table supports ingestion-time transformations but not Basic Logs
- `StorageBlobLogs`: look for one requester, app, auth pattern, or noisy operation; this table supports Basic Logs and ingestion-time transformations
