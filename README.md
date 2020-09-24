# Overview


555 RTC SDK provides APIs which allow android developers to create apps with different features like Audio/Video/PSTN call, 
conference call, chat, screen sharing, etc. The services offered by SDK are supported by 555 platform's reliable and scalable infrastructure designed to handle a large number of calls. 

Below steps allow the user to make an end-to-end PSTN call. This includes user authentication, login, notification creating media connection.  

1) Follow [installation guide](https://github.com/555platform/555-rtc-android-sdk/wiki/Installation-Guide) to integrate SDK into your project.
2) From [555 portal](https://developer.555.comcast.com/), get an API key and access token.
3) Complete [login](https://github.com/555platform/555-rtc-android-sdk/wiki/User-Authentication) to get the JWT which is needed to access any SDK or backend APIs.
4) Follow [notification guide](https://github.com/555platform/555-rtc-android-sdk/wiki/Notification-Guide) to add support for APNS notification in your app.
4) Use below SDK API document to make incoming/outgoing PSTN calls.
  
# Usage

## PSTN Call

This document list down all the SDK API calls and their usage for making incoming/outgoing PSTN calls and using advance call features like call hold/unhold, mute/unmute, etc.  

Before initiating or accepting a PSTN call, make sure you have [logged in](https://github.com/555platform/555-rtc-android-sdk/wiki/User-Authentication) to 555 platform. 
  
  ## SDK initialization

Below API initializes and sets the context for the SDK. It is advisable to call this API at the launch of the application.

```java
Rtc555Sdk.initializeLibrary(appContext);
```

|Parameters| |  
  |------|-----|
  |appContext|Application context|
  
  ## Adding SDK configuration.

  To make PSTN/Voice related API call, SDK should have below mentioned mandatory configuration data to 
  establish a connection with the 555 platform backend.

  ```java
       Rtc555Config config = Rtc555Sdk.getInstance().getConfig();
       config.phoneNumber =  <Source Telephone Number>;
       config.routingId =  <555 platform login response will provide unique id for the logged in user>;
       config.url =  <Event manager URL>;
       config.token =  <555 platform login response will provide unique jwt token for the logged in user>;

       Rtc555Sdk.setConfig(config);
```
     
## Initiate an Outgoing Call

To make outgoing calls, pass destination telephone number, notification payload, instance of Rtc555Result observer to *dial* API and implement the *onSuccess()* and *onError()* methods of Rtc555Result class. Notification payload contains two fields called *data* and *notification*. Users can add any custom data as part *data* key which will get delivered to the remote end as part of the notification. The notification key contains the notification type and federation type of the notification (Both values are already populated in the below example).

  *dial* API returns a single value response. Success case of response will return callid .failure case returns Rtc555Exception object.
  ```java
   Rtc555Voice.dial(targetTelephoneNum, buildNotificationPayload(), new Rtc555Result() {
                        @Override
                        public void onSuccess(String callId) {
                            
                        }

                        @Override
                        public void onError(Rtc555Exception e) {

                        }
                    });
   ... 
   ... 
   
 //build notification  payload
 String buildNotificationPayload() {
     Context context = mAppContext;
  
     HashMap<String,HashMap> userdata=new HashMap<String,HashMap>(); 
     HashMap<String,String> notification=new HashMap<String,String>();
     HashMap<String,String> data=new HashMap<String,String>();
  
     try {
        data.put("cname", <Source Telephone Number>); // source telephone number
        data.put("cid", <Source Telephone Number>); // source telephone number
        data.put("tar", <Target Telephone Number>); // target telephone number
   
        notification.put("type", "pstn");
        notification.put("topic", "federation/pstn");
   
        userdata.put("data", data);
        userdata.put("notification", notification);
   
     } catch (JSONException e) {
        e.printStackTrace();
     }
 }
  ```

|Parameters| |
|------|-----|
|targetTelephoneNum|Target telephone number in e.164 format.|
|notificationPayload|Notification payload data.|
|new Rtc555Result()|instance of observer|


Below is the Notification payload need to be build for outgoing notificationData:


| Property          | Type   | Description           |
|-------------------|--------|-----------------------|
| data              | HashMap                                                               |  <ul> <li>cname</li><li>cid</li><li>targetId</li></ul> |
| notification | HashMap     |   <ul> <li>type:"pstn"</li><li>topic :"federation/pstn" </li></ul>    |


  ***Note:*** When using the Rtc555Sdk class, it will ask user to allow to use microphone while running. Please make sure access is allowed. 
  The run time permission dialog is shown after Android version Marshmallow onwards.

  ## Accept/Reject an Incoming Call

  When user receives an incoming PSTN call alert, the user can choose either to:
  -   [Accept Call](#accept-call)
  -   [Reject Call](#reject-call)
  
### Accept Call
  If the user wants to accept the incoming PSTN call request, use *accept* API and pass the notification payload received from incoming push notification ,instance of RTC555Result observer. Also implement the *onSuccess()* and *onError()* methods of Rtc555Result class.
  *accept* API returns a single value response. Success case of response will return callid and failure case return Rtc555Exception object.
  
  ```java
    Rtc555Voice.accept(HashMap notificationdata, new Rtc555Result() {
            @Override
            public void onSuccess(String callId) {
                            
            }

            @Override
            public void onError(Rtc555Exception e) {

            }
         });
```
Pass notification payload received at incoming notification.

 |Parameters| |
 |------|-----|
 |trace_id|trace id|
 |room_id|Room name that needs to be joined which is recieved in notification payload.|
 |room_token|Room token which is recieved in notification payload.|
 |room_token_expiry_time|Room token expiry time retrieved from notification payload.|
 |to_routing_id|toRoutingId |
 |rtc_server|RTC server URL extracted from notification payload.|
 
### Reject Call
  Users can reject an incoming call using *reject* API. Pass notification payload received in incoming push notification and instance of RTC555Result observer to *reject* API. Also implement the *onSuccess()* and *onError()* methods of Rtc555Result class.
  
```java
    Rtc555Voice.reject(HashMap notificationdata, new Rtc555Result() {
            @Override
            public void onSuccess(String callId) {
                            
            }

            @Override
            public void onError(Rtc555Exception e) {

            }
         });
```

|Parameters| |
 |------|-----|
 |trace_id|trace id|
 |room_id|Room name that needs to be joined which is recieved in notification payload.|
 |room_token|Room token which is received in notification payload.|
 |room_token_expiry_time|Room token expiry time retrieved from notification payload.|
 |to_routing_id|toRoutingId |
 |rtc_server|RTC server URL extracted from notification payload.|

## End an Active Call

Users need to invoke *hangup* API to end the active call.

```java
  Rtc555Voice.hangup(String callId);
```  
|Parameters| |
|-------|---|
|callId| callId is unique id for this call which was returned from dial/accept API|

## Clean up the session.

Invoke below API in Rtc555Sdk to release all the resources used by SDK. This call also allows the client to disconnect with the 555 platform backend. The app developer should call this API if the app goes to the background or kill state. 

```java
  Rtc555Sdk.cleanup();
``` 
   
## On-call Features
Features offered by Android SDK for PSTN call are:
- [Hold Call](#hold-call)
- [Unhold Call](#unhold-call)
- [Mute Local Audio](#mute-local-audio)
- [Unmute Local Audio](#unmute-local-audio)

### Hold Call

This API call allows users to hold the ongoing call. Both caller and callee won't able to hear to each other after invoking this call.

```java
    Rtc555Voice.hold(String callId);
```
|Parameters| |
|-------|---|
|callId| callId is unique id for this call which was returned from dial/accept API|

### Unhold Call

This API call allows users to un-hold the already held call. Both caller and callee will hear each other after invoking this call.

```java
    Rtc555Voice.unhold(String callId);
```
|Parameters| |
|-------|---|
|callId| callId is unique id for this call which was returned from dial/accept API|

### Mute Local Audio

This API call allows users to mute it's audio in the call.

```java
    Rtc555Voice.mute(String callId);
```
|Parameters| |
|-------|---|
|callId| callId is unique id for this call which was returned from dial/accept API|

### Unmute Local Audio

This API call allow user to un-mute it's audio in the call.

```java
    Rtc555Voice.unmute(String callId);
```
|Parameters| |
|-------|---|
|callId| callId is unique id for this call which was returned from dial/accept API|
     
## Audio Call Callbacks

To get call status or error report during the call, implement Rtc555SdkObserver interface and initialize the observer as below

```java
Rtc555Voice.setObserver(this);
```

### onStatus

This callback gets invoked when we receive the status of an ongoing PSTN call.

```java
    void onStatus(Rtc555Sdk.CallStatus callStatus, String callId);
```

|Parameters| |
|-------|----|
|status| status of ongoing call|
|callId| callId received from backend|

Status:
- initializing&emsp;&emsp;&emsp;- &emsp; When the call is initializing
- connecting&nbsp;&emsp;&emsp;- &emsp; When the call is connecting
- connected&nbsp;&emsp;&emsp; - &emsp; When the call is connected
- disconnected&emsp;&nbsp;- &emsp; When the call is disconnected
- hold&emsp;&emsp;&emsp;&emsp;&ensp;&ensp; - &emsp; When the call is hold

### onError

This callback gets invoked when we receives Rtc555Exception from an ongoing PSTN call.

```java
    void onError(Rtc555Exception rtc555Exception, String callId);
```

|Parameters| |
|-------|----|
|rtc555Exception| error object consists of error code and error message|
|callId| callId received from backend|

### onNotification

In case you want to use XMPP notification (Instead of GCM), use this callback. The client needs to be connected to 555 backend to avail of this notification callback.

```java
    void onNotification(HashMap notifyData);
```

 |Parameters| |
 |-------|----|
 |notifyData| notification payload for incoming notification|

 

