# NetBrain Download API Design

## ***GET*** /V1/NBDownload/Libraries/{libraryId}/Tasks
This API call is used to list the tasks of a NetBrain Download library, with their install status in the current domain and their prerequisites. This is where you read the result of an [Install](Install%20NetBrain%20Download%20Library%20API.md) or [Uninstall](Uninstall%20NetBrain%20Download%20Library%20API.md).

<b>Note</b>:
- Available from R13.0.1.0.
- This API acts on the current domain of your session only. It has no domain parameter. To work in another domain, first call the [Specify A Working Domain API](../Authentication%20and%20Authorization/Specify%20A%20Working%20Domain%20API.md) (`PUT /V1/Session/CurrentDomain`), then call this API.
- Permission: **Shared Resource Management** in the current domain.
- Reading a library updates the domain's copy of that library, the same as opening it on the NetBrain Download page.

## Detail Information

> **Title** : List NetBrain Download Library Tasks API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/NBDownload/Libraries/{libraryId}/Tasks

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
|libraryId* | string | The `id` of a library returned by the [List NetBrain Download Libraries API](List%20NetBrain%20Download%20Libraries%20API.md). |

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
|tasks| list of object | The tasks of the library. |
|tasks.id| string | Task ID, used in `taskIds` of the Install and Uninstall APIs. |
|tasks.name| string | Task name. |
|tasks.folder| string | The folder that contains the task. |
|tasks.status| integer | Install status of the task in the current domain: <br>▪ `0` - None: never installed in the current domain. <br>▪ `1` - Installing: being installed. <br>▪ `2` - Installed: installed. <br>▪ `3` - InstallFailed: the last install failed. <br>▪ `4` - Uninstalled: installed before and since uninstalled. <br>▪ `5` - WaitingInstall: part of a running install, waiting for its turn. |
|tasks.deleted| bool | `true` when the task is no longer in the published library. A deleted task cannot be installed but can be uninstalled. When a library has been withdrawn from the catalogue, all of its tasks are `deleted`. |
|tasks.prerequisites| list of string | Tasks that must be installed before this one, or in the same install, as `Folder/Task`. |
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

listTasks_URL = nb_url + "/ServicesAPI/API/V1/NBDownload/Libraries/" + library_id + "/Tasks"

def listNetBrainDownloadLibraryTasks(listTasks_URL, headers):
    try:
        # Do the HTTP request
        response = requests.get(listTasks_URL, headers=headers, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to List NetBrain Download Library Tasks! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = listNetBrainDownloadLibraryTasks(listTasks_URL, headers)
pprint.pprint(result)
```
```python
    {'libraryId': '5f1c2a9e8b3d4c0012ab34cd',
     'statusCode': 790200,
     'statusDescription': 'Success.',
     'tasks': [{'deleted': False,
                'folder': 'IOS',
                'id': '6a01b7c2d9e8f70011aa22bb',
                'name': 'Golden Config Baseline',
                'prerequisites': ['IOS/Common Variables'],
                'status': 2}]}
```

# cURL Code from Postman
```python
curl -X GET \
  http://192.168.28.79/ServicesAPI/API/V1/NBDownload/Libraries/5f1c2a9e8b3d4c0012ab34cd/Tasks \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7'
```

# Error Examples
## Error Example 1: Unknown library ID
```python
Input:
    
        library_id = "000000000000000000000000" # No library has this ID in the current domain.
          
Response:
    
    Failed to List NetBrain Download Library Tasks! - {"statusCode":791006,"statusDescription":"Library 000000000000000000000000 does not exist."}
```
> Check the ID and the current domain of the session.

## Error Example 2: Blank library ID
```python
Input:
    
        library_id = "%20" # Only whitespace.
          
Response:
    
    Failed to List NetBrain Download Library Tasks! - <!DOCTYPE html>
    <html>
        <head>
            <title>The resource cannot be found.</title>
            ...
        </head>
        <body bgcolor="white">
                <span><H1>Server Error in '/ServicesAPI' Application.<hr width=100% size=1 color=silver></H1>
                <h2> <i>The resource cannot be found.</i> </h2></span>
                ...
                <b> Description: </b>HTTP 404. The resource you are looking for (or one of its dependencies) could have been removed, had its name changed, or is temporarily unavailable. &nbsp;Please review the following URL and make sure that it is spelled correctly.
                <br><br>
                <b> Requested URL: </b>/ServicesAPI/API/V1/NBDownload/Libraries/ /Tasks<br><br>
                ...
        </body>
    </html>
```
> A `libraryId` that is only whitespace does not reach this API. The web server returns HTTP 404 with an HTML error page, not a JSON response, so there is no `statusCode`. Always pass the `id` returned by the [List NetBrain Download Libraries API](List%20NetBrain%20Download%20Libraries%20API.md).

# Error Codes

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | Success. |
| 791006 | 400 | `Library {libraryId} does not exist.` No library has this ID in the current domain. |
| 795003 | 403 | The user lacks Shared Resource Management in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793404 | 404 | `libraryId` is empty (`.../Libraries//Tasks`). The server returns `No resource`. |
| — | 404 | `libraryId` is only whitespace (e.g. `%20`). The web server returns an HTML `The resource cannot be found.` page, not JSON. |
| 793001 | 500 | `An unexpected internal error occurred. Please retry. If the issue persists, contact support with the time of the call.` (unexpected result from NetBrain Download) or `Inner exception.` (unexpected server error). |
