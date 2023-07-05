# Local testing of an Envoy proxy with a lua filter

In trying to debug some Envoy behaviour, I wanted to run a local proxy
and mess around with some lua filters.

After a few false starts, I got things working.

Sources:
* Pretty sure this repo is the origin of this config: https://github.com/ibm-cloud-architecture/tutorial-istio-envoy-lua-filters/tree/master
* This blog post, which has typos and is old, and seems very similar to the
  above repo, but is a few years newer: https://varunksaini.com/http-proxy-lua-filter/
* This GitHub issue which helped resolve errors: https://github.com/envoyproxy/envoy/issues/21464

To avoid this breaking, I've pinned the versions of the docker images used,
since if you're using this you probably don't care about the specific version
of Envoy you're using, and probably care more about testing some quirk of the
lua filter.

## Usage

```
docker-compose up --build

# This curls the echo container directly
curl -v localhost:8080
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.88.1
> Accept: */*
>
< HTTP/1.1 200 OK
< X-Powered-By: Express
< Content-Type: application/json; charset=utf-8
< Content-Length: 359
< ETag: W/"167-VUuUz5gio5UBbiCjtWNCNIQd2Js"
< Date: Wed, 05 Jul 2023 07:59:56 GMT
< Connection: keep-alive
<
{
  "path": "/",
  "headers": {
    "host": "localhost:8080",
    "user-agent": "curl/7.88.1",
    "accept": "*/*"
  },
  "method": "GET",
  "body": "",
  "fresh": false,
  "hostname": "localhost",
  "ip": "::ffff:172.26.0.1",
  "ips": [],
  "protocol": "http",
  "query": {},
  "subdomains": [],
  "xhr": false,
  "os": {
    "hostname": "7c8ecda1ab77"
  }
* Connection #0 to host localhost left intact
}%      

# This curls the container via Envoy
# By default, you will see an additional foo: bar header, and some envoy headers added
curl -v localhost:8000
*   Trying 127.0.0.1:8000...
* Connected to localhost (127.0.0.1) port 8000 (#0)
> GET / HTTP/1.1
> Host: localhost:8000
> User-Agent: curl/7.88.1
> Accept: */*
>
< HTTP/1.1 200 OK
< x-powered-by: Express
< content-type: application/json; charset=utf-8
< content-length: 517
< etag: W/"205-h6BllTwWoJsCG9r12MCWQDTYDyw"
< date: Wed, 05 Jul 2023 08:00:23 GMT
< x-envoy-upstream-service-time: 8
< server: envoy
<
{
  "path": "/",
  "headers": {
    "host": "localhost:8000",
    "user-agent": "curl/7.88.1",
    "accept": "*/*",
    "x-forwarded-proto": "http",
    "x-request-id": "c375004b-e854-46e3-8675-34b0f0779bdb",
    "foo": "bar", # This header is new
    "x-envoy-expected-rq-timeout-ms": "15000"
  },
  "method": "GET",
  "body": "",
  "fresh": false,
  "hostname": "localhost",
  "ip": "::ffff:172.26.0.2",
  "ips": [],
  "protocol": "http",
  "query": {},
  "subdomains": [],
  "xhr": false,
  "os": {
    "hostname": "7c8ecda1ab77"
  }
* Connection #0 to host localhost left intact
}%    
```

Make changes to the `envoy.yaml` file, then run docker compose again to alter
the lua script in the envoy filter.
