# Plugin API Design

## ***POST*** /V1/CMDB/Plugin
This API call is used to add a new private plugin to the current domain, or to update an existing one.

- If `pluginInfo.id` does not match an existing plugin, a new plugin is created with that ID.
- If `pluginInfo.id` matches an existing plugin, that plugin is updated.

<b>Note</b>: `pluginInfo.id` is required. To create a plugin, generate your own GUID (e.g. `str(uuid.uuid4())` in Python) and send it. If `pluginInfo.id` is omitted, the request fails with HTTP 500 and `statusCode` 793000 "Unexpected Error.".

<b>Note</b>: Only private (user-created) plugins are supported. Built-in plugins maintained by NetBrain cannot be created or modified through this API.

## Detail Information

> **Title** : Add or Update Plugin API<br>

> **Version** : 10/19/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Plugin

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|parentId* | string | The ID (GUID) of the plugin folder to place the plugin in. <br>Either `parentId` or `parentPath` is required. If both are provided, `parentId` takes precedence. |
|parentPath* | string | The path of the plugin folder to place the plugin in, e.g. `/My Plugins/My Folder`. <br>Either `parentId` or `parentPath` is required. <br>The first segment must be a root folder: `My Plugins` or `All Plugins`. Each following segment must be an existing folder name. Matching is case-insensitive. <br>**Note:** The folder must already exist; this API does not create folders. To place the plugin directly under the root, use `/My Plugins`. If any segment cannot be found, the request fails with HTTP 500 / `793000` "Unexpected Error.". |
|pluginInfo* | object | The plugin definition. |
|pluginInfo.id* | string | The ID (GUID) of the plugin. <br>To create a plugin, send a new client-generated GUID; the server keeps the ID you send. To update an existing plugin, send its ID. <br>**Note:** This field cannot be omitted. Omitting it returns HTTP 500 / `793000`. |
|pluginInfo.name* | string | The name of the plugin. <br>Must match `^[a-zA-Z_]\w*$` (letters, digits and underscores, not starting with a digit), up to 255 characters. |
|pluginInfo.description^ | object | The description of the plugin. <br>▪ `content` (string) - plain-text description. |
|pluginInfo.input^ | object | The input settings of the plugin. <br>▪ `enabled` (bool) - whether the plugin accepts user input. <br>▪ `sample` (string) - a sample input value. |
|pluginInfo.files* | list of object | The source files of the plugin (Python script and any companion JSON/text files). |
|pluginInfo.files[].name* | string | The file name, e.g. `main.py`. |
|pluginInfo.files[].content* | string | The file content. <br>**Note:** The content is stored as-is. Python syntax is **not** checked when saving; an invalid script only fails when the plugin runs. <br>The Python entry point is `def run(input):`. It must return `True` for a run to show as **Succeeded**; returning `False` or nothing shows the run as **Failed**. |

> ### ***Example***


```python
body = {
    "parentPath": "/My Plugins/APIPlugins",
    "pluginInfo": {
        "id": "6f1c2b3a-4d5e-4f60-8a7b-9c0d1e2f3a4b",
        "name": "my_plugin",
        "description": {"content": "Plugin created by REST API"},
        "input": {"enabled": False, "sample": ""},
        "files": [
            {"name": "main.py", "content": "from netbrain.sysapi import pluginfw\n\ndef run(input):\n    pluginfw.AddLog('hello')\n    return True\n"}
        ]
    }
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
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |
|plugin| object | The created or updated plugin. |
|plugin.id| string | The ID of the plugin. |
|plugin.name| string | The name of the plugin. |
|plugin.path| string | The full path of the plugin, including the `?domainId=<guid>` suffix. Use this exact value for the `path` query parameter of the [Get Plugin API](Get%20Plugin%20API.md) and for `pluginGroup.plugins[].path` in the [Add or Update Schedule Plugin Task API](Add%20or%20Update%20Schedule%20Plugin%20Task%20API.md). |
|plugin.description| object | The description of the plugin (`content`). |
|plugin.input| object | The input settings of the plugin (`enabled`, `sample`). |
|plugin.files| list of object | The source files of the plugin (`name`, `content`). |


# Full Example

```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
import uuid
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

nb_url = "http://192.168.28.79"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
token = "bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7"
headers["Token"] = token

# Always send your own plugin ID; reuse it to update the same plugin later.
plugin_id = str(uuid.uuid4())

# run(input) is the plugin entry point. It must return True for the run to show as Succeeded;
# returning False or nothing (None) shows the run as Failed.
script = """from netbrain.sysapi import pluginfw

def run(input):
    pluginfw.AddLog('hello from my_plugin')
    return True
"""

body = {
    "parentPath": "/My Plugins/APIPlugins",   # folder must already exist; use "/My Plugins" for the root
    "pluginInfo": {
        "id": plugin_id,
        "name": "my_plugin",
        "description": {"content": "Plugin created by REST API"},
        "input": {"enabled": False, "sample": ""},
        "files": [
            {"name": "main.py", "content": script}
        ]
    }
}

addPlugin_URL = nb_url + "/ServicesAPI/API/V1/CMDB/Plugin"

def addOrUpdatePlugin(addPlugin_URL, body, headers):
    try:
        # Do the HTTP request
        response = requests.post(addPlugin_URL, headers=headers, data=json.dumps(body), verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Failed to Add or Update Plugin! - " + str(response.text))
    except Exception as e:
        return (str(e))

result = addOrUpdatePlugin(addPlugin_URL, body, headers)
pprint.pprint(result)
```
```python
    {'plugin': {'createInfo': {'dateTime': '2026-10-19T18:23:03.2771442Z',
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
            'operateInfo': {'opTime': '2026-10-19T18:23:03.2771442Z',
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
curl -X POST \
  http://192.168.28.79/ServicesAPI/API/V1/CMDB/Plugin \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: bf414c1e-a3b8-4ef1-9812-b2ba8529e6e7' \
  -d '{
    "parentPath": "/My Plugins/APIPlugins",
    "pluginInfo": {
        "id": "6f1c2b3a-4d5e-4f60-8a7b-9c0d1e2f3a4b",
        "name": "my_plugin",
        "description": {"content": "Plugin created by REST API"},
        "input": {"enabled": false, "sample": ""},
        "files": [
            {"name": "main.py", "content": "from netbrain.sysapi import pluginfw\n\ndef run(input):\n    pluginfw.AddLog(\u0027hello\u0027)\n    return True\n"}
        ]
    }
}'
```

# Error Examples
## Error Example 1: Missing `parentId` and `parentPath`
```python
Input:
    
        body = {
            "pluginInfo": {"id": plugin_id, "name": "my_plugin", "files": [{"name": "main.py", "content": script}]}
        } # Either parentId or parentPath is required.
          
Response:
    
    ('Failed to Add or Update Plugin! - '
    '{"statusCode":791000,"statusDescription":"Null parameter: the parameter '
    '\'parentId or parentPath\' cannot be null."}')
```
## Error Example 2: `parentPath` cannot be resolved
```python
Input 1:
    
        "parentPath": "/My Plugins/Folder That Does Not Exist" # The folder must already exist.
          
Response 1:   # HTTP 500
    
    ('Failed to Add or Update Plugin! - '
    '{"statusCode":793000,"statusDescription":"Unexpected Error."}')
```
```python
Input 2:
    
        "parentPath": "/No Such Root/My Folder" # The root must be "My Plugins" or "All Plugins".
          
Response 2:   # HTTP 500
    
    ('Failed to Add or Update Plugin! - '
    '{"statusCode":793000,"statusDescription":"Unexpected Error."}')
```
## Error Example 3: `pluginInfo.id` omitted
```python
Input:
    
        "pluginInfo": {"name": "my_plugin", "files": [{"name": "main.py", "content": script}]} # No "id". A client-generated GUID is required.
          
Response:   # HTTP 500
    
    ('Failed to Add or Update Plugin! - '
    '{"statusCode":793000,"statusDescription":"Unexpected Error."}')
```
<!-- ## Error Example 4: Invalid `pluginInfo.name`
```python
Input 1:
    
        "name": "" # Name cannot be empty.
          
Response 1:
    
    Failed to Add or Update Plugin! - {"statusCode":470100,"statusDescription":"..."}
```
```python
Input 2:
    
        "name": "a" * 256 # Name cannot exceed 255 characters.
          
Response 2:
    
    Failed to Add or Update Plugin! - {"statusCode":470101,"statusDescription":"..."}
```
```python
Input 3:
    
        "name": "1-my plugin" # Name must match ^[a-zA-Z_]\w*$
          
Response 3:
    
    Failed to Add or Update Plugin! - {"statusCode":791001,"statusDescription":"Invalid parameter: ..."}
``` -->
