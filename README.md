# Introduction 

This demo is a basic extension of an existing Azure IoT tutorial to demonstrate SAP connectivity leveraging a public SAP Gateway demo system. 

The intention is to have a repeatable and public SAP and Azure IoT demo based on:

* Azure Raspberry emulator

* Azure IoT Hub and Logic App

* Public SAP demo system 





![Demo high level overview](https://github.com/ROBROICH/REPO1/blob/master/images/DEMO_ARCHITECTURE.jpg)

# Demo scenario and basic story line 

* Imagine the Raspberry Emulator as IoT device used in shipping of heat-sensitive vaccines

* If the temperature is measured above 30°, the vaccine is damaged and must be replaced  

* To replace the vaccine, a sales order is automatically created in SAP ECC in case the measured temperature is above 30° 

* For excel users the current sales order SAP ECC will be displayed in Excel 

1
# Introduction 
2
​
3
This demo is a basic extension of an existing Azure IoT tutorial to demonstrate SAP connectivity leveraging a public SAP Gateway demo system. 
4
​
5
The intention is to have a repeatable and public SAP and Azure IoT demo based on:
6
​
7
* Azure Raspberry emulator
8
​
9
* Azure IoT Hub and Logic App
10
​
11
* Public SAP demo system 
12
​
13
​
14
​
15
​
16
​
17
![Demo high level overview](https://github.com/ROBROICH/REPO1/blob/master/images/DEMO_ARCHITECTURE.jpg)
18
​
19
# Demo scenario and basic story line 
20
​
21
* Imagine the Raspberry Emulator as IoT device used in shipping of heat-sensitive vaccines
22
​
23
* If the temperature is measured above 30°, the vaccine is damaged and must be replaced  
24
​
25
* To replace the vaccine, a sales order is automatically created in SAP ECC in case the measured temperature is above 30° 
26
​
27
* For excel users the current sales order SAP ECC will be displayed in Excel 
28
​
29
# Demo implementation
30
​
31
The following steps are required to implement the demo:
32
* Raspberry PI emulator
33
​
34
* Notifications with Azure Logic Apps
35
​
36
* Get access to public SAP system
37
​
38
* Adjust Logic-App
39
​
40
* Show sales order in Excel
41
​
42
![Demo flow](https://github.com/ROBROICH/REPO1/blob/master/images/DEMO_FLOW.jpg)
43
​
44
## Raspberry PI emulator & Notifications with Azure Logic Apps
45
​
46
* Implement Raspberry PI emulator tutorial ([Link](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-raspberry-pi-web-simulator-get-started))
47
​
48
* Implement IoT remote monitoring and notifications with Azure Logic Apps connecting your IoT hub and mailbox tutorial ([Link](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps)) _ Stop at the paragraph “Configure the logic app trigger”_

# Demo implementation

The following steps are required to implement the demo:
* Raspberry PI emulator

* Notifications with Azure Logic Apps

* Get access to public SAP system

* Adjust Logic-App

* Show sales order in Excel

![Demo flow](https://github.com/ROBROICH/REPO1/blob/master/images/DEMO_FLOW.jpg)

## Raspberry PI emulator & Notifications with Azure Logic Apps

* Implement Raspberry PI emulator tutorial ([Link](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-raspberry-pi-web-simulator-get-started))

* Implement IoT remote monitoring and notifications with Azure Logic Apps connecting your IoT hub and mailbox tutorial ([Link](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps)) _ Stop at the paragraph “Configure the logic app trigger”_

## Access to public SAP demo system

* Get access to SAP Gateway demo system([Link](https://blogs.sap.com/2017/12/05/new-sap-gateway-demo-system-available/comment-page-1/))

* Get familiar with [GWSampleBasic](https://sapes5.sapdevcenter.com/sap/opu/odata/iwbep/GWSAMPLE_BASIC/SalesOrderSet?(%270500000001%27)/ToLineItems) Odata Service ([Visual-Code REST Client](https://github.com/Huachao/vscode-restclient))

Coding for VSCode Restclient

```

# sapes5.sapdevcenter.com
# Authorization: YOURUSERNAME:YOURPWD

# @name login
GET https://sapes5.sapdevcenter.com/sap/opu/odata/iwbep/GWSAMPLE_BASIC/$metadata HTTP/1.1
Authorization: Basic YOURUSERNAME:YOURPWD
X-CSRF-Token: fetch

@authToken = {{login.response.headers.X-CSRF-Token}}

# @name salesOrder
POST https://sapes5.sapdevcenter.com/sap/opu/odata/iwbep/GWSAMPLE_BASIC/SalesOrderSet HTTP/1.1
Authorization: YOURUSERNAME:YOURPWD
X-CSRF-Token: {{authToken}}
Content-Type: application/json


{
   "SalesOrderID" : "0500000011",
    "Note" : "SAP MFST DEMO 1234",
	"NoteLanguage" : "E",
	"CustomerID" : "0100000010",
	"CustomerName" : "SAP",
	"CurrencyCode" : "EUR",
	"GrossAmount" : "99",
	"NetAmount" : "100",
	"TaxAmount" : "200",
	"LifecycleStatus" : "N",
	"LifecycleStatusDescription" : "New",
	"BillingStatus" : "",
	"BillingStatusDescription" : "Initial",
	"DeliveryStatus" : null,
	"DeliveryStatusDescription" : "Initial",
	"CreatedAt" : "2012-10-10T00:00:00",
	"ChangedAt" : "2014-10-10T00:00:00"
}

```

## Logic App implementation 
Remark: Without the support of Bartosz Jarkowski the demo would end here. Further details [here](https://blogs.sap.com/2019/05/28/your-sap-on-azure-part-18-the-story-of-a-missing-csrf-token/). 

Finish the [tutorial](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps) until : Configure the logic app trigger / 5. Create a service bus connection.
Instead of creating an STMP connection, create a HTTP GET Action

### GET Request 

```
URI: https://sapes5.sapdevcenter.com/sap/opu/odata/iwbep/GWSAMPLE_BASIC/SalesOrderSet/
Header parameter:
Connection: keep-alive
X-CSRF-Token: fetch
Authentication: Basic
User/Pwd: Credentials for SAP demo system
```

![GET Request](https://github.com/ROBROICH/REPO1/blob/master/images/Get1.jpg)

### POST Request 

* Append a HTTP POST Action to the GET-request 
* The logic app creates an SAP sales order via the Odata Gateway

```
Headers: 
"Connection": "keep-alive",
"X-Requested-With": "XMLHttpRequest",
"x-csrf-token": "@outputs('GET')['headers']['x-csrf-token']"

Body:
                        "BillingStatus": "",
                        "BillingStatusDescription": "Initial",
                        "ChangedAt": "2014-10-10T00:00:00",
                        "CreatedAt": "2012-10-10T00:00:00",
                        "CurrencyCode": "EUR",
                        "CustomerID": "0100000000",
                        "CustomerName": "SAP",
                        "DeliveryStatus": null,
                        "DeliveryStatusDescription": "Initial",
                        "GrossAmount": "2000",
                        "LifecycleStatus": "N",
                        "LifecycleStatusDescription": "New",
                        "NetAmount": "100",
                        "Note": "SAP and Azure in Berlin",
                        "NoteLanguage": "E",
                        "SalesOrderID": "0500000011",
                        "TaxAmount": "200"

Cookie: 
@{replace(outputs('GET')['headers']['Set-cookie'], ',', ';')}

```
![Post request](https://github.com/ROBROICH/REPO1/blob/master/images/post.png)

## Run the demo 
* Run the logic app
* Run the Raspberry emulator. Wait until the emulator sends an alert, when temperature is >30°
* The logic app gets executed 

![Logic App](https://github.com/ROBROICH/REPO1/blob/master/images/post_finished.png)

## Check data in excel 
After the logic app got executed, the data is stored in SAP and available for consumption by business users. 

![Run the demo](https://github.com/ROBROICH/REPO1/blob/master/images/run%20the%20demo.png)

Connect to Odata datasource

`URL:  https://sapes5.sapdevcenter.com/sap/opu/odata/iwbep/GWSAMPLE_BASIC/SalesOrderSet`

![Excel screen](https://github.com/ROBROICH/REPO1/blob/master/images/excel.png)

Display results: 

![Result](https://github.com/ROBROICH/REPO1/blob/master/images/excel1.png)

## Demo finished!

