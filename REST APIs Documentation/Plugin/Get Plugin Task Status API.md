# Plugin API Design

## ***GET*** /V1/CMDB/Plugin/Status/{executionId}
This API call is used to get the status of a plugin run started by the [Run Debug Plugin API](Run%20Debug%20Plugin%20API.md).

<b>Note</b>:
- `{executionId}` must be the `executionId` returned by the Run Debug Plugin API. This API does not track schedule plugin tasks: a schedule task's `lastExecId` from the [Get Schedule Plugin Task API](Get%20Schedule%20Plugin%20Task%20API.md) returns `status` `-1`.
- The Run Debug Plugin API returns before the run has started. Until the worker starts the run, this API returns `status` `-1`, so poll it instead of reading `-1` right after Run as a wrong ID. See **Poll until the run finishes** below.
- An unknown ID also returns `status` `-1` with `790200` (Success), not an error code.

## Detail Information

> **Title** : Get Plugin Task Status API<br>

> **Version** : 27/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Plugin/Status/{executionId}

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
|status| integer | The status of the run. See **Status values** below. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> **Status values**

|**status**|**Name**|**Meaning**|**Finished**|
|------|------|------|------|
|<img width=50/>|<img width=150/>|<img width=350/>|<img width=100/>|
|-1 | NoJob | No run record yet: the run has not started, or the ID is unknown. | No |
|3 | Executing | The plugin is running. | No |
|4 | CompletedNormally | The plugin finished and `run(input)` returned `True`. | Yes, succeeded |
|5 | CompletedWithError | The run failed: the plugin path could not be found, `run(input)` raised an exception, or `run(input)` returned `False` or nothing. Check the run's log with the [Get Plugin Task Log API](Get%20Plugin%20Task%20Log%20API.md). | Yes, failed |
|6 | CompletedWithCrash | The run crashed. | Yes, failed |
|7 | Canceled | The run was stopped. | Yes |
|8 | CompletedWithWarning | The run succeeded with a warning. | Yes, succeeded |

>>**Note:** A run started by the Run Debug Plugin API ends in `4`, `5` or `7`. Values `6` and `8` are part of the same status list but are not set for these runs. Values `1` and `2` are not used. `0` (NoStart) appears only in the Run Debug Plugin API response.

> ***Example***


```python
{
    "executionId": "85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "status": 4,
    "statusCode": 790200,
    "statusDescription": "Success."
}
```

> ### ***Poll until the run finishes***

A run goes through these steps:
1. The Run Debug Plugin API returns the `executionId`. This API returns `status` `-1`.
2. The worker starts the run. This API returns `status` `3` (Executing).
3. The run finishes. This API returns `4`, `5` or `7`.

Keep polling while `status` is `-1` or `3`, and stop when it is `4` or higher. Because an unknown ID also returns `-1`, set a limit on how long you keep polling.

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

getPluginStatus_URL = nb_url + "/ServicesAPI/API/V1/CMDB/Plugin/Status/" + executionId

def getPluginTaskStatus(getPluginStatus_URL, headers):
    try:
        # Do the HTTP request
        response = requests.get(getPluginStatus_URL, headers=headers, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Get Plugin Task Status! - " + str(response.text))
    except Exception as e:
        return (str(e))

status_names = {-1: "NoJob", 3: "Executing", 4: "CompletedNormally", 5: "CompletedWithError",
                6: "CompletedWithCrash", 7: "Canceled", 8: "CompletedWithWarning"}

# Poll every 5 seconds, up to 5 minutes, until the run finishes (status 4 or higher)
for i in range(60):
    result = getPluginTaskStatus(getPluginStatus_URL, headers)
    if not isinstance(result, dict):
        print(result)
        break
    status = result.get("status")
    print(status, status_names.get(status))
    if status is not None and status >= 4:
        pprint.pprint(result)
        break
    time.sleep(5)
```
```python
    4 CompletedNormally
    {'executionId': '85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
     'status': 4,
     'statusCode': 790200,
     'statusDescription': 'Success.'}
```

# cURL Code from Postman
```python
curl -X GET \
  http://192.168.28.79/ServicesAPI/API/V1/CMDB/Plugin/Status/85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7'
```

# Error Examples
## Error Example 1: Unknown ID, or a schedule task's `lastExecId`
```python
Input:
    
        executionId = "34f9f68d-xxxx-xxxx-xxxx-xxxxxxxxxxxx" # Not an executionId returned by the Run Debug Plugin API.
          
Response:
    
    {'executionId': '34f9f68d-xxxx-xxxx-xxxx-xxxxxxxxxxxx', 'status': -1, 'statusCode': 790200, 'statusDescription': 'Success.'} # -1 = NoJob. Not an error code.
```
