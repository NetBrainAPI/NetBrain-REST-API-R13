
# Benchmark API Design

## ***PUT*** /V1/CMDB/Benchmark/Tasks
This API call is used to update an existing benchmark task.

## Detail Information

> **Title** : Update Benchmark Task API<br>

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
|newTaskName | string  | the new task name, optional.  |
|description | string  | The description of the task. This field is optional.  |
|startDate | string  | The date when the task starts to run. The standard time format is required, for example, '2017-07-13', '2017/07/13'. This field is optional. Current date will be used by default.  |
|schedule | object  | The schedule to run the task. The following sub parameters are included in this object: <br>▪ frequency* (string) - the frequency to run the task. This field is required and includes ”once”, “hourly”,” daily”, “weekly” and “monthly” options.<br>▪ interval(string) - the interval to run the task (optional). This field is only valid for “hourly”,” daily”, and “weekly” options and the default value is 1, such as every 1 hour, 1 week.<br>▪ startTime* (string) - the time to run the task. This field is required and startTime should be in format: ["HH:mm:ss"], if you put date time format such as "2018/04/04 19:20:20 ", "19:20:20" will be used and the date part "2018/04/04" will be ignored.<br> **Note:** Set the time according to your IIS server time zone since the time zone of your ISS server rather than your physical time zone is adopted by the benchmark task.<br>▪ weekday(integer) - the day of the week to run the task. This field is optional and only valid when the frequency is weekly.  0 stands for Sunday, 6 for Saturday and 1-5 for Monday to Friday respectively.<br>▪ dayOfMonth(integer) - which day of a month to run the task. This field is optional and only valid when the frequency is monthly. The default is 1.<br>▪ Months(integer) - which month to run the task. This field is optional and only valid when the frequency is monthly. The default is all 12 months.|
|deviceScope | string  | the devices included in this task. |
|deviceScope.scopeType | string  | scope type options:<br>"all" for all devices of current domain, deviceScope.scopes will be ignored if this field is set to "all";<br>"deviceGroup" for specified group, if set deviceScope.scopes would be list of full path to device groups, such as \["Public/devgrp1", "Private/devgrp2", "System/devgrp3"\];<br>"site" for a particular site. if set deviceScope.scopes would be list of full path to sites, for example: \["My Networks/US/MA/Boston", "My Networks/US/ME/Portland"\]  |
|deviceScope.scopes | list of string  | ignored if deviceScope.scopeType is set to "all";<br>full path to device groups, such as \["Public/devgrp1", "Private/devgrp2", "System/devgrp3"\] if deviceScope.scopeType is set to "deviceGroup";<br>&nbsp;&nbsp;&nbsp;&nbsp;Use the Public/, Private/ or System/ prefix. The names returned by the [Get Device Group API](../Device%20Group%20Management/Get%20Device%20Group%20API.md) start with "Shared Device Groups/", "My Device Groups/" or "System Device Groups/" (type 0, 1, 2); replace that first part with Public, Private or System. For example, "My Device Groups/F5" is "Private/F5". Otherwise the request fails with 791006 "device group prefix: ... does not exist. options: Public, Private, System".;<br>full path to sites, such as \["My Networks/US/MA/Boston", "My Networks/US/ME/Portland"\] if deviceScope.scopeType is set to "site";  |
|limitRunMins | string  | The time used to retrieve the data. When it reaches the specified time, the task will stop retrieving more data. This field is optional.  |
|cliCommands | string  | The customized CLI commands to retrieve data (for example, ["show version", "show arp"]. This field is optional.  |
|retrieveData | list of string  | The built-in live data to retrieve. This corresponds to *Built-in Live Data* in the *Batch Definition* tab of the task in the UI. This field is optional. If omitted, the task's current selection is kept.<br>**When sent, it replaces the task's current selection**, e.g. sending ["macTable"] to a task that retrieves the Configuration File leaves only the MAC Table. Supported values:<br>▪ "config" - Configuration File<br>▪ "routingTable" - Route Table<br>▪ "arpTable" - ARP Table<br>▪ "macTable" - MAC Table<br>▪ "cdpTable" - NDP Table<br>▪ "stpTable" - STP Table<br>▪ "inventoryInfo" - Inventory Information of Device/Interface/Module<br>▪ The name of an NCT table, exactly as shown in the *Built-in Data* tab of the UI, for example "BGP Neighbors Table" or "Xconnect Table". NCT tables are shown in the task summary as a count, e.g. "NCT Table: 2".<br>**Note:** The values are not validated. An unsupported value is silently ignored and the request still returns success. If none of the values is supported, the task retrieves no built-in live data at all, not even the Configuration File.  |
|isBuildIPv4L3Topo | bool  | Determine whether to build IPv4 L3 topology. This field is optional and the default value is false.  |
|isBuildIPv6L3Topo | bool  | Determine whether to build IPv6 L3 topology. This field is optional and the default value is false.  |
|isBuildL2Topo | bool  | Determine whether to build L2 topology. This field is optional and the default value is false.  |
|isBuildIPsecVPNTopology | bool  | Determine whether to build IPsecVPN topology. This field is optional and the default value is false. |
|isRecalculateDynamicDeviceGroups | bool  | Determine whether to recalculate dynamic device groups. This field is optional and the default value is false.  |
|isRecalculateSite | bool  | Determine whether to rebuild sites. This field is optional and the default value is false.|
|isRecalculateMPLSVirtualRouteTables | bool  | Determine whether to recalculate MPLS Virtual Route Table. This field is optional and the default value is false.  |
|isbuildDefaultDeviceDataView | bool  | Determine whether to build default device data view. This field is optional and the default value is false.  |
|isEnable | bool  | Determine whether to enable the task. This field is optional and the default value is true. |
|pluginExecPoints | list of object  | The Analysis Plugins to run during the task, grouped by execution point. This corresponds to the *Plugins* tab of the benchmark task in the UI. This field is optional. If omitted, the task's current plugin configuration is kept.<br>**When sent, it replaces the task's entire plugin configuration**: execution points that are not included are removed. If the request is rejected, the task's existing plugin configuration is left unchanged.  |
|pluginExecPoints[].execPoint* | string  | The execution point, i.e. the stage of the benchmark at which the plugins run. Each execution point can be used once per task. Options: StartBenchmarkTask, BeforeRetrieve, AfterRetrieve, BeforeBuildTopo, BeforeBuildVlanGroup, AfterBuildVlanGroup, BeforeBuildL3Topo, AfterBuildL3Topo, BeforeBuildL2Topo, AfterBuildL2Topo, BeforeBuildMPLSCEList, AfterBuildMPLSEList, AfterBuildTopo, BeforeSystemOperations, AfterSystemOperations, BeforeRebuildVisualSpace, AfterRebuildVisualSpace, BeforeUpdateMap, AfterUpdateMap, EndBenchmarkTask.  |
|pluginExecPoints[].isParallel | bool  | Determine whether to run the plugins of this execution point in parallel. This field is optional and the default value is false.  |
|pluginExecPoints[].plugins* | list of object  | The plugins to run at this execution point. It cannot be empty.  |
|pluginExecPoints[].plugins[].path* | string  | The plugin path. Use the exact `plugin.path` value returned by the [Get Plugin API](../Plugin/Get%20Plugin%20API.md), for example "My Plugins/interface_capacity?domainId=&lt;domain-guid&gt;": no leading "/", and including the `?domainId=` suffix. A path such as "/My Plugins/interface_capacity" is not recognized and returns 791006.  |
|pluginExecPoints[].plugins[].input | string  | The input passed to the plugin when it runs. This field is optional. It only takes effect if the plugin accepts input (`pluginInfo.input.enabled` is true in the [Add or Update Plugin API](../Plugin/Add%20or%20Update%20Plugin%20API.md)); otherwise it is ignored and the UI shows N/A.  |
|pluginExecPoints[].plugins[].enabled | bool  | Determine whether the plugin runs. This field is optional and **the default value is false**, so set it to true for every plugin that should run.  |

> ### ***Example***


```python
# Update task name:
body = {
        "taskName":taskName, #The name of the task.
        "newTaskName":newTaskName, #The new task name.
        "startDate":startDate, #The date when the task starts to run. The standard time format is required, for example, '2017-07-13', '2017/07/13'.
        "schedule":{
            "frequency":frequency, #The frequency to run the task. This field is required and includes ”once”, “hourly”,” daily”, “weekly” and “monthly” options.
            "startTime":[startTime] #The time to run the task. This field is required and startTime should be in format: ["HH:mm:ss"], if you put date time format such as "2018/04/04 19:20:20 ", "19:20:20" will be used and the date part "2018/04/04" will be ignored.
            },
        "deviceScope" : {
            "scopeType" : scopeType
            }
        }


# Update other attribute:
body = {
        "taskName":taskName, #The name of the task.
        "startDate":newStartDate, #The date when the task starts to run. The standard time format is required, for example, '2017-07-13', '2017/07/13'.
        "schedule":{
            "frequency":newFrequency, #The frequency to run the task. This field is required and includes ”once”, “hourly”,” daily”, “weekly” and “monthly” options.
            "startTime":[startTime] #The time to run the task. This field is required and startTime should be in format: ["HH:mm:ss"], if you put date time format such as "2018/04/04 19:20:20 ", "19:20:20" will be used and the date part "2018/04/04" will be ignored.
            },
        "deviceScope" : {
            "scopeType" : scopeType
            }
        }


# Update the Analysis Plugins (replaces the task's existing plugin configuration):
body = {
        "taskName":taskName, #The name of the task.
        "pluginExecPoints" : [
            {
                "execPoint" : "AfterRetrieve", #Required. The stage of the benchmark at which the plugins run.
                "isParallel" : False, #Optional. Default is false.
                "plugins" : [ #Required. Cannot be empty.
                    {"path" : "My Plugins/interface_capacity?domainId=<domain-guid>", "input" : "threshold=80", "enabled" : True} #enabled defaults to false.
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

# Set the request parameters
token = "dbd4e523-5964-4c2d-ba8f-da018cfb6299"
nb_url = "http://192.168.28.79"
taskName = "Scheduled System DiscoveryGDL"
startDate = "2019-01-16"
#frequency = "weekly" #Update the benchmark running frequency.
startTime = "14:40:20"
scopeType = "all"

full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Benchmark/Tasks"

headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

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
    
try:
    response = requests.put(full_url, data=json.dumps(body), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print(result)
    else:
        print ("Benchmark Task updated Failed! - " + str(response.text))

except Exception as e:
    print (str(e)) 
```

    {'statusCode': 790200, 'statusDescription': 'Success.'}
    

# Full Example 2 : Update the Analysis Plugins of a Benchmark Task

Sending `pluginExecPoints` replaces the task's entire plugin configuration. Any execution point not listed in the request is removed from the task.


```python
# import python modules 
import requests
import urllib3
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request parameters
token = "dbd4e523-5964-4c2d-ba8f-da018cfb6299"
nb_url = "http://192.168.28.79"
taskName = "Scheduled System DiscoveryGDL"

full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Benchmark/Tasks"

headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

body = {
    "taskName":taskName, #The name of the task.
    "pluginExecPoints" : [
        {
            "execPoint" : "AfterRetrieve",
            "isParallel" : True, #Run the plugins of this execution point in parallel.
            "plugins" : [
                {"path" : "My Plugins/interface_capacity?domainId=<domain-guid>", "input" : "threshold=80", "enabled" : True},
                {"path" : "My Plugins/eigrp_health?domainId=<domain-guid>", "enabled" : False} #Keep the plugin on the task but do not run it.
                ]
            },
        {
            "execPoint" : "EndBenchmarkTask",
            "plugins" : [
                {"path" : "My Plugins/interface_capacity?domainId=<domain-guid>", "input" : "threshold=80", "enabled" : True}
                ]
            }
        ]
}
    
try:
    response = requests.put(full_url, data=json.dumps(body), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print(result)
    else:
        print ("Benchmark Task updated Failed! - " + str(response.text))

except Exception as e:
    print (str(e)) 
```

    {'statusCode': 790200, 'statusDescription': 'Success.'}
    

# Full Example 3 : Update the Built-in Live Data of a Benchmark Task

Sending `retrieveData` replaces the task's current selection. List every table the task should keep retrieving, including "config".


```python
# import python modules
import requests
import urllib3
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request parameters
token = "dbd4e523-5964-4c2d-ba8f-da018cfb6299"
nb_url = "http://192.168.28.79"
taskName = "Scheduled System DiscoveryGDL"

full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Benchmark/Tasks"

headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

body = {
    "taskName":taskName, #The name of the task.
    "retrieveData" : ["config", "arpTable", "macTable", "stpTable"] #Replaces the task's current Built-in Live Data selection.
}

try:
    response = requests.put(full_url, data=json.dumps(body), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print(result)
    else:
        print ("Benchmark Task updated Failed! - " + str(response.text))

except Exception as e:
    print (str(e))
```

    {'statusCode': 790200, 'statusDescription': 'Success.'}


# cURL Code from Postman


```python
curl -X PUT \
  http://192.168.28.79/ServicesAPI/API/V1/CMDB/Benchmark/Tasks \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: b277650f-f646-474d-954c-104a028f9a9f' \
  -H 'cache-control: no-cache' \
  -H 'token: c00de805-9210-44a9-9a26-f0c1e944ea36' \
  -d 'body = {
    "taskName":"Scheduled System DiscoveryGDL1", 
    "startDate":"2020-01-16", 
    "schedule":{
        "frequency":"weekly", 
        "startTime":["14:40:20"]
        },
    "deviceScope" : {
        "scopeType" : "all"
        }
}'
```


```python
newTaskName = ""
frequency = ""

body = {
    "taskName":taskName, #The name of the task.
    "newTaskName":newTaskName,
    "startDate":startDate, #The date when the task starts to run. The standard time format is required, for example, '2017-07-13', '2017/07/13'.
    "schedule":{
        "frequency":frequency, #The frequency to run the task. This field is required and includes ”once”, “hourly”,” daily”, “weekly” and “monthly” options.
        "startTime":[startTime] #The time to run the task. This field is required and startTime should be in format: ["HH:mm:ss"], if you put date time format such as "2018/04/04 19:20:20 ", "19:20:20" will be used and the date part "2018/04/04" will be ignored.
        },
    "deviceScope" : {
        "scopeType" : scopeType
        }
}
    
try:
    response = requests.put(full_url, data=json.dumps(body), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print(result)
    else:
        print ("Benchmark Task updated Failed! - " + str(response.text))

except Exception as e:
    print (str(e)) 
```

    {'statusCode': 790200, 'statusDescription': 'Success.'}
    

# Error Examples and Note

## Error Example 1: `newTaskName` is the same as the current task name
```python
Input:

        taskName = "Scheduled System DiscoveryGDL"
        newTaskName = "Scheduled System DiscoveryGDL" # Must be different from the current name.
        startDate = "2019-01-16"
        frequency = "weekly"
        startTime = "14:40:20"
        scopeType = "all"

Response:

    Benchmark Task updated Failed! - {"statusCode":791007,"statusDescription":"newTaskName already exists."}

```
## Error Example 2: Required attributes updated as empty
```python
Input:

        taskName = "Scheduled System DiscoveryGDL"
        newTaskName = "" # New task name updated as empty.
        startDate = "2019-01-16"
        frequency = "" # Benchmark running frequency updated as empty.
        startTime = "14:40:20"
        scopeType = "all"

Response:

    {'statusCode': 790200, 'statusDescription': 'Success.'}

```
## Error Example 3: `startDate` in the wrong format
```python
Input:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "20190116" # Wrong format of start date.
        frequency = "weekly"
        startTime = "14:40:20"
        scopeType = "all"

Response:

    {'statusCode': 790200, 'statusDescription': 'Success.'}

```
## Error Example 4: Invalid `execPoint`
```python
Input:

        pluginExecPoints = [{"execPoint": "AfterRetrive", # Typo: must be one of the execution points, e.g. "AfterRetrieve".
                             "plugins": [{"path": "My Plugins/interface_capacity?domainId=<domain-guid>", "enabled": True}]}]

Response:

    Benchmark Task updated Failed! - {"statusCode":791001,"statusDescription":"Invalid parameter: the parameter 'pluginExecPoints.execPoint' is invalid. Options: StartBenchmarkTask, BeforeRetrieve, AfterRetrieve, BeforeBuildTopo, BeforeBuildVlanGroup, AfterBuildVlanGroup, BeforeBuildL3Topo, AfterBuildL3Topo, BeforeBuildL2Topo, AfterBuildL2Topo, BeforeBuildMPLSCEList, AfterBuildMPLSEList, AfterBuildTopo, BeforeSystemOperations, AfterSystemOperations, BeforeRebuildVisualSpace, AfterRebuildVisualSpace, BeforeUpdateMap, AfterUpdateMap, EndBenchmarkTask"}

```
## Error Example 5: Plugin `path` not found
```python
Input:

        pluginExecPoints = [{"execPoint": "AfterRetrieve",
                             "plugins": [{"path": "/My Plugins/interface_capacity", "enabled": True}]}] # Wrong format: use the exact plugin.path from the Get Plugin API, e.g. "My Plugins/interface_capacity?domainId=<domain-guid>".

Response:

    Benchmark Task updated Failed! - {"statusCode":791006,"statusDescription":"plugin path /My Plugins/interface_capacity does not exist."}

```
## Error Example 6: Empty `execPoint` or plugin `path`
```python
Input 1:

        pluginExecPoints = [{"execPoint": "", # Cannot be empty.
                             "plugins": [{"path": "My Plugins/interface_capacity?domainId=<domain-guid>", "enabled": True}]}]

Response 1:

    Benchmark Task updated Failed! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'PluginExecPoints.ExecPoint'. The ExecPoint field is required."}

```
```python
Input 2:

        pluginExecPoints = [{"execPoint": "AfterRetrieve",
                             "plugins": [{"path": "  ", "enabled": True}]}] # Cannot be empty or only spaces.

Response 2:

    Benchmark Task updated Failed! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'PluginExecPoints.Plugins.Path'. The Path field is required."}

```
## Error Example 7: Empty `plugins` list
```python
Input:

        pluginExecPoints = [{"execPoint": "AfterRetrieve", "plugins": []}] # Must contain at least one plugin.

Response:

    Benchmark Task updated Failed! - {"statusCode":791000,"statusDescription":"Null parameter: the parameter 'pluginExecPoints.plugins' cannot be null."}

```
## Note 1: No further update (`newTaskName` not considered)
```python
Input:

        taskName = "Scheduled System DiscoveryGDL"
        startDate = "2019-01-16"
        frequency = "weekly"
        startTime = "14:40:20"
        scopeType = "all"
        # All inputs are the same as the existing benchmark task.

Response:

    {'statusCode': 790200, 'statusDescription': 'Success.'}

```
