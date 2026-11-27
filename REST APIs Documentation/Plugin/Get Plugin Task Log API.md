# Plugin API Design

## ***GET*** /V1/CMDB/Plugin/Log/{executionId}
This API call is used to get the log of a plugin run started by the [Run Debug Plugin API](Run%20Debug%20Plugin%20API.md). Use it to see why a run ended in status `5` (CompletedWithError).

<b>Note</b>:
- `{executionId}` must be the `executionId` returned by the Run Debug Plugin API. A schedule task's `lastExecId` from the [Get Schedule Plugin Task API](Get%20Schedule%20Plugin%20Task%20API.md) is not supported; there is no API for the log of a scheduled run.
- Every log entry of the run is returned in one response. There is no paging.
- The ID is not checked: an unknown ID returns an empty `logs` list with `790200` (Success). An empty list is also returned before the run starts. Use the [Get Plugin Task Status API](Get%20Plugin%20Task%20Status%20API.md) to tell "no logs yet" from "no such run".

## Detail Information

> **Title** : Get Plugin Task Log API<br>

> **Version** : 27/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Plugin/Log/{executionId}

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

> No request body.

## Path Parameters(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|executionId* | string | The `executionId` returned by the [Run Debug Plugin API](Run%20Debug%20Plugin%20API.md). |

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
|executionId| string | The ID of the run. |
|logs| list of object | Every log entry of the run, oldest first. Empty if the run has not started or the ID is unknown. |
|logs[].id| string | The ID of the log entry. |
|logs[].execId| string | The ID of the run the entry belongs to. |
|logs[].level| integer | The log level: <br>▪ `0` - Debug <br>▪ `1` - Information <br>▪ `2` - Warning <br>▪ `3` - Error |
|logs[].msg| string | The log message. |
|logs[].time| string | When the entry was written (UTC). |
|logs[].errorDetails| string | Error details. Present only when set. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

>>**Note:** A run's log starts with `Begin: run plugin "<path>".` and ends with `End: run plugin "<path>".` followed by the result:
>>- `The result is successful.` (level `1`): the run ended in status `4`.
>>- `The result is failed.` (level `3`): the run ended in status `5`, for example because `run(input)` returned `False` or returned nothing. If `run(input)` raised an exception, the traceback is also logged at level `3`.
>>- `The result is canceled.`: the run was stopped (status `7`).
>>
>>Messages the plugin writes with `pluginfw.AddLog(...)` appear between the Begin and End entries, at level `1`. Output from Python's `print()` is **not** written to this log.

> ***Example***


```python
{
    "executionId": "85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "logs": [
        {
            "execId": "85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "id": "4fe0db7f-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "level": 1,
            "msg": "Begin: run plugin \"My Plugins/APIPlugins/my_plugin?domainId=510a7b9f-xxxx-xxxx-xxxx-xxxxxxxxxxxx\".",
            "time": "2026-11-27T16:03:21.846Z"
        },
        {
            "execId": "85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "id": "96e4a90b-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "level": 1,
            "msg": "hello from my_plugin",
            "time": "2026-11-27T16:03:21.882Z"
        },
        {
            "execId": "85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "id": "5eff0c01-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "level": 1,
            "msg": "End: run plugin \"My Plugins/APIPlugins/my_plugin?domainId=510a7b9f-xxxx-xxxx-xxxx-xxxxxxxxxxxx\".  The result is successful.",
            "time": "2026-11-27T16:03:21.885Z"
        }
    ],
    "statusCode": 790200,
    "statusDescription": "Success."
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

executionId = "85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx"   # "executionId" returned by the Run Debug Plugin API

getPluginLog_URL = nb_url + "/ServicesAPI/API/V1/CMDB/Plugin/Log/" + executionId

def getPluginTaskLog(getPluginLog_URL, headers):
    try:
        # Do the HTTP request
        response = requests.get(getPluginLog_URL, headers=headers, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Get Plugin Task Log! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = getPluginTaskLog(getPluginLog_URL, headers)

level_names = {0: "DEBUG", 1: "INFO", 2: "WARNING", 3: "ERROR"}

# Print one line per log entry
if isinstance(result, dict):
    for log in result.get("logs", []):
        print(log.get("time"), level_names.get(log.get("level")), log.get("msg"))
else:
    print(result)
```
```python
    2026-11-27T16:03:21.846Z INFO Begin: run plugin "My Plugins/APIPlugins/my_plugin?domainId=510a7b9f-xxxx-xxxx-xxxx-xxxxxxxxxxxx".
    2026-11-27T16:03:21.882Z INFO hello from my_plugin
    2026-11-27T16:03:21.885Z INFO End: run plugin "My Plugins/APIPlugins/my_plugin?domainId=510a7b9f-xxxx-xxxx-xxxx-xxxxxxxxxxxx".  The result is successful.
```

# cURL Code from Postman
```python
curl -X GET \
  http://192.168.28.79/ServicesAPI/API/V1/CMDB/Plugin/Log/85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7'
```

# Error Examples
## Error Example 1: Unknown `executionId`
```python
Input:
    
        executionId = "00000000-0000-0000-0000-000000000000" # Not an executionId returned by the Run Debug Plugin API.
          
Response:
    
    {'executionId': '00000000-0000-0000-0000-000000000000', 'logs': [], 'statusCode': 790200, 'statusDescription': 'Success.'} # Empty list, not an error code.
```
## Error Example 2: Log of a failed run (`run(input)` did not return `True`)
```python
Input:
    
        # The plugin's run(input) only calls print() and has no "return True", so the run ended in status 5 (CompletedWithError).
        executionId = "a1593ec1-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          
Response:
    
    2026-11-27T15:54:07.745Z INFO Begin: run plugin "My Plugins/APIPlugins/my_plugin?domainId=510a7b9f-xxxx-xxxx-xxxx-xxxxxxxxxxxx".
    2026-11-27T15:54:07.759Z ERROR End: run plugin "My Plugins/APIPlugins/my_plugin?domainId=510a7b9f-xxxx-xxxx-xxxx-xxxxxxxxxxxx".  The result is failed.
    # The print() output does not appear in the log. Use pluginfw.AddLog(...) to write to the log, and end run(input) with "return True".
```
