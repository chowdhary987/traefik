# IPWhiteList

Limiting Clients to Specific IPs
{: .subtitle }

![IpWhiteList](../assets/img/middleware/ipwhitelist.png)

IPWhitelist accepts / refuses requests based on the client IP.

## Configuration Examples

```yaml tab="Docker"
# Accepts request from defined IP
labels:
- "traefik.http.middlewares.test-ipwhitelist.IPWhiteList.SourceRange=127.0.0.1/32, 192.168.1.7"
```

```yaml tab="Kubernetes"
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: test-ipwhitelist
spec:
  ipWhiteList:
    sourceRange:
    - 127.0.0.1/32
    - 192.168.1.7
```

```json tab="Marathon"
"labels": {
  "traefik.http.middlewares.test-ipwhitelist.IPWhiteList.SourceRange": "127.0.0.1/32,192.168.1.7"
}
```

```yaml tab="Rancher"
# Accepts request from defined IP
labels:
- "traefik.http.middlewares.test-ipwhitelist.IPWhiteList.SourceRange=127.0.0.1/32, 192.168.1.7"
```

```toml tab="File"
# Accepts request from defined IP
[http.middlewares]
  [http.middlewares.test-ipwhitelist.ipWhiteList]
    sourceRange = ["127.0.0.1/32", "192.168.1.7"]
```

## Configuration Options

### `sourceRange`

The `sourceRange` option sets the allowed IPs (or ranges of allowed IPs).

### `ipStrategy`

The `ipStrategy` option defines two parameters that sets how Traefik will determine the client IP: `depth`, and `excludedIPs`.

#### `ipStrategy.depth`

The `depth` option tells Traefik to use the `X-Forwarded-For` header and take the IP located at the `depth` position (starting from the right).

!!! note "Examples of Depth & X-Forwarded-For"

    If `depth` was equal to 2, and the request `X-Forwarded-For` header was `"10.0.0.1,11.0.0.1,12.0.0.1,13.0.0.1"` then the "real" client IP would be `"10.0.0.1"` (at depth 4) but the IP used for the whitelisting would be `"12.0.0.1"` (`depth=2`).
    
    ??? note "More examples"
    
        | `X-Forwarded-For`                       | `depth` | clientIP     |
        |-----------------------------------------|---------|--------------|
        | `"10.0.0.1,11.0.0.1,12.0.0.1,13.0.0.1"` | `1`     | `"13.0.0.1"` |
        | `"10.0.0.1,11.0.0.1,12.0.0.1,13.0.0.1"` | `3`     | `"11.0.0.1"` |
        | `"10.0.0.1,11.0.0.1,12.0.0.1,13.0.0.1"` | `5`     | `""`         |
    
    ```yaml tab="Docker"
    # Whitelisting Based on `X-Forwarded-For` with `depth=2`
    labels:
        - "traefik.http.middlewares.testIPwhitelist.ipWhiteList.SourceRange=127.0.0.1/32, 192.168.1.7"
        - "traefik.http.middlewares.testIPwhitelist.ipwhitelist.ipstrategy.depth=2"
    ```
    
    ```yaml tab="Kubernetes"
    # Whitelisting Based on `X-Forwarded-For` with `depth=2`
    apiVersion: traefik.containo.us/v1alpha1
    kind: Middleware
    metadata:
      name: testIPwhitelist
    spec:
      ipWhiteList:
        SourceRange:
        - 127.0.0.1/32
        - 192.168.1.7
        ipstrategy:
          depth: 2
    ```
    
    ```yaml tab="Rancher"
    # Whitelisting Based on `X-Forwarded-For` with `depth=2`
    labels:
        - "traefik.http.middlewares.testIPwhitelist.ipWhiteList.SourceRange=127.0.0.1/32, 192.168.1.7"
        - "traefik.http.middlewares.testIPwhitelist.ipwhitelist.ipstrategy.depth=2"
    ```
    
    ```toml tab="File"
    # Whitelisting Based on `X-Forwarded-For` with `depth=2`
    [http.middlewares]
      [http.middlewares.test-ipwhitelist.ipWhiteList]
        sourceRange = ["127.0.0.1/32", "192.168.1.7"]
        [http.middlewares.test-ipwhitelist.ipWhiteList.ipStrategy]
            depth = 2
    ```

!!! note

    - If `depth` is greater than the total number of IPs in `X-Forwarded-For`, then the client IP will be empty.
    - `depth` is ignored if its value is is lesser than or equal to 0.

#### `ipStrategy.excludedIPs`

`excludedIPs` tells Traefik to scan the `X-Forwarded-For` header and pick the first IP not in the list.

!!! note "Examples of ExcludedIPs & X-Forwarded-For"

    | `X-Forwarded-For`                       | `excludedIPs`         | clientIP     |
    |-----------------------------------------|-----------------------|--------------|
    | `"10.0.0.1,11.0.0.1,12.0.0.1,13.0.0.1"` | `"12.0.0.1,13.0.0.1"` | `"11.0.0.1"` |
    | `"10.0.0.1,11.0.0.1,12.0.0.1,13.0.0.1"` | `"15.0.0.1,13.0.0.1"` | `"12.0.0.1"` |
    | `"10.0.0.1,11.0.0.1,12.0.0.1,13.0.0.1"` | `"10.0.0.1,13.0.0.1"` | `"12.0.0.1"` |
    | `"10.0.0.1,11.0.0.1,12.0.0.1,13.0.0.1"` | `"15.0.0.1,16.0.0.1"` | `"13.0.0.1"` |
    | `"10.0.0.1,11.0.0.1"`                   | `"10.0.0.1,11.0.0.1"` | `""`         |

!!! important
    If `depth` is specified, `excludedIPs` is ignored.

```yaml tab="Docker"
# Exclude from `X-Forwarded-For`
labels:
    - "traefik.http.middlewares.test-ipwhitelist.ipwhitelist.ipstrategy.excludedIPs=127.0.0.1/32, 192.168.1.7"
```

```yaml tab="Kubernetes"
# Exclude from `X-Forwarded-For`
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: test-ipwhitelist
spec:
  ipWhiteList:
    ipstrategy:
      excludedIPs:
      - 127.0.0.1/32
      - 192.168.1.7
```

```yaml tab="Rancher"
# Exclude from `X-Forwarded-For`
labels:
    - "traefik.http.middlewares.test-ipwhitelist.ipwhitelist.ipstrategy.excludedIPs=127.0.0.1/32, 192.168.1.7"
```

```toml tab="File"
# Exclude from `X-Forwarded-For`
[http.middlewares]
  [http.middlewares.test-ipwhitelist.ipWhiteList]
    [http.middlewares.test-ipwhitelist.ipWhiteList.ipStrategy]
      excludedIPs = ["127.0.0.1/32", "192.168.1.7"]
```
