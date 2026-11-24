# GES Assessment Library API Design

## ***POST*** /V1/GES/Assessment-Library/Uninstall
This API call is used to start uninstalling installed assessment assets of the Golden Assessment Library (GAL) from the current domain. It is the API equivalent of **Uninstall** on the **Assessment Library** page in IE.

<b>Note</b>:
- Available from R13.0.1.0.
- A selector is **required**, so a call can never remove everything by accident. Send `localVersions` **or** `libraryVersions`, and/or `afIds`. `localVersions` and `libraryVersions` cannot be sent together. A missing body or an empty body (`{}`) is rejected.
- This API removes the assessment assets from the domain. It does not uninstall the library from NetBrain Download.
- This API acts on the current domain of your session only. It has no domain parameter. To work in another domain, first call the [Specify A Working Domain API](../Authentication%20and%20Authorization/Specify%20A%20Working%20Domain%20API.md) (`PUT /V1/Session/CurrentDomain`), then call this API. One call never spans domains.
- Permission: **Shared Resource Management** in the current domain.
- The task runs asynchronously. The response returns a task ID as soon as the task has started. Use the [Get Assessment Library Task Status API](Get%20Assessment%20Library%20Task%20Status%20API.md) to wait for it to end, then check the assets with the [List Assessment Library Assets API](List%20Assessment%20Library%20Assets%20API.md).
- Only one Uninstall task can run in a domain at a time, whether it was started through this API or in IE. A second call returns 794001 with the running task's ID.
- Only eligible assets are uninstalled: assets with status `Ignored` or `ToBeInstalledLocalConflict` are excluded, and so are the ARs of such an AF.
- Each call that passes the permission, request-body and list-entry checks writes one entry to the system audit log (module Golden Assessment), whether the task started or not (`Task ID: not started` when it did not start), e.g. `Library Uninstall (API) initiated. Local versions: 1.0; Library versions: null; Assessment feature IDs: null; Task ID: 5e4d3c2b1a0948f7e6d5c4b3a2918070.` The entry can take up to about a minute to appear.

## Detail Information

> **Title** : Uninstall Assessment Library Assets API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/GES/Assessment-Library/Uninstall

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

> The body is required, and at least one field must be non-empty. See the selector rules below.

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|localVersions^ | list of string | Uninstall the installed assets whose **local** version is listed, e.g. `["1.0"]`. Exact match on the `localVersion` returned by the [List Assessment Library Assets API](List%20Assessment%20Library%20Assets%20API.md). Cannot be combined with `libraryVersions`. |
|libraryVersions^ | list of string | Uninstall the installed assets whose **library** version is listed, e.g. `["2.0"]`. Exact match on `libraryVersion`. Cannot be combined with `localVersions`. |
|afIds^ | list of string | Uninstall the installed assets of these assessment features (AF), all versions, with all their ARs. Use the `id` of an `AF` item returned by the [List Assessment Library Assets API](List%20Assessment%20Library%20Assets%20API.md). |

> A null or blank entry in any list, or an `afIds` entry that is an AR ID or not an AF of the library, rejects the whole call with 791001 and nothing starts.

> ### ***Selector rules***

|**Request body**|**Result**|
|------|------|
|<img width=300/>|<img width=500/>|
| No body | 791000. Nothing starts. |
| `{}`, or every list empty | 791001. Nothing starts. |
| `localVersions` and `libraryVersions` both non-empty | 791001. Nothing starts. |
| `localVersions` or `libraryVersions` | Uninstall the installed assets of those versions. |
| `afIds` | Uninstall the installed assets of those AFs, all versions. |
| `localVersions` or `libraryVersions`, and `afIds` | Uninstall the installed assets of those versions within those AFs. |

> ### ***Example***


```python
body = {
    "localVersions": ["1.0"],
    "afIds": ["6a1f0c2e9b7d4a0012cd34ef"]
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
|runningTaskId| string | Only with statusCode 794001: the ID of the Uninstall task that is already running, when known. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> ***Example***


```python
{
    "taskId": "5e4d3c2b1a0948f7e6d5c4b3a2918070",
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

# Uninstall the assets installed at local version 1.0.
body = {
    "localVersions": ["1.0"]
}

uninstall_URL = nb_url + "/ServicesAPI/API/V1/GES/Assessment-Library/Uninstall"

def uninstallAssessmentLibraryAssets(uninstall_URL, body, headers):
    try:
        # Do the HTTP request
        response = requests.post(uninstall_URL, headers=headers, data=json.dumps(body), verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Uninstall Assessment Library Assets! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = uninstallAssessmentLibraryAssets(uninstall_URL, body, headers)
pprint.pprint(result)

# Keep the task ID to check the task with the Get Assessment Library Task Status API
task_id = result["taskId"]
```
```python
    {'statusCode': 790200,
     'statusDescription': 'Success.',
     'taskId': '5e4d3c2b1a0948f7e6d5c4b3a2918070'}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.28.79/ServicesAPI/API/V1/GES/Assessment-Library/Uninstall \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7' \
  -d '{
    "localVersions": ["1.0"]
}'
```

# Error Examples
## Error Example 1: No selector
```python
Input:
    
        body = {} # At least one of localVersions, libraryVersions or afIds is required.
          
Response:
    
    Failed to Uninstall Assessment Library Assets! - {"statusCode":791001,"statusDescription":"Invalid parameter: 'request'. Specify at least one of the following: localVersions, libraryVersions, or afIds."}
```
## Error Example 2: Both `localVersions` and `libraryVersions`
```python
Input:
    
        body = {"localVersions": ["1.0"], "libraryVersions": ["2.0"]} # Send only one of them.
          
Response:
    
    Failed to Uninstall Assessment Library Assets! - {"statusCode":791001,"statusDescription":"Invalid parameter: 'libraryVersions'. Specify either localVersions or libraryVersions, but not both."}
```
## Error Example 3: No request body
```python
Input:
    
        response = requests.post(uninstall_URL, headers=headers, verify=False) # No body at all.
          
Response:
    
    Failed to Uninstall Assessment Library Assets! - {"statusCode":791000,"statusDescription":"Invalid parameter: 'request' cannot be null."}
```
## Error Example 4: None of the selected assets is installed
```python
Input:
    
        body = {"afIds": ["6a1f0c2e9b7d4a0012cd34ef"]} # This AF is not installed in the domain.
          
Response:
    
    Failed to Uninstall Assessment Library Assets! - {"statusCode":791006,"statusDescription":"None of the selected library assets are installed."}
```
## Error Example 5: Uninstall is already running in the domain
```python
Input:
    
        body = {"localVersions": ["1.0"]} # Another Uninstall task, started through the API or in IE, has not ended.
          
Response:
    
    Failed to Uninstall Assessment Library Assets! - {"runningTaskId":"5e4d3c2b1a0948f7e6d5c4b3a2918070","statusCode":794001,"statusDescription":"Another task is already running. Task ID: 5e4d3c2b1a0948f7e6d5c4b3a2918070."}
```
> Wait until the running task ends, then retry.

# Error Codes

> When several checks fail, the first one is returned, in this order: permission; a missing body and blank list entries; the selector rules; `afIds`; an empty selection (791006); starting the task (794001, 794005).

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | The task started. |
| 791000 | 400 | No request body. |
| 791001 | 400 | No selector; both version lists; a null or blank list entry (`Invalid parameter: '{field}'.`); or an `afIds` entry that is not an AF of the library. Nothing starts. |
| 791006 | 400 | No eligible asset (`No library asset is eligible for this operation.`), no eligible asset with the requested versions (`No library asset was found matching the requested versions.`), or none of the selected assets is installed. Nothing starts. |
| 794001 | 409 | An Uninstall task is already running in the domain. See `runningTaskId`. When the ID of the running task is unknown, the description is `Another task is already running.` and `runningTaskId` is omitted. |
| 794005 | 400 | The task could not be started: `Failed to start the Library Uninstall task. Error code {N}.` No task exists. Retry later. |
| 795003 | 403 | The user lacks Shared Resource Management in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793001 | 500 | `An unexpected internal error occurred. Please retry. If the issue persists, contact support with the time of the call.` |
