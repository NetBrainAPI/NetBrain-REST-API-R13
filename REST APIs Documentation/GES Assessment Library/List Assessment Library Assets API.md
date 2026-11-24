# GES Assessment Library API Design

## ***GET*** /V1/GES/Assessment-Library/Assets
This API call is used to list the assets of the Golden Assessment Library (GAL) in the current domain, as a tree of assessment features (AF) and their assessment rules (AR). It returns the same data as the **Assessment Library** page in IE: library and local versions, library status, and the relevant-device scope and count of each asset.

Use it to:
- see which versions and AFs are available before you call [Analyze Relevance](Analyze%20Relevance%20API.md), [Generate Golden Assets](Generate%20Golden%20Assets%20API.md) or [Uninstall Assessment Library Assets](Uninstall%20Assessment%20Library%20Assets%20API.md);
- get the AF IDs (`afIds`) those calls accept;
- read the result of a finished task. The start calls return only a task ID, so asset status, local version and relevant devices are read here.

<b>Note</b>:
- Available from R13.0.1.0.
- This API acts on the current domain of your session only. It has no domain parameter. To work in another domain, first call the [Specify A Working Domain API](../Authentication%20and%20Authorization/Specify%20A%20Working%20Domain%20API.md) (`PUT /V1/Session/CurrentDomain`), then call this API.
- Permission: **Domain User** in the current domain (the same as the Assessment Library page).
- This API returns all assets, including `Ignored` and `ToBeInstalledLocalConflict` ones. Analyze, Generate and Uninstall skip those assets.

## Detail Information

> **Title** : List Assessment Library Assets API<br>

> **Version** : 23/11/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/GES/Assessment-Library/Assets

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

> No request body.

## Query Parameters(****required***)

> Repeat a parameter to send several values, e.g. `?libraryVersions=1.0&libraryVersions=2.0`. A comma-separated value is **not** split.

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|libraryVersions^ | string | Return only assets of this library version, e.g. `2.0`. Exact match: case-sensitive and not trimmed. <br>An AR also matches when the library version of its AF is listed. An AF that does not match itself but has matching ARs is returned with only those ARs. <br>Default: no version filter. |
|afIds^ | string | Return only this assessment feature (AF) and its ARs. Use the `id` of an `AF` item returned by this API. An AR ID, or an ID that is not an AF of the current domain's library, rejects the call with 791001. <br>Default: all AFs. |

> When both filters are given, an asset must match both. A null or blank value is invalid (791001).

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
|assets| list of object | The matching AFs, in the order of the Assessment Library page. Empty (`[]`) when nothing matches or the library is empty. |
|assets.id| string | The asset ID. An AF ID can be used in `afIds`. |
|assets.name| string | The asset name, e.g. `NTP`. |
|assets.type| string | `AF` (assessment feature) or `AR` (assessment rule). |
|assets.parentId| string | AR only: the ID of its AF. |
|assets.libraryVersion| string | The version of the asset as shipped in the library. |
|assets.localVersion| string | The version of the domain's installed copy. Not returned when the asset is not installed. |
|assets.status| string | The library status of the asset. Not returned when the library gives none. <br>▪ `ToBeInstalled` - not installed in the domain. <br>▪ `Installed` - installed. <br>▪ `InstalledLocalResourceModified` - installed, and changed locally since. <br>▪ `ToBeUpdated` - installed; the library has a newer version. <br>▪ `InstalledNoChange` - installed; the library version brings no change. <br>▪ `Ignored` - ignored in the domain. <br>▪ `ToBeInstalledLocalConflict` - not installed; conflicts with a local asset. |
|assets.relevantDeviceScope| object | Where the relevant devices are taken from. Not returned when the library gives none. |
|assets.relevantDeviceScope.type| string | ▪ `AllNetwork` - all devices of the network. <br>▪ `FeatureDefinition` - devices matched by a feature definition. <br>▪ `ReferenceCluster` - devices of a reference cluster. |
|assets.relevantDeviceScope.id| string | The ID of the feature definition or reference cluster, when there is one. |
|assets.relevantDeviceCount| integer | The number of relevant devices, as shown on the Assessment Library page. Computed by [Analyze Relevance](Analyze%20Relevance%20API.md). |
|assets.children| list of object | AF only: its matching ARs, with the same fields. |
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

# Optional filters. Use a list to send a parameter more than once.
params = {
    "libraryVersions": ["2.0"]
    # "afIds": ["6a1f0c2e9b7d4a0012cd34ef"]
}

listAssets_URL = nb_url + "/ServicesAPI/API/V1/GES/Assessment-Library/Assets"

def listAssessmentLibraryAssets(listAssets_URL, params, headers):
    try:
        # Do the HTTP request
        response = requests.get(listAssets_URL, headers=headers, params=params, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to List Assessment Library Assets! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = listAssessmentLibraryAssets(listAssets_URL, params, headers)
pprint.pprint(result)

# Print one line per asset
if isinstance(result, dict):
    for af in result["assets"]:
        print(af["type"], af["id"], af["name"], af.get("localVersion"), "->", af["libraryVersion"], af.get("status"))
        for ar in af.get("children", []):
            print("   ", ar["type"], ar["name"], ar.get("status"), ar.get("relevantDeviceCount"))
```
```python
    {'assets': [{'children': [{'id': '6a1f0c2e9b7d4a0012cd3501',
                               'libraryVersion': '2.0',
                               'localVersion': '1.0',
                               'name': 'NTP Server',
                               'parentId': '6a1f0c2e9b7d4a0012cd34ef',
                               'relevantDeviceCount': 3,
                               'relevantDeviceScope': {'id': '6a1f0c2e9b7d4a0012cd3601',
                                                       'type': 'FeatureDefinition'},
                               'status': 'ToBeUpdated',
                               'type': 'AR'}],
                 'id': '6a1f0c2e9b7d4a0012cd34ef',
                 'libraryVersion': '2.0',
                 'localVersion': '1.0',
                 'name': 'NTP',
                 'relevantDeviceCount': 12,
                 'relevantDeviceScope': {'type': 'AllNetwork'},
                 'status': 'ToBeUpdated',
                 'type': 'AF'}],
     'statusCode': 790200,
     'statusDescription': 'Success.'}
    AF 6a1f0c2e9b7d4a0012cd34ef NTP 1.0 -> 2.0 ToBeUpdated
        AR NTP Server ToBeUpdated 3
```

# cURL Code from Postman
```python
curl -X GET \
  'http://192.168.28.79/ServicesAPI/API/V1/GES/Assessment-Library/Assets?libraryVersions=1.0&libraryVersions=2.0' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7'
```

# Error Examples
## Error Example 1: `afIds` holds an AR ID
```python
Input:
    
        params = {"afIds": ["6a1f0c2e9b7d4a0012cd3501"]} # An AR ID. Only AF IDs are accepted.
          
Response:
    
    Failed to List Assessment Library Assets! - {"statusCode":791001,"statusDescription":"Invalid parameter: 'afIds'. The following IDs are not valid assessment feature IDs: 6a1f0c2e9b7d4a0012cd3501."}
```
## Error Example 2: Blank `libraryVersions` value
```python
Input:
    
        params = {"libraryVersions": [""]} # A blank value is invalid.
          
Response:
    
    Failed to List Assessment Library Assets! - {"statusCode":791001,"statusDescription":"Invalid parameter: 'libraryVersions'."}
```
## Error Example 3: No Domain User permission
```python
Input:
    
        token = "<token of a user without Domain User in the current domain>" # Permission is checked first.
          
Response:
    
    Failed to List Assessment Library Assets! - {"statusCode":795003,"statusDescription":"Insufficient permissions: the current user has insufficient permissions to perform the requested operation."}
```

# Error Codes

|**statusCode**|**HTTP**|**When**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| 790200 | 200 | Success. No match returns `"assets": []`. |
| 791001 | 400 | A blank filter value, or an `afIds` value that is an AR ID or not an AF of the library. |
| 795003 | 403 | The user lacks Domain User in the current domain. |
| 795026 | 403 | API access control is enabled and the user has not been granted this API. |
| 793001 | 500 | Unexpected server error. Retry; if it persists, contact NetBrain Support with the time of the call. |
