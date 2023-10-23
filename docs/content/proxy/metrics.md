# Metrics

Starting from the v0.3.0 release, Capsule Proxy exposes Prometheus metrics available at `http://0.0.0.0:8080/metrics`.

The offered metrics are related to the internal `controller-manager` code base, such as work queue and REST client requests, and the Go runtime ones.

Along with these, metrics `capsule_proxy_response_time_seconds` and `capsule_proxy_requests_total` have been introduced and are specific to the Capsule Proxy code-base and functionalities.

`capsule_proxy_response_time_seconds` offers a bucket representation of the HTTP request duration.
The available variables for these metrics are the following ones:
- `path`: the HTTP path of every single request that Capsule Proxy passes to the upstream

`capsule_proxy_requests_total` counts the global requests that Capsule Proxy is passing to the upstream with the following labels.
- `path`: the HTTP path of every single request that Capsule Proxy passes to the upstream
- `status`: the HTTP status code of the request

Example output of the metrics:

```
# HELP capsule_proxy_requests_total Number of requests
# TYPE capsule_proxy_requests_total counter
capsule_proxy_requests_total{path="/api/v1/namespaces",status="403"} 1
# HELP capsule_proxy_response_time_seconds Duration of capsule proxy requests.
# TYPE capsule_proxy_response_time_seconds histogram
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="0.005"} 0
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="0.01"} 0
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="0.025"} 0
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="0.05"} 0
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="0.1"} 0
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="0.25"} 0
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="0.5"} 0
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="1"} 0
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="2.5"} 1
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="5"} 1
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="10"} 1
capsule_proxy_response_time_seconds_bucket{path="/api/v1/namespaces",le="+Inf"} 1
capsule_proxy_response_time_seconds_sum{path="/api/v1/namespaces"} 2.206192787
capsule_proxy_response_time_seconds_count{path="/api/v1/namespaces"} 1
```
