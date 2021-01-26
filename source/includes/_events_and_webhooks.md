# Events & Webhooks

Some of the services that make up this API need to notify interested consumers of events so they can respond accordingly (e.g. notifying a staff member when a patient responds to their file request). Events and webhooks provide a publish-subscribe pattern to accommodate this requirement.

Events are sent via an HTTP `POST` request with a JSON request body. The request will be sent with the a `Content-Type` header of `application/json`. A webhook should process an event and respond with a status code between `200` and `299` (inclusive).

## File Request Files Updated

> Sample webhook event JSON for `file-request.files.update` event type:

```json
{
  "event": {
    "id": "2e0db25d-8b39-40fa-81fc-cab625fa130f",
    "createdAt": "2021-01-15T14:33:30.929534+00:00",
    "type": "file-request.files.update",
    "fileRequest": {
      "id": "f4d47426-c99a-43df-81ea-8304a62c5127",
      "accountId": "12",
      "createdAt": "2021-01-15T14:32:17.452104+00:00",
      "files": [
        {
          "id": "7fecfb58-bfd6-462e-9e6e-f9ede76c7c95",
          "createdAt": "2021-01-15T14:33:29.145322+00:00",
          "expiresAt": "2021-01-15T15:33:29.145322+00:00",
          "mimeType": "image/jpeg",
          "originalName": "IMG_0292.JPG",
          "size": 2904221
        },
        {
          "id": "196dccdb-079a-4bbd-a082-bb03cc8efe47",
          "createdAt": "2021-01-15T14:33:29.145322+00:00",
          "expiresAt": "2021-01-15T15:33:29.145322+00:00",
          "mimeType": "image/jpeg",
          "originalName": "IMG_0293.JPG",
          "size": 2738706
        }
      ],
      "patientId": "9d6ad525-90c6-4c03-8f05-63584ee20a29",
      "prompt": "Please take a close-up photo of the rash on the back of your right hand.",
      "shortLinkExpiresAt": null,
      "shortLinkId": "a49ekq3j",
      "staffId": "c46c3407-9c79-498f-a713-07b7d8f1fabf",
      "staffName": "Dr Rachel Williams",
      "type": "photo"
    },
    "addedFileIds": [
      "7fecfb58-bfd6-462e-9e6e-f9ede76c7c95",
      "196dccdb-079a-4bbd-a082-bb03cc8efe47"
    ],
    "removedFileIds": []
  }
}
```

This event is generated whenever the files associated with a particular file request are added or removed, for example, when a patient responds to a file request by uploading one or more files.

The JSON includes the FileRequest whose files changed, and arrays of IDs for the files that were added and removed.

<aside class="notice">
While we might offer the ability to dynamically configure webhooks in the future, for now events of this type are hardcoded to be sent to the following API webhook:<br>
<code>https://sc-api.x-onweb.com/api/v2/patient-notifications</code>
</aside>

### Event Body

Parameter | Type | Description
--------- | ---- | -----------
`event.id` | String | The UUID of the event. This may be used to identify & protect against duplicate events.
`event.createdAt` | String | The timestamp when the event was created in ISO 8601 format.
`event.type` | String | The event type. This value will always be `file-request.files.update`.
`event.fileRequest` | Object | The file request whose files changed, as a [FileRequest](#filerequest) JSON Object.
`event.addedFileIds` | Array | An array of UUIDs (Strings) of the files that were added to the file request.
`event.removedFileIds` | Array | An array of UUIDs (Strings) of the files that were removed from the file request.


## User Caller Details Updated

> Sample webhook event JSON for `user.caller-details.update` event type:

```json
{
  "event": {
    "id": "bcbecf9e-590d-4a18-ba00-f8b0caa3216b",
    "createdAt": "2021-01-14T11:16:47.17502+00:00",
    "type": "user.caller-details.update",
    "user": {
      "id": "9d6ad525-90c6-4c03-8f05-63584ee20a29",
      "mobileNumber": "+447777123456",
      "isMobileVerified": true,
      "firstName": "John",
      "lastName": "Smith"
    },
    "oldMobileNumber": "+447111222333"
  }
}
```

This event is generated whenever a user updates their name, mobile number, or mobile verification status.

The JSON includes all the new user details, and the user's old mobile number.

<aside class="notice">
While we might offer the ability to dynamically configure webhooks in the future, for now events of this type are hardcoded to be sent to the following API webhook:<br>
<code>https://platform.x-onweb.com/.../patient-notifications</code> (TBC)
</aside>

### Event Body

Parameter | Type | Description
--------- | ---- | -----------
`event.id` | String | The UUID of the event. This may be used to identify & protect against duplicate events.
`event.createdAt` | String | The timestamp when the event was created in ISO 8601 format.
`event.type` | String | The event type. This value will always be `user.caller-details.update`.
`event.user` | Object | The user whose caller details have been updated. All fields of this object represent the *updated* user attributes.
`event.user.id` | String | The ID of the user.
`event.user.mobileNumber` | String | The user's mobile number in E.164 format.
`event.user.isMobileVerified` | Boolean | Whether or not the user's mobile number has been verified via SMS verification.
`event.user.firstName` | String | The user's first name.
`event.user.lastName` | String | The user's last name.
`event.oldMobileNumber` | String | The user's old mobile number (before the update), in E.164 format.

### Discussion - SIP Accounts

This webhook event is provided primarily as a way of communicating with the X-on Platform to allow SIP user accounts & subscriber records to be created for each patient user. Unlike existing business accounts where users and numbers are pre-configured, patient accounts require SIP user accounts to be created dynamically so that a user can register their device to make and receive calls.

In order for the platform to identify SIP UAs using only a patient mobile number, a patient's SIP username will always be their mobile number in E.164 format. By publishing this event in response the a user updating their mobile number, we can ensure a SIP user account always exists for patient users.

The webhook processing this event should perform the following actions:

  1. Create a SIP user account with `event.user.mobileNumber` as the username.
  2. If the mobile number hasn't changed (`event.user.mobileNumber == event.oldMobileNumber`), update the existing user account with the user's name (`event.user.{first|last}Name`).
  3. Delete any existing SIP user account with the username `event.oldMobileNumber`.

Once this has been done, the patient user device will be able to SIP register, associating a device ID with that registration, and allowing the X-on Platform to initiate VoIP calls to the device.

See [Call a User Device](#call-a-user-device) for details on makings calls to a patient device.
