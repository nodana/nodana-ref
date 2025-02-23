# Nodana API Reference

Welcome to the API reference for Nodana. You should have everything you need to deploy apps on Nodana using the API.

> If you're looking for the Nodana CLI then you can find it [here](https://github.com/nodana/nodana-cli).

## Jump to Endpoint

[Create a project](#create-project)

[Retrieve a project](#retrieve-project)

[Update a project](#update-project)

[Delete a project](#delete-project)

[List projects](#list-projects)

[List project apps](#list-project-apps)

[Create an app](#create-app)

[Retrieve an app](#retrieve-app)

[Update an app](#update-app)

[Start an app](#start-app)

[Stop an app](#stop-app)

[Restart an app](#restart-app)

[Delete an app](#delete-app)

## Getting Started

You will need to create a token to be able to use this API. This can be done on the Nodana website from the account section. Please be very careful with your tokens as they provide full read/write access to your projects and apps.

## Concepts

The API should be very easy to use but there are a few things to know:

- Only GET and POST methods are supported
- Uses method name in the route (/update, /delete etc)
- All responses (including errors) are 200 http status codes

See endpoint documentation below for a more detailed explanation.

## Base Url

`https://api.nodana.io/<version>`

## Authentication

You will need an api key to make requests. Please add this key to the password part of the Basic Authentication header. You should leave the username part of the header empty.

All calls should be made over https.

Keep your keys safe as they provide access to your projects and apps.

## Rate Limiting

In general, requests are limited to 1 per second. Some endpoints may enforce stricter rates.
Please do not repeatedly abuse the rate limits or keys will be banned.

## Errors

If there is an internal error with the api (which shouldn't be very often) then there will be a 500 response so ensure you requests logic can catch unexpected errors.

All other responses are considered successful (200) so you need to examine the response payload to understand the outcome of the request:

Here's an example response:

```json
{
  "message": "Request is forbidden",
  "error": "ForbiddenError",
  "statusCode": 403
}
```

Always check for the existence of `error` on the response object. If it exists then your request hasn't been successful. If there isn't an `error` property then your request has been successful and there will be a `data` property instead. Responses never include both `data` and `error` properties.

## Projects

Projects allow you to group related apps and these apps can communicate with each other over a private network. Billing is applied to projects and you will be able to scope API keys to a project in the future.

## Create Project

Create a project.

`POST /projects`

### Request

```bash
curl -i -X POST \\
  -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
  "${BASE_URL}/v1/projects" \\
  -d '{
    "name"="My Project",
  }'
```

### Response

```json
{
  "data": {
    "id": "<project-id>",
    "name": "My Project"
  }
}
```

## Retrieve Project

Retrieve a project.

`GET /projects/:id`

### Request

```bash
curl -i -X GET \\
  -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
  "${BASE_URL}/v1/projects/:id"
```

### Response

```json
{
  "data": {
    "id": "<project-id>",
    "userId": "<user-id>",
    "name": "New Project",
    "deleted": false,
    "createdAt": "2025-05-01T07:46:12.368Z",
    "updatedAt": "2025-05-01T07:46:12.368Z"
  }
}
```

## Update Project

Update a project.

`POST /projects/:id/update`

### Request

```bash
curl -i -X POST \\
  -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
  "${BASE_URL}/v1/projects/:id/update" \\
  -d '{
    "name"="My Updated Project",
  }'
```

### Response

```json
{
  "data": {
    "id": "<project-id>",
    "updated": true
  }
}
```

## Delete Project

Delete a project. Note that only empty projects can be deleted.

`POST /projects/:id/delete`

### Request

```bash
curl -i -X POST \\
  -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
  "${BASE_URL}/v1/projects/:id/delete"
```

### Response

```json
{
  "data": {
    "id": "<project-id>",
    "deleted": true
  }
}
```

## List Projects

List all projects.

`GET /projects`

### Request

```bash
curl -i -X GET \\
  -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
  "${BASE_URL}/v1/projects" \\
```

### Response

```json
{
  "data": [
    {
      "id": "<project-id>",
      "userId": "<user-id>",
      "name": "New Project 1",
      "deleted": false,
      "createdAt": "2025-05-01T07:46:12.368Z",
      "updatedAt": "2025-05-01T07:46:12.368Z"
    },
    {
      "id": "<project-id>",
      "userId": "<user-id>",
      "name": "New Project 2",
      "deleted": false,
      "createdAt": "2025-05-01T07:46:12.368Z",
      "updatedAt": "2025-05-01T07:46:12.368Z"
    }
  ]
}
```

## List Project Apps

List all apps within a project.

`GET /projects/:id/apps`

### Request

```bash
curl -i -X GET \\
  -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
  "${BASE_URL}/v1/projects/:id/apps" \\
```

### Response

```json
{
  "data": [
    {
      "id": "<app-id>",
      "userId": "<user-id>",
      "projectId": "<project-id>",
      "template": "template-name",
      "rate": 0.000233,
      "url": "https://app-url.nodana.app:8000",
      "status": "running",
      "createdAt": "2025-05-24T08:07:10.509Z",
      "updatedAt": "2025-05-24T08:07:10.509Z"
    }
  ]
}
```

## Apps

Apps are the wrapper around the machine that run your chosen software.

## Create App

Create an app. You can provide a `project_id` if you want the app to be assigned to an existing project. If you don't provide a `project_id` then a new project will be created automatically.

`POST /apps`

### Request

```bash
curl -i -X POST \\
    -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
    "${BASE_URL}/v1/apps" \\
  -d '{
    "project_id"="prj-5fd3",
    "template": "template-name",
    "params": {
      "prop-1": "value-1",
      "prop-2": "value-2,
    },
  }'
```

`params` is an object but the properties differ for each template. If the template isn't listed below then it doesn't accept config values.

> You will receive a response to the Create App request immediately but it doesn't mean the app is running. It usually takes about 10 seconds for an app to start (but this can vary). You should use the Retrieve App endpoint to know when the status is "available".

#### alby-hub-phoenixd

| Property         | Type   | Possible Values |
| ---------------- | ------ | --------------- |
| `auto_liquidity` | string | 2m, 5m, 10m     |

#### cdk-mintd

| Property           | Type   | Possible Values |
| ------------------ | ------ | --------------- |
| `mint_name`        | string |                 |
| `mint_description` | string |                 |

#### lnbits-phoenixd

| Property         | Type   | Possible Values |
| ---------------- | ------ | --------------- |
| `auto_liquidity` | string | 2m, 5m, 10m     |

#### nutshell

| Property            | Type   | Possible Values |
| ------------------- | ------ | --------------- |
| `mint_name`         | string |                 |
| `mint_description`  | string |                 |
| `mint_lnd_rest_url` | string |                 |
| `mint_lnd_macaroon` | string |                 |

#### phoenixd

| Property         | Type   | Possible Values |
| ---------------- | ------ | --------------- |
| `auto_liquidity` | string | 2m, 5m, 10m     |
| `webhook`        | string |                 |

### Response

```json
{
  "projectId": "<project-id>",
  "id": "<app-id>",
  "url": "https://app-url-1234.nodana.app:8080"
}
```

> The full response will differ depending on the template provided. For example, if you create a phoenixd node then the response will include a seed phrase, passwords etc.

## Retrieve App

Retrieve an app.

`GET /apps/:id`

### Request

```bash
curl -i -X GET \\
  -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
  "${BASE_URL}/v1/apps/:id"
```

### Response

```json
{
  "data": {
    "id": "<app-id>",
    "userId": "<user-id>",
    "projectId": "<project-id>",
    "template": "template-name",
    "rate": 0.000233,
    "url": "https://template-id-ad4f.nodana.app:8000",
    "status": "available",
    "createdAt": "2025-05-24T08:07:10.509Z",
    "updatedAt": "2025-05-24T08:07:10.509Z"
  }
}
```

> App status can either be "available", "cordoned" or "deleted".

## Update App

Update an app to use the latest supported version.

`POST /apps/:id/update`

### Request

```bash
curl -i -X POST \\
  -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
  "${BASE_URL}/v1/apps/:id/update"
```

### Response

```json
{
  "data": {
    "id": "<app-id>",
    "image": "nodana/phoenixd:0.7.2",
    "version": "0.7.2"
  }
}
```

## Start App

Start an app.

`POST /apps/:id/start`

### Request

```bash
curl -i -X POST \\
    -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
    "${BASE_URL}/v1/apps/:id/start"
```

### Response

```json
{
  "data": {
    "id": "<app-id>"
  }
}
```

## Stop App

Stop an app.

`POST /apps/:id/stop`

### Request

```bash
curl -i -X POST \\
    -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
    "${BASE_URL}/v1/apps/:id/stop"
```

### Response

```json
{
  "data": {
    "id": "<app-id>"
  }
}
```

## Restart App

Restart an app.

`POST /apps/:id/restart`

### Request

```bash
curl -i -X POST \\
    -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
    "${BASE_URL}/v1/apps/:id/restart"
```

### Response

```json
{
  "data": {
    "id": "<app-id>"
  }
}
```

## Delete App

Delete an app.

`POST /apps/:id/delete`

### Request

```bash
curl -i -X POST \\
    -H "Authorization: Basic :${API_KEY}" -H "Content-Type: application/json" \\
    "${BASE_URL}/v1/apps/:id/delete"
```

```json
{
  "data": {
    "id": "<app-id>"
  }
}
```
