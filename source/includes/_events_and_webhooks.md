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
          "description": null,
          "expiresAt": "2021-01-15T15:33:29.145322+00:00",
          "imageHeight": 4032,
          "imageWidth": 3024,
          "mimeType": "image/jpeg",
          "originalName": "IMG_0292.JPG",
          "photoCreatedAt": "2021-01-12T10:30:12.153234+00:00",
          "size": 2904221
        },
        {
          "id": "196dccdb-079a-4bbd-a082-bb03cc8efe47",
          "createdAt": "2021-01-15T14:33:29.145322+00:00",
          "description": "This rash is looking better here than it was yesterday.",
          "expiresAt": "2021-01-15T15:33:29.145322+00:00",
          "imageHeight": 4032,
          "imageWidth": 3024,
          "mimeType": "image/jpeg",
          "originalName": "IMG_0293.JPG",
          "photoCreatedAt": "2021-01-12T10:30:13.342424+00:00",
          "size": 2738706
        }
      ],
      "patientUser": {
        "id": "9d6ad525-90c6-4c03-8f05-63584ee20a29",
        "displayName": null,
        "firstName": "John",
        "lastName": "Smith",
        "profilePictureUrl": "https://surgeryconnect-patients-public.s3.eu-west-2.amazonaws.com/8285e660-f6d9-4e0f-ac03-d62f3f1cc2e0"
      },
      "staffUser": {
        "id": "c46c3407-9c79-498f-a713-07b7d8f1fabf",
        "displayName": "Dr Rachel Williams",
        "firstName": null,
        "lastName": null,
        "profilePictureUrl": null
      },
      "prompt": "Please take a photo of your rash.",
      "expiresAt": null,
      "shortLinkExpiresAt": null,
      "shortLinkId": "a49ekq3j",
      "type": "photo",
      "filesUpdateWebhookUrl": null
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
If the field `filesUpdateWebhookUrl` was specified when creating the file request, that URL is used as the webhook URL for this event. Otherwise, events of this type are hardcoded to be sent to the following API webhook:<br>
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


## Create Callable User

This event is generated whenever a user updates their mobile number, mobile verification status, or devices such that the user is now callable by VoIP on a particular mobile number.

<aside class="notice">
While we might offer the ability to dynamically configure webhooks in the future, for now events of this type are hardcoded to be sent to the following API webhook:<br>
<code>https://sc-api.x-onweb.com/api/v2/patient-notifications</code>
</aside>

### Event Body

> Sample webhook event JSON for `calls.user.create` event type:

```json
{
  "event": {
    "id": "bcbecf9e-590d-4a18-ba00-f8b0caa3216b",
    "createdAt": "2021-01-14T11:16:47.17502+00:00",
    "type": "calls.user.create",
    "mobileNumber": "+447777123456"
  }
}
```

Parameter | Type | Description
--------- | ---- | -----------
`event.id` | String | The UUID of the event. This may be used to identify & protect against duplicate events.
`event.createdAt` | String | The timestamp when the event was created in ISO 8601 format.
`event.type` | String | The event type. This value will always be `calls.user.create`.
`event.mobileNumber` | String | The callable user's mobile number in E.164 format.

### Discussion

This webhook event is provided as a way of communicating with the X-on Platform that a particular mobile number is now associated with an active patient app user. This allows the platform to maintain a list of callable patient MSISDNs locally, used for fast route selection when it receives a call request to a mobile phone.

In order for the platform to identify SIP UAs using only a patient mobile number, a patient's SIP username will always be their mobile number in E.164 format. The SIP proxy should not require subscriber records to be created in advance, and authentication should not be necessary. Patient SIP UAs will be prompted to register just in time by use of push notifications (see [Calls](#calls)).


## Delete Callable User

This event is generated whenever a user updates their mobile number, mobile verification status, or devices such that the user is no longer callable by VoIP on a particular mobile number.

<aside class="notice">
While we might offer the ability to dynamically configure webhooks in the future, for now events of this type are hardcoded to be sent to the following API webhook:<br>
<code>https://sc-api.x-onweb.com/api/v2/patient-notifications</code>
</aside>

### Event Body

> Sample webhook event JSON for `calls.user.delete` event type:

```json
{
  "event": {
    "id": "6ac9365c-b937-4532-aeab-80b88ae2bc4b",
    "createdAt": "2021-01-29T15:34:12.13042+00:00",
    "type": "calls.user.delete",
    "mobileNumber": "+447777123456"
  }
}
```

Parameter | Type | Description
--------- | ---- | -----------
`event.id` | String | The UUID of the event. This may be used to identify & protect against duplicate events.
`event.createdAt` | String | The timestamp when the event was created in ISO 8601 format.
`event.type` | String | The event type. This value will always be `calls.user.delete`.
`event.mobileNumber` | String | The mobile number (in E.164 format) of the user that is no longer callable.

### Discussion

This webhook event is provided as a way of communicating with the X-on Platform that a particular mobile number is no longer associated with an active patient app user. This allows the platform to update its list of callable patient MSISDNs locally, allowing it to bypass an attempt to call the patient over VoIP and use the PSTN instead.
