* TOC
{:toc}

ThingsBoard provides a way to send commands from the dashboard or server-side application to the device and vise-versa.
For example, you may turn the lights on and off using the widget on the [dashboard](/docs/{{docsPrefix}}user-guide/dashboards/).
Similar, your device may send the request to ThingsBoard to get the weather forecast. 
Also, command from one device may be forwarded to related device. 

This guide covers the API to send, receive and process the commands with few examples.

## Terminology

It is important to use same terminology in regard to the remote commands:

 - RPC - the remote procedure call. For simplicity, we will often replace it with a word "command". Basically the command from or to the device.
 - Server-side RPC - Command that is originated from the platform and sent to the device.
 - Client-side RPC - Command that is originated from the device and sent to the platform.
 - One-way RPC - Command that is sent without expecting any result.
 - Two-way RPC - Command that is expecting some result of the execution to be sent backward.

## Server-side RPC

According to our [terminology](#terminology), this is a command originated **from the platform** and sent **to the device**. 
The server-side RPC command consist of the method name and parameters. The *method name* is always a text string while *parameters* may be of primitive type or JSON.
ThingsBoard supports one-way and two-way server-side RPC commands. This means that you can choose whether to wait for the reply from device or not. 

For example, the command below may be used to set the GPIO pin to certain value (0 or 1):

```json
{
  "method": "setGpio",
  "params": {
    "pin": "23",
    "value": 1
  }
}
```
{: .copy-code}

As you may notice, the command name is *setGpio* while parameters (params) is a JSON with two fields.

### Send command from Dashboards



### Send command using REST API

In order to send an RPC request you need to execute an HTTP POST request to the following URL:

{% if docsPrefix == "paas/" %}

```shell
https://thingsboard.cloud/api/plugins/rpc/{callType}/{deviceId}
```

{% else %}

```shell
http(s)://host:port/api/plugins/rpc/{callType}/{deviceId}
```

{% endif %}

where

- **callType** is either **oneway** or **twoway**;
- **deviceId** is your target [Device ID](/docs/{{docsPrefix}}user-guide/ui/devices/#get-device-id).

The request body should be a valid JSON object with two elements:

- **method** - method name, JSON string;
- **params** - method parameters, JSON object or primitive.

For example:

```shell
curl 'https://thingsboard.cloud/api/plugins/rpc/oneway/854ad1b0-43a5-11eb-938b-338066692c79' \
  -H 'accept: application/json, text/plain, */*' \
  -H 'x-authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhc2h2YXlrYUB0aGluZ3Nib2FyZC5pbyIsInNjb3BlcyI6WyJURU5BTlRfQURNSU4iXSwidXNlcklkIjoiYWMxYmJkMTAtM2Y5OC0xMWViLWE4ZDYtZjVhODdmMDdkNGJlIiwiZmlyc3ROYW1lIjoiSm9obiIsImxhc3ROYW1lIjoiRG9lIiwiZW5hYmxlZCI6dHJ1ZSwiaXNQdWJsaWMiOmZhbHNlLCJpc0JpbGxpbmdTZXJ2aWNlIjpmYWxzZSwicHJpdmFjeVBvbGljeUFjY2VwdGVkIjp0cnVlLCJ0ZXJtc09mVXNlQWNjZXB0ZWQiOnRydWUsInRlbmFudElkIjoiYWI5MzdhNDAtM2Y5OC0xMWViLWE4ZDYtZjVhODdmMDdkNGJlIiwiY3VzdG9tZXJJZCI6IjEzODE0MDAwLTFkZDItMTFiMi04MDgwLTgwODA4MDgwODA4MCIsImlzcyI6InRoaW5nc2JvYXJkLmNsb3VkIiwiaWF0IjoxNjE4MzA5MDIxLCJleHAiOjE2MTgzMzc4MjF9.V03unHAB1Ahmyk3efOiSW0rXxhKiNh2EKSQGjSNWvDfQy_MBEL5mp5W-RlHIXkiLYOj3F3Meeod0sPx96-RRow' \
  -H 'content-type: application/json' \
  --data-raw '{"method":"setGpio","params":"{\"pin\": 23, \"value\": 1}", "timeout":5000}' \
```

{% capture tabspec %}mqtt-rpc-from-client
A,set-gpio-request.sh,shell,resources/set-gpio-request.sh,/docs/{{docsPrefix}}user-guide/resources/set-gpio-request.sh
B,set-gpio-request.json,json,resources/set-gpio-request.json,/docs/{{docsPrefix}}user-guide/resources/set-gpio-request.json{% endcapture %}  
{% include tabs.html %}

#### Response codes


#### Oneway vs twoway commands

You should use **oneway** command if you are not interested in response from the device.
The caller will receive successful result if  

The **twoway** command request is blocked until    

### Command persistence and TTL



=======================================================================================================

## RPC call types

Thinsboard RPC feature can be divided into two types based on a originator: device-originated and server-originated RPC.
In order to use more familiar names, we will name device-originated RPC calls as a **client-side** RPC 
and server-originated RPC as **server-side** RPC.
  
   {:refdef: style="text-align: center;"}
   ![image](/images/user-guide/client-side-rpc.svg)
   {: refdef}  

Server-side RPC can be divided into one-way and two-way:
 
 - One-way RPC request is sent to the device without delivery confirmation and obviously does not provide any response from the device. 
   RPC may fail only if there is no active connection with the target device within a configurable timeout period.
   
   {:refdef: style="text-align: center;"}
   ![image](/images/user-guide/one-way-rpc.svg)
   {: refdef}
   
 - Two-way RPC request is sent to the device and expects to receive a response from the device within a certain timeout. 
   The Server-side request is blocked until the target device replies to the request.

   {:refdef: style="text-align: center;"}
   ![image](/images/user-guide/two-way-rpc.svg)
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

## RPC Rule Nodes
It is possible to integrate RPC actions into processing workflow. There are 2 Rule Nodes for working with RPC requests. 

-  [RPC reply](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/action-nodes/#rpc-call-reply-node) 
-  [RPC request](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/action-nodes/#rpc-call-request-node) 

## RPC widgets

See [widgets library](/docs/{{docsPrefix}}user-guide/ui/widget-library/#gpio-widgets) for more details.