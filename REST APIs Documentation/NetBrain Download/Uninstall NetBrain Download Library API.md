# NetBrain Download API Design

## ***POST*** /V1/NBDownload/Libraries/{libraryId}/Uninstall
This API call is used to start uninstalling the installed tasks of a NetBrain Download library, or selected tasks of it, from the current domain. It is the API equivalent of **Uninstall** on the **NetBrain Download** page in IE.

<b>Note</b>:
- Available from R13.0.1.0.
- This API acts on the current domain of your session only. It has no domain parameter. To work in another domain, first call the [Specify A Working Domain API](../Authentication%20and%20Authorization/Specify%20A%20Working%20Domain%20API.md) (`PUT /V1/Session/CurrentDomain`), then call this API.
- Permission: **Shared Resource Management** in the current domain.
- The uninstall runs asynchronously. The call returns as soon as the uninstall has started. Poll the [Get NetBrain Download Library Status API](Get%20NetBrain%20Download%20Library%20Status%20API.md) until `activity` is `0` (Idle), then check that the tasks have `status` `4` (Uninstalled) with the [List NetBrain Download Library Tasks API](List%20NetBrain%20Download%20Library%20Tasks%20API.md).
- Tasks that depend on an uninstalled task are not checked, the same as on the NetBrain Download page.
- To remove assessment assets that were generated from the Golden Assessment Library, use the [Uninstall Assessment Library Assets API](../GES%20Assessment%20Library/Uninstall%20Assessment%20Library%20Assets%20API.md) first.
- Each call that passes the permission and `libraryId` checks writes one entry to the system audit log (module Intent Library), whether the uninstall started or was rejected, e.g. `NetBrain Download uninstall (API) initiated. Library: Cisco Assessment Library; Task IDs: 6a01b7c2d9e8f70011aa22bb, 6a01b7c2d9e8f70011aa22cc.` A rejected call writes `Failed to initiate NetBrain Download uninstall (API). Library: ...; Task IDs: ...; Reason: ...`. `Task IDs` is `not specified` when `taskIds` is omitted or empty.

## Detail Information

> **Title** : Uninstall NetBrain Download Library API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/NBDownload/Libraries/{libraryId}/Uninstall

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
|taskIds^ | list of string | Tasks to uninstall. Use the `id` of a task returned by the [List NetBrain Download Library Tasks API](List%20NetBrain%20Download%20Library%20Tasks%20API.md). <br>Default (omitted or empty): every task that has been installed in the domain and is not already Uninstalled, including `deleted` tasks. <br>When given, each listed task that has been installed in the domain is uninstalled, whatever its current status. Listed tasks never installed in the domain are skipped and returned in `ignoredTaskIds`. |

> ***Example***


```python
body = {
    "taskIds": ["6a01b7c2d9e8f70011aa22bb", "6a01b7c2d9e8f70011aa22cc"]
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
|taskIds| list of string | The tasks being uninstalled. |
|ignoredTaskIds| list of string | Listed tasks that were skipped because they have never been installed in the domain. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> ***Example***


```python
{
    "libraryId": "5f1c2a9e8b3d4c0012ab34cd",
    "taskIds": ["6a01b7c2d9e8f70011aa22bb"],
    "ignoredTaskIds": ["6a01b7c2d9e8f70011aa22cc"],
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

library_id = "5f1c2a9e8b3d4c0012ab34cd"   # "id" returned by List NetBrain Download Libraries
body = {
    "taskIds": ["6a01b7c2d9e8f70011aa22bb", "6a01b7c2d9e8f70011aa22cc"]
}

uninstall_URL = nb_url + "/ServicesAPI/API/V1/NBDownload/Libraries/" + library_id + "/Uninstall"

def uninstallNetBrainDownloadLibrary(uninstall_URL, body, headers):
    try:
        # Do the HTTP request
        response = requests.post(uninstall_URL, headers=headers, data=json.dumps(body), verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Uninstall NetBrain Download Library! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = uninstallNetBrainDownloadLibrary(uninstall_URL, body, headers)
pprint.pprint(result)
```
```python
    {'ignoredTaskIds': ['6a01b7c2d9e8f70011aa22cc'],
     'libraryId': '5f1c2a9e8b3d4c0012ab34cd',
     'statusCode': 790200,
     'statusDescription': 'Success.',
     'taskIds': ['6a01b7c2d9e8f70011aa22bb']}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.28.79/ServicesAPI/API/V1/NBDownload/Libraries/5f1c2a9e8b3d4c0012ab34cd/Uninstall \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7' \
  -d '{
    "taskIds": ["6a01b7c2d9e8f70011aa22bb", "6a01b7c2d9e8f70011aa22cc"]
}'
```

# Error Examples
## Error Example 1: Nothing installed to uninstall
```python
Input:
    
        body = {} # No task of the library is installed in the domain.
          
Response:
    
    Failed to Uninstall NetBrain Download Library! - {"statusCode":791006,"statusDescription":"Library 5f1c2a9e8b3d4c0012ab34cd has no installed tasks to uninstall."}
```
> The same response is returned when `taskIds` is given and none of the listed tasks has been installed.

## Error Example 2: Unknown task
```python
Input:
    
        body = {"taskIds": ["000000000000000000000000"]} # Not a task of this library.
          
Response:
    
    Failed to Uninstall NetBrain Download Library! - {"statusCode":791006,"statusDescription":"Task 000000000000000000000000 does not exist."}
```
## Error Example 3: Another install or uninstall of this library is running
```python
Input:
    
        body = {} # An install or uninstall of this library, started through the API or on the NetBrain Download page, has not ended.
          
Response:
    
    Failed to Uninstall NetBrain Download Library! - {"statusCode":794001,"statusDescription":"Another task is already running."}
```
> Poll the Get NetBrain Download Library Status API until `activity` is `0` (Idle), then retry.

# Error Codes

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | The uninstall started. |
| 791000 | 400 | `libraryId` is blank (only whitespace, e.g. `%20`). |
| 791006 | 400 | `Library {libraryId} does not exist.`; `Task {taskId} does not exist.` (not a task of the library); or `Library {libraryId} has no installed tasks to uninstall.` |
| 794001 | 409 | An install or uninstall of this library is running. |
| 794005 | 400 | `Failed to start the NetBrain Download uninstall task. Error code {N}.` Retry later or check the library on the NetBrain Download page. |
| 795003 | 403 | The user lacks Shared Resource Management in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793404 | 404 | `libraryId` is empty (`.../Libraries//Uninstall`). The server returns `No resource`. |
| 793001 | 500 | `An unexpected internal error occurred. Please retry. If the issue persists, contact support with the time of the call.` or `Inner exception.` |
