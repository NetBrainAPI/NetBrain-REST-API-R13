# GES Assessment Library API Design

## ***POST*** /V1/GES/Assessment-Library/Analyze
This API call is used to start **Analyze Relevance** on the Golden Assessment Library (GAL) of the current domain. Using the live network, it finds which devices each selected assessment feature (AF) and assessment rule (AR) applies to. Nothing is installed. It is the API equivalent of **Analyze Relevance** on the **Assessment Library** page in IE.

Run it before [Generate Golden Assets](Generate%20Golden%20Assets%20API.md): by default, Generate skips assets that have no relevant devices.

<b>Note</b>:
- Available from R13.0.1.0.
- This API acts on the current domain of your session only. It has no domain parameter. To work in another domain, first call the [Specify A Working Domain API](../Authentication%20and%20Authorization/Specify%20A%20Working%20Domain%20API.md) (`PUT /V1/Session/CurrentDomain`), then call this API. One call never spans domains.
- Permission: **Domain User** and **Access to Live Network** in the current domain.
- The task runs asynchronously. The response returns a task ID as soon as the task has started. Use the [Get Assessment Library Task Status API](Get%20Assessment%20Library%20Task%20Status%20API.md) to wait for it to end, then read `relevantDeviceScope` and `relevantDeviceCount` with the [List Assessment Library Assets API](List%20Assessment%20Library%20Assets%20API.md).
- Only one Analyze task can run in a domain at a time, whether it was started through this API or in IE. A second call returns 794001 with the running task's ID.
- Only eligible assets are analyzed, the same as **select all** on the Assessment Library page: assets with status `Ignored` or `ToBeInstalledLocalConflict` are excluded, and so are the ARs of such an AF.
- Each call that passes the permission, list-entry and Access to Live Network checks writes one entry to the system audit log (module Golden Assessment), whether the task started or not (`Task ID: not started` when it did not start), e.g. `Library Analyze (API) initiated. Library versions: 2.0; Assessment feature IDs: null; Task ID: 8c2d41f7a3b94e6f9d0a1b2c3d4e5f60.` The entry can take up to about a minute to appear.
- You can see the task details in IE.

## Detail Information

> **Title** : Analyze Relevance API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/GES/Assessment-Library/Analyze

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

> The request body is optional. Without a body, or with an empty body, every eligible asset in the library is analyzed.

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|libraryVersions^ | list of string | Library versions to analyze, e.g. `["2.0"]`. Exact match (case-sensitive, not trimmed) on the `libraryVersion` returned by the [List Assessment Library Assets API](List%20Assessment%20Library%20Assets%20API.md). An AR also matches when the library version of its AF is listed. <br>Default: all versions. |
|afIds^ | list of string | IDs of the assessment features (AF) to analyze. Each AF is analyzed with all its ARs. Use the `id` of an `AF` item returned by the [List Assessment Library Assets API](List%20Assessment%20Library%20Assets%20API.md). <br>Default: all AFs. |

> When both fields are given, an asset must match both. A null or blank entry, or an `afIds` entry that is an AR ID or not an AF of the library, rejects the whole call with 791001 and nothing starts.

> ### ***Example***


```python
body = {
    "libraryVersions": ["2.0"]
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
|taskId| string | The ID of the started task. Pass it to the [Get Assessment Library Task Status API](Get%20Assessment%20Library%20Task%20Status%20API.md). Not returned on error. |
|runningTaskId| string | Only with statusCode 794001: the ID of the Analyze task that is already running, when known. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> ***Example***


```python
{
    "taskId": "8c2d41f7a3b94e6f9d0a1b2c3d4e5f60",
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

# Analyze library version 2.0 only. Send {} (or no body) to analyze every eligible asset.
body = {
    "libraryVersions": ["2.0"]
}

analyze_URL = nb_url + "/ServicesAPI/API/V1/GES/Assessment-Library/Analyze"

def analyzeRelevance(analyze_URL, body, headers):
    try:
        # Do the HTTP request
        response = requests.post(analyze_URL, headers=headers, data=json.dumps(body), verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Analyze Relevance! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = analyzeRelevance(analyze_URL, body, headers)
pprint.pprint(result)

# Keep the task ID to check the task with the Get Assessment Library Task Status API
task_id = result["taskId"]
```
```python
    {'statusCode': 790200,
     'statusDescription': 'Success.',
     'taskId': '8c2d41f7a3b94e6f9d0a1b2c3d4e5f60'}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.28.79/ServicesAPI/API/V1/GES/Assessment-Library/Analyze \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7' \
  -d '{
    "libraryVersions": ["2.0"]
}'
```

# Error Examples
## Error Example 1: Analyze is already running in the domain
```python
Input:
    
        body = {} # Another Analyze task, started through the API or in IE, has not ended.
          
Response:
    
    Failed to Analyze Relevance! - {"runningTaskId":"8c2d41f7a3b94e6f9d0a1b2c3d4e5f60","statusCode":794001,"statusDescription":"Another task is already running. Task ID: 8c2d41f7a3b94e6f9d0a1b2c3d4e5f60."}
```
> Wait until the running task ends, then retry. When the running task's ID is unknown, `runningTaskId` is not returned and the description is `Another task is already running.`

## Error Example 2: No asset has the requested version
```python
Input:
    
        body = {"libraryVersions": ["9.9"]} # No eligible asset has library version 9.9.
          
Response:
    
    Failed to Analyze Relevance! - {"statusCode":791006,"statusDescription":"No library asset was found matching the requested versions."}
```
## Error Example 3: `afIds` holds an AR ID
```python
Input:
    
        body = {"afIds": ["6a1f0c2e9b7d4a0012cd3501"]} # An AR ID. Only AF IDs are accepted.
          
Response:
    
    Failed to Analyze Relevance! - {"statusCode":791001,"statusDescription":"Invalid parameter: 'afIds'. The following IDs are not valid assessment feature IDs: 6a1f0c2e9b7d4a0012cd3501."}
```
## Error Example 4: No Access to Live Network
```python
Input:
    
        token = "<token of a user without Access to Live Network in the current domain>"
          
Response:
    
    Failed to Analyze Relevance! - {"statusCode":795003,"statusDescription":"Insufficient permissions: the current user has insufficient permissions to perform the requested operation."}
```
## Error Example 5: The task failed to start
```python
Input:
    
        body = {} # The server could not start the task, e.g. a Worker Server queue failure.
          
Response:
    
    Failed to Analyze Relevance! - {"statusCode":794005,"statusDescription":"Failed to start the Library Analyze task. Error code 230492."}
```
> No task exists. Retry later. The `Error code N.` sentence is present only when the underlying product error code is known.

# Error Codes

> When several checks fail, the first one is returned, in this order: permission; blank list entries; Access to Live Network; `afIds`; an empty selection (791006); starting the task (794001, 794005).

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | The task started. |
| 791001 | 400 | A null or blank entry in `libraryVersions` or `afIds` (`Invalid parameter: '{field}'.`), or an `afIds` entry that is not an AF of the library. Nothing starts. |
| 791006 | 400 | No eligible asset in the selected AFs or the library (`No library asset is eligible for this operation.`), or no eligible asset has the requested versions. Nothing starts. |
| 794001 | 409 | An Analyze task is already running in the domain. See `runningTaskId`. When the ID of the running task is unknown, the description is `Another task is already running.` and `runningTaskId` is omitted. |
| 794005 | 400 | The task could not be started. No task exists. |
| 795003 | 403 | The user lacks Domain User or Access to Live Network in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793001 | 500 | `An unexpected internal error occurred. Please retry. If the issue persists, contact support with the time of the call.` |
