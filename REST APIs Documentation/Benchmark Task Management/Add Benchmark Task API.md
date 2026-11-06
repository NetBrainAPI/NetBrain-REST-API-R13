
# Benchmark API Design

## ***POST*** /V1/CMDB/Benchmark/Tasks
This API call is used to add a benchmark task to a domain. 

Note that, as the key, task name should be unique system wide. 

The option for data to be retrieved in this task is 'Built-in Live Data' which the user can see from UI.

<b>Note</b>: A task created by this API has the following settings by default, which can be checked in the *Summary* tab of the task in the UI:
- **Built-in Live Data** retrieves the **Configuration File** only. Other built-in data such as Route, ARP, MAC, NDP and STP tables is not retrieved unless it is listed in `retrieveData`.
- In **Additional Operations after Benchmark > System Operations**, **Fully synchronize the data from MongoDB to Elasticsearch** is enabled. This API has no parameter to disable it.

## Detail Information

> **Title** : Add Benchmark Task API<br>

> **Version** : 28/10/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Benchmark/Tasks	

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

 ## Request body(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|taskName* | string  | The name of the task.  |
|description | string  | The description of the task. This field is optional.  |
|startDate* | string  | The date when the task starts to run. The standard time format is required, for example, '2017-07-13', '2017/07/13'. This field is optional. Current date will be used by default.  |
|schedule* | object  | The schedule to run the task. The following sub parameters are included in this object: <br>▪ frequency* (string) - the frequency to run the task. This field is required and includes ”once”, “hourly”,” daily”, “weekly” and “monthly” options.<br>▪ interval(string) - the interval to run the task (optional). This field is only valid for “hourly”,” daily”, and “weekly” options and the default value is 1, such as every 1 hour, 1 week.<br>▪ startTime* (string) - the time to run the task. This field is required and startTime should be in format: ["HH:mm:ss"], if you put date time format such as "2018/04/04 19:20:20 ", "19:20:20" will be used and the date part "2018/04/04" will be ignored.<br> **Note:** Set the time according to your IIS server time zone since the time zone of your ISS server rather than your physical time zone is adopted by the benchmark task.<br>▪ weekday(integer) - the day of the week to run the task. This field is optional and only valid when the frequency is weekly.  0 stands for Sunday, 6 for Saturday and 1-5 for Monday to Friday respectively.<br>▪ dayOfMonth(integer) - which day of a month to run the task. This field is optional and only valid when the frequency is monthly. The default is 1.<br>▪ Months(integer) - which month to run the task. This field is optional and only valid when the frequency is monthly. The default is all 12 months.|
|deviceScope* | string  |  The devices included in this task.   |
|deviceScope.scopeType | string  | scope type options:<br>"all" for all devices of current domain, deviceScope.scopes will be ignored if this field is set to "all";<br>"deviceGroup" for specified group, if set deviceScope.scopes would be list of full path to device groups, such as \["Public/devgrp1", "Private/devgrp2", "System/devgrp3"\];<br>"site" for a particular site. if set deviceScope.scopes would be list of full path to sites, for example: \["My Networks/US/MA/Boston", "My Networks/US/ME/Portland"\]  |
|deviceScope.scopes | list of string  | ignored if deviceScope.scopeType is set to "all";<br>full path to device groups, such as \["Public/devgrp1", "Private/devgrp2", "System/devgrp3"\] if deviceScope.scopeType is set to "deviceGroup";<br>&nbsp;&nbsp;&nbsp;&nbsp;Use the Public/, Private/ or System/ prefix. The names returned by the [Get Device Group API](../Device%20Group%20Management/Get%20Device%20Group%20API.md) start with "Shared Device Groups/", "My Device Groups/" or "System Device Groups/" (type 0, 1, 2); replace that first part with Public, Private or System. For example, "My Device Groups/F5" is "Private/F5". Otherwise the request fails with 791006 "device group prefix: ... does not exist. options: Public, Private, System".;<br>full path to sites, such as \["My Networks/US/MA/Boston", "My Networks/US/ME/Portland"\] if deviceScope.scopeType is set to "site";  |
|limitRunMins | string  | The time used to retrieve the data. When it reaches the specified time, the task will stop retrieving more data. This field is optional.  |
|cliCommands | string  | The customized CLI commands to retrieve data (for example, ["show version", "show arp"]. This field is optional.  |
|retrieveData | list of string  | The built-in live data to retrieve. This corresponds to *Built-in Live Data* in the *Batch Definition* tab of the task in the UI. This field is optional. If omitted, only the Configuration File is retrieved. Supported values:<br>▪ "config" - Configuration File<br>▪ "routingTable" - Route Table<br>▪ "arpTable" - ARP Table<br>▪ "macTable" - MAC Table<br>▪ "cdpTable" - NDP Table<br>▪ "stpTable" - STP Table<br>▪ "inventoryInfo" - Inventory Information of Device/Interface/Module<br>▪ The name of an NCT table, exactly as shown in the *Built-in Data* tab of the UI, for example "BGP Neighbors Table" or "Xconnect Table". NCT tables are shown in the task summary as a count, e.g. "NCT Table: 2".<br>**Note:** The values are not validated. An unsupported value is silently ignored and the request still returns success. If none of the values is supported, the task retrieves no built-in live data at all, not even the Configuration File.  |
|isBuildIPv4L3Topo | bool  | Determine whether to build IPv4 L3 topology. This field is optional and the default value is false.  |
|isBuildIPv6L3Topo | bool  | Determine whether to build IPv6 L3 topology. This field is optional and the default value is false.  |
|isBuildL2Topo | bool  | Determine whether to build L2 topology. This field is optional and the default value is false.  |
|isBuildIPsecVPNTopology | bool  | Determine whether to build IPsecVPN topology. This field is optional and the default value is false. |
|isRecalculateDynamicDeviceGroups | bool  | Determine whether to recalculate dynamic device groups. This field is optional and the default value is false.  |
|isRecalculateSite | bool  | Determine whether to rebuild sites. This field is optional and the default value is false.|
|isRecalculateMPLSVirtualRouteTables | bool  | Determine whether to recalculate MPLS Virtual Route Table. This field is optional and the default value is false.  |
|isbuildDefaultDeviceDataView | bool  | Determine whether to build default device data view. This field is optional and the default value is false.  |
|isEnable | bool  | Determine whether to enable the task. This field is optional and the default value is true. |
|pluginExecPoints | list of object  | The Analysis Plugins to run during the task, grouped by execution point. This corresponds to the *Plugins* tab of the benchmark task in the UI. This field is optional. If omitted, no plugin runs.  |
|pluginExecPoints[].execPoint* | string  | The execution point, i.e. the stage of the benchmark at which the plugins run. Each execution point can be used once per task. Options: StartBenchmarkTask, BeforeRetrieve, AfterRetrieve, BeforeBuildTopo, BeforeBuildVlanGroup, AfterBuildVlanGroup, BeforeBuildL3Topo, AfterBuildL3Topo, BeforeBuildL2Topo, AfterBuildL2Topo, BeforeBuildMPLSCEList, AfterBuildMPLSEList, AfterBuildTopo, BeforeSystemOperations, AfterSystemOperations, BeforeRebuildVisualSpace, AfterRebuildVisualSpace, BeforeUpdateMap, AfterUpdateMap, EndBenchmarkTask.  |
|pluginExecPoints[].isParallel | bool  | Determine whether to run the plugins of this execution point in parallel. This field is optional and the default value is false.  |
|pluginExecPoints[].plugins* | list of object  | The plugins to run at this execution point. It cannot be empty.  |
|pluginExecPoints[].plugins[].path* | string  | The plugin path. Use the exact `plugin.path` value returned by the [Get Plugin API](../Plugin/Get%20Plugin%20API.md), for example "My Plugins/interface_capacity?domainId=&lt;domain-guid&gt;": no leading "/", and including the `?domainId=` suffix. A path such as "/My Plugins/interface_capacity" is not recognized and returns 791006.  |
|pluginExecPoints[].plugins[].input | string  | The input passed to the plugin when it runs. This field is optional. It only takes effect if the plugin accepts input (`pluginInfo.input.enabled` is true in the [Add or Update Plugin API](../Plugin/Add%20or%20Update%20Plugin%20API.md)); otherwise it is ignored and the UI shows N/A.  |
|pluginExecPoints[].plugins[].enabled | bool  | Determine whether the plugin runs. This field is optional and **the default value is false**, so set it to true for every plugin that should run.  |

> ### ***Example***


```python
body = {
        "taskName":taskName, #The name of the task.
        "startDate":startDate, #The date when the task starts to run. The standard time format is required, for example, '2017-07-13', '2017/07/13'.
        "schedule":{
            "frequency":frequency, #The frequency to run the task. This field is required and includes ”once”, “hourly”,” daily”, “weekly” and “monthly” options.
            "startTime":[startTime] #The time to run the task. This field is required and startTime should be in format: ["HH:mm:ss"], if you put date time format such as "2018/04/04 19:20:20 ", "19:20:20" will be used and the date part "2018/04/04" will be ignored.
            },
        "deviceScope" : {
            "scopeType" : scopeType
            }
        }


# Add a task that runs Analysis Plugins:
body = {
        "taskName":taskName, #The name of the task.
        "startDate":startDate,
        "schedule":{
            "frequency":frequency,
            "startTime":[startTime]
            },
        "deviceScope" : {
            "scopeType" : scopeType
            },
        "pluginExecPoints" : [ #Optional. The Analysis Plugins to run, grouped by execution point.
            {
                "execPoint" : "AfterRetrieve", #Required. The stage of the benchmark at which the plugins run.
                "isParallel" : False, #Optional. Run the plugins in parallel. Default is false.
                "plugins" : [ #Required. Cannot be empty.
                    {"path" : "My Plugins/interface_capacity?domainId=<domain-guid>", "input" : "threshold=80", "enabled" : True}, #enabled defaults to false, so set it to true for each plugin that should run.
                    {"path" : "My Plugins/eigrp_health?domainId=<domain-guid>", "enabled" : True}
                    ]
                }
            ]
        }
```

## Parameters(****required***)

> No parameters required.

## Headers

> **Data Format Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| Content-Type | string  | support "application/json" |
| Accept | string  | support "application/json" |

> **Authorization Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| token | string  | Authentication token, get from login API. |


## Response

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> ***Example***


```python
{
    "statusCode": 790200,
    "statusDescription": "Success."
}
```

# Full Example :


```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request inputs
token = "dbd4e523-5964-4c2d-ba8f-da018cfb6299"
nb_url = "http://192.168.28.79"
taskName = "Scheduled System DiscoveryGDL11"
startDate = "2019-01-16"
frequency = "once"
startTime = "14:40:20"
scopeType = "all"

# Add a new Benchmark
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Benchmark/Tasks"

headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

body = {
    "taskName":taskName, 
    "startDate":startDate, 
    "schedule":{
        "frequency":frequency, 
        "startTime":[startTime]
        },
    "deviceScope" : {
        "scopeType" : scopeType
        }
}

try:
    response = requests.post(full_url, data=json.dumps(body), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Benchmark Task added Failed! - " + str(response.text))

except Exception as e:
    print (str(e))

```

    {'statusCode': 790200, 'statusDescription': 'Success.'}
    

# Full Example 2 : Add a Benchmark Task that runs Analysis Plugins


```python
# import python modules 
import requests
import urllib3
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request inputs
token = "dbd4e523-5964-4c2d-ba8f-da018cfb6299"
nb_url = "http://192.168.28.79"
taskName = "Nightly Benchmark with Plugins"
startDate = "2026-11-04"
frequency = "daily"
startTime = "02:00:00"
scopeType = "all"

# Add a new Benchmark that runs two Analysis Plugins
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Benchmark/Tasks"

headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

body = {
    "taskName":taskName, 
    "startDate":startDate, 
    "schedule":{
        "frequency":frequency, 
        "startTime":[startTime]
        },
    "deviceScope" : {
        "scopeType" : scopeType
        },
    "pluginExecPoints" : [
        {
            "execPoint" : "AfterRetrieve",
            "isParallel" : False,
            "plugins" : [
                {"path" : "My Plugins/interface_capacity?domainId=<domain-guid>", "input" : "threshold=80", "enabled" : True},
                {"path" : "My Plugins/eigrp_health?domainId=<domain-guid>", "enabled" : True}
                ]
            }
        ]
}

try:
    response = requests.post(full_url, data=json.dumps(body), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Benchmark Task added Failed! - " + str(response.text))

except Exception as e:
    print (str(e))

```

    {'statusCode': 790200, 'statusDescription': 'Success.'}
    

# Full Example 3 : Add a Benchmark Task that retrieves selected Built-in Live Data


```python
# import python modules
import requests
import urllib3
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request inputs
token = "dbd4e523-5964-4c2d-ba8f-da018cfb6299"
nb_url = "http://192.168.28.79"
taskName = "Nightly Benchmark with L2 and L3 Tables"
startDate = "2026-11-04"
frequency = "daily"
startTime = "02:00:00"
scopeType = "all"

# Add a new Benchmark that retrieves the configuration, route, ARP and MAC tables
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Benchmark/Tasks"

headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

body = {
    "taskName":taskName,
    "startDate":startDate,
    "schedule":{
        "frequency":frequency,
        "startTime":[startTime]
        },
    "deviceScope" : {
        "scopeType" : scopeType
        },
    "retrieveData" : ["config", "routingTable", "arpTable", "macTable"] #List "config" too, otherwise the configuration file is not retrieved.
}

try:
    response = requests.post(full_url, data=json.dumps(body), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Benchmark Task added Failed! - " + str(response.text))

except Exception as e:
    print (str(e))

```

    {'statusCode': 790200, 'statusDescription': 'Success.'}


# Full Example 4 : Add a Benchmark Task that passes an input to a plugin

`pluginExecPoints[].plugins[].input` is a string that is passed to the plugin's `run(input)` function when the plugin runs during the benchmark. It is the same value as the plugin's **Input** column in the *Plugins* tab of the task in the UI. The plugin decides what the string means: here, `threshold=80` is a setting the plugin can read, so the same plugin can be reused by different tasks with different values.

The plugin must accept input, i.e. it is created with `pluginInfo.input.enabled` set to true in the [Add or Update Plugin API](../Plugin/Add%20or%20Update%20Plugin%20API.md). Otherwise the input is ignored and the UI shows N/A.

The plugin used in this example (`main.py` of `My Plugins/api_input_demo`) logs the input it receives, and fails if it receives none:

```python
from netbrain.sysapi import pluginfw

def run(input):
    if input:
        pluginfw.AddLog('api_input_demo received input: %s' % input)
        return True
    pluginfw.AddLog('api_input_demo received NO input')
    return False
```

The task below runs this plugin with the input `threshold=80` before the benchmark starts, on the devices of one device group.


```python
# import python modules
import requests
import urllib3
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request inputs
token = "dbd4e523-5964-4c2d-ba8f-da018cfb6299"
nb_url = "http://192.168.28.79"
taskName = "Input Plugin Test"
startDate = "2026-11-04"
frequency = "once"
startTime = "23:59:00"
pluginPath = "My Plugins/api_input_demo?domainId=<domain-guid>" # The exact plugin.path returned by the Get Plugin API.

full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Benchmark/Tasks"

headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

body = {
    "taskName":taskName,
    "startDate":startDate,
    "schedule":{
        "frequency":frequency,
        "startTime":[startTime]
        },
    "deviceScope" : {
        "scopeType" : "deviceGroup",
        "scopes" : ["Private/F5"] # Use the Public/Private/System prefix, not "Shared Device Groups" or "My Device Groups".
        },
    "pluginExecPoints" : [
        {
            "execPoint" : "StartBenchmarkTask", # "Before executing benchmark task" in the UI.
            "plugins" : [
                {"path" : pluginPath, "input" : "threshold=80", "enabled" : True} # Passed to run(input) as "threshold=80".
                ]
            }
        ]
}

try:
    response = requests.post(full_url, data=json.dumps(body), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Benchmark Task added Failed! - " + str(response.text))

except Exception as e:
    print (str(e))

```

    {'statusCode': 790200, 'statusDescription': 'Success.'}


# cURL Code from Postman


```python
curl -X POST \
  http://192.168.28.79/ServicesAPI/API/V1/CMDB/Benchmark/Tasks \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: a6b7d2b7-f331-4418-bcc0-d1a90f01dbd8' \
  -H 'Token: c00de805-9210-44a9-9a26-f0c1e944ea36' \
  -H 'cache-control: no-cache' \
  -d 'body = {
                "taskName":"Scheduled System DiscoveryGDL1", 
                "startDate":"2019-01-16", 
                "schedule":{
                    "frequency":"once", 
                    "startTime":["14:40:20"]
                    },
                "deviceScope" : {
                    "scopeType" : "all"
                    }
            }'
```

# Error Examples

## Error Example 1: Empty `taskName`
```python
Input:

        taskName = "" # Cannot be empty.
        startDate = "2019-01-16"
        frequency = "once"
        startTime = "14:40:20"
        scopeType = "all"

Response:

    Benchmark Task added Failed! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'TaskName'. The TaskName field is required."}

```
## Error Example 2: Duplicate `taskName`
```python
Input:

        taskName = "Scheduled System Discovery" # A benchmark task with this name already exists.
        startDate = "2019-01-16"
        frequency = "once"
        startTime = "14:40:20"
        scopeType = "all"

Response:

    Benchmark Task added Failed! - {"statusCode":791007,"statusDescription":"Benchmark task with name Scheduled System Discovery already exists."}

```
## Error Example 3: Empty or wrongly formatted `startDate`
```python
Input 1:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "" # Cannot be empty.
        frequency = "once"
        startTime = "14:40:20"
        scopeType = "all"

Response 1:

    Benchmark Task added Failed! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'StartDate'. The StartDate field is required."}

```
```python
Input 2:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "20190116" # Wrong format: use "2019-01-16" or "2019/01/16".
        frequency = "once"
        startTime = "14:40:20"
        scopeType = "all"

Response 2:

    Benchmark Task added Failed! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'startDate'."}

```
## Error Example 4: Empty or invalid `schedule.frequency`
```python
Input 1:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "2019-01-16"
        frequency = "" # Cannot be empty.
        startTime = "14:40:20"
        scopeType = "all"

Response 1:

    Benchmark Task added Failed! - {"statusCode":791000,"statusDescription":"Null parameter: the parameter 'schedule frequency' cannot be null."}

```
```python
Input 2:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "2019-01-16"
        frequency = "1" # Must be once, hourly, daily, weekly or monthly.
        startTime = "14:40:20"
        scopeType = "all"

Response 2:

    Benchmark Task added Failed! - {"statusCode":791001,"statusDescription":"Invalid parameter: the parameter 'schedule.frequency' is invalid.  Options: once, hourly, daily, weekly, monthly "}

```
## Error Example 5: Empty or wrongly formatted `schedule.startTime`
```python
Input 1:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "2019-01-16"
        frequency = "once"
        startTime = "" # Cannot be empty.
        scopeType = "all"

Response 1:

    Benchmark Task added Failed! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'schedule.startTime'."}

```
```python
Input 2:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "2019-01-16"
        frequency = "once"
        startTime = "14/40/20" # Wrong format: use "HH:mm:ss".
        scopeType = "all"

Response 2:

    Benchmark Task added Failed! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'schedule.startTime'."}

```
## Error Example 6: Empty or invalid `deviceScope.scopeType`
```python
Input 1:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "2019-01-16"
        frequency = "once"
        startTime = "14:40:20"
        scopeType = "" # Cannot be empty.

Response 1:

    Benchmark Task added Failed! - {"statusCode":792009,"statusDescription":"Invalid data type. deviceScope.scopeType, options: all,deviceGroup,site"}

```
```python
Input 2:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "2019-01-16"
        frequency = "once"
        startTime = "14:40:20"
        scopeType = "whole" # Must be all, deviceGroup or site.

Response 2:

    Benchmark Task added Failed! - {"statusCode":792009,"statusDescription":"Invalid data type. deviceScope.scopeType, options: all,deviceGroup,site"}

```
## Error Example 7: Invalid `execPoint`
```python
Input:

        pluginExecPoints = [{"execPoint": "AfterRetrive", # Typo: must be one of the execution points, e.g. "AfterRetrieve".
                             "plugins": [{"path": "My Plugins/interface_capacity?domainId=<domain-guid>", "enabled": True}]}]

Response:

    Benchmark Task added Failed! - {"statusCode":791001,"statusDescription":"Invalid parameter: the parameter 'pluginExecPoints.execPoint' is invalid. Options: StartBenchmarkTask, BeforeRetrieve, AfterRetrieve, BeforeBuildTopo, BeforeBuildVlanGroup, AfterBuildVlanGroup, BeforeBuildL3Topo, AfterBuildL3Topo, BeforeBuildL2Topo, AfterBuildL2Topo, BeforeBuildMPLSCEList, AfterBuildMPLSEList, AfterBuildTopo, BeforeSystemOperations, AfterSystemOperations, BeforeRebuildVisualSpace, AfterRebuildVisualSpace, BeforeUpdateMap, AfterUpdateMap, EndBenchmarkTask"}

```
## Error Example 8: Plugin `path` not found
```python
Input:

        pluginExecPoints = [{"execPoint": "AfterRetrieve",
                             "plugins": [{"path": "/My Plugins/interface_capacity", "enabled": True}]}] # Wrong format: use the exact plugin.path from the Get Plugin API, e.g. "My Plugins/interface_capacity?domainId=<domain-guid>".

Response:

    Benchmark Task added Failed! - {"statusCode":791006,"statusDescription":"plugin path /My Plugins/interface_capacity does not exist."}

```
## Error Example 9: Empty `execPoint` or plugin `path`
```python
Input 1:

        pluginExecPoints = [{"execPoint": "", # Cannot be empty.
                             "plugins": [{"path": "My Plugins/interface_capacity?domainId=<domain-guid>", "enabled": True}]}]

Response 1:

    Benchmark Task added Failed! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'PluginExecPoints.ExecPoint'. The ExecPoint field is required."}

```
```python
Input 2:

        pluginExecPoints = [{"execPoint": "AfterRetrieve",
                             "plugins": [{"path": "  ", "enabled": True}]}] # Cannot be empty or only spaces.

Response 2:

    Benchmark Task added Failed! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'PluginExecPoints.Plugins.Path'. The Path field is required."}

```
## Error Example 10: Empty `plugins` list
```python
Input:

        pluginExecPoints = [{"execPoint": "AfterRetrieve", "plugins": []}] # Must contain at least one plugin.

Response:

    Benchmark Task added Failed! - {"statusCode":791000,"statusDescription":"Null parameter: the parameter 'pluginExecPoints.plugins' cannot be null."}

```
## Error Example 11: Unsupported `retrieveData` value
```python
Input:

        taskName = "Nightly Benchmark"
        startDate = "2026-11-04"
        frequency = "daily"
        startTime = "02:00:00"
        scopeType = "all"
        retrieveData = ["routeTable"] # Typo: the supported value is "routingTable".

Response:

    {'statusCode': 790200, 'statusDescription': 'Success.'}

    # The request succeeds, but "routeTable" is silently ignored. Because no supported value was sent,
    # the task retrieves no built-in live data at all, not even the Configuration File:
    # the Built-in Live Data list in the task's Summary tab is empty.

```
