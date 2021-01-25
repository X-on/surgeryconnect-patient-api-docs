# Notifications




## Call a User Device

Initiates a VoIP call to a patient user.

### URI

`POST /v1/notifications/call/{userDeviceId}`

### Request Parameters

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`userDeviceId` | Yes | String | The UUID of the user device to call. Specified as a path parameter (see URI above).

### Response

> Example JSON for 403 Forbidden response:

```json
{
  "error": "User has configured device to refuse inbound VoIP calls."
}
```

Code | Description
---- | -----------
202 Accepted | A user device was found and a push notification dispatched to initiate a VoIP call. The response body will be empty.
403 Forbidden | A user device was found, but the user has configured it to refuse inbound VoIP calls. The response body will contain an `error` field with a message describing why the call was forbidden.
404 Not Found | The specified user device doesn't exist, and so cannot be called using VoIP. The response body will be empty.

### Discussion

Signalling for calls uses SIP. Unlike deskphones, it is not practical for patient mobile devices to remain perpetually SIP registered, since apps are backgrounded or suspended entirely to preserve device battery life. To overcome this limitation, the device must be SIP registered on demand using a push notification.

**Call Sequence**

> Example SIP REGISTER request made from client app in response to the push notification:

```http
REGISTER sip:+447777123456@x-onsip.net SIP/2.0
Via: SIP/2.0/UDP 10.11.228.67:5060;branch=z9hG4bKnashds7
Max-Forwards: 70
To: John Smith <sip:+447777123456@x-onsip.net>
From: John Smith <sip:+447777123456@x-onsip.net>;tag=456248
Call-ID: 843817637684230@998sdasdh09
CSeq: 1826 REGISTER
Contact: <sip:+447777123456@x-onsip.net;
  pn-prid=c1a21fce-8660-4c69-a98d-820a50154f93>;
  +sip.pnsreg
Expires: 7200
Content-Length: 0
```

This endpoint causes the following sequence of events:

  1. Details of the specified user device are fetched and checked to ensure it is configured to accept inbound calls.
  2. A push notification is dispatched to the device.
  3. If the push notification is delivered successfully (i.e. the device is switched on with access to the internet, and the app is still installed), the app immediately shows that an inbound call is trying to connect, and registers with the SIP network.
  4. The SIP proxy should observe the successful SIP registration of the device, and send a SIP invite to the registered SIP URI.
  5. The device receives the SIP invite and updates its calling UI with the caller ID. At this point, the patient user can accept the call to begin talking.

When attempting to call a patient, a VoIP call should be considered to have failed and a PSTN call made instead if any of the following circumstances:

  - None of the calls made to this endpoint return 202 Accepted. This indicates this patient user has no devices configured to accept an inbound VoIP call.
  - None of the calls made to this endpoint result in a SIP registration within before a reasonable timeout. In this case, the device could not register in reasonable time, and a PSTN call should be made to maintain a reasonable user experience for the caller.

> Note the `pn-prid=c1a21fce-8660-4c69-a98d-820a50154f93` field in the `Contact` header - this Push Resource ID (PRID) value is the same as `userDeviceId`, and is used by the SIP proxy to associate SIP registration records with a push-capable device.

**RFC 8599**

For the purposes of RFC 8599 (Push Notifications with SIP), the `userDeviceId` should be considered to be the Push Resource ID (PRID), and will be included as the value for the `pn-prid` field inside the `Contact` header.

<aside class="notice">
RFC 8599 also specifies SIP URI parameters for `pn-provider` and `pn-param`, which are designed to specify the specific platform push notification provider to use, and the respective account through which the push notification should be sent.

In our implementation, these details are the concern of this patient API, not the SIP proxy / API consumer. As such, we will not specify these additional parameters, unless mandated through the use of a pre-configured software module conforming to the RFC 8599 spec.
</aside>