# Plugin API Design

## ***GET*** /V1/CMDB/SchedulePlugin
This API call is used to get a schedule plugin task in the current domain by its ID.

## Detail Information

> **Title** : Get Schedule Plugin Task API<br>

> **Version** : 10/19/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/SchedulePlugin

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

> No request body.

## Query Parameters(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|id* | string | The ID (GUID) of the schedule plugin task. |

>>**Note:** `id` is required. A call without `id` returns HTTP 405 (`statusCode` 793405) instead of a null-parameter error.

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
|schedulePlugin| object | The schedule plugin task. The fields are the same as the request body of the [Add or Update Schedule Plugin Task API](Add%20or%20Update%20Schedule%20Plugin%20Task%20API.md), plus `lastExecId`. |
|schedulePlugin.id| string | The ID of the task. |
|schedulePlugin.name| string | The name of the task. |
|schedulePlugin.enabled| bool | Whether the task is enabled. |
|schedulePlugin.description| string | The description of the task. |
|schedulePlugin.schedule| object | The schedule of the task (`startDate`, `timeZone`, `start`, `enableEnd`, `endDate`, `end`, `frequency`). |
|schedulePlugin.pluginGroup| object | The plugins to run (`isParallel`, `plugins[].path`, `plugins[].input`, `plugins[].enabled`). |
|schedulePlugin.lastExecId| string | The ID of the last execution of this task. |

>>**Note:** `lastExecId` shows that the task has run, but it cannot be passed to the existing plugin status or log APIs (`/V1/CMDB/Plugin/Status/{id}`, `/V1/CMDB/Plugin/Log/{id}`); those return `NoJob` for a scheduled execution ID.

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
        },
        "lastExecId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
}
```

# Full Example

```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

nb_url = "http://192.168.28.79"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
token = "bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7"
headers["Token"] = token

params = {"id": "43c5c65e-810e-4ab6-9d80-fdec285d3d54"}

getSchedulePlugin_URL = nb_url + "/ServicesAPI/API/V1/CMDB/SchedulePlugin"

def getSchedulePlugin(getSchedulePlugin_URL, params, headers):
    try:
        # Do the HTTP request
        response = requests.get(getSchedulePlugin_URL, params=params, headers=headers, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Get Schedule Plugin Task! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = getSchedulePlugin(getSchedulePlugin_URL, params, headers)
pprint.pprint(result)
```
```python
    {'schedulePlugin': {'description': 'Runs health check plugin daily',
                        'enabled': True,
                        'id': '43c5c65e-810e-4ab6-9d80-fdec285d3d54',
                        'lastExecId': 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
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

# cURL Code from Postman
```python
curl -X GET \
  'http://192.168.28.79/ServicesAPI/API/V1/CMDB/SchedulePlugin?id=43c5c65e-810e-4ab6-9d80-fdec285d3d54' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7'
```

# Error Examples
## Error Example 1: Missing `id`
```python
Input:
    
        params = {} # id is required.
          
Response:   # HTTP 405
    
    ('Failed to Get Schedule Plugin Task! - '
    '{"statusCode":791009,"statusDescription":"Invalid value for parameter '
    '\'String\'."}')
```
## Error Example 2: Schedule plugin task not found
```python
Input:
    
        params = {"id": "00000000-0000-0000-0000-000000000000"} # There is no task with this ID.
          
Response:   # HTTP 500
    
    ('Failed to Get Schedule Plugin Task! - '
    '{"statusCode":793000,"statusDescription":"Unexpected Error."}')

```
