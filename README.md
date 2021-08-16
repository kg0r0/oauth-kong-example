# oauth-kong-example

## Add Service to Kong

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"name":"step-on-api-server", "url":"http://api:3000"}' https://localhost:8001/services
{"retries":5,"port":3000,"protocol":"http","client_certificate":null,"tls_verify_depth":null,"created_at":1629142483,"updated_at":1629142483,"connect_timeout":60000,"write_timeout":60000,"read_timeout":60000,"tags":null,"name":"step-on-api-server","path":null,"tls_verify":null,"ca_certificates":null,"id":"89eebfb2-dc68-4300-b834-e66e435c18e7","host":"api"}⏎
```

## Add Route to Kong

```
$ curl -k -X POST -H "Content-Type: application/json" -d '{"name":"step-on-route", "service": {"name":"step-on-api-server"}, "paths": ["/stepon"]}' https://localhost:8001/routes
{"hosts":null,"paths":["/stepon"],"methods":null,"sources":null,"path_handling":"v0","request_buffering":true,"response_buffering":true,"updated_at":1629142497,"created_at":1629142497,"id":"7a70a6e5-63fd-4d10-a2a8-97318c301645","https_redirect_status_code":426,"service":{"id":"89eebfb2-dc68-4300-b834-e66e435c18e7"},"regex_priority":0,"snis":null,"strip_path":true,"destinations":null,"tags":null,"headers":null,"protocols":["http","https"],"preserve_host":false,"name":"step-on-route"}⏎
```

## Test Endpoint Through Kong Gateway

```
$ curl -k -H "mock-logged-in-as: clark" https://localhost:8000/stepon/stepcounts
[{"date":"2021-01-01","count":2500},{"date":"2021-01-02","count":12000},{"date":"2021-01-03","count":9500}]
```


## References
- https://konghq.com/blog/kong-gateway-oauth2/
- https://docs.konghq.com/install/docker/
- https://github.com/Kong/docker-kong