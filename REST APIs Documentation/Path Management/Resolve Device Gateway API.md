
# Path API Design

## ***GET*** /V1/CMDB/Path/Gateways
Call this API to retrieve the gateways for a device used as path source.

## Detail Information

> **Title** : Resolve Device Gateway API<br>

> **Version** : 30/01/2019.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Path/Gateways

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
| ipOrHost* | string  | IP address or hostname of the path source. To choose the gateway of a client manually, use any unused host IP in the subnet of the gateway interface (see Note 1). |

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
| token | string  | Authentication token, retrieved from Login API. |

## Response

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |
|gatewayList| List of Object | Gateway candidates for `ipOrHost`. Pass one entry, unchanged, as `sourceGateway` in the [Calculate Path API](https://github.com/NetBrainAPI/NetBrain-REST-API-R13/blob/main/REST%20APIs%20Documentation/Path%20Management/Calculate%20Path%20API.md).  |
|gatewayList.gatewayName| string | Name of the gateway, in the format `<device>.<interface>(<interface IP>)`.  |
|gatewayList.type| string | Gateway type, such as `Device Interface`. The same gateway can also be listed with the type `Multicast`.  |
|gatewayList.payload| string | JSON-encoded string with the gateway device and interface (`ip`, `device`, `deviceId`, `interface`, `interfaceId`, `prefixLen`, `endPointInfo`, `details`). Internal data structure, pass it unchanged. `endPointInfo` is `null` when the entry is the gateway *for* `ipOrHost`; it is set when `ipOrHost` is the device interface itself (see Note 1).  |
|gatewayList.natInfo| List of Object | NAT information of the gateway. Empty list if none.  |
|gatewayList.isCutomized| bool | Internal flag. `false` for gateways resolved by NetBrain.  |
|transip| string | The IP address the gateways were resolved for.  |


> ***Example***
```python
{
    "gatewayList": [
        {
            "gatewayName": "Core-Service.Ethernet1/3(10.8.197.7)",
            "type": "Device Interface",
            "payload": "{\"ip\": \"10.8.197.7\", \"endPointInfo\": null, \"device\": \"Core-Service\", \"deviceId\": \"5993df20-fd6b-4c14-905e-803e395f9b5c\", \"interface\": \"Ethernet1/3\", \"interfaceId\": \"f611da85-141a-457c-a5dd-9f99baccfed2\", \"prefixLen\": 24, \"details\": [{\"device\": \"Core-Service\", \"deviceId\": \"5993df20-fd6b-4c14-905e-803e395f9b5c\", \"interface\": \"Ethernet1/3\", \"interfaceId\": \"f611da85-141a-457c-a5dd-9f99baccfed2\"}]}",
            "natInfo": [],
            "isCutomized": false
        }
    ],
    "transip": "10.8.197.11",
    "statusCode": 790200,
    "statusDescription": "Success."
}
```

 # Full Example:
```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request inputs
token = "34713abf-71d6-41c6-8fd4-d3b8f3066959"
nb_url = "http://192.168.28.79"
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Path/Gateways"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

ipOrHost = "10.8.197.11"

data = {"ipOrHost":ipOrHost}

try:
    response = requests.get(full_url, params = data, headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Resolve Device Gateway! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```

    {'gatewayList': [{'gatewayName': 'CA-TOR-R1.Ethernet0/3(10.8.197.9)', 'type': 'Device Interface', 'payload': '{"ip": "10.8.197.9", "endPointInfo": null, ...}', 'natInfo': [], 'isCutomized': False}, {'gatewayName': 'Core-Service.Ethernet1/3(10.8.197.7)', 'type': 'Device Interface', ...}, ...], 'transip': '10.8.197.11', 'statusCode': 790200, 'statusDescription': 'Success.'}
    

 # cURL Code form Postman:
```python
curl -X GET \
  'http://192.168.28.79/ServicesAPI/API/V1/CMDB/Path/Gateways?ipOrHost=10.1.13.2' \
  -H 'Postman-Token: 67f40439-549a-4689-929e-26d9651597ad' \
  -H 'cache-control: no-cache' \
  -H 'token: c4edcb21-8d27-42a3-be0c-7e3b53b608c7'
```

 # Error Examples:

## Error Example 1: Empty `ipOrHost`
```python
Input:

        ipOrHost = "" # cannot be empty

Response:

    Failed to Resolve Device Gateway! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'IpOrHost'. The IpOrHost field is required."}

```

## Error Example 2: No gateway found for `ipOrHost`
```python
Input:

        ipOrHost = "198.51.100.25" # not in any subnet that NetBrain has discovered

Response:

    Failed to Resolve Device Gateway! - {"statusCode":792040,"statusDescription":"Gateway was not found."}

```

## Note 1: Gateway of a client vs. the device interface itself
- `ipOrHost` = a **host IP** in a discovered subnet (for example `10.8.197.11`): the API returns the gateways *for* that host, with `"endPointInfo": null` in `payload`. Use this to choose the gateway of a client manually: pass any unused host IP in the subnet of the gateway interface you want, pick the entry by `gatewayName`, and send it as `sourceGateway` with the real client IP as `sourceIP` in the [Calculate Path API](https://github.com/NetBrainAPI/NetBrain-REST-API-R13/blob/main/REST%20APIs%20Documentation/Path%20Management/Calculate%20Path%20API.md).
- `ipOrHost` = a **device's own IP or hostname** (for example `10.8.197.7` or `Core-Service`): the API returns that device interface as the source itself, with `endPointInfo` set. If this entry is sent as `sourceGateway`, the path is calculated from the device's IP, not from `sourceIP`.
