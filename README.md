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
curl localhost:8080
# This curls the container via Envoy
# By default, you will see an additional foo: bar header
curl localhost:8000
```

Make changes to the `envoy.yaml` file, then run docker compose again to alter
the lua script in the envoy filter.
