# GES Assessment Library API Design

## ***POST*** /V1/GES/Assessment-Library/Generate
This API call is used to start **Generate Golden Assets** on the Golden Assessment Library (GAL) of the current domain. Using the live network, it installs the selected library assets into the domain and generates golden assets from them. It is the API equivalent of **Generate Golden Assets** on the **Assessment Library** page in IE, with the same generate options.

Run [Analyze Relevance](Analyze%20Relevance%20API.md) first and wait for it to end. By default (`requireRelevantDevice` = `true`), assets without relevant devices are skipped. An asset has relevant devices when its `relevantDeviceScope.type` is `AllNetwork` or its `relevantDeviceCount` is greater than 0.

<b>Note</b>:
- Available from R13.0.1.0.
- This API acts on the current domain of your session only. It has no domain parameter. To work in another domain, first call the [Specify A Working Domain API](../Authentication%20and%20Authorization/Specify%20A%20Working%20Domain%20API.md) (`PUT /V1/Session/CurrentDomain`), then call this API. One call never spans domains.
- Permission: **Shared Resource Management** and **Access to Live Network** in the current domain.
- The task runs asynchronously. The response returns a task ID as soon as the task has started. Use the [Get Assessment Library Task Status API](Get%20Assessment%20Library%20Task%20Status%20API.md) to wait for it to end, then check `status` and `localVersion` of the assets with the [List Assessment Library Assets API](List%20Assessment%20Library%20Assets%20API.md).
- A Generate of many assets can run for a long time (more than an hour for a full library). Poll the status at a long interval.
- Only one Generate task can run in a domain at a time, whether it was started through this API or in IE. A second call returns 794001 with the running task's ID.
- Only eligible assets are generated, the same as **select all** on the Assessment Library page: assets with status `Ignored` or `ToBeInstalledLocalConflict` are excluded, and so are the ARs of such an AF. Assets with status `InstalledLocalResourceModified` (installed and changed locally) are also skipped, so local changes are kept, unless `overwriteLocalModified` is `true`.
- Each call that passes the permission, list-entry and Access to Live Network checks writes one entry to the system audit log (module Golden Assessment), whether the task started or not (`Task ID: not started` when it did not start), e.g. `Library Generate (API) initiated. Library versions: 2.0; Assessment feature IDs: null; requireRelevantDevice: true; verifyGoldenConfig: true; runMemberIntent: true; importAIInsight: true; overwriteLocalModified: false; Task ID: 1f9e8d7c6b5a49382716a5b4c3d2e1f0.` The entry can take up to about a minute to appear.
- You can see the task details in IE.

## Detail Information

> **Title** : Generate Golden Assets API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/GES/Assessment-Library/Generate

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

> The request body is optional. Every field is optional; a missing field takes the default shown.

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|libraryVersions^ | list of string | Library versions to generate, e.g. `["2.0"]`. Exact match (case-sensitive, not trimmed) on the `libraryVersion` returned by the [List Assessment Library Assets API](List%20Assessment%20Library%20Assets%20API.md). An AR also matches when the library version of its AF is listed. <br>Default: all versions. |
|afIds^ | list of string | IDs of the assessment features (AF) to generate. Each AF is generated with all its eligible ARs. Use the `id` of an `AF` item returned by the [List Assessment Library Assets API](List%20Assessment%20Library%20Assets%20API.md). <br>Default: all AFs. |
|requireRelevantDevice^ | bool | Generate only the assets that have relevant devices. <br>Default: `true`. |
|verifyGoldenConfig^ | bool | Verify the golden configuration after generating. <br>Default: `true`. |
|runMemberIntent^ | bool | Run the member intents after generating. <br>Default: `true`. |
|importAIInsight^ | bool | Import AI insights. <br>Default: `true`. |
|overwriteLocalModified^ | bool | Also generate the assets with local modifications (`InstalledLocalResourceModified`), overwriting the local changes. <br>Default: `false`. |

> When `libraryVersions` and `afIds` are both given, an asset must match both. A null or blank entry, or an `afIds` entry that is an AR ID or not an AF of the library, rejects the whole call with 791001 and nothing starts.

> ### ***Example***


```python
body = {
    "afIds": ["6a1f0c2e9b7d4a0012cd34ef"],
    "requireRelevantDevice": True,
    "verifyGoldenConfig": True,
    "runMemberIntent": False,
    "importAIInsight": True,
    "overwriteLocalModified": False
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
|runningTaskId| string | Only with statusCode 794001: the ID of the Generate task that is already running, when known. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> ***Example***


```python
{
    "taskId": "1f9e8d7c6b5a49382716a5b4c3d2e1f0",
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

# Generate one AF without running the member intents. Other options keep their defaults.
body = {
    "afIds": ["6a1f0c2e9b7d4a0012cd34ef"],
    "runMemberIntent": False
}

generate_URL = nb_url + "/ServicesAPI/API/V1/GES/Assessment-Library/Generate"

def generateGoldenAssets(generate_URL, body, headers):
    try:
        # Do the HTTP request
        response = requests.post(generate_URL, headers=headers, data=json.dumps(body), verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Generate Golden Assets! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = generateGoldenAssets(generate_URL, body, headers)
pprint.pprint(result)

# Keep the task ID to check the task with the Get Assessment Library Task Status API
task_id = result["taskId"]
```
```python
    {'statusCode': 790200,
     'statusDescription': 'Success.',
     'taskId': '1f9e8d7c6b5a49382716a5b4c3d2e1f0'}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.28.79/ServicesAPI/API/V1/GES/Assessment-Library/Generate \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7' \
  -d '{
    "afIds": ["6a1f0c2e9b7d4a0012cd34ef"],
    "runMemberIntent": false
}'
```

# Error Examples
## Error Example 1: No selected asset has relevant devices
```python
Input:
    
        body = {"requireRelevantDevice": True} # Analyze Relevance has not been run, or found no relevant devices.
          
Response:
    
    Failed to Generate Golden Assets! - {"statusCode":791006,"statusDescription":"None of the selected library assets have relevant devices. Run Analyze first, or set requireRelevantDevice to false."}
```
## Error Example 2: Every selected asset has local modifications
```python
Input:
    
        body = {"afIds": ["6a1f0c2e9b7d4a0012cd34ef"]} # This AF is InstalledLocalResourceModified; overwriteLocalModified defaults to false.
          
Response:
    
    Failed to Generate Golden Assets! - {"statusCode":791006,"statusDescription":"All selected library assets have local modifications. Set overwriteLocalModified to true to include them in generation."}
```
> Setting `overwriteLocalModified` to `true` overwrites the local changes.

## Error Example 3: Generate is already running in the domain
```python
Input:
    
        body = {} # Another Generate task, started through the API or in IE, has not ended.
          
Response:
    
    Failed to Generate Golden Assets! - {"runningTaskId":"1f9e8d7c6b5a49382716a5b4c3d2e1f0","statusCode":794001,"statusDescription":"Another task is already running. Task ID: 1f9e8d7c6b5a49382716a5b4c3d2e1f0."}
```
> Wait until the running task ends, then retry.

## Error Example 4: No Shared Resource Management permission
```python
Input:
    
        token = "<token of a user without Shared Resource Management in the current domain>"
          
Response:
    
    Failed to Generate Golden Assets! - {"statusCode":795003,"statusDescription":"Insufficient permissions: the current user has insufficient permissions to perform the requested operation."}
```
## Error Example 5: The task failed to start
```python
Input:
    
        body = {} # The server could not start the task, e.g. a Worker Server queue failure.
          
Response:
    
    Failed to Generate Golden Assets! - {"statusCode":794005,"statusDescription":"Failed to start the Library Generate task. Error code 230492."}
```
> No task exists. Retry later. The `Error code N.` sentence is present only when the underlying product error code is known.

# Error Codes

> When several checks fail, the first one is returned, in this order: permission; blank list entries; Access to Live Network; `afIds`; an empty selection (791006); starting the task (794001, 794005).

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | The task started. |
| 791001 | 400 | A null or blank entry in `libraryVersions` or `afIds` (`Invalid parameter: '{field}'.`), or an `afIds` entry that is not an AF of the library. Nothing starts. |
| 791006 | 400 | Nothing can be generated: no eligible asset (`No library asset is eligible for this operation.`), no eligible asset with the requested versions (`No library asset was found matching the requested versions.`), every selected asset has local modifications, or no selected asset has relevant devices. Nothing starts. |
| 794001 | 409 | A Generate task is already running in the domain. See `runningTaskId`. When the ID of the running task is unknown, the description is `Another task is already running.` and `runningTaskId` is omitted. |
| 794005 | 400 | The task could not be started. No task exists. |
| 795003 | 403 | The user lacks Shared Resource Management or Access to Live Network in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793001 | 500 | `An unexpected internal error occurred. Please retry. If the issue persists, contact support with the time of the call.` |
