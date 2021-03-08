# Files


## Upload Files

> Example request JSON:

```json
{
  "files": [
    {
      "name": "IMG_0292.JPG",
      "description": null,
      "type": "image/jpeg",
      "contents": "..."
    },
    {
      "name": "IMG_0293.JPG",
      "description": "This rash is looking better here than it was yesterday.",
      "type": "image/jpeg",
      "contents": "..."
    }
  ],
  "expiryInterval": 3600,
  "fileRequestId": "f4d47426-c99a-43df-81ea-8304a62c5127",
  "userId": "9d6ad525-90c6-4c03-8f05-63584ee20a29"
}
```

> Example response JSON:

```json
[
  { 
    "id": "7fecfb58-bfd6-462e-9e6e-f9ede76c7c95",
    "createdAt": "2021-01-15T14:33:29.145322+00:00",
    "description": null,
    "expiresAt": "2021-01-15T15:33:29.145322+00:00",
    "fileRequestId": "f4d47426-c99a-43df-81ea-8304a62c5127",
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
    "fileRequestId": "f4d47426-c99a-43df-81ea-8304a62c5127",
    "imageHeight": 4032,
    "imageWidth": 3024,
    "mimeType": "image/jpeg",
    "originalName": "IMG_0293.JPG",
    "size": 2738706
  }
]
```

Uploads one or more files, optionally associating them with a specific file request.

If an associated file request is specified, a [`file-request.files.update`](#file-request-files-updated) event is published to registered webhooks.

### URI

`POST /v1/files`

### Request Body

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`files` | Yes | Array | An array of JSON objects representing files to be uploaded.
`files[x].name` | Yes | String | The original name of the file.
`files[x].description` | No | String | An optional description of the file.
`files[x].type` | Yes | String | The MIME type of the file, e.g. "image/jpeg".
`files[x].contents` | Yes | String | The Base64 encoded contents of the file.
`expiryInterval` | No | Integer | The time interval (in seconds) after which the file will expire and subsequently be removed from storage. If you don't specify a value, the file will not expire automatically.
`userId` | Yes | String | The UUID of the user uploading the file.
`fileRequestId` | No | String | The UUID of a file request to associate the files with (i.e. as a response to the file request). If `null`, the files will be uploaded without assocating them with an existing file request.

### Response Body

Contains a [File](#file) JSON Object.


## Download a Specific File

Downloads an existing file as a stream of data. Optionally specify image dimensions to download a thumbnail of the file (only works with image files, obviously!).

For patient privacy, files stored in the cloud are not accessible using a publicly accessible URL. Access to files is only possible using this endpoint.

### URI

`GET /v1/files/{fileId}?imageSize=200x140`

### Request Parameters

Parameter | Required | Type | Description
--------- | -------- | ---- | -----------
`fileId` | Yes | String | The UUID of the file to download. Specified as a path parameter (see URI above).
`imageSize` | No | String | The desired dimensions of the downloaded image, e.g. `200x140`. Use to obtain a thumbnail of the image without having to download the full image. Omit this parameter to download the image at its original size.<br><br>The server does not perform cropping, so the returned image dimensions are not guaranteed to be exactly those specified in this parameter. The image is resized using aspect fill scaling, so that neither the width nor height of the returned image is smaller than the requested dimensions (unless the resultant size is *larger* than the original image size, in which case the original image size is returned).<br><br>If the specified file is not an image, this parameter is ignored.

### Response 

A file stream for downloading the specified file data.

## File JSON Objects

### File

Field | Type | Description
----- | ---- | -----------
`id` | String | The UUID of the file.
`createdAt` | String | The timestamp when this file was created (uploaded) in ISO 8601 format.
`description` | String | An optional description of the file, provided by the file uploader.
`expiresAt` | String | The timestamp when access to this file will expire in ISO 8601 format. After this time, the file will remain in file storage, but will be inaccessible to all APIs.
`fileRequestId` | String | The UUID of the file request this file is associated with, or `null` if not associated with any file request.
`imageHeight` | Integer | The height of the image file. This is `null` if the file is not an image, or if the dimensions could not otherwise be obtained.
`imageWidth` | Integer | The width of the image file. This is `null` if the file is not an image, or if the dimensions could not otherwise be obtained.
`mimeType` | String | The MIME type of the file.
`originalName` | String | The original name of the file when uploaded.
`size` | Integer | The size of the file in bytes.