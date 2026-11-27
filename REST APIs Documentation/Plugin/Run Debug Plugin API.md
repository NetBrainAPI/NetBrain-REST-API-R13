# Plugin API Design

## ***POST*** /V1/CMDB/Plugin/Run
This API call is used to run a plugin once, on demand. It is the API equivalent of **Debug Run** in Plugin Manager (right-click a plugin > **Debug Run**).

<b>Note</b>:
- This API is for running a plugin once. It cannot create or run a schedule plugin task. To run a plugin on a schedule, use the [Add or Update Schedule Plugin Task API](Add%20or%20Update%20Schedule%20Plugin%20Task%20API.md).
- The call only submits the run and returns at once; it does not wait for the plugin. Use the `executionId` in the response with the [Get Plugin Task Status API](Get%20Plugin%20Task%20Status%20API.md) to follow the run.
- Only `path` is checked. The API does not check that the plugin exists, is built-in, is locked or is already running, and every call starts a new run. A plugin that cannot be found is reported by the run itself, which ends in status `5` (CompletedWithError), not by an error code from this API.
- Whether the run succeeds depends on the plugin's code, not on `input`. The run ends in status `4` (CompletedNormally) when the plugin's `run(input)` function returns `True`, and in status `5` (CompletedWithError) when it returns `False`, returns nothing (no `return` statement), or raises an exception. Always end `run(input)` with an explicit `return True`. Use the [Get Plugin Task Log API](Get%20Plugin%20Task%20Log%20API.md) to see why a run failed. See the [Add or Update Plugin API](Add%20or%20Update%20Plugin%20API.md) for a sample plugin.
- This API applies no memory limit to the run. A schedule plugin task applies its plugin group's memory limit (500 MB by default in the UI), so a plugin that succeeds here can still fail when scheduled.

## Detail Information

> **Title** : Run Debug Plugin API<br>

> **Version** : 27/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Plugin/Run

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
|path* | string | The full path of the plugin to run. <br>Use the exact `plugin.path` value returned by the [Get Plugin API](Get%20Plugin%20API.md), e.g. `My Plugins/APIPlugins/my_plugin?domainId=<domain-guid>`. Private plugins carry the `?domainId=` suffix. <br>There is no `id` field: the plugin is found by `path` only. |
|input^ | string | The input passed to the plugin's `run(input)` function, as text. Same as the **Input** box in the Debug Run dialog. To pass JSON, send it as a string. |
|execId^ | string | The ID (GUID) to give this run. If omitted, the server generates one. It is returned as `executionId`. |
|debugMode^ | bool | Whether to run the plugin in debug mode. |
|options^ | object | Run options, forwarded to the worker unchanged: <br>▪ `maxMemory` (integer) <br>▪ `maxVirtualMemory` (integer) <br>▪ `maxRunDuration` (integer) <br>**Note:** This API does not enforce these values. They are not applied as memory or time limits on the run. |

> ### ***Example***


```python
body = {
    "path": "My Plugins/APIPlugins/my_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "input": "hello"
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
|executionId| string | The ID of this run (the `execId` you sent, or a generated one). Pass it to the [Get Plugin Task Status API](Get%20Plugin%20Task%20Status%20API.md). |
|status| integer | Always `0` (NoStart). It only means the run was submitted; it does not mean the run was queued or started. Use the Get Plugin Task Status API for the real status. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> ***Example***


```python
{
    "executionId": "85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "status": 0,
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

# Plugin path: use the exact "plugin.path" returned by the Get Plugin API.
# my_plugin's run(input) logs a message and returns True, so this run ends in status 4 (CompletedNormally).
body = {
    "path": "My Plugins/APIPlugins/my_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "input": "hello"   # passed to run(input) as text; it does not decide the result
}

runPlugin_URL = nb_url + "/ServicesAPI/API/V1/CMDB/Plugin/Run"

def runPlugin(runPlugin_URL, body, headers):
    try:
        # Do the HTTP request
        response = requests.post(runPlugin_URL, headers=headers, data=json.dumps(body), verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Run Plugin! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = runPlugin(runPlugin_URL, body, headers)
pprint.pprint(result)

# Keep the execution ID to follow the run with the Get Plugin Task Status API
if isinstance(result, dict) and "executionId" in result:
    executionId = result["executionId"]
```
```python
    {'executionId': '85656772-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
     'status': 0,
     'statusCode': 790200,
     'statusDescription': 'Success.'}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.28.79/ServicesAPI/API/V1/CMDB/Plugin/Run \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7' \
  -d '{
    "path": "My Plugins/APIPlugins/my_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "input": "hello"
}'
```

# Error Examples
## Error Example 1: Missing `path`
```python
Input:
    
        body = {"input": "hello"} # path is required.
          
Response:
    
    ('Failed to Run Plugin! - '
    '{"status":0,"statusCode":791000,"statusDescription":"Null parameter: the '
    'parameter \'path\' cannot be null."}')
```

## Error Example 2: Plugin not found (the call still succeeds)
```python
Input:
    
        "path": "My Plugins/APIPlugins/no_such_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx" # There is no plugin at this path.
          
Response:
    
    {'executionId': '...', 'status': 0, 'statusCode': 790200, 'statusDescription': 'Success.'} # The run is still submitted. It then ends in status 5 (CompletedWithError), and the error is in the run's log.
```
