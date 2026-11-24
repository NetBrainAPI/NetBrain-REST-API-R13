# NetBrain Download API Design

## ***GET*** /V1/NBDownload/Libraries/{libraryId}/Status
This API call is used to get whether an install or uninstall of a NetBrain Download library is running in the current domain. The status covers every operation on the library, whether it was started through the API or on the **NetBrain Download** page in IE.

<b>Note</b>:
- Available from R13.0.1.0.
- This API acts on the current domain of your session only. It has no domain parameter. To work in another domain, first call the [Specify A Working Domain API](../Authentication%20and%20Authorization/Specify%20A%20Working%20Domain%20API.md) (`PUT /V1/Session/CurrentDomain`), then call this API.
- Permission: **Shared Resource Management** in the current domain.
- This API only says whether an operation is running. When `activity` is back to `0` (Idle), read the result of each task with the [List NetBrain Download Library Tasks API](List%20NetBrain%20Download%20Library%20Tasks%20API.md).

## Detail Information

> **Title** : Get NetBrain Download Library Status API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/NBDownload/Libraries/{libraryId}/Status

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
|activity| integer | Current activity of the library: <br>▪ `0` - Idle: no install or uninstall of the library is running. <br>▪ `1` - Installing: an install of the library is running. <br>▪ `2` - Uninstalling: an uninstall of the library is running. |
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

getStatus_URL = nb_url + "/ServicesAPI/API/V1/NBDownload/Libraries/" + library_id + "/Status"

def getNetBrainDownloadLibraryStatus(getStatus_URL, headers):
    try:
        # Do the HTTP request
        response = requests.get(getStatus_URL, headers=headers, verify=False)
        # An error response still carries a JSON body with statusCode
        return (response.json())
    except Exception as e:
        return ({"statusCode": None, "statusDescription": str(e)})

# Poll every 10 seconds until the library is Idle (activity 0).
while True:
    result = getNetBrainDownloadLibraryStatus(getStatus_URL, headers)
    pprint.pprint(result)
    if result.get("statusCode") != 790200 or result.get("activity") == 0:
        break
    time.sleep(10)
```
```python
    {'activity': 1,
     'libraryId': '5f1c2a9e8b3d4c0012ab34cd',
     'statusCode': 790200,
     'statusDescription': 'Success.'}
    {'activity': 0,
     'libraryId': '5f1c2a9e8b3d4c0012ab34cd',
     'statusCode': 790200,
     'statusDescription': 'Success.'}
```

# Typical Workflow

1. [List NetBrain Download Libraries](List%20NetBrain%20Download%20Libraries%20API.md) with `name` to find the library ID.
2. [List NetBrain Download Library Tasks](List%20NetBrain%20Download%20Library%20Tasks%20API.md) to choose the task IDs and check their prerequisites.
3. [Install](Install%20NetBrain%20Download%20Library%20API.md) (or [Uninstall](Uninstall%20NetBrain%20Download%20Library%20API.md)) the library with the chosen `taskIds`.
4. Poll this API until `activity` is `0` (Idle).
5. [List NetBrain Download Library Tasks](List%20NetBrain%20Download%20Library%20Tasks%20API.md) again and check the `status` of the tasks you sent: `2` (Installed) after an install, `4` (Uninstalled) after an uninstall. `3` (InstallFailed) means the install of that task failed.
6. For an assessment library, continue with the [GES Assessment Library APIs](../GES%20Assessment%20Library/Get%20Assessment%20Library%20Task%20Status%20API.md#typical-workflow) to analyze and generate the assets.

# Example: Install a library and check the result

```python
import requests
import time
import urllib3
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

nb_url = "http://192.168.28.79"
api = nb_url + "/ServicesAPI/API/V1/NBDownload"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = "bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7"   # token from the Login API

STATUS = {0: "None", 1: "Installing", 2: "Installed", 3: "InstallFailed", 4: "Uninstalled", 5: "WaitingInstall"}

# 1. Find the library
libs = requests.get(api + "/Libraries", headers=headers,
                    params={"name": "Cisco Assessment Library"}, verify=False).json()["libraries"]
library_id = libs[0]["id"]

# 2. Install every task that is not deleted
started = requests.post(api + "/Libraries/" + library_id + "/Install", headers=headers,
                        data=json.dumps({}), verify=False).json()
if started.get("statusCode") != 790200:
    raise SystemExit("Install not started: " + str(started.get("statusCode")) + " " + started.get("statusDescription"))

# 3. Wait until the library is Idle
while requests.get(api + "/Libraries/" + library_id + "/Status", headers=headers,
                   verify=False).json().get("activity") != 0:
    time.sleep(10)

# 4. Check the result of each task
tasks = requests.get(api + "/Libraries/" + library_id + "/Tasks", headers=headers, verify=False).json()["tasks"]
for t in tasks:
    if t["id"] in started["taskIds"]:
        print(t["folder"] + "/" + t["name"], STATUS[t["status"]])
```
```python
    IOS/Common Variables Installed
    IOS/Golden Config Baseline Installed
```

# cURL Code from Postman
```python
curl -X GET \
  http://192.168.28.79/ServicesAPI/API/V1/NBDownload/Libraries/5f1c2a9e8b3d4c0012ab34cd/Status \
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
    
    {'statusCode': 791006, 'statusDescription': 'Library 000000000000000000000000 does not exist.'}
```
## Error Example 2: No Shared Resource Management permission
```python
Input:
    
        token = "<token of a user without Shared Resource Management in the current domain>"
          
Response:
    
    {'statusCode': 795003, 'statusDescription': 'Insufficient permissions: the current user has insufficient permissions to perform the requested operation.'}
```

# Error Codes

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | Returned the activity. |
| 791000 | 400 | `libraryId` is blank (only whitespace, e.g. `%20`). |
| 791006 | 400 | `Library {libraryId} does not exist.` No library has this ID in the current domain. |
| 795003 | 403 | The user lacks Shared Resource Management in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793404 | 404 | `libraryId` is empty (`.../Libraries//Status`). The server returns `No resource`. |
| 793001 | 500 | `An unexpected internal error occurred. Please retry. If the issue persists, contact support with the time of the call.` or `Inner exception.` |
