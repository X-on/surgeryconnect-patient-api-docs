# File Requests


## Create a File Request

Creates a new file request and notifies the patient or a proxy (e.g. carer or paramedic).

A *patient* is the person that is associated with the resulting file request. A *recipient* is the person that is notified about file request (the recipient of the `message` the file request is wrapped in). In most cases the patient and recipient are the same person, but they might be different if a carer, paramedic, guardian, or other proxy needs to respond on behalf of the patient.

### URI

`POST /v1/file-requests`

### Request Body

> Example request JSON to send file request to patient via mobile app if the patient is an app user, otherwise SMS:

```json
{
  "prompt": "Please take a photo of your rash.",
  "type": "photo",
  "accountUserId": "42",
  "accountId": "12",
  "staffName": "Dr Rachel Williams",
  "patientDateOfBirth": "1980-06-17",
  "patientFirstName": "John",
  "patientLastName": "Smith",
  "patientMobile": "+447777123456",
  "recipientMobile": "+447777123456"
}
```
> In the above example, the recipient user is identified using `recipientMobile`. They will be notified in the app if they're an app user, otherwise by SMS. In this example it happens that the patient and recipient are the same person (clear because `patientMobile` and `recipientMobile` are identical).

> Example request JSON to send file request via email:

```json
{
  "prompt": "Please take a photo of your rash.",
  "type": "photo",
  "accountUserId": "42",
  "accountId": "12",
  "staffName": "Dr Rachel Williams",
  "patientDateOfBirth": "1980-06-17",
  "patientFirstName": "John",
  "patientLastName": "Smith",
  "patientMobile": "+447777123456",
  "recipientEmail": "johnsmith@example.com",
  "attemptAppDelivery": true
}
```
> In the above example, a non-app recipient user is identified using `recipientEmail` (even if they're an app user), since `recipientMobile` is not provided. The recipient—assumed to be the patient themselves since `recipientIsProxy` defaults to `false`—will only be notified of this request by email.

> Example request JSON to send file request via email, where patient does not have a mobile number at all:

```json
{
  "prompt": "Please take a photo of your rash.",
  "type": "photo",
  "accountUserId": "42",
  "accountId": "12",
  "staffName": "Dr Rachel Williams",
  "patientDateOfBirth": "1980-06-17",
  "patientFirstName": "John",
  "patientLastName": "Smith",
  "patientExternalId": "EMISIM1-5005730",
  "recipientEmail": "johnsmith@example.com"
}
```
> In the above example, the patient user is identified/created using `patientExternalIdentifier` since `patientMobile` is not provided. As with the previous example, a non-app recipient will be identified and notified of the file request by email.

> Example request JSON to send file request via SMS (NOT to mobile app):

```json
{
  "prompt": "Please take a photo of your rash.",
  "type": "photo",
  "accountUserId": "42",
  "accountId": "12",
  "staffName": "Dr Rachel Williams",
  "patientDateOfBirth": "1980-06-17",
  "patientFirstName": "John",
  "patientLastName": "Smith",
  "patientMobile": "+447777123456",
  "recipientMobile": "+447755012345",
  "attemptAppDelivery": false
}
```
> In the above example, a non-app recipient user is identified/created using `recipientMobile` even if an app-using patient user exists with that mobile, since `attemptAppDelivery` is `false`. The recipient will be notified of the file request by SMS.

> Example request JSON to send file request to a proxy (e.g. carer or paramedic) via the mobile app or SMS:

```json
{
  "prompt": "Please take a photo of your rash.",
  "type": "photo",
  "accountUserId": "42",
  "accountId": "12",
  "staffName": "Dr Rachel Williams",
  "patientDateOfBirth": "1980-06-17",
  "patientFirstName": "John",
  "patientLastName": "Smith",
  "patientMobile": "+447777123456",
  "recipientMobile": "+47123456789",
  "recipientIsProxy": true
}
```
> In the above example, a recipient user is identified using `recipientMobile`. They will be notified in the app if they're an app user, otherwise by SMS. Since `recipientIsProxy` is `true`, the message will indicate the file request is relating to a patient in their care.

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`prompt` | No | String | A textual prompt to show to the patient, directing them to upload a particular file.
`type` | Yes | String | Determines what file type/size restrictions should be used when validating user-selected files. The only option at the moment is "photo".
`staffId` | No | String | The UUID of the staff user in the patient database (where known).<br><br>If `staffId` is not provided, then `accountUserId` will be required.
`accountUserId` | No | String | The ID of the X-on account user sending the message.<br><br>If `accountUserId` is not provided, then `staffId` will be required.
`accountId` | Yes | String | The ID of the X-on account sending the message. Used to associate a file request with a particular account, so that any SMS notifications are sent using that account's SMS gateway.
`staffName` | Yes | String | The name of the staff member requesting a file, shown to the patient.
`patientDateOfBirth` | Yes | String | Patient date of birth in ISO 8601 date format, such as `YYYY-MM-DD`. Used to verify a patient's identity when the patient or proxy responds to a file request.
`patientFirstName` | No | String | The patient's first name.
`patientLastName` | No | String | The patient's last name.
`patientMobile` | No | String | Mobile number of the patient in E.164 format. Used to identify the patient user this file request should be linked to.<br><br>`patientMobile` is preferred for patient user identification, so it should be passed whenever available. If `patientMobile` is not provided, then `patientExternalId` will be required.
`patientExternalId` | No | String | An identifier that can be used to consistently identify this patient over time. Used to identify the patient user this file request should be linked to in the case that a patient mobile number is not available.<br><br>If `patientExternalId` is not provided, then `patientMobile` will be required.
`recipientMobile` | No | String | Mobile number of the recipient in E.164 format. Used to identify the user who should be notified of this file request. In most cases this will be the patient user, but it could also be a proxy (e.g. carer or paramedic).<br><br>When `recipientMobile` is provided and `attemptAppDelivery` is `true`, the recipient will be notified in the mobile app if they're an app user, else via SMS.<br><br>When `recipientMobile` is provided and `attemptAppDelivery` is `false`, the recipient will be notified with an SMS, whether or not they're an app user.<br><br>If `recipientMobile` is not provided, then `recipientEmail` will be required.
`recipientEmail` | No | String | Email address of the recipient. Used to identify the user who should be notified of this file request. In most cases this will be the patient user, but it could also be a proxy (e.g. carer or paramedic).<br><br>When `recipientEmail` is provided, the recipient will be notified of the file request by email.<br><br>If `recipientEmail` is not provided, then `recipientMobile` will be required.<br><br>N.B. This parameter does **not** affect in-app/SMS delivery. If `recipientMobile` matches an app-using patient and `attemptAppDelivery` is `true`, the file request will be delivered to the mobile app, regardless of this parameter. Similarly if both `recipientMobile` and `recipientEmail` are provided, the patient will be notified on their mobile (in-app/SMS) **and** email.
`recipientIsProxy` | No | Boolean | Indicates whether this request is being sent to a proxy (e.g. carer or paramedic) rather than the patient themselves.<br><br>When `true`, the message about the file request may contain additional copy indicating that the file request pertains to a patient in their care.<br><br>Defaults to `false`.
`attemptAppDelivery` | No | Boolean | Indicates whether or not this file request should be sent to the mobile app (if the recipient is using the app).<br><br>When `false`, the message containing the file request will **not** be sent to an app-using recipient user, but to a non-app user. This ensures the file request is not received in app, but by SMS and/or email (according to the inclusion of `recipientMobile` and/or `recipientEmail`).<br><br>Defaults to `true`. You might set this to `false` if you know the recipient is a proxy, and you want to notify them by SMS/email so they respond via a web page, rather than inside the app.

### Response Body

> Example response JSON:

```json
{
  "id": "f4d47426-c99a-43df-81ea-8304a62c5127",
  "accountId": "12",
  "createdAt": "2021-01-15T14:32:17.452104+00:00",
  "files": [],
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
  "type": "photo"
}
```

Contains a [FileRequest](#filerequest) JSON Object.

### Discussion

The patient user is identified by matching their (verified) mobile number, date of birth, and name to the values in this request. If a patient app user already exists where the verified mobile number, DOB, and name don't conflict, the file request is associated with them. If no patient already exists with that verified mobile number, a new non-app user is created, and the file request is associated with that user.

A mobile number for a patient might not exist (e.g. none recorded against their patient record in the clinical system). In this case it is assumed the patient is not using the mobile app, and a non-app user is retrieved or created. To prevent creating a new non-app user every time, a `patientExternalId` must be provided that is unique to that patient, such as one vended by the clinical system containing the patient record prefixed with an identifier for that clinical system, e.g. "EMISIM1-5005730".

The recipient user is identified by matching their (verified) mobile number to the `recipientMobile` request parameter, if provided. If no user already exists with that verified mobile number, a non-app user with that mobile number is retrieved or created. If `recipientMobile` is not provided, a non-app user with the value in `recipientEmail` is retrieved or created.

The resulting recipient user is associated with a newly created message wrapping the file request, and they are notified by mobile (in-app/SMS) and/or email according to the values specified in `recipientMobile`, `recipientEmail`, and `attemptAppDelivery`.

The staff user is identified using their user ID in the patient database (if known), else using their X-on account user ID. If no staff user is found with the provided ID, a new non-app user is created.

## Get a Specific File Request

> Example response JSON:

```json
{
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


## File Request JSON Objects

### FileRequest

Field | Type | Description
----- | ---- | -----------
`id` | String | The UUID of the created file request.
`accountId` | String | The ID of the X-on account sending the message.
`createdAt` | String | The file request creation date in ISO 8601 format.
`files` | Array | The list of files sent in response to this file request. Initially empty.
`files[x].id` | String | The UUID of the file.
`files[x].createdAt` | String | The timestamp when this file was created (uploaded) in ISO 8601 format.
`files[x].description` | String | An optional description of the file, provided by the file uploader.
`files[x].expiresAt` | String | The timestamp when access to this file will expire in ISO 8601 format. After this time, the file will remain in file storage, but will be inaccessible to all APIs.
`files[x].imageHeight` | Integer | The height of the image file. This is `null` if the file is not an image, or if the dimensions could not otherwise be obtained.
`files[x].imageWidth` | Integer | The width of the image file. This is `null` if the file is not an image, or if the dimensions could not otherwise be obtained.
`files[x].mimeType` | String | The MIME type of the file.
`files[x].originalName` | String | The original name of the file when uploaded.
`files[x].size` | Integer | The size of the file in bytes.
`patientUser` | String | The patient user the file request was sent to.
`patientUser.id` | String | The UUID of the patient user.
`patientUser.displayName` | String | The display name of the patient. This should be the preferred name to use for the patient, however it can be `null`, in which case `firstName` and `lastName` should be used.
`patientUser.firstName` | String | The patient's first name. If available, `displayName` should be used instead. This value can be `null`.
`patientUser.lastName` | String | The patient's last name. If available, `displayName` should be used instead. This value can be `null`.
`patientUser.profilePictureUrl` | String | The URL to a profile picture for this patient. Can be `null`.
`staffUser` | String | The staff user the file request was sent from.
`staffUser.id` | String | The UUID of the staff user the file request was sent from.
`staffUser.displayName` | String | The display name of the staff member. This should be the preferred name to use for the staff member, however it can be `null`, in which case `firstName` and `lastName` should be used.
`staffUser.firstName` | String | The staff member's first name. If available, `displayName` should be used instead. This value can be `null`.
`staffUser.lastName` | String | The staff member's last name. If available, `displayName` should be used instead. This value can be `null`.
`staffUser.profilePictureUrl` | String | The URL to a profile picture for this staff member. Can be `null`.
`prompt` | String | A textual prompt to show to the patient, directing them to upload a particular file.
`expiresAt` | String | The timestamp when this file request expires. After this time, files can no longer be added to it.
`shortLinkExpiresAt` | String | The timestamp when access to this file request via the short link URL will expire. After this time, the patient will be unable to respond to this file request using a web browser, though an authenticated patient app user might still be able to respond as long as `expiresAt` is `null` or in the future.
`shortLinkId` | String | An ID that can be used to form a public URL to the file request. This allows an unauthenticated user to access a webpage through which they can view/respond to the file request.
`type` | String | Determines what file type/size restrictions should be used when validating user-selected files.