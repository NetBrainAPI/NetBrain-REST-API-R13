# GES Assessment Library API Design

## ***GET*** /V1/GES/Assessment-Library/Tasks/{taskId}/Status
This API call is used to get the status of an Analyze, Generate or Uninstall task of the current domain, using the `taskId` returned by the [Analyze Relevance](Analyze%20Relevance%20API.md), [Generate Golden Assets](Generate%20Golden%20Assets%20API.md) or [Uninstall Assessment Library Assets](Uninstall%20Assessment%20Library%20Assets%20API.md) API. The status is the same as IE shows for the task.

<b>Note</b>:
- Available from R13.0.1.0.
- This API only answers for Analyze, Generate and Uninstall tasks of the **current domain**. Call it in the same domain that started the task.
- **Right after a task is started, its status may not be available for a few seconds**, and this API returns 791006 meanwhile. This does not mean the task failed. Keep polling, and treat 791006 as an unknown task only if it persists (for example, for more than 30 seconds).
- There is no batch status call. Query one task at a time.
- Permission: **Domain User** in the current domain.

## Detail Information

> **Title** : Get Assessment Library Task Status API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/GES/Assessment-Library/Tasks/{taskId}/Status

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
|taskId* | string | The `taskId` returned by the Analyze Relevance, Generate Golden Assets or Uninstall Assessment Library Assets API. |

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
|taskId| string | The requested task ID. |
|status| string | The task status: <br>▪ `NotStart` - not started yet. <br>▪ `Running` - running. <br>▪ `Completed` - completed. <br>▪ `CompletedWithWarning` - completed with warnings. <br>▪ `CompletedWithError` - completed with errors. <br>▪ `Crashed` - ended abnormally. <br>▪ `Cancelled` - cancelled. <br>The task has ended when the status is anything other than `NotStart` or `Running`. The details of a task are shown in IE. |
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

task_id = "8c2d41f7a3b94e6f9d0a1b2c3d4e5f60"   # "taskId" returned by Analyze, Generate or Uninstall

getTaskStatus_URL = nb_url + "/ServicesAPI/API/V1/GES/Assessment-Library/Tasks/" + task_id + "/Status"

def getAssessmentLibraryTaskStatus(getTaskStatus_URL, headers):
    try:
        # Do the HTTP request
        response = requests.get(getTaskStatus_URL, headers=headers, verify=False)
        # An error response still carries a JSON body with statusCode
        return (response.json())
    except Exception as e:
        return ({"statusCode": None, "statusDescription": str(e)})

ACTIVE = ("NotStart", "Running")

# Poll every 10 seconds. Tolerate 791006 for the first 30 seconds (the task is not visible yet).
start = time.time()
while True:
    result = getAssessmentLibraryTaskStatus(getTaskStatus_URL, headers)
    if result.get("statusCode") == 791006 and time.time() - start < 30:
        time.sleep(5)
        continue
    pprint.pprint(result)
    if result.get("statusCode") != 790200 or result.get("status") not in ACTIVE:
        break
    time.sleep(10)
```
```python
    {'status': 'Running',
     'statusCode': 790200,
     'statusDescription': 'Success.',
     'taskId': '8c2d41f7a3b94e6f9d0a1b2c3d4e5f60'}
    {'status': 'Completed',
     'statusCode': 790200,
     'statusDescription': 'Success.',
     'taskId': '8c2d41f7a3b94e6f9d0a1b2c3d4e5f60'}
```

# Typical Workflow

1. Install the assessment library with NetBrain Download first (manually, from Knowledge Cloud, or with the NetBrain Download API `POST /V1/NBDownload/Libraries/{libraryId}/Install`).
2. [List Assessment Library Assets](List%20Assessment%20Library%20Assets%20API.md) to find the versions and AF IDs to work on.
3. [Analyze Relevance](Analyze%20Relevance%20API.md) with the selection, then poll this API until the task ends.
4. [Generate Golden Assets](Generate%20Golden%20Assets%20API.md) with the same selection, then poll this API until the task ends.
5. [List Assessment Library Assets](List%20Assessment%20Library%20Assets%20API.md) with the same filters, and check `status` and `localVersion` of the assets.
6. To manage several domains, switch the current domain and repeat, one domain at a time.

# Example: Analyze and Generate in several domains

This example runs steps 3 and 4 of the workflow in each domain of a tenant, one domain at a time.

```python
import requests
import time
import urllib3
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

nb_url = "http://192.168.28.79"
api = nb_url + "/ServicesAPI/API/V1"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = "bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7"   # token from the Login API

tenant_id = "fb24f3f0-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
domain_ids = ["0201adc4-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "1b7e9d33-xxxx-xxxx-xxxx-xxxxxxxxxxxx"]

ACTIVE = ("NotStart", "Running")

def switch_domain(domain_id):
    r = requests.put(api + "/Session/CurrentDomain", headers=headers,
                     data=json.dumps({"tenantId": tenant_id, "domainId": domain_id}), verify=False)
    return r.json()

def start(operation, body):
    # operation: "Analyze", "Generate" or "Uninstall"
    r = requests.post(api + "/GES/Assessment-Library/" + operation, headers=headers,
                      data=json.dumps(body), verify=False)
    return r.json()

def wait(task_id, interval=30):
    begin = time.time()
    while True:
        js = requests.get(api + "/GES/Assessment-Library/Tasks/" + task_id + "/Status",
                          headers=headers, verify=False).json()
        if js.get("statusCode") == 791006 and time.time() - begin < 30:
            time.sleep(5)           # not visible yet
            continue
        if js.get("statusCode") != 790200:
            return "Unknown (" + str(js.get("statusCode")) + ")"
        if js["status"] not in ACTIVE:
            return js["status"]
        time.sleep(interval)

for domain_id in domain_ids:
    print("Domain", domain_id, switch_domain(domain_id).get("statusDescription"))

    analyze = start("Analyze", {})
    if analyze.get("statusCode") != 790200:
        print("  Analyze not started:", analyze.get("statusCode"), analyze.get("statusDescription"))
        continue
    status = wait(analyze["taskId"])
    print("  Analyze:", status)
    if status not in ("Completed", "CompletedWithWarning"):
        continue

    generate = start("Generate", {})   # default options
    if generate.get("statusCode") != 790200:
        print("  Generate not started:", generate.get("statusCode"), generate.get("statusDescription"))
        continue
    print("  Generate:", wait(generate["taskId"], interval=60))   # a full Generate can take over an hour
```
```python
    Domain 0201adc4-xxxx-xxxx-xxxx-xxxxxxxxxxxx Success.
      Analyze: Completed
      Generate: Completed
    Domain 1b7e9d33-xxxx-xxxx-xxxx-xxxxxxxxxxxx Success.
      Analyze: Completed
      Generate not started: 791006 None of the selected library assets have relevant devices. Run Analyze first, or set requireRelevantDevice to false.
```

# cURL Code from Postman
```python
curl -X GET \
  http://192.168.28.79/ServicesAPI/API/V1/GES/Assessment-Library/Tasks/8c2d41f7a3b94e6f9d0a1b2c3d4e5f60/Status \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7'
```

# Error Examples
## Error Example 1: Unknown task ID
```python
Input:
    
        task_id = "00000000000000000000000000000000" # Not an Analyze, Generate or Uninstall task of this domain.
          
Response:
    
    {'statusCode': 791006, 'statusDescription': 'Task 00000000000000000000000000000000 does not exist.'}
```
> The same response is returned for a task that has expired or belongs to another domain, and, for a few seconds, for a task that was just started.

## Error Example 2: No Domain User permission
```python
Input:
    
        token = "<token of a user without Domain User in the current domain>"
          
Response:
    
    {'statusCode': 795003, 'statusDescription': 'Insufficient permissions: the current user has insufficient permissions to perform the requested operation.'}
```

# Error Codes

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | Returned the status. |
| 791006 | 400 | The task does not exist, has expired, is not an Analyze, Generate or Uninstall task of the current domain, or is not visible yet. |
| 795003 | 403 | The user lacks Domain User in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793404 | 404 | `taskId` is empty (`.../Tasks//Status`). The request does not reach this API, and the server returns `No resource`. |
| 793001 | 500 | `An unexpected internal error occurred. Please retry. If the issue persists, contact support with the time of the call.` |
