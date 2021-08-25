* TOC
{:toc}

ThingsBoard allows you to send commands (Remote Procedure Calls or [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call)) 
from the [dashboards](/docs/{{docsPrefix}}user-guide/dashboards/) or [server-side applications]() to devices and vice versa. 
This guide covers ThingsBoard RPC feature capabilities. After reading this guide, you will get familiar with the following topics:

- RPC types;
- Basic RPC use-cases;
- RPC client-side and server-side APIs;
- RPC widgets.


## Server-side RPC

Server-side RPC is the type of RPC call that is sent from the server to the device. 
For example, you may use this type of RPC to trigger reboot of the device or remotely turn the engine on or off.

Server-side RPC can be divided into one-way and two-way:

- One-way RPC request does not expect device to send any response back to the server.
  For example, request to reboot the device;

  {:refdef: style="text-align: center;"}
  ![image](/images/user-guide/one-way-rpc.svg)
  {: refdef}

- Two-way RPC request is sent to the device and expects to receive a response from the device within a certain timeout.
  For example, change the current engine state or get the current value of a certain parameter;

  {:refdef: style="text-align: center;"}
  ![image](/images/user-guide/two-way-rpc.svg)
  {: refdef}
  
### Lightweight RPC

By default, the RPC you send to the device via ThingsBoard is not persisted to any database and is very lightweight. 
ThingsBoard will attempt to send the call to device if device is online. 
The RPC may fail if there is no active connection with the target device within a configurable timeout period.
The RPC may also fail in case of network issue between the server and the device or reboot of the server while the command is being delivered.

To summarize, using lightweight RPC is preferable, if you would like to minimize load on the server and can tolerate that some commands may be lost when device is offline 
or there is a server outage.

### Persistent RPC

Since version 3.3, ThingsBoard provides support of persistent RPC which are stored in the database. 
The RPC will be delivered to the device once it goes online. 
In case of any network failure, the RPC will be re-send to device for configurable number of times until it is delivered or expired.

ThingsBoard supports following persistent RPC states:

 * **QUEUED** - RPC was saved to the database;
 * **SENT** - The platform performed attempt to deliver the RPC. For example, sent the confirmable CoAP message; 
 * **DELIVERED** - The platform received confirmation from the device that the command was delivered. This is the final state of processing for the one-way command;
 * **TIMEOUT** - The platform has not received confirmation from the device that the command was delivered within configurable time. The platform may try again if device will become online;
 * **SUCCESSFUL** - The platform has received successful response from the device for a two-way RPC command;   
 * **FAILED** - The platform has received unsuccessful response from the device for a two-way RPC command;
 * **EXPIRED** - The RPC command has expired.

#### Rule Engine events



#### TTL Configuration

To enable periodic cleanup of the RPC calls from the database, alter following [configuration](/docs/{{docsPrefix}}user-guide/install/config/) parameters:

```
export SQL_TTL_RPC_ENABLED=true
export SQL_RPC_TTL_CHECKING_INTERVAL=7200000
```
{: .copy-code}

Where:

1. **SQL_TTL_RPC_ENABLED** <br>enables periodic RPC cleanup from the database.

2. **SQL_RPC_TTL_CHECKING_INTERVAL** <br>configures how often persistent RPC cleanup procedure will be executed. By default, this parameter is set to two hours (in milliseconds).

The system administrator can configure how many days the RPC calls are stored in the database for each Tenant Profile. This is **RPC TTL days configuration** parameter.
See the screenshot below:

{% include images-gallery.html imageCollection="tenant-profile-rpc" %}


## Client-side RPC


Thinsboard RPC feature can be divided into two types based on a originator: device-originated and server-originated RPC.
In order to use more familiar names, we will name device-originated RPC calls as a **client-side** RPC 
and server-originated RPC as **server-side** RPC.
  
   {:refdef: style="text-align: center;"}
   ![image](/images/user-guide/client-side-rpc.svg)
   {: refdef}  



## Device RPC API

ThingsBoard provides a convenient API to send and receive RPC commands from applications running on the device.
This API is specific for each supported network protocol.
You can review API and examples on the corresponding reference page:

 - [MQTT RPC API reference](/docs/{{docsPrefix}}reference/mqtt-api/#rpc-api)
 - [CoAP RPC API reference](/docs/{{docsPrefix}}reference/coap-api/#rpc-api)
 - [HTTP RPC API reference](/docs/{{docsPrefix}}reference/http-api/#rpc-api) 

## Server-side RPC API

ThingsBoard provides **System RPC Service** that allows you to send RPC calls from server-side applications to the device.
In order to send an RPC request you need to execute an HTTP POST request to the following URL:

```shell
http(s)://host:port/api/plugins/rpc/{callType}/{deviceId}
```

where 

 - **callType** is either **oneway** or **twoway**;
 - **deviceId** is your target [Device ID](/docs/{{docsPrefix}}user-guide/ui/devices/#get-device-id).

The request body should be a valid JSON object with two elements: 
 
 - **method** - method name, JSON string;
 - **params** - method parameters, JSON object.

For example:

{% capture tabspec %}mqtt-rpc-from-client
A,set-gpio-request.sh,shell,resources/set-gpio-request.sh,/docs/{{docsPrefix}}user-guide/resources/set-gpio-request.sh
B,set-gpio-request.json,json,resources/set-gpio-request.json,/docs/{{docsPrefix}}user-guide/resources/set-gpio-request.json{% endcapture %}  
{% include tabs.html %}

**Please note** that in order to execute this request, you will need to substitute **$JWT_TOKEN** with a valid JWT token.
This token should belong to either 

 - user with **TENANT_ADMIN** role;
 - user with **CUSTOMER_USER** role that owns the device identified by **$DEVICE_ID**.
 
You can use the following [guide](/docs/{{docsPrefix}}reference/rest-api/#rest-api-auth) to get the token.

## Persistent RPC

Since version 3.3, ThingsBoard provides the new feature: **Persistent RPC**.
Unlike basic RPC, Persistent RPC has an increased timeout and the command is stored in the database for configurable amount of time.
Persistent RPC is extremely useful when your device is in power-saving mode. 
Power-saving mode (or PSM) is when the device temporary is turning off to save the battery energy.
You can set the PSM in the device profile or device configuration. This feature is available for [CoAP](/docs/{{docsPrefix}}reference/coap-api/) and [LWM2M](/docs/{{docsPrefix}}reference/lwm2m-api/) only.
After you send an RPC request to this device, the request will be saved in the database for the time you configured and the device will receive the request and send the response when it is turned on again.  
In addition, every time you send the Persistent RPC, the response will contain RPC ID. Whenever you need to find a specific RPC and view its states and responses, you can do it with that ID through the database.

#### RPC Rule chain events 

In the Rule chain, you are able to configure events that will be dispatched every time you send an RPC request: RPC queued, RPC delivered, RPC successful, RPC timeout, RPC failed.
Configured RPC events reflect [RPC states](/docs/{{docsPrefix}}user-guide/rpc/#rpc-states).

{% include images-gallery.html imageCollection="rule-chain" %}

#### Usage of Persistent RPC

To send the Persistent RPC through ThingsBoard, you need to add RPC Debug Terminal widget to your dashboard. 
How to add RPC Debug Terminal and use this widget, you can read [here](/docs/{{docsPrefix}}reference/lwm2m-api/#rpc-commands).
Then, follow these steps to test the Persistent RPC:

{% include images-gallery.html imageCollection="rpc-test" showListImageTitles="true" %}

## RPC Rule Nodes
It is possible to integrate RPC actions into processing workflow. There are 2 Rule Nodes for working with RPC requests. 

-  [RPC reply](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/action-nodes/#rpc-call-reply-node) 
-  [RPC request](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/action-nodes/#rpc-call-request-node) 

## RPC widgets

See [widgets library](/docs/{{docsPrefix}}user-guide/ui/widget-library/#gpio-widgets) for more details.