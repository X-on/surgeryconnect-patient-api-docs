# Getting Started


## Authentication

```shell
curl "api_endpoint_here" \
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

Unless otherwise specified, all request URIs should be appended to the following base URL:

`https://api.x-on.app`

## Status Codes

Each API may document specific status codes and provide a specific reason for returning that status code. This is a general overview of the status codes you may expect from an API and what they will mean to you.

Code | Description
---- | -----------
200 | The request was successful. The response will contain a JSON body.
400 | The request was invalid and/or malformed. The response will contain JSON describing the specific errors.
401 | You did not supply a valid Authorization header. The header was omitted or your API key was not valid. The response will be empty. See [Authentication](#authentication).
404 | The object you requested doesn’t exist. The response will be empty.
500 | There was an internal error. The response will be empty. This is generally a server error condition. If possible, please open a GitHub Issue so we can help you resolve the issue.
503 | The requested action cannot be completed due the current rate of requests. Retry the request later.