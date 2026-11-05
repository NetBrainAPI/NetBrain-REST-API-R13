
# Path API Design

## ***POST*** /V1/CMDB/Path/Calculation
Call this API to calculate the path from endpoint A (source) to endpoint B (destination). <br> 
It returns the result of the calculated path in the form of a path ID (a string), which can be used as the input parameter to get each hop information of the path via [Get Path Calculation Result API](https://github.com/NetBrainAPI/NetBrain-REST-API-R13/blob/main/REST%20APIs%20Documentation/Path%20Management/Get%20Path%20Calculation%20Result%20API.md) or [Get Path Calculation Overview API](https://github.com/NetBrainAPI/NetBrain-REST-API-R13/blob/main/REST%20APIs%20Documentation/Path%20Management/Get%20Path%20Calculation%20Overview%20API.md).

## Detail Information

> **Title** : Calculate Path API<br>

> **Version** : 05/09/2023.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Path/Calculation

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|sourceIP* | string  | Input the IP address of the source device.  |
|sourcePort | integer  | Specify the source protocol port If TCP/UDP is selected such as 23 for telnet. This parameter can be null.  |
|sourceGateway* | Object  | The gateway of the source: one entry of the `gatewayList` returned by the [Resolve Device Gateway API](https://github.com/NetBrainAPI/NetBrain-REST-API-R13/blob/main/REST%20APIs%20Documentation/Path%20Management/Resolve%20Device%20Gateway%20API.md), sent unchanged. An empty object (`{}`) or a gateway object built by hand is accepted, but the path calculation then fails with `Gateway device was not found` (see Note 2). To use a gateway you choose, see Note 1.  |
|sourceGateway.type* | string | `type` of the Resolve Device Gateway API entry, such as `Device Interface`.|
|sourceGateway.gatewayName* | string  | `gatewayName` of the Resolve Device Gateway API entry, such as `Core-Service.Ethernet1/3(10.8.197.7)`.  |
|sourceGateway.payload*| string | `payload` of the Resolve Device Gateway API entry: a JSON-encoded string. Internal data structure, pass it unchanged.  |
|destIP* | string  | Input IP address of the destination device.  |
|destPort* | integer  | Specify the destination protocol port If TCP/UDP is selected, such as 23 for telnet. This parameter can be null.  |
|protocol* | integer  | Specify the application protocol. see list_of_ip_protocol_numbers, such as 4 for IPv4.  |
|isLive* | bool  | ▪ `False` - Use data From current Baseline<br>▪ `True` - Use data via live access |
|advanced |object |advance setting.|
|advanced.debugMode | bool	|The debug mode of trigger API.|
|advanced.calcWhenDeniedByACL| bool | Whether to keep calculate when the process been denied by ACL.|
|advanced.calcWhenDeniedByPolicy |bool |Whether to keep calculate when the process been denied by policy.|
|advanced.calcL3ActivePath| bool |Whether to calculate L3 active path.|
|advanced.useCommandsWithArguments| bool |Whether to use the commands with arguments inside.|
|advanced.enablePathFixup |bool |Whether to enable the path fixup feature.|
|routingScheme |integer |Scheme for routing process and with two values 0 and 1 can be selected. 0 - `UNICAST`, 1 - `MULTICAST`.|
|group |string |Group name for Unicast calculation. (The group must have value when the routing scheme is 0).|

## Parameters(****required***)

>No parameters required.

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
|taskID| string | The task ID of the calculated path. <br> Use this as `taskID` input parameter in Get Path Calculation Result API or Get Path Calculation Overview API to get the hop information of the path. |
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |

> ***Example***
```python
{
    'taskID': 'b0bae173-af8c-418b-8dc4-1ffdb0e897fa',
    'statusCode': 790200,
    'statusDescription': 'Success.'
}
```

# Full Example:
```python
Calculate_Path_url = nb_url + "/ServicesAPI/API/V1/CMDB/Path/Calculation"

gwName = source_gateway["gatewayList"][0]["gatewayName"]
gwType = source_gateway["gatewayList"][0]["type"]
gw = source_gateway["gatewayList"][0]["payload"]

sourceIP = source_device
sourcePort = None #can be 8080
destIP = destination_device
destPort = 0
pathAnalysisSet = 2
protocol = 4
isLive = True

body = {
    "sourceIP" : sourceIP,                # IP address of the source device.
    "sourcePort" : sourcePort,
    "sourceGateway" : {
        "type" : gwType,
        "gatewayName" : gwName,
        "payload" : gw
    },    
    "destIP" : destIP,                    # IP address of the destination device.
    "destPort" : destPort,
    "protocol" : protocol,                # Specify the application protocol, check online help, such as 4 for IPv4.
    "isLive" : isLive                     # False: Current Baseline; True: Live access
} 


def calculate_path(Calculate_Path_url, body, headers, token):
    headers["Token"] = token
    
    try:
        response = requests.post(Calculate_Path_url, data = json.dumps(body), headers = headers, verify = False)
        if response.status_code == 200:
            result = response.json()
            return (result)
        else:
            return ("Failed to Calculate Path! - " + str(response.text))

    except Exception as e:
        return (str(e)) 

res = calculate_path(Calculate_Path_url, body, headers, token)
res
```
```python
    {'taskID': 'dcf25655-81a9-4cfe-82ca-aef80a698971',
     'statusCode': 790200,
     'statusDescription': 'Success.'}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.28.173/ServicesAPI/API/V1/CMDB/Path/Calculation \
  -H 'Accept: */*' \
  -H 'Cache-Control: no-cache' \
  -H 'Connection: keep-alive' \
  -H 'Content-Type: application/json' \
  -H 'Host: 192.168.28.173' \
  -H 'Postman-Token: 8e0bc75f-58c8-48fe-869a-0535f6385590,b98e9443-bf6d-4b12-be03-bf580231fa38' \
  -H 'User-Agent: PostmanRuntime/7.13.0' \
  -H 'accept-encoding: gzip, deflate' \
  -H 'cache-control: no-cache' \
  -H 'content-length: 581' \
  -H 'token: 76f95ffa-fc1b-4eb6-a503-75760185f2a5' \
  -d '{
"sourceIP": "172.24.33.10",
 "sourcePort": None,
 "sourceGateway": {
 	"type": "Device Interface",
    "gatewayName": "BJ_L2_test_1.Vlan10(172.24.33.10)",
    "payload": "{"ip": "172.24.33.10", "endPointInfo": {"deviceId": "101deae6-8d11-47d2-b87f-b69cbe7ba2ce", "interfaceId": "9a40c2e8-12ba-4556-bb2f-9545f776afb7"}, "device": "BJ_L2_test_1", "deviceId": "101deae6-8d11-47d2-b87f-b69cbe7ba2ce", "interface": "Vlan10", "interfaceId": "9a40c2e8-12ba-4556-bb2f-9545f776afb7", "prefixLen": 26}"},
 "destIP": "172.24.33.135",
 "destPort": 0,
 "protocol": 4,
 "isLive": True	
}'
```

# Error Examples and Note

## Error Example 1: `sourceGateway` is missing
```python
Input:

        body = {
            "sourceIP" : "198.51.100.25",
            "destIP" : "10.8.1.51",
            "protocol" : 4,
            "isLive" : False
        } # sourceGateway is required

Response:

    Failed to Calculate Path! - {"statusCode":791009,"statusDescription":"Invalid value for parameter 'SourceGateway'. The SourceGateway field is required."}

```

## Note 1: Use a gateway you choose for a client IP
When NetBrain cannot resolve the gateway of the source IP, the [Resolve Device Gateway API](https://github.com/NetBrainAPI/NetBrain-REST-API-R13/blob/main/REST%20APIs%20Documentation/Path%20Management/Resolve%20Device%20Gateway%20API.md) returns `792040 Gateway was not found.` To calculate the path anyway:

1. Call the Resolve Device Gateway API with any unused host IP in the subnet of the gateway interface you want (for example `10.8.197.11` for the gateway interface `10.8.197.7/24`).
2. Pick the `gatewayList` entry whose `gatewayName` is that gateway.
3. Call this API with `sourceIP` set to the client IP and `sourceGateway` set to that entry.

```python
Input:

        body = {
            "sourceIP" : "198.51.100.25", # client IP, no gateway found for it
            "sourcePort" : None,
            "sourceGateway" : {
                "type" : "Device Interface",
                "gatewayName" : "Core-Service.Ethernet1/3(10.8.197.7)",
                "payload" : "{\"ip\": \"10.8.197.7\", \"endPointInfo\": null, \"device\": \"Core-Service\", ...}" # from the Resolve Device Gateway API, ipOrHost = "10.8.197.11"
            },
            "destIP" : "10.8.1.51",
            "destPort" : None,
            "protocol" : 4,
            "isLive" : False
        }

Response:

    {'taskID': '...', 'statusCode': 790200, 'statusDescription': 'Success.'}
    # Get Path Calculation Overview API: 'description': '198.51.100.25 -> 10.8.1.51', 'status': 'Success'

```

Do **not** resolve the gateway with the gateway's own IP address or hostname. That entry has a non-null `endPointInfo` in its `payload`, meaning the gateway interface is the source itself, and the path is then calculated from the gateway's IP (`10.8.197.7 -> 10.8.1.51`) instead of from `sourceIP`. See the [Get Path By Calling APIs](https://github.com/NetBrainAPI/NetBrain-REST-API-R13/blob/main/Golden%20Use%20Case%20Templates/Get%20Path%20By%20Calling%20APIs.md#choose-the-source-gateway-manually) use case for the full sample code.

## Note 2: Empty or hand-built `sourceGateway`
An empty `sourceGateway`, or one with only a `gatewayName`, is accepted and returns a taskID, but the path calculation fails. Check the path status, not only the HTTP status code.
```python
Input 1:

        "sourceGateway" : {} # empty

Response 1:

    {'taskID': '...', 'statusCode': 790200, 'statusDescription': 'Success.'}
    # Get Path Calculation Status API: {'result': {'resultCode': 3, 'resultDescription': 'Failed'}, ...}
    # Get Path Calculation Overview API: 'failure_reason': 'Gateway device was not found'

```
```python
Input 2:

        "sourceGateway" : {
            "gatewayName" : "Core-Service.Ethernet1/3(10.8.197.7)" # type and payload are missing
        }

Response 2:

    {'taskID': '...', 'statusCode': 790200, 'statusDescription': 'Success.'}
    # Get Path Calculation Overview API: 'failure_reason': 'Gateway device was not found'

```
