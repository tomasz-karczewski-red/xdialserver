# DIAL 2.1 Server Implementation

The RDK DIAL server supports XCAST_2 API through thunder plugin interface

## XCAST_2 API

### setEnabled

```
void setEnabled(boolean);
```
Enable or disable XCAST service. Default value is __true__. When disabled, the customer should not be able to discover CPE as a cast target for any client application. After enable(true) server application manger must re-register all app that are available for user to cast.

The value is persisted on device after each call. Upon reboot or reconnect, the persisted value is used until the next call.

### getEnabled
```
object getEnabled(void)
```

Return the persisted value in json object
Object Parameter
```
{
  "enabled" :  "true"|"false"
}
```

### onApplicationLaunchRequest
```
void onApplicationLaunchRequest(Object)
```
Fired when service receives a launch request from client.  If not already running, the requested app is started. if already running and in background mode, the requested app enters foreground mode (optimus::running, xcast::running). If already in foreground mode, the request does not change app state.

This request must result in an `onApplicationStateChanged` event

Object parameter
```
{
  serviceName: [string] (optional) name of the service that the request originates from (DIAL, Alexa etc.)
  applicationName : <string> registered application name
  parameters: [object]  {
    "query"   : <string> url-encoded query string from dial client HTTP POST Request, if any.
    "payload" : <string> url-encoded payload string from dial client HTTP POST Request, if any.
  }
}
```

### onApplicationStopRequest
```
void onApplicationStopRequest(Object)
```
Fired when service receives a stop request from client.  If app is in running or hidden state,  the requested app is stopped.
This request result in an `onApplicationStateChanged` event when the application is considered stopped. Until then the client can query the state of the app and may get intermediate states.

Object parameter
```
{
  applicationName : <string> registered application name
  applicationId: [string] (optional) application instance Id.
}
```

### onApplicationStateRequest
```
void onApplicationStateRequest(object)
```
Fired when service needs an update of application state.

This request must result in an `onApplicationStateChanged` event

Object parameter
```
{
  applicationName: <string> name of the application whose state is being requested
  applicationId: [string] (optional) application instance Id.
}
```

### onApplicationStateChanged
```
void onApplicationStateRequest(object)

```
Notify whenever app changes state (due to user activity or internal error or other reasons). For singleton, applicationId is optional

If application launch/stop request is denied or fails to reach the target state or the state change is triggered by an internal error, a predefined error string should be included. This error may be translated from DIAL http error code to a XCAST client's error code.

Object parameter
```
{
  serviceName: [string] (optional) name of the service that the request originates from (DIAL, Alexa etc.)
  applicationName: <string> name of the application whose state has changed.
  applicationId: [string] (optional) application instance Id,
  state: <string> Predefined state strings. [running|stopped],
  error: [string] (optional) Predefined Error string from cast target app [none|forbidden|unavailable|invalid|internal]
}
```

Client Error Mapping Example:

|XCastService Error| Definition | DIAL Client Error|
|------------------|------------|------------------|
|none (or absent)  |request (start/stop) is fulfilled successfully|HTTP 200 OK|
|forbidden         |user is not allowed to change the state of the app (This is not related to user account authentication of the native app)|HTTP 403 Forbidden|
|unavailable       |	the target native app is not available on the device|HTTP 404 Not Found|
|invalid           |	the request is invalid (bad param for example)|	HTTP 400 Bad Request|
|internal          |	the server fail to fulfill the request (server error)|	HTTP 500 Internal|
