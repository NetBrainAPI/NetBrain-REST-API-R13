# NetBrain Download API Design

## ***POST*** /V1/NBDownload/Libraries/{libraryId}/Install
This API call is used to start installing a whole NetBrain Download library, or selected tasks of it, in the current domain, with the default install variables of each task. It is the API equivalent of **Install** on the **NetBrain Download** page in IE.

<b>Note</b>:
- Available from R13.0.1.0.
- This API acts on the current domain of your session only. It has no domain parameter. To work in another domain, first call the [Specify A Working Domain API](../Authentication%20and%20Authorization/Specify%20A%20Working%20Domain%20API.md) (`PUT /V1/Session/CurrentDomain`), then call this API.
- Permission: **Shared Resource Management** in the current domain.
- The install runs asynchronously. The call returns as soon as the install has started. Poll the [Get NetBrain Download Library Status API](Get%20NetBrain%20Download%20Library%20Status%20API.md) until `activity` is `0` (Idle), then check the `status` of the tasks with the [List NetBrain Download Library Tasks API](List%20NetBrain%20Download%20Library%20Tasks%20API.md).
- A task that is already installed is installed again from the published library.
- Every prerequisite of a selected task must be selected in the same call or already installed. Otherwise nothing is started (791001).
- Only one library can be installing in a domain at a time, whether the install was started through this API or on the NetBrain Download page.
- Each call that passes the permission and `libraryId` checks writes one entry to the system audit log (module Intent Library), whether the install started or was rejected, e.g. `NetBrain Download install (API) initiated. Library: Cisco Assessment Library; Task IDs: 6a01b7c2d9e8f70011aa22bb.` A rejected call writes `Failed to initiate NetBrain Download install (API). Library: ...; Task IDs: ...; Reason: ...`. `Task IDs` is `not specified` when `taskIds` is omitted or empty.

## Detail Information

> **Title** : Install NetBrain Download Library API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/NBDownload/Libraries/{libraryId}/Install

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Path Parameters(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|libraryId* | string | The `id` of a library returned by the [List NetBrain Download Libraries API](List%20NetBrain%20Download%20Libraries%20API.md). |

## Request body(****required***)

> The request body is optional.

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|taskIds^ | list of string | Tasks to install. Use the `id` of a task returned by the [List NetBrain Download Library Tasks API](List%20NetBrain%20Download%20Library%20Tasks%20API.md). <br>Default (omitted or empty): every task of the library that is not `deleted`. |

> ***Example***


```python
body = {
    "taskIds": ["6a01b7c2d9e8f70011aa22bb"]
}
```

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
|libraryId| string | The requested library ID. |
|taskIds| list of string | The tasks being installed. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

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

library_id = "5f1c2a9e8b3d4c0012ab34cd"   # "id" returned by List NetBrain Download Libraries
body = {
    "taskIds": ["6a01b7c2d9e8f70011aa22bb"]
}

install_URL = nb_url + "/ServicesAPI/API/V1/NBDownload/Libraries/" + library_id + "/Install"

def installNetBrainDownloadLibrary(install_URL, body, headers):
    try:
        # Do the HTTP request
        response = requests.post(install_URL, headers=headers, data=json.dumps(body), verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Install NetBrain Download Library! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = installNetBrainDownloadLibrary(install_URL, body, headers)
pprint.pprint(result)
```
```python
    {'libraryId': '5f1c2a9e8b3d4c0012ab34cd',
     'statusCode': 790200,
     'statusDescription': 'Success.',
     'taskIds': ['6a01b7c2d9e8f70011aa22bb']}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.28.79/ServicesAPI/API/V1/NBDownload/Libraries/5f1c2a9e8b3d4c0012ab34cd/Install \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7' \
  -d '{
    "taskIds": ["6a01b7c2d9e8f70011aa22bb"]
}'
```

# Error Examples
## Error Example 1: Missing prerequisite task
```python
Input:
    
        body = {"taskIds": ["6a01b7c2d9e8f70011aa22bb"]} # Its prerequisite IOS/Common Variables is neither selected nor installed.
          
Response:
    
    Failed to Install NetBrain Download Library! - {"statusCode":791001,"statusDescription":"Invalid parameter: 'taskIds'. Missing prerequisite tasks: IOS/Common Variables. Add them to taskIds or install them first."}
```
> Nothing is started.

## Error Example 2: Unknown or deleted task
```python
Input:
    
        body = {"taskIds": ["000000000000000000000000"]} # Not a task of this library, or a deleted task.
          
Response:
    
    Failed to Install NetBrain Download Library! - {"statusCode":791006,"statusDescription":"Task 000000000000000000000000 does not exist."}
```
## Error Example 3: Every task of the library is deleted
```python
Input:
    
        body = {} # No taskIds, and the library has been withdrawn from the catalogue.
          
Response:
    
    Failed to Install NetBrain Download Library! - {"statusCode":791006,"statusDescription":"Library 5f1c2a9e8b3d4c0012ab34cd has no tasks that can be installed."}
```
## Error Example 4: Another install or uninstall is running
```python
Input:
    
        body = {} # This library is being installed or uninstalled, or another library is being installed in the domain.
          
Response:
    
    Failed to Install NetBrain Download Library! - {"statusCode":794001,"statusDescription":"Another task is already running."}
```
> Poll the Get NetBrain Download Library Status API until `activity` is `0` (Idle), then retry.

## Error Example 5: Invalid default install variable
```python
Input:
    
        body = {"taskIds": ["6a01b7c2d9e8f70011aa22bb"]} # A default install variable of this task is invalid.
          
Response:
    
    Failed to Install NetBrain Download Library! - {"statusCode":791002,"statusDescription":"Invalid value. A default install variable of a selected task is invalid. Correct it on the NetBrain Download page. Error code 230492."}
```
> The `Error code N.` sentence is present only when the underlying product error code is known.

# Error Codes

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | The install started. |
| 791000 | 400 | `libraryId` is blank (only whitespace, e.g. `%20`). |
| 791001 | 400 | A prerequisite of a selected task is neither selected nor installed. Nothing is started. |
| 791002 | 400 | A default install variable of a selected task is invalid. Correct it on the NetBrain Download page. Nothing is started. |
| 791006 | 400 | `Library {libraryId} does not exist.`; `Task {taskId} does not exist.` (not a task of the library, or a `deleted` task); or `Library {libraryId} has no tasks that can be installed.` (no `taskIds` and every task is `deleted`). |
| 794001 | 409 | An install or uninstall of this library is running, or another library is being installed in the domain. |
| 794005 | 400 | `Failed to start the NetBrain Download install task. Error code {N}.` Retry later or check the library on the NetBrain Download page. |
| 795003 | 403 | The user lacks Shared Resource Management in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793404 | 404 | `libraryId` is empty (`.../Libraries//Install`). The server returns `No resource`. |
| 793001 | 500 | `An unexpected internal error occurred. Please retry. If the issue persists, contact support with the time of the call.` or `Inner exception.` |
