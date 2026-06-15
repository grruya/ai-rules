---
name: rest-client
description: >-
    VS Code REST Client guidance for concise, runnable, and safe `.http` or `.rest`
    API request files. Use when creating, editing, explaining, or reviewing REST Client
    requests, variables, auth headers, GraphQL calls, cURL snippets, or request chaining.
---

**Do not use this skill when**:

- The task does not involve `.http` or `.rest` files.
- The user needs production API client code, SDK code, or backend route implementation.
- The task is only general API design, OpenAPI docs, Postman collections, or shell `curl` usage outside REST Client files.

# REST Client

Use VS Code REST Client style. Keep `.http` / `.rest` files short, runnable, and safe.

## Core Syntax

- One request per block.
- Split requests with `###`.
- Request line: `METHOD URL HTTP/1.1`. If method missing, REST Client treats request as `GET`.
- Headers go after request line: `Header-Name: value`.
- Body starts after one blank line.
- Comments: `# comment` or `// comment`.

```http
GET https://api.example.com/users/1 HTTP/1.1
Accept: application/json

###

POST https://api.example.com/users HTTP/1.1
Content-Type: application/json

{
  "name": "Ada"
}
```

## Query Params

Use multiline params for readable URLs.

```http
GET https://api.example.com/users
    ?page=1
    &limit=20
```

## Variables

Use variables to avoid repeated URLs, tokens, IDs.

- File variable: `@name = value`
- Reference custom variable: `{{name}}`
- Reference system variable: `{{$guid}}`
- Percent-encode variable: `{{%name}}`
- Prompt variable before request: `# @prompt token`

```http
@baseUrl = https://api.example.com
@token = {{$dotenv API_TOKEN}}

GET {{baseUrl}}/users
Authorization: Bearer {{token}}
X-Request-Id: {{$guid}}
```

Useful system variables:

- `{{$guid}}`
- `{{$randomInt min max}}`
- `{{$timestamp}}`
- `{{$datetime iso8601}}`
- `{{$localDatetime iso8601}}`
- `{{$processEnv NAME}}`
- `{{$dotenv NAME}}`

## Named Requests + Chaining

Name request with `# @name name`. Later requests can read latest response.

```http
@baseUrl = https://api.example.com

# @name login
POST {{baseUrl}}/login
Content-Type: application/json

{
  "email": "dev@example.com",
  "password": "secret"
}

###

@authToken = {{login.response.body.$.token}}

GET {{baseUrl}}/me
Authorization: Bearer {{authToken}}
```

Reference forms:

- Body JSONPath: `{{login.response.body.$.token}}`
- Header: `{{login.response.headers.X-AuthToken}}`
- Full body: `{{login.response.body.*}}`
- Request data: `{{login.request.body.*}}`

Named request must be run once before response variables resolve.

## Auth

Prefer explicit `Authorization` headers.

```http
GET https://api.example.com/me
Authorization: Bearer {{token}}

###

GET https://httpbin.org/basic-auth/user/passwd
Authorization: Basic user passwd

###

GET https://httpbin.org/digest-auth/auth/user/passwd
Authorization: Digest user passwd
```

AWS Signature v4:

```http
GET https://service.region.amazonaws.com/path
Authorization: AWS <accessId> <accessKey> [token:<sessionToken>] [region:<regionName>] [service:<serviceName>]
```

AWS Cognito:

```http
GET https://api.example.com/protected
Authorization: COGNITO <Username> <Password> <Region> <UserPoolId> <ClientId>
```

## Body From File

Use file content as request body.

```http
POST https://api.example.com/upload
Content-Type: application/json

< ./payload.json
```

Process variables inside file:

```http
POST https://api.example.com/upload
Content-Type: application/json

<@ ./payload.json
```

## Forms + Multipart

```http
POST https://api.example.com/login
Content-Type: application/x-www-form-urlencoded

email=dev@example.com
&password=secret
```

```http
POST https://api.example.com/upload
Content-Type: multipart/form-data; boundary=----BOUNDARY

------BOUNDARY
Content-Disposition: form-data; name="title"

Example
------BOUNDARY
Content-Disposition: form-data; name="file"; filename="image.png"
Content-Type: image/png

< ./image.png
------BOUNDARY--
```

## GraphQL

Add `X-Request-Type: GraphQL`. Query body first, optional variables after blank line.

```http
POST https://api.example.com/graphql
Content-Type: application/json
Authorization: Bearer {{token}}
X-Request-Type: GraphQL

query ($id: ID!) {
  user(id: $id) {
    id
    name
  }
}

{
  "id": "123"
}
```

## cURL

REST Client can run common cURL commands.

Supported useful flags:

- `-X`, `--request`
- `-H`, `--header`
- `-d`, `--data`, `--data-raw`, `--data-binary`
- `-u`, `--user`
- `-I`, `--head`
- `-L`, `--location`

```http
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Ada"}'
```

## Per-Request Settings

Put before request.

```http
# @name deleteUser
# @note Confirm destructive request
# @no-redirect
# @no-cookie-jar
DELETE https://api.example.com/users/123
Authorization: Bearer {{token}}
```

## Useful Commands

- Send request: click `Send Request`, run `Rest Client: Send Request`, or use `Cmd+Alt+R` on macOS.
- Cancel request: `Rest Client: Cancel Request` or `Cmd+Alt+K`.
- Rerun last: `Rest Client: Rerun Last Request` or `Cmd+Alt+L`.
- Switch env: `Rest Client: Switch Environment` or `Cmd+Alt+E`.
- Copy request as cURL: `Rest Client: Copy Request As cURL`.
- Generate snippet: `Rest Client: Generate Code Snippet`.

## Defaults

- Use `.http` or `.rest`.
- Keep secrets in `.env`, shell env, or VS Code REST Client environment variables. Do not commit secrets.
- Use named requests for login/token flows.
- Use `# @note` before destructive calls.
- Keep examples runnable against current project API, not generic docs clutter.
