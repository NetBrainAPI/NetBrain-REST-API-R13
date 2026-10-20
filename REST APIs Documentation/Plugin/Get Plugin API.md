# Plugin API Design

## ***GET*** /V1/CMDB/Plugin
This API call is used to get a private plugin in the current domain by its ID or path.

## Detail Information

> **Title** : Get Plugin API<br>

> **Version** : 10/19/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Plugin

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
|id* | string | The ID (GUID) of the plugin. |
|path* | string | The full path of the plugin (alternative to `id`). <br>Must be the exact `plugin.path` value returned by this API, which is the same value used for `pluginGroup.plugins[].path` in the [Add or Update Schedule Plugin Task API](Add%20or%20Update%20Schedule%20Plugin%20Task%20API.md), e.g. `My Plugins/APIPlugins/my_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. |

>>**Note:** Either `id` or `path` must be provided. <br>`path` must match the full value exactly: no leading `/`, and including the `?domainId=<guid>` suffix. A partial path such as `/My Plugins/APIPlugins/my_plugin` or `My Plugins/APIPlugins/my_plugin` (without the suffix) is not recognized.

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
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |
|plugin| object | The plugin. |
|plugin.id| string | The ID of the plugin. |
|plugin.name| string | The name of the plugin. |
|plugin.path| string | The full path of the plugin. <br>**Note:** Use this value as-is for `pluginGroup.plugins[].path` when calling the [Add or Update Schedule Plugin Task API](Add%20or%20Update%20Schedule%20Plugin%20Task%20API.md). For private plugins it carries a `?domainId=<guid>` suffix; do not rebuild it from the folder path and plugin name. |
|plugin.description| object | The description of the plugin (`content`). |
|plugin.input| object | The input settings of the plugin (`enabled`, `sample`). |
|plugin.files| list of object | The source files of the plugin (`name`, `content`). |
|plugin.author| string | The user who created the plugin. |

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

params = {"id": "6f1c2b3a-4d5e-4f60-8a7b-9c0d1e2f3a4b"}
##OR##
#params = {"path": "My Plugins/APIPlugins/my_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx"}   # full plugin.path value, including ?domainId=

getPlugin_URL = nb_url + "/ServicesAPI/API/V1/CMDB/Plugin"

def getPlugin(getPlugin_URL, params, headers):
    try:
        # Do the HTTP request
        response = requests.get(getPlugin_URL, params=params, headers=headers, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Get Plugin! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = getPlugin(getPlugin_URL, params, headers)
pprint.pprint(result)

# Keep the returned path for scheduling this plugin later
plugin_path = result["plugin"]["path"]
```
```python
    {'plugin': {'createInfo': {'dateTime': '2026-10-19T18:23:03.277Z',
                           'userId': 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
                           'userName': 'xxx'},
            'description': {'content': 'Plugin created by REST API'},
            'files': [{'content': 'from netbrain.sysapi import pluginfw\n'
                                  '\n'
                                  'def run(input):\n'
                                  "    pluginfw.AddLog('hello from my_plugin')\n"
                                  '    return True\n',
                       'name': 'main.py'}],
            'id': '6ba5adfc-949d-4c4b-9cd9-56a56ee88ec4',
            'input': {'enabled': False, 'sample': ''},
            'name': 'my_plugin',
            'operateInfo': {'opTime': '2026-10-19T18:23:03.277Z',
                            'opUser': 'xxx',
                            'opUserId': 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
                            'source': None},
            'path': 'My '
                    'Plugins/APIPlugins/my_plugin?domainId=3e04a0e9-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
            'version': '8.0'},
 'statusCode': 790200,
 'statusDescription': 'Success.'}
```

# cURL Code from Postman
```python
curl -X GET \
  'http://192.168.28.79/ServicesAPI/API/V1/CMDB/Plugin?id=6f1c2b3a-4d5e-4f60-8a7b-9c0d1e2f3a4b' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7'
```

# Error Examples
## Error Example 1: Missing `id` and `path`
```python
Input:
    
        params = {} # Either id or path is required.
          
Response:
    
('Failed to Get Plugin! - {"statusCode":791000,"statusDescription":"Null '
 'parameter: the parameter \'id or path\' cannot be null."}')
```
## Error Example 2: Plugin not found
```python
Input:
    
        params = {"id": "00000000-0000-0000-0000-000000000000"} # There is no plugin with this ID.
          
Response:
    
('Failed to Get Plugin! - {"statusCode":791006,"statusDescription":"Plugin '
 'does not exist."}')
```
