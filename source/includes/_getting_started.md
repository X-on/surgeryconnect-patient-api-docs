# Getting Started


## Authentication

```shell
curl "api-endpoint-here" \
  -H "Authorization: Bearer MyApiKey"
```

> Replace `MyApiKey` with your API key.

You will be issued with an API key that is used to authenticate access to this API.

Your API key must be provided in all requests to the API using the Authorization header.

`Authorization: Bearer MyApiKey`

<aside class="notice">
Replace <code>MyApiKey</code> with your API key.
</aside>


## Constructing Requests

Unless otherwise specified, all request URIs should be appended to the following base URL, which uses the production environment:

`https://api.x-on.app`

### Staging Environment

For staging APIs, use the following base URL:

`https://staging.api.x-on.app`

Staging APIs are considered complete and ready for testing. They should behave according to the API documentation, and should be used to test integrations before coordinating a deployment to the production environment.

### Development Environment

For development APIs, use with the following base URL:

`https://dev.api.x-on.app`

<aside class="warning">
Development APIs are undocumented and may behave in an incorrect or unexpected way. Only use these APIs in close consultation with the API author.
</aside>


## Response Status Codes

Each API may document specific status codes and provide a specific reason for returning that status code. This is a general overview of the status codes you may expect from an API and what they will mean to you.

Code | Description
---- | -----------
200 OK | The request was successful. The response will contain a JSON body.
400 Bad Request | The request was invalid and/or malformed. The response will contain JSON describing the specific errors.
401 Unauthorized | You did not supply a valid Authorization header. The header was omitted or your API key was not valid. The response will be empty. See [Authentication](#authentication).
404 Not Found | The object you requested doesn’t exist. The response will be empty.
429 Too Many Attempts | Too many requests have been made within a short time period. Retry the request after the number of seconds indicated in the `Retry-After` header. 
500 Internal Server Error | There was an internal error. The response will be empty. This is generally a server error condition. If possible, please open a GitHub Issue so we can help you resolve the issue.
503 Service Unavailable | The requested action cannot be completed right now due server load or maintenance. Retry the request later, or after the number of seconds indicated in the `Retry-After` header.

## Error Responses

> Example response JSON for 400 Bad Request error:

```json
{
  "error": "Either staffId or accountUserId is required."
}
```

Where appropriate, error responses may also include a body describing the error in further detail.
