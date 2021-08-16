# oauth-kong-example

## Usage

```
$ git clone https://github.com/kg0r0/oauth-kong-example.git
$ cd oauth-kong-example
$ docker-compose up -d 
```

##  Put Your API Server Behind Kong Gateway

### Add Service to Kong

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"name":"step-on-api-server", "url":"http://api:3000"}' https://localhost:8001/services
{
  "retries": 5,
  "port": 3000,
  "protocol": "http",
  "client_certificate": null,
  "tls_verify_depth": null,
  "created_at": 1629142483,
  "updated_at": 1629142483,
  "connect_timeout": 60000,
  "write_timeout": 60000,
  "read_timeout": 60000,
  "tags": null,
  "name": "step-on-api-server",
  "path": null,
  "tls_verify": null,
  "ca_certificates": null,
  "id": "89eebfb2-dc68-4300-b834-e66e435c18e7",
  "host": "api"
}
```

### Add Route to Kong

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"name":"step-on-route", "service": {"name":"step-on-api-server"}, "paths": ["/stepon"]}' https://localhost:8001/routes
{
  "hosts": null,
  "paths": [
    "/stepon"
  ],
  "methods": null,
  "sources": null,
  "path_handling": "v0",
  "request_buffering": true,
  "response_buffering": true,
  "updated_at": 1629142497,
  "created_at": 1629142497,
  "id": "7a70a6e5-63fd-4d10-a2a8-97318c301645",
  "https_redirect_status_code": 426,
  "service": {
    "id": "89eebfb2-dc68-4300-b834-e66e435c18e7"
  },
  "regex_priority": 0,
  "snis": null,
  "strip_path": true,
  "destinations": null,
  "tags": null,
  "headers": null,
  "protocols": [
    "http",
    "https"
  ],
  "preserve_host": false,
  "name": "step-on-route"
}
```

### Test Endpoint Through Kong Gateway

```
$ curl -k -H "mock-logged-in-as: clark" https://localhost:8000/stepon/stepcounts
[
  {
    "date": "2021-01-01",
    "count": 2500
  },
  {
    "date": "2021-01-02",
    "count": 12000
  },
  {
    "date": "2021-01-03",
    "count": 9500
  }
]
```

## Enable the Kong Gateway OAuth 2 Plugin

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"name":"oauth2", "config": {"scopes":["user_profile", "biometric", "step_counts"], "mandatory_scope": true, "enable_authorization_code": true}, "protocols": ["https"]}' https://localhost:8001/services/step-on-api-server/plugins
{
  "created_at": 1629143302,
  "id": "f07fdaee-251c-41d6-8fb0-680cc70f7a9b",
  "name": "oauth2",
  "service": {
    "id": "89eebfb2-dc68-4300-b834-e66e435c18e7"
  },
  "route": null,
  "enabled": true,
  "tags": null,
  "protocols": [
    "https"
  ],
  "config": {
    "enable_client_credentials": false,
    "auth_header_name": "authorization",
    "accept_http_if_already_terminated": false,
    "scopes": [
      "user_profile",
      "biometric",
      "step_counts"
    ],
    "anonymous": null,
    "provision_key": "KB8zmkuKDd8lzt3u8y4sYE47H9SFgOrJ",
    "enable_authorization_code": true,
    "reuse_refresh_token": false,
    "token_expiration": 7200,
    "refresh_token_ttl": 1209600,
    "global_credentials": false,
    "pkce": "lax",
    "enable_password_grant": false,
    "hide_credentials": false,
    "mandatory_scope": true,
    "enable_implicit_grant": false
  },
  "consumer": null
}
```

### Add Client Application (API Consumer) to the Plugin

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"username": "shoeflyshoe"}' https://localhost:8001/consumers
{
  "username": "shoeflyshoe",
  "id": "73411a66-15b6-4680-a44c-4f3daaa86b52",
  "custom_id": null,
  "tags": null,
  "created_at": 1629143473
}
```

### Add OAuth Credentials 

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"name": "Shoe Fly Shoe Customer Rewards", "redirect_uris": ["https://shoeflyshoe.store/oauth_return"]}' https://localhost:8001/consumers/shoeflyshoe/oauth2
{
  "created_at": 1629143669,
  "redirect_uris": [
    "https://shoeflyshoe.store/oauth_return"
  ],
  "name": "Shoe Fly Shoe Customer Rewards",
  "client_id": "1wyDxCGUgq0ugdezJy5EN4iktJ5197B5",
  "hash_secret": false,
  "client_type": "confidential",
  "tags": null,
  "id": "8fe80872-a63b-49fe-a46a-0e3ecb7fd63a",
  "consumer": {
    "id": "73411a66-15b6-4680-a44c-4f3daaa86b52"
  },
  "client_secret": "UFqTO2mmpNrm82aJLeWocwi8akM92FIH"
}
```

## Walk Through OAuth2 Flow of Requests

### User Submits POST Request to Authorize Application

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"client_id": "1wyDxCGUgq0ugdezJy5EN4iktJ5197B5", "response_type": "code", "scope": "step_counts", "provision_key": "KB8zmkuKDd8lzt3u8y4sYE47H9SFgOrJ", "authenticated_userid": "clark", "redirect_url": "https://shoeflyshoe.store/oauth_return" }' https://localhost:8000/stepon/oauth2/authorize
{
  "redirect_uri": "https://shoeflyshoe.store/oauth_return?code=HIbSOblERaBIVZxepeZEfqezzCrXRa9m"
}
```

### Client Application Exchanges Code for Access Token

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"grant_type": "authorization_code", "code": "HIbSOblERaBIVZxepeZEfqezzCrXRa9m", "client_id": "1wyDxCGUgq0ugdezJy5EN4iktJ5197B5", "client_secret": "UFqTO2mmpNrm82aJLeWocwi8akM92FIH" }' https://localhost:8000/stepon/oauth2/token
{
  "expires_in": 7200,
  "access_token": "dMaYaT2uHOOi1YrSZ1NEb5agzwhtL7jY",
  "token_type": "bearer",
  "refresh_token": "Cxu5Oq55ZKoAv3E0S6Nx4QZbjrZYGfN0"
}
```

### Client Application Uses Access Token to Request Resource


```
$ curl -k -H "Authorization: Bearer dMaYaT2uHOOi1YrSZ1NEb5agzwhtL7jY" https://localhost:8000/stepon/stepcounts
[
  {
    "date": "2021-01-01",
    "count": 2500
  },
  {
    "date": "2021-01-02",
    "count": 12000
  },
  {
    "date": "2021-01-03",
    "count": 9500
  }
]

$ curl -k -H "Authorization: Bearer dummy" https://localhost:8000/stepon/stepcounts
{
  "error": "invalid_token",
  "error_description": "The access token is invalid or has expired"
}
```

## Requesting a Regreshed Access Token

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"grant_type": "refresh_token", "refresh_token": "Cxu5Oq55ZKoAv3E0S6Nx4QZbjrZYGfN0", "client_id": "1wyDxCGUgq0ugdezJy5EN4iktJ5197B5", "client_secret": "UFqTO2mmpNrm82aJLeWocwi8akM92FIH" }' https://localhost:8000/stepon/oauth2/token
{
  "expires_in": 7200,
  "access_token": "aMpSs6BoXZMF60s4BUAOIjpN1L7uKvqg",
  "token_type": "bearer",
  "refresh_token": "CS8ODf7IVI6yNtI62FX5FLlGOaiaqZGj"
}
```

## References
- https://konghq.com/blog/kong-gateway-oauth2/
- https://docs.konghq.com/install/docker/
- https://github.com/Kong/docker-kong