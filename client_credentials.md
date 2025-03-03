---
Client Credentials Flow
---

```mermaid
sequenceDiagram
  actor a as Microservice API 1
  participant a0 as Auth0
  participant api as Microservice API 2 <br>(API Gateway / middleware)
  autonumber

  a ->> a0: Authenticate with M2M Application Credentials
  note right of a: POST /oauth/token<br/>grant_type=client_credentials<br/>client_id=client_id<br/>client_secret=client_secret<br/>audience=api_identifier

  a0 ->> a0: Validate Client Credentials

  a0 -->> a: Return Access Token
  note right of a: { "access_token": "token", "token_type": "Bearer", "expires_in": 3600 }

  a ->> api: Use Access Token to Access Protected Resource
  note right of a: GET /resource<br/>Authorization: Bearer access_token

  critical Validate Access Token
  alt JWKS Cached?
    api ->> api: Validate JWT using Cached JWKS Keys
  else JWKS Not Cached
    api ->> a0: Fetch JWKS from Auth0
    a0 -->> api: Return JWKS Response
    api ->> api: Validate JWT using JWKS
  end
  end

  alt [Token Validation Successful]
    api -->> a: Return Resource Data
    note right of a: { "data": "resource_data" }
  else [Token Validation Failed]
    api -->> a: Return 401 Unauthorized
  end

```


### Image Link : [Client Credentials Flow](https://github.com/Tanver-Hasan/oauth2.0-flows-/blob/main/images/client_credentails_flow.png)
