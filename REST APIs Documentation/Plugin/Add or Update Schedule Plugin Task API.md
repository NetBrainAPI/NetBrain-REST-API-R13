# Plugin API Design

## ***POST*** /V1/CMDB/SchedulePlugin
This API call is used to add a new schedule plugin task to the current domain, or to update an existing one.

- If `id` is omitted or does not match an existing task, a new task is created.
- If `id` matches an existing task, that task is updated.

<b>Note</b>: `pluginGroup.plugins[].path` is stored as-is and is not validated. A path that does not resolve to a plugin produces a task that never runs.

## Detail Information

> **Title** : Add or Update Schedule Plugin Task API<br>

> **Version** : 10/19/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/SchedulePlugin

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|id* | string | The ID (GUID) of the task. <br>Recommended: always send a client-generated GUID. If a create request fails, the task may still be saved while the response returns `id: null`; your own GUID lets you find or clean it up. To update an existing task, send its ID. |
|name* | string | The name of the task. |
|enabled* | bool | Whether the task is enabled. |
|description^ | string | The description of the task. |
|schedule* | object | The schedule of the task. See **Schedule object** below. |
|pluginGroup* | object | The plugins to run. |
|pluginGroup.isParallel^ | bool | Whether to run the plugins in parallel. <br>Default: `false` |
|pluginGroup.plugins* | list of object | The plugins to run. |
|pluginGroup.plugins[].path* | string | The plugin path. <br>For private plugins, use the exact `plugin.path` value returned by the [Get Plugin API](Get%20Plugin%20API.md), which includes a `?domainId=<guid>` suffix, e.g. `My Plugins/my_plugin?domainId=<domain-guid>`. |
|pluginGroup.plugins[].input^ | string | The input value passed to the plugin. |
|pluginGroup.plugins[].enabled^ | bool | Whether this plugin entry is enabled. |

> **Schedule object** (`schedule`)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|startDate* | string | The local start date in `yyyy-M-d` format, e.g. `2026-1-1` or `2026-01-01`. |
|timeZone* | string | A Windows time zone ID, e.g. `Eastern Standard Time`, `Pacific Standard Time`, `China Standard Time`. `Coordinated Universal Time` is accepted and treated as `UTC`. |
|start* | string | The start date at **local midnight, expressed in UTC** (ISO 8601). This is **not** the run time; the time of day is set by `startTime` in the frequency object. <br>e.g. `startDate` = `2026-01-01` and `timeZone` = `Eastern Standard Time` (UTC-5) → `start` = `2026-01-01T05:00:00Z`. <br>**Note:** `start` is required and is not derived from `startDate`. If it is omitted, the task is saved with a start of `0001-01-01` (shown as Start Date `0001-12-31` in the UI) and may run early, late, repeatedly, or not at all. |
|enableEnd^ | bool | Whether the task has an end date. |
|endDate^ | string | The local end date in `yyyy-M-d` format. Required when `enableEnd` is `true`. |
|end^ | string | The end date at local midnight, expressed in UTC. Same rules as `start`. |
|frequency* | object | How often the task runs. See **Frequency object** below. |

> **Frequency object** (`schedule.frequency`)

`frequency` contains a `type` field plus **one** matching member object. The member key must be the exact name in the table below (`runOnce`, `byHour`, `byDay`, `byWeek`, `byMonth`), not the UI label (`Once`, `Hourly`, ...).

>>**Note:** `type`* is **required**. It is not derived from the member object. If `type` is omitted it defaults to `0` (Once), so `byHour`, `byDay`, `byWeek` and `byMonth` are rejected with `791001`, and the task shows as **Once** in the Edit Plugin Task dialog.

|**type**|**Member**|**UI label**|**Member fields**|
|------|------|------|------|
|<img width=50/>|<img width=100/>|<img width=100/>|<img width=450/>|
|0 | runOnce | Once | ▪ `startTime`* (int) - seconds since local midnight, `>= 0`. |
|2 | byHour | Hourly | ▪ `interval`* (int) - every N hours, `> 0`. <br>▪ `startTime`* (int) - seconds since local midnight, `>= 0`. |
|3 | byDay | Daily | ▪ `interval`* (int) - every N days, `> 0`. <br>▪ `timeRange`* (list of TimeRange) - non-empty. |
|4 | byWeek | Weekly | ▪ `interval`* (int) - every N weeks, `> 0`. <br>▪ `day`* (list of int) - non-empty; 0 = Sunday ... 6 = Saturday. <br>▪ `timeRange`* (list of TimeRange) - non-empty. |
|5 | byMonth | Monthly | ▪ `month`* (list of int) - non-empty; values 1-12. <br>▪ `day`* (int) - day of the month, 1-31. <br>▪ `timeRange`* (list of TimeRange) - non-empty. <br>**Note:** `byMonth` has no `interval` field. |

**TimeRange**: ▪ `startTime`* (int) - seconds since local midnight. ▪ `endTime`^ (int) - seconds since local midnight.

>>**Note:** `startTime` is the number of seconds after local midnight in `schedule.timeZone`, e.g. `28800` = 08:00:00, `36960` = 10:16:00, `71136` = 19:45:36. <br>`type = 1` (`bySecond`) is not supported for plugin tasks.

> ### ***Example***

```python
# Daily at 08:00 Eastern
body = {
    "id": "43c5c65e-810e-4ab6-9d80-fdec285d3d54",
    "name": "Daily Health Check",
    "enabled": True,
    "description": "Runs health check plugin daily",
    "schedule": {
        "startDate": "2026-10-20",
        "timeZone": "Eastern Standard Time",
        "start": "2026-10-20T04:00:00Z",
        "enableEnd": False,
        "frequency": {
            "type": 3,
            "byDay": {"interval": 1, "timeRange": [{"startTime": 28800}]}
        }
    },
    "pluginGroup": {
        "isParallel": False,
        "plugins": [
            {"path": "My Plugins/my_plugin?domainId=<domain-guid>", "input": "", "enabled": True}
        ]
    }
}
```

> **Frequency examples**

```python
# type 0 - Once at 10:16:00
"frequency": {"type": 0, "runOnce": {"startTime": 36960}}

# type 2 - Every 1 hour, starting 19:45:36
"frequency": {"type": 2, "byHour": {"interval": 1, "startTime": 71136}}

# type 3 - Every 1 day at 08:00:00
"frequency": {"type": 3, "byDay": {"interval": 1, "timeRange": [{"startTime": 28800}]}}

# type 4 - Every 1 week on Monday, Wednesday, Friday at 08:00:00
"frequency": {"type": 4, "byWeek": {"interval": 1, "day": [1, 3, 5], "timeRange": [{"startTime": 28800}]}}

# type 5 - On the 15th of January, April, July, October at 08:00:00
"frequency": {"type": 5, "byMonth": {"month": [1, 4, 7, 10], "day": 15, "timeRange": [{"startTime": 28800}]}}
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
|schedulePlugin| object | The created or updated task, with the same fields as the request body. |

>>**Note:** Check the returned `schedulePlugin.schedule.frequency`. If `type` is not what you sent, or the member object is missing or empty, the task was saved but **will not run**. This happens, for example, when the member key is a UI label (`"Daily"`) instead of the member name (`"byDay"`), or when `type` is missing.

> ***Example***


```python
{
    "statusCode": 790200,
    "statusDescription": "Success.",
    "schedulePlugin": {
        "id": "43c5c65e-810e-4ab6-9d80-fdec285d3d54",
        "name": "Daily Health Check",
        "enabled": True,
        "description": "Runs health check plugin daily",
        "schedule": {
            "startDate": "2026-10-20",
            "timeZone": "Eastern Standard Time",
            "start": "2026-10-20T04:00:00Z",
            "enableEnd": False,
            "frequency": {
                "type": 3,
                "byDay": {"interval": 1, "timeRange": [{"startTime": 28800}]}
            }
        },
        "pluginGroup": {
            "isParallel": False,
            "plugins": [
                {"path": "My Plugins/my_plugin?domainId=<domain-guid>", "input": "", "enabled": True}
            ]
        }
    }
}
```

# Full Example 1: Run daily at 08:00 AM Eastern

```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
import uuid
from datetime import datetime, timedelta
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

nb_url = "http://192.168.28.79"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
token = "bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7"
headers["Token"] = token

# Plugin path: use the exact "plugin.path" returned by the Get Plugin API
plugin_path = "My Plugins/my_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# Schedule: start on 2026-10-20, run daily at 08:00 Eastern
start_date = "2026-10-20"
windows_tz = "Eastern Standard Time"   # Windows time zone ID sent to NetBrain
utc_offset_hours = -4                  # UTC offset of windows_tz ON start_date (Eastern: -4 in daylight time, -5 in standard time)

# "start" = local midnight of startDate, converted to UTC
local_midnight = datetime.strptime(start_date, "%Y-%m-%d")
start_utc = (local_midnight - timedelta(hours=utc_offset_hours)).strftime("%Y-%m-%dT%H:%M:%SZ")   # 2026-10-20T04:00:00Z

def seconds_since_midnight(hhmmss):
    h, m, s = (int(x) for x in hhmmss.split(":"))
    return h * 3600 + m * 60 + s

body = {
    "id": str(uuid.uuid4()),   # always send your own ID
    "name": "Daily Health Check",
    "enabled": True,
    "description": "Runs health check plugin daily",
    "schedule": {
        "startDate": start_date,
        "timeZone": windows_tz,
        "start": start_utc,
        "enableEnd": False,
        "frequency": {
            "type": 3,
            "byDay": {"interval": 1, "timeRange": [{"startTime": seconds_since_midnight("08:00:00")}]}
        }
    },
    "pluginGroup": {
        "isParallel": False,
        "plugins": [
            {"path": plugin_path, "input": "", "enabled": True}
        ]
    }
}

addSchedulePlugin_URL = nb_url + "/ServicesAPI/API/V1/CMDB/SchedulePlugin"

def addOrUpdateSchedulePlugin(addSchedulePlugin_URL, body, headers):
    try:
        # Do the HTTP request
        response = requests.post(addSchedulePlugin_URL, headers=headers, data=json.dumps(body), verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Add or Update Schedule Plugin Task! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = addOrUpdateSchedulePlugin(addSchedulePlugin_URL, body, headers)
pprint.pprint(result)

# Confirm the frequency was stored as sent
stored = result["schedulePlugin"]["schedule"]["frequency"]
assert stored.get("type") == 3 and stored.get("byDay"), "Frequency not stored - task will not run"
```
```python
    {'schedulePlugin': {'description': 'Runs health check plugin daily',
                        'enabled': True,
                        'id': '43c5c65e-810e-4ab6-9d80-fdec285d3d54',
                        'name': 'Daily Health Check',
                        'pluginGroup': {'isParallel': False,
                                        'plugins': [{'enabled': True,
                                                     'input': '',
                                                     'path': 'My Plugins/my_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx'}]},
                        'schedule': {'enableEnd': False,
                                     'frequency': {'byDay': {'interval': 1, 'timeRange': [{'startTime': 28800}]},
                                                   'type': 3},
                                     'start': '2026-10-20T04:00:00Z',
                                     'startDate': '2026-10-20',
                                     'timeZone': 'Eastern Standard Time'}},
     'statusCode': 790200,
     'statusDescription': 'Success.'}
```

# Full Example 2: Run once at 11:15 AM Eastern

This example reuses `headers`, `plugin_path`, `seconds_since_midnight()`, `addSchedulePlugin_URL` and `addOrUpdateSchedulePlugin()` from Full Example 1.

```python
# Schedule: run once on 2026-10-21 at 11:15 AM Eastern
start_date = "2026-10-21"
windows_tz = "Eastern Standard Time"   # Windows time zone ID; covers both EST and EDT
utc_offset_hours = -4                  # 2026-10-21 is in daylight time (EDT, UTC-4); use -5 from Nov 1, 2026

# "start" = local midnight of startDate, converted to UTC (NOT 11:15)
local_midnight = datetime.strptime(start_date, "%Y-%m-%d")
start_utc = (local_midnight - timedelta(hours=utc_offset_hours)).strftime("%Y-%m-%dT%H:%M:%SZ")   # 2026-10-21T04:00:00Z

# The run time goes in runOnce.startTime, as seconds since local midnight
run_time = seconds_since_midnight("11:15:00")   # 40500

body = {
    "id": str(uuid.uuid4()),   # always send your own ID
    "name": "Run Once 1115",
    "enabled": True,
    "description": "Runs the plugin once at 11:15 AM Eastern",
    "schedule": {
        "startDate": start_date,
        "timeZone": windows_tz,
        "start": start_utc,
        "enableEnd": False,
        "frequency": {
            "type": 0,
            "runOnce": {"startTime": run_time}
        }
    },
    "pluginGroup": {
        "isParallel": False,
        "plugins": [
            {"path": plugin_path, "input": "", "enabled": True}
        ]
    }
}

result = addOrUpdateSchedulePlugin(addSchedulePlugin_URL, body, headers)
pprint.pprint(result)

# Confirm the frequency was stored as sent
stored = result["schedulePlugin"]["schedule"]["frequency"]
assert stored.get("type") == 0 and stored.get("runOnce"), "Frequency not stored - task will not run"
```
```python
    {'schedulePlugin': {'description': 'Runs the plugin once at 11:15 AM Eastern',
                        'enabled': True,
                        'id': '9b2e7c41-5a3d-4f8e-b1c6-2d7a8e9f0a1b',
                        'name': 'Run Once 1115',
                        'pluginGroup': {'isParallel': False,
                                        'plugins': [{'enabled': True,
                                                     'input': '',
                                                     'path': 'My Plugins/my_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx'}]},
                        'schedule': {'enableEnd': False,
                                     'frequency': {'runOnce': {'startTime': 40500},
                                                   'type': 0},
                                     'start': '2026-10-21T04:00:00Z',
                                     'startDate': '2026-10-21',
                                     'timeZone': 'Eastern Standard Time'}},
     'statusCode': 790200,
     'statusDescription': 'Success.'}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.28.79/ServicesAPI/API/V1/CMDB/SchedulePlugin \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7' \
  -d '{
    "id": "43c5c65e-810e-4ab6-9d80-fdec285d3d54",
    "name": "hourly_report",
    "enabled": true,
    "schedule": {
        "startDate": "2026-10-15",
        "timeZone": "Eastern Standard Time",
        "start": "2026-10-15T04:00:00Z",
        "enableEnd": false,
        "frequency": {
            "type": 2,
            "byHour": {"interval": 1, "startTime": 71136}
        }
    },
    "pluginGroup": {
        "isParallel": false,
        "plugins": [
            {"path": "My Plugins/hourly_report?domainId=<domain-guid>", "input": "", "enabled": true}
        ]
    }
}'
```

# Error Examples
## Error Example 1: Missing `name`
```python
Input:
    
        "name": "" # Task name cannot be empty.
          
Response:
    
('Failed to Add or Update Schedule Plugin Task! - '
 '{"statusCode":791000,"statusDescription":"Null parameter: the parameter '
 '\'schedulePlugin.name\' cannot be null."}')
 ```
## Error Example 2: `frequency.type` omitted
```python
Input:
    
        "frequency": {"byDay": {"interval": 1, "timeRange": [{"startTime": 28800}]}} # No "type"; it defaults to 0 (Once).
          
Response:
    
    Failed to Add or Update Schedule Plugin Task! - {"statusCode":791001,"statusDescription":"Invalid parameter: ..."}
```
## Error Example 3: `frequency.type` does not match the member object
```python
Input:
    
        "frequency": {"type": 2, "byDay": {"interval": 1, "timeRange": [{"startTime": 28800}]}} # type 2 expects "byHour".
          
Response:
    
    Failed to Add or Update Schedule Plugin Task! - {"statusCode":791001,"statusDescription":"Invalid parameter: ..."}
```
## Error Example 4: Invalid member fields
```python
Input 1:
    
        "frequency": {"type": 3, "byDay": {"interval": 1}} # byDay requires a non-empty timeRange.
          
Response 1:
    
    Failed to Add or Update Schedule Plugin Task! - {"statusCode":791001,"statusDescription":"Invalid parameter: ..."}
```
```python
Input 2:
    
        "frequency": {"type": 5, "byMonth": {"month": [1], "day": 32, "timeRange": [{"startTime": 28800}]}} # byMonth.day must be 1..31.
          
Response 2:
    
    Failed to Add or Update Schedule Plugin Task! - {"statusCode":791001,"statusDescription":"Invalid parameter: ..."}
```
## Error Example 5: UI label used as the member key (accepted, but the task never runs)
```python
Input:
    
        "frequency": {"type": 3, "Daily": {"interval": 1, "timeRange": [{"startTime": 28800}]}} # Should be "byDay", not "Daily".
          
Response:
    
    {'schedulePlugin': {..., 'schedule': {..., 'frequency': {...}}}, 'statusCode': 790200, 'statusDescription': 'Success.'} # The stored frequency member is empty.
```
## Error Example 6: `schedule.start` omitted (accepted, but runs at unpredictable times)
```python
Input:
    
        "schedule": {"startDate": "2026-10-20", "timeZone": "Eastern Standard Time", "frequency": {...}} # No "start".
          
Response:
    
    {'schedulePlugin': {..., 'schedule': {..., 'start': '0001-01-01T00:00:00'}}, 'statusCode': 790200, 'statusDescription': 'Success.'} # The task may run early, late, repeatedly, or not at all.
```
