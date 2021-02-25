# Calls

## Get Callable Devices

For a specified mobile number, returns an array of patient user devices that are able to receive VoIP calls.

### URI

`GET /v1/callable-devices?mobileNumber={mobileNumber}`

### Request Parameters

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`mobileNumber` | Yes | String | A mobile number in E.164 format. Specified as a path parameter (see URI above).

### Response

> Example JSON for 200 OK response:

```json
[
  { "id": "cbc16ed8-0422-456f-9e15-f269b8a25811" },
  { "id": "7d7e2fdb-416d-4dc8-a545-e42ee157ef03" }
]
```

A successful response will return an array of devices (each containing just the `id` field). If the specified mobile number is not associated with any active devices that are able to receive VoIP calls, the response will contain an empty array.


## Call a User Device

Initiates a VoIP call to a patient user device.

### URI

`POST /v1/call`

### Request Parameters

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`deviceId` | Yes | String | The UUID of the user device to call. This should be a device ID obtained from a request to the [callable-devices](#get-callable-devices) endpoint.
`callId` | Yes | String | The call ID, as obtained from a SIP INVITE.
`callerName` | Yes | String | The display name of the caller, as obtained from a SIP INVITE.

### Response

> Example JSON for 403 Forbidden response:

```json
{
  "error": "device rejected inbound call"
}
```

Code | Description
---- | -----------
202 Accepted | A callable user device was found and a push notification dispatched to initiate a VoIP call. The response body will be empty.
403 Forbidden | A valid user device was found, but the user has configured it to refuse inbound VoIP calls. The response body will contain an `error` field with a message describing why the call was forbidden.
404 Not Found | A valid user device could not be found, and so cannot be called using VoIP. Note that even if a device exists, it is only considered valid if it has an associated VoIP push token, and if the device user is considered to be an active app user (according to this API). The response body will be empty.

### Discussion

Signalling for calls uses SIP. Unlike deskphones, it is not practical for patient mobile devices to remain perpetually SIP registered, since apps are backgrounded or suspended entirely to preserve device battery life. To overcome this limitation, the device must be SIP registered on demand using a push notification.

**Call Sequence**

> Example SIP REGISTER request made from client app in response to the push notification:

```http
REGISTER sip:+447777123456@sip.x-on.app SIP/2.0
Via: SIP/2.0/UDP 10.11.228.67:5060;branch=z9hG4bKnashds7
Max-Forwards: 70
To: John Smith <sip:+447777123456@sip.x-on.app>
From: John Smith <sip:+447777123456@sip.x-on.app>;tag=456248
Call-ID: 843817637684230@998sdasdh09
CSeq: 1826 REGISTER
Contact: <sip:+447777123456@sip.x-on.app;
  pn-provider=xon;
  pn-prid=c1a21fce-8660-4c69-a98d-820a50154f93>
Expires: 7200
Content-Length: 0
```

> Note the `pn-prid=c1a21fce-8660-4c69-a98d-820a50154f93` field in the `Contact` header - this Push Resource ID (PRID) value is the same as `userDeviceId`, and is used by the SIP proxy to associate SIP registration records with a push-capable device. The `pn-provider=xon` field is also specified to tell the SIP proxy to use the X-on push notification service.

This endpoint causes the following sequence of events:

  1. Details of the specified user device are fetched and checked to ensure it is configured to accept inbound calls.
  2. A push notification is dispatched to the device.
  3. If the push notification is delivered successfully (i.e. the device is switched on with access to the internet, and the app is still installed), the app immediately shows that an inbound call is trying to connect, and registers with the SIP proxy.
  4. The SIP proxy should observe the successful SIP registration of the device, and send a SIP INVITE to the registered SIP URI.
  5. The device receives the SIP INVITE and updates its calling UI with the caller ID. At this point, the patient user can accept the call to begin talking.

When attempting to call a patient, a VoIP call should be considered to have failed and a PSTN call made instead if any of the following circumstances occur:

  - None of the calls made to this endpoint return 202 Accepted. This indicates this patient user has no devices configured to accept an inbound VoIP call.
  - None of the calls made to this endpoint result in a SIP registration before a reasonable timeout. In this case, the device could not register in reasonable time, and a PSTN call should be made to maintain a reasonable user experience for the caller.

**RFC 8599**

For the purposes of RFC 8599 (Push Notifications with SIP), the `userDeviceId` should be considered to be the Push Resource ID (PRID), and will be included as the value for the `pn-prid` field inside the `Contact` header.

Additionally, the `pn-provider` field with the fixed value `xon` should be included in the `Contact` header.

<aside class="notice">
RFC 8599 also specifies a SIP URI parameter for <code>pn-param</code>, which is designed to specify the platform account through which the push notification should be sent (e.g. the app bundle ID and push notification service ID).<br>
<br>
In our implementation, that detail is the concern of this patient API, not the SIP proxy / API consumer. As such, we will not specify this additional parameter, unless mandated through the use of a pre-configured software module conforming to the RFC 8599 spec.
</aside>


## Update a Call With a Device

Updates a VoIP call with a patient user device.

### URI

`POST /v1/update-call`

### Request Parameters

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`deviceId` | Yes | String | The UUID of the user device to call. This should be a device ID obtained from a request to the [callable-devices](#get-callable-devices) endpoint.
`callId` | Yes | String | The call ID, as obtained from a SIP INVITE.
`callerName` | Yes | String | The display name of the caller, as obtained from a SIP INVITE.

### Response

Code | Description
---- | -----------
202 Accepted | A callable user device was found and a push notification dispatched to update a VoIP call. The response body will be empty.
404 Not Found | A valid user device could not be found, and so cannot be called using VoIP. Note that even if a device exists, it is only considered valid if it has an associated push token, and if the device user is considered to be an active app user (according to this API). The response body will be empty.

### Discussion

This endpoint provided to support mid-dialog requests, such as hold, transfer, keep-switch to video, or regular keep-alive messages. It causes a push notification to be sent to the specified device, which responds by registering with the SIP proxy. The proxy listens for this registration, and then forwards the SIP request on to the device.


## Cancel a Call With a Device

Cancels a VoIP call to a patient user device.

### URI

`POST /v1/cancel-call`

### Request Parameters

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`deviceId` | Yes | String | The UUID of the user device to call. This should be a device ID obtained from a request to the [callable-devices](#get-callable-devices) endpoint.
`callId` | Yes | String | The call ID, as obtained from a SIP INVITE.
`reason` | Yes | String | A code describing the reason the call to this device is cancelled, either `cancelled` or `answered_elsewhere`.

### Response

Code | Description
---- | -----------
202 Accepted | A callable user device was found and a push notification dispatched to cancel a VoIP call. The response body will be empty.
404 Not Found | A valid user device could not be found, and so we cannot cancel a VoIP call. Note that even if a device exists, it is only considered valid if it has an associated push token, and if the device user is considered to be an active app user (according to this API). The response body will be empty.

### Discussion

This endpoint should be used to cancel a call initiated with the [call](#call-a-user-device) endpoint in the event that the caller hung up before the call was connected, or when another user device accepted the call first.

This push notification mechanism is required since a device may have begun ringing in response to a push notification from the [call](#call-a-user-device) endpoint, but not yet registered with the SIP proxy and so would not receive the SIP CANCEL request.