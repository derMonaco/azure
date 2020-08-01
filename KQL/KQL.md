## KQL Query to report non-reporting VM'S.

let Location = "-WE-";
let Timeframe = 15m;
let VMname = "vm_naming convention"
Heartbeat
| summarize LastCall = max(TimeGenerated) by Resource
| where LastCall < ago(Timeframe)
| project LastCall, Resource
|join kind = fullouter
(
    AzureActivity
    | where ResourceProvider == "Microsoft.Compute"
    | where Resource contains VMname and ResourceGroup contains Location
    | summarize LastOps = max(TimeGenerated) by OperationName, Resource, ResourceGroup
    | summarize argmax(LastOps, *) by Resource
    | project Resource, max_LastOps, max_LastOps_OperationName
)on Resource
| extend vm_status = iif(max_LastOps_OperationName contains "DEALLOCATE" or max_LastOps_OperationName contains "Delete Virtual Machine", "down", "running")
| where vm_status == "running"
| where isnotempty(LastCall)
