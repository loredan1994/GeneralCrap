# Docs Map

Use this folder as the documentation index. If you want the repo-level entry point, start at [README.md](/Users/loredan/Downloads/GeneralCrap/README.md).

## Which Doc To Open
| Need | Open |
| --- | --- |
| Weekly SRE review flow | [sre-weekly-runbook.md](/Users/loredan/Downloads/GeneralCrap/docs/sre-weekly-runbook.md) |
| Full end-to-end cost triage playbook | [log-analytics-cost-playbook.md](/Users/loredan/Downloads/GeneralCrap/docs/log-analytics-cost-playbook.md) |
| `Event` table deep dive | [event-table-drilldown-guide.md](/Users/loredan/Downloads/GeneralCrap/docs/event-table-drilldown-guide.md) |
| App Insights and Storage hot tables | [appinsights-storage-hot-tables-guide.md](/Users/loredan/Downloads/GeneralCrap/docs/appinsights-storage-hot-tables-guide.md) |
| Unknown built-in table routing | [builtin-table-family-map.md](/Users/loredan/Downloads/GeneralCrap/docs/builtin-table-family-map.md) |
| Official Microsoft source links | [microsoft-learn-reference-map.md](/Users/loredan/Downloads/GeneralCrap/docs/microsoft-learn-reference-map.md) |
| Platform and governance operator pack | [operator-pack-first-wave.md](/Users/loredan/Downloads/GeneralCrap/docs/operator-pack-first-wave.md) |
| Weekly review capture template | [weekly-review-template.md](/Users/loredan/Downloads/GeneralCrap/docs/weekly-review-template.md) |
| One-incident notes template | [triage-notes-template.md](/Users/loredan/Downloads/GeneralCrap/docs/triage-notes-template.md) |

## Recommended Reading Order
1. Open [event-table-drilldown-guide.md](/Users/loredan/Downloads/GeneralCrap/docs/event-table-drilldown-guide.md) immediately if `Event` is the hot table.
2. Open [appinsights-storage-hot-tables-guide.md](/Users/loredan/Downloads/GeneralCrap/docs/appinsights-storage-hot-tables-guide.md) immediately if `AppExceptions`, `AppTraces`, or `StorageBlobLogs` is the hot table.
3. Otherwise open [sre-weekly-runbook.md](/Users/loredan/Downloads/GeneralCrap/docs/sre-weekly-runbook.md) for routine review.
4. Open [log-analytics-cost-playbook.md](/Users/loredan/Downloads/GeneralCrap/docs/log-analytics-cost-playbook.md) once a hot table or consumer needs investigation.
5. Open [builtin-table-family-map.md](/Users/loredan/Downloads/GeneralCrap/docs/builtin-table-family-map.md) when the table is unfamiliar.

## Surface Reminder
- The docs mostly describe Log Analytics workflows.
- Queries under `kql/resource-graph/` are different: run them in Azure Resource Graph, not in the Log Analytics query blade.
