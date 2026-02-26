# ta_nginx_ingress

Technology Add-on for Kubernetes Nginx Ingress Controller logs.

## Sourcetypes

| Sourcetype | Description |
|---|---|
| `nginx:ingress:access` | Nginx Ingress Controller access logs |
| `nginx:ingress:error` | Nginx Ingress Controller error logs |

## Expected Log Format

This TA parses the default nginx ingress controller `log-format-upstream`:

```
$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $request_length $request_time [$proxy_upstream_name] [$proxy_alternative_upstream_name] $upstream_addr $upstream_response_length $upstream_response_time $upstream_status $req_id
```

Example:

```
10.244.0.1 - - [25/Feb/2026:10:30:15 +0000] "GET /api/v1/users HTTP/1.1" 200 1234 "https://example.com" "Mozilla/5.0" 456 0.032 [default-myapp-svc-80] [default-myapp-svc-80] 192.168.22.1:8080 1234 0.031 200 abc123def456
```

## Extracted Fields

### Standard Web Fields (CIM)

| Field | Description |
|---|---|
| `src` | Client IP address (`$remote_addr`) |
| `dest` | Upstream backend IP (parsed from `$upstream_addr`) |
| `dest_port` | Upstream backend port (parsed from `$upstream_addr`) |
| `http_method` | HTTP request method |
| `uri_path` | Request URI path |
| `uri_query` | Request query string |
| `status` | HTTP response status code |
| `bytes_in` | Request size in bytes (`$request_length`) |
| `bytes_out` | Response body size in bytes (`$body_bytes_sent`) |
| `bytes` | Total bytes (request + response) |
| `http_referrer` | HTTP Referer header |
| `http_user_agent` | HTTP User-Agent header |
| `response_time` | Request duration in milliseconds |
| `url` | Reconstructed URL |
| `user` | Authenticated user (null-coerced) |

### Kubernetes-Specific Fields

| Field | Description |
|---|---|
| `proxy_upstream_name` | Kubernetes service backend (`namespace-service-port`) |
| `proxy_alternative_upstream_name` | Alternative upstream name |
| `upstream_addr` | Backend pod address (`ip:port`) |
| `upstream_response_length` | Response length from upstream |
| `upstream_response_time` | Response time from upstream (seconds) |
| `upstream_status` | HTTP status from upstream |
| `req_id` | Unique request ID for tracing |

## CIM Compatibility

- **Web** data model: access events tagged with `web`
- HTTP status lookup provides `status_description`, `status_type`, and `action`

## Installation

Install on search heads for field extractions and CIM compliance. Events are expected to be ingested via HEC.

### RKE2 Sourcetype Configuration

Set the sourcetype on the `rke2-ingress-nginx-controller` DaemonSet via pod annotation:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: rke2-ingress-nginx-controller
  namespace: kube-system
spec:
  template:
    metadata:
      annotations:
        splunk.com/sourcetype: "nginx:ingress:access"
```

Apply with `kubectl patch`:

```bash
kubectl patch daemonset rke2-ingress-nginx-controller -n kube-system \
  --type merge \
  -p '{"spec":{"template":{"metadata":{"annotations":{"splunk.com/sourcetype":"nginx:ingress:access"}}}}}'
```

