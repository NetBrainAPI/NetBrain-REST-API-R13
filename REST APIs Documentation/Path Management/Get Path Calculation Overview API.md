
# Path API Design

## ***GET*** /V1/CMDB/Path/Calculation/{taskID}/OverView
Call this API to get the hop information of the calculated path achieved through [Calculate Path API](https://github.com/NetBrainAPI/NetBrain-REST-API-R13/blob/main/REST%20APIs%20Documentation/Path%20Management/Calculate%20Path%20API.md). 

If the [Calculation Path](https://github.com/NetBrainAPI/NetBrain-REST-API-R13/blob/main/REST%20APIs%20Documentation/Path%20Management/Calculate%20Path%20API.md) task is not yet finished or failed, the API will prompt an error with message accordingly. 

All directed links in the result consists of a directed path graph, which contains all possible reachable paths from the original source to the destination specified in path calculation.


## Detail Information

> **Title** : Get Path Calculation Overview API<br>

> **Version** : 01/30/2019.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Path/Calculation/{taskID}/OverView	

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
|taskID* | string  | Input the task ID returned by the Calculate Path API. |

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
The below table mainly includes the fields meant for customer's use. <br>
To view all the generated fields, please refer to the examples below.
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| path_overview | list of Object | A list of path_list.|
| path_list | list of Object | A list of path.|
| branch_list | list of Object | A list of branch.|
| hop_detail_list | list of Object | A list of hop with hop detail information.|
| acl | string | Access Control List of the hop.|
| pbr | string | Policy-Based Routing of the hop.|
| policy | string | Policy of the hop.|
| fromDev | Object | Detail information of source device. |
| devId | string | ID of source device. |
| devName | string | Hostname of source device. |
| devType | integer | Type of source device. |
| fromIntf | Object | The detailed information of source device interface. |
| hopId | string | The ID of current hop. |
| isP2P | boolean | `True` - P2P <br> `False` - not P2P |
| mediaInfo | Object | The detailed information of media. |
| failure_reason | string | Explanation of the failure path. |
| description | string | Path description. |
| status | string | Status of the path calculation. |
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |

> ***Example***
```python
[
  {
    "path_overview": [
      {
        "path_list": [
          {
            "branch_list": [
              {
                "hop_detail_list": [
                  {
                    "acl": "",
                    "pbr": "",
                    "policy": "Policy Default Policy on device BUR-R206-Forti200F-1",
                    "fromDev": {
                      "devId": "8914cf5c-abe2-426e-9d72-fb16eebd9f10",
                      "devName": "BUR-R206-Forti200F-1",
                      "devType": 10380,
                      "domainId": ""
                    },
                    "fromIntf": {
                      "intfKeyObj": {
                        "schema": "ipIntfs._id",
                        "value": "68aeaabc-8ed5-4399-8021-d6be25e80a9f"
                      },
                      "intfDisplaySchemaObj": {
                        "schema": "ipIntfs.name",
                        "value": "mgmt 192.168.0.10/22"
                      },
                      "PhysicalInftName": "mgmt",
                      "ipLoc": "192.168.0.10/22"
                    },
                    "hopId": "0511b746-31af-46f9-a358-5bb7d990b316",
                    "isComplete": false,
                    "isP2P": false,
                    "mediaId": "433a2fec-c77c-4aa4-9118-93873e8e06a0",
                    "mediaInfo": {
                      "mediaName": "192.168.0.0/22",
                      "mediaType": "Lan",
                      "neat": false,
                      "topoType": "L3_Topo_Type"
                    },
                    "preHopId": "00000000-0000-0000-0000-000000000000",
                    "sequnce": 0,
                    "toIntf": {
                      "intfKeyObj": {
                        "schema": "",
                        "value": ""
                      },
                      "intfDisplaySchemaObj": {
                        "schema": "",
                        "value": ""
                      },
                      "PhysicalInftName": "",
                      "ipLoc": ""
                    },
                    "toDev": {
                      "devId": "00000000-0000-0000-0000-003232235777",
                      "devName": "192.168.1.1",
                      "devType": 1036,
                      "domainId": ""
                    },
                    "topoType": "L3_Topo_Type",
                    "trafficState": {
                      "acl": "",
                      "alg": -1,
                      "dev_name": "BUR-R206-Forti200F-1",
                      "dev_type": 10380,
                      "in_intf": "mgmt",
                      "in_intf_schema": "intfs",
                      "in_intf_topo_type": "L3_Topo_Type",
                      "next_dev_in_intf": "",
                      "next_dev_in_intf_schema": "",
                      "next_dev_in_intf_topo_type": "L3_Topo_Type",
                      "next_dev_name": "192.168.1.1",
                      "next_dev_type": 1036,
                      "next_hop_ip": "192.168.1.1",
                      "next_hop_mac": "",
                      "out_intf": "mgmt 192.168.0.10/22",
                      "out_intf_schema": "ipIntfs",
                      "out_intf_topo_type": "L3_Topo_Type",
                      "pbr": "",
                      "vrf": ""
                    },
                    "parentHopId": "",
                    "isGateway": false,
                    "techs": []
                  },
                  {
                    "acl": "",
                    "pbr": "",
                    "policy": "",
                    "fromDev": {
                      "devId": "00000000-0000-0000-0000-003232235777",
                      "devName": "192.168.1.1",
                      "devType": 1036,
                      "domainId": ""
                    },
                    "fromIntf": {
                      "intfKeyObj": {
                        "schema": "",
                        "value": ""
                      },
                      "intfDisplaySchemaObj": {
                        "schema": "",
                        "value": ""
                      },
                      "PhysicalInftName": "",
                      "ipLoc": ""
                    },
                    "hopId": "a720a049-b23a-4bc1-89ae-4a894ad2fe92",
                    "isComplete": false,
                    "isP2P": false,
                    "mediaId": "",
                    "mediaInfo": {
                      "mediaName": "",
                      "mediaType": "",
                      "neat": false,
                      "topoType": ""
                    },
                    "preHopId": "0511b746-31af-46f9-a358-5bb7d990b316",
                    "sequnce": 1,
                    "toIntf": {
                      "intfKeyObj": {
                        "schema": "",
                        "value": ""
                      },
                      "intfDisplaySchemaObj": {
                        "schema": "",
                        "value": ""
                      },
                      "PhysicalInftName": "",
                      "ipLoc": ""
                    },
                    "toDev": {
                      "devId": "",
                      "devName": "",
                      "devType": 0,
                      "domainId": ""
                    },
                    "topoType": "",
                    "trafficState": {
                      "acl": "",
                      "alg": -1,
                      "dev_name": "192.168.1.1",
                      "dev_type": 1036,
                      "in_intf": "",
                      "in_intf_schema": "",
                      "in_intf_topo_type": "L3_Topo_Type",
                      "next_dev_in_intf": "",
                      "next_dev_in_intf_schema": "",
                      "next_dev_in_intf_topo_type": "",
                      "next_dev_name": "",
                      "next_dev_type": 0,
                      "next_hop_ip": "",
                      "next_hop_mac": "",
                      "out_intf": "",
                      "out_intf_schema": "",
                      "out_intf_topo_type": "",
                      "pbr": "",
                      "vrf": ""
                    },
                    "parentHopId": "",
                    "isGateway": false,
                    "techs": []
                  }
                ],
                "failure_reason": "",
                "status": "Success",
                "category": "",
                "error_code": -1,
                "priority": 0
              }
            ],
            "failure_reasons": [],
            "path_name": "L3 Path",
            "description": "192.168.0.10 -> 192.168.1.1",
            "status": "Success"
          }
        ],
        "failure_reasons": [],
        "status": "Success"
      }
    ],
    "statusCode": 790200,
    "statusDescription": "Success."
  }
]
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
taskID = "498c10a7-0011-4a59-a4cd-0258af3edd19"

token = "c4edcb21-8d27-42a3-be0c-7e3b53b608c7"
nb_url = "http://192.168.28.79"
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Path/Calculation/" + str(taskID) + "/Result"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

try:
    response = requests.get(full_url, headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        
        print ("Failed to Get Path Calculation Overview! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```
```python
    {"path_overview":[{"path_list":[{"branch_list":[{"hop_detail_list":[{"acl":"","pbr":"","policy":"Policy Default Policy on device BUR-R206-Forti200F-1","fromDev":{"devId":"8914cf5c-abe2-426e-9d72-fb16eebd9f10","devName":"BUR-R206-Forti200F-1","devType":10380,"domainId":""},"fromIntf":{"intfKeyObj":{"schema":"ipIntfs._id","value":"68aeaabc-8ed5-4399-8021-d6be25e80a9f"},"intfDisplaySchemaObj":{"schema":"ipIntfs.name","value":"mgmt 192.168.0.10/22"},"PhysicalInftName":"mgmt","ipLoc":"192.168.0.10/22"},"hopId":"0511b746-31af-46f9-a358-5bb7d990b316","isComplete":false,"isP2P":false,"mediaId":"433a2fec-c77c-4aa4-9118-93873e8e06a0","mediaInfo":{"mediaName":"192.168.0.0/22","mediaType":"Lan","neat":false,"topoType":"L3_Topo_Type"},"preHopId":"00000000-0000-0000-0000-000000000000","sequnce":0,"toIntf":{"intfKeyObj":{"schema":"","value":""},"intfDisplaySchemaObj":{"schema":"","value":""},"PhysicalInftName":"","ipLoc":""},"toDev":{"devId":"00000000-0000-0000-0000-003232235777","devName":"192.168.1.1","devType":1036,"domainId":""},"topoType":"L3_Topo_Type","trafficState":{"acl":"","alg":-1,"dev_name":"BUR-R206-Forti200F-1","dev_type":10380,"in_intf":"mgmt","in_intf_schema":"intfs","in_intf_topo_type":"L3_Topo_Type","next_dev_in_intf":"","next_dev_in_intf_schema":"","next_dev_in_intf_topo_type":"L3_Topo_Type","next_dev_name":"192.168.1.1","next_dev_type":1036,"next_hop_ip":"192.168.1.1","next_hop_mac":"","out_intf":"mgmt 192.168.0.10/22","out_intf_schema":"ipIntfs","out_intf_topo_type":"L3_Topo_Type","pbr":"","vrf":""},"parentHopId":"","isGateway":false,"techs":[]},{"acl":"","pbr":"","policy":"","fromDev":{"devId":"00000000-0000-0000-0000-003232235777","devName":"192.168.1.1","devType":1036,"domainId":""},"fromIntf":{"intfKeyObj":{"schema":"","value":""},"intfDisplaySchemaObj":{"schema":"","value":""},"PhysicalInftName":"","ipLoc":""},"hopId":"a720a049-b23a-4bc1-89ae-4a894ad2fe92","isComplete":false,"isP2P":false,"mediaId":"","mediaInfo":{"mediaName":"","mediaType":"","neat":false,"topoType":""},"preHopId":"0511b746-31af-46f9-a358-5bb7d990b316","sequnce":1,"toIntf":{"intfKeyObj":{"schema":"","value":""},"intfDisplaySchemaObj":{"schema":"","value":""},"PhysicalInftName":"","ipLoc":""},"toDev":{"devId":"","devName":"","devType":0,"domainId":""},"topoType":"","trafficState":{"acl":"","alg":-1,"dev_name":"192.168.1.1","dev_type":1036,"in_intf":"","in_intf_schema":"","in_intf_topo_type":"L3_Topo_Type","next_dev_in_intf":"","next_dev_in_intf_schema":"","next_dev_in_intf_topo_type":"","next_dev_name":"","next_dev_type":0,"next_hop_ip":"","next_hop_mac":"","out_intf":"","out_intf_schema":"","out_intf_topo_type":"","pbr":"","vrf":""},"parentHopId":"","isGateway":false,"techs":[]}],"failure_reason":"","status":"Success","category":"","error_code":-1,"priority":0}],"failure_reasons":[],"path_name":"L3 Path","description":"192.168.0.10 -> 192.168.1.1","status":"Success"}],"failure_reasons":[],"status":"Success"}],"statusCode":790200,"statusDescription":"Success."},
```

# cURL Code from Postman:
```python
curl -X GET \
  http://192.168.28.173/ServicesAPI/API/V1/CMDB/Path/Calculation/dcf25655-81a9-4cfe-82ca-aef80a698971/OverView \
  -H 'Accept: */*' \
  -H 'Cache-Control: no-cache' \
  -H 'Connection: keep-alive' \
  -H 'Host: 192.168.28.173' \
  -H 'Postman-Token: b0cd60b6-d29d-4665-9469-8969c925ee92,c4d2df42-2573-402b-8327-174c45ee1356' \
  -H 'User-Agent: PostmanRuntime/7.13.0' \
  -H 'accept-encoding: gzip, deflate' \
  -H 'cache-control: no-cache' \
  -H 'token: db5757cb-aaa6-4efb-ad25-ce4918def0ce'
```
