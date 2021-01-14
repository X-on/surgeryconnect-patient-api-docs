# File Requests


## Create a File Request

> Example request JSON:

```json
{
  "prompt": "Please take a close-up photo of the rash on the back of your right hand.",
  "type": "photo",
  "accountUserId": 42,
  "accountId": 12,
  "staffName": "Dr Rachel Williams",
  "patientMobile": "+447777123456",
  "patientDateOfBirth": "1980-06-17",
  "patientFirstName": "John",
  "patientLastName": "Smith"
}
```

> Example response JSON:

```json
{
  "id": "f4d47426-c99a-43df-81ea-8304a62c5127",
  "account_id": "12",
  "created_at": "2021-01-15T14:32:17.452104+00:00",
  "files": [],
  "patient_id": "9d6ad525-90c6-4c03-8f05-63584ee20a29",
  "prompt": "Please take a close-up photo of the rash on the back of your right hand.",
  "short_link_expires_at": null,
  "short_link_id": "a49ekq3j",
  "staff_id": "c46c3407-9c79-498f-a713-07b7d8f1fabf",
  "staff_name": "Dr Rachel Williams",
  "type": "photo"
}
```

Creates a new file request and notifies the patient.

The patient is identified using their mobile number. If a patient app user already exists with that mobile number the file request is associated with them and they receive a notification to the app. If no patient already exists with that mobile number, a new 'shell' user is created, and the patient receives an SMS with a link to respond to the file request in a web browser.

The staff user is identified using their user ID in the patient database (if known), else using their X-on account user ID. If no staff user is found with the provided ID, a new 'shell' user is created.

### URI

`POST /v1/file-requests`

### Request Body

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`prompt` | Yes | String | A textual prompt to show to the patient, directing them to upload a particular file.
`type` | Yes | String | Determines what file type/size restrictions should be used when validating user-selected files. The only option at the moment is "photo".
`staffId` | No | String | The UUID of the staff user in the patient database (where known).
`accountUserId` | Yes, if `staffId` is not specified; otherwise no. | String | The ID of the X-on account user sending the message.
`accountId` | Yes | String | The ID of the X-on account sending the message. Used to associate a file request with a particular account, so that any SMS notifications are sent using that account's SMS gateway.
`staffName` | Yes | String | The name of the staff member requesting a file, shown to the patient.
`patientMobile` | Yes | String | Mobile number of the patient in E.164 format.
`patientDateOfBirth` | Yes | String | Patient date of birth in ISO 8601 date format. Used to allow patient to verify their identity to respond to a file request.
`patientFirstName` | No | String | The patient's first name.
`patientLastName` | No | String | The patient's last name.

### Response Body

Contains a [FileRequest](#filerequest) JSON Object.


## Get a Specific File Request

> Example response JSON:

```json
{
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
}
```

Fetches an existing file request.

If the patient has responded to the file request, the response will include one or more files.

### URI

`GET /v1/file-requests/{fileRequestId}`

### Request Parameters

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`fileRequestId` | Yes | String | The UUID of the file request to fetch.

### Response Body

Contains a [FileRequest](#filerequest) JSON Object.


## JSON Objects

### FileRequest

Field | Type | Description
----- | ---- | -----------
`id` | String | The UUID of the created file request.
`account_id` | String | The ID of the X-on account sending the message.
`created_at` | String | The file request creation date in ISO 8601 format.
`files` | Array | The list of files sent in response to this file request. Initially empty.
`files[x].id` | String | The UUID of the file.
`files[x].created_at` | String | The timestamp when this file was created (uploaded) in ISO 8601 format.
`files[x].expires_at` | String | The timestamp when access to this file will expire in ISO 8601 format. After this time, the file will remain in file storage, but will be inaccessible to all APIs.
`files[x].mime_type` | String | The MIME type of the file.
`files[x].original_name` | String | The original name of the file when uploaded.
`files[x].size` | Integer | The size of the file in bytes.
`patient_id` | String | The UUID of the patient user the file request was sent to.
`prompt` | String | A textual prompt to show to the patient, directing them to upload a particular file.
`short_link_expires_at` | String | The timestamp when access to this file request via the short link URL will expire. After this time, the patient will be unable to respond to this file request.
`short_link_id` | String | An ID that can be used to form a public URL to the file request. This allows an unauthenticated user to access a webpage through which they can view/respond to the file request.
`staff_id` | String | The UUID of the staff user the file request was sent from.
`staff_name` | String | The name of the staff member requesting a file, shown to the patient.
`type` | String | Determines what file type/size restrictions should be used when validating user-selected files.