# NetBrain Download API Design

## ***GET*** /V1/NBDownload/Libraries
This API call is used to list the NetBrain Download libraries visible in the current domain, newest first (by `publishTime`). It is the API equivalent of the library list on the **NetBrain Download** page in IE.

<b>Note</b>:
- Available from R13.0.1.0.
- This API acts on the current domain of your session only. It has no domain parameter. To work in another domain, first call the [Specify A Working Domain API](../Authentication%20and%20Authorization/Specify%20A%20Working%20Domain%20API.md) (`PUT /V1/Session/CurrentDomain`), then call this API.
- Permission: **Shared Resource Management** in the current domain, the same permission as the NetBrain Download page.
- Use the `id` of a library in the other NetBrain Download APIs.

## Detail Information

> **Title** : List NetBrain Download Libraries API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/NBDownload/Libraries

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

> No request body.

## Query Parameters(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|name^ | string | Return only the library with this exact name (case-insensitive), e.g. `Cisco Assessment Library`. <br>Default: all libraries. An empty value is the same as omitting it. |

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
|libraries| list of object | The matching libraries. Empty when none match. |
|libraries.id| string | Library ID, used in the other NetBrain Download APIs. |
|libraries.name| string | Library name. |
|libraries.version| string | Library version. |
|libraries.description| string | Library description. |
|libraries.publishTime| string | Publish time, ISO 8601 without a time-zone offset, as stored by NetBrain Download, e.g. `2026-09-30T08:00:00`. |
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

params = {"name": "Cisco Assessment Library"}

listLibraries_URL = nb_url + "/ServicesAPI/API/V1/NBDownload/Libraries"

def listNetBrainDownloadLibraries(listLibraries_URL, params, headers):
    try:
        # Do the HTTP request
        response = requests.get(listLibraries_URL, headers=headers, params=params, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to List NetBrain Download Libraries! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = listNetBrainDownloadLibraries(listLibraries_URL, params, headers)
pprint.pprint(result)
```
```python
    {'libraries': [{'description': 'Assessment content for Cisco devices',
                    'id': '5f1c2a9e8b3d4c0012ab34cd',
                    'name': 'Cisco Assessment Library',
                    'publishTime': '2026-09-30T08:00:00',
                    'version': '13.0'}],
     'statusCode': 790200,
     'statusDescription': 'Success.'}
```

# cURL Code from Postman
```python
curl -X GET \
  'http://192.168.28.79/ServicesAPI/API/V1/NBDownload/Libraries?name=Cisco%20Assessment%20Library' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7'
```

# Error Examples
## Error Example 1: No Shared Resource Management permission
```python
Input:
    
        token = "<token of a user without Shared Resource Management in the current domain>"
          
Response:
    
    Failed to List NetBrain Download Libraries! - {"statusCode":795003,"statusDescription":"Insufficient permissions: the current user has insufficient permissions to perform the requested operation."}
```
> A `name` that matches no library is not an error: the call returns 790200 with an empty `libraries` list.

# Error Codes

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | Success, including when no library matches. |
| 795003 | 403 | The user lacks Shared Resource Management in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793001 | 500 | `Inner exception.` Unexpected server error. Retry; if it persists, contact support with the time of the call. |
