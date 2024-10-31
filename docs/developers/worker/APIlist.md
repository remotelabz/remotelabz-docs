#API commands
Remotelabz workers also use a REST API to communicate certain data to the front 
## Get
Here a list of this API's get commands:

 * `apiGetCPUUsage()` : return cpu usage in percentage
 * `apiGetDiskUsage()` : return disk usage in percentage
 * `apiGetOldMemUsage()` : RAM usage in percentage as cache or data or -1 if not valid (Deprecated)
 * `apiGetMemUsage()` : RAM usage in percentage as cache or data
 * `apiGetRunningWrappers()` : Returns the Wrappers running states for iol,dynamips,qemu,docker,vcps.The results are returned into an array of int. 
 * `apiGetSwapUsage()`: Return the SWAP usage percentage.

## Set
Here a list of this API's set commands:

 * `apiSetUksm($processor)` : Set UKSM status for processor `$processor`.
 * `apiSetKsm($processor)` : Set KSM status for processor `$processor`.
 * `apiSetCPULimit($processor)` : Set CPULimit for processor `$processor`.
