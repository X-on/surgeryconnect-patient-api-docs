# Events & Webhooks

Some of the services that make up this API need to notify interested consumers of events so they can respond accordingly (e.g. notifying a staff member when a patient responds to their file request). Events and webhooks provide a publish-subscribe pattern to accommodate this requirement.

Events are sent via an HTTP `POST` request with a JSON request body. The request will be sent with the a `Content-Type` header of `application/json`. A webhook should process an event and respond with a status code between `200` and `299` (inclusive).

<aside class="notice">
While we might offer the ability to dynamically configure webhooks in the future, for now all events are hardcoded to be sent to the following X-on Platform API webhook:<br>
<code>https://sc-api.x-onweb.com/api/v2/patient-notifications</code>
</aside>

## File Request Files Updated

```json
{
  "event": {
    "id": "2e0db25d-8b39-40fa-81fc-cab625fa130f",
    "createdAt": "2021-01-15T14:33:30.929534+00:00",
    "type": "file-request.files.update",
    "fileRequest": {
      "id": "f4d47426-c99a-43df-81ea-8304a62c5127",
      "account_id": "12",
      "created_at": "2021-01-15T14:32:17.452104+00:00",
      "files": [
        {
          "id": "7fecfb58-bfd6-462e-9e6e-f9ede76c7c95",
          "created_at": "2021-01-15T14:33:29.145322+00:00",
          "expires_at": "2021-01-15T15:33:29.145322+00:00",
          "mime_type": "image/jpeg",
          "original_name": "IMG_0292.JPG",
          "size": 2904221
        },
        {
          "id": "196dccdb-079a-4bbd-a082-bb03cc8efe47",
          "created_at": "2021-01-15T14:33:29.145322+00:00",
          "expires_at": "2021-01-15T15:33:29.145322+00:00",
          "file_request_id": "f4d47426-c99a-43df-81ea-8304a62c5127",
          "mime_type": "image/jpeg",
          "original_name": "IMG_0293.JPG",
          "size": 2738706
        }
      ],
      "patient_id": "9d6ad525-90c6-4c03-8f05-63584ee20a29",
      "prompt": "Please take a close-up photo of the rash on the back of your right hand.",
      "short_link_expires_at": null,
      "short_link_id": "a49ekq3j",
      "staff_id": "c46c3407-9c79-498f-a713-07b7d8f1fabf",
      "staff_name": "Dr Rachel Williams",
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

### Event Body

Parameter | Type | Description
--------- | ---- | -----------
`event.id` | String | The UUID of the event. This may be used to identify & protect against duplicate events.
`event.createdAt` | String | The timestamp when the event was created in ISO 8601 format.
`event.type` | String | The event type. This value will always be `file-request.files.update`.
`event.fileRequest` | Object | The file request whose files changed, as a [FileRequest](#filerequest) JSON Object.
`event.addedFileIds` | Array | An array of UUIDs (Strings) of the files that were added to the file request.
`event.removedFileIds` | Array | An array of UUIDs (Strings) of the files that were removed from the file request.
