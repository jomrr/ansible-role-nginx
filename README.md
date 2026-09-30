# Ansible Role: nginx

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-nginx)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-nginx)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-nginx)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-nginx/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-nginx/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-nginx/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-nginx/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for a minimal hardened nginx reverse proxy baseline.

## Purpose

Installs and runs nginx with a minimal reverse proxy baseline, default-deny HTTP
and HTTPS listeners,
verified HTTPS upstreams, and native nginx application configuration supplied
through `nginx_vhosts`.
The complete candidate is validated before replacing `/etc/nginx/nginx.conf`.

## Scope

### Managed

- nginx and CA certificate packages
- Complete nginx main configuration and declared native vhosts, including
  application-specific security policies
- Private temporary directories for request and response buffering
- An inaccessible default document root and configurable edge-owned security
  headers
- Optional runtime DNS resolvers and global request rate-limit zones
- Explicit persistent SELinux booleans and their Python bindings
- Service enablement, startup, and reload on configuration changes

### Not Managed

- Certificate issuance, deployment, renewal, and ACME challenges
- Application services, authentication, firewall, and SELinux ports or file
  contexts
- Distribution include directories and dynamic modules

## Requirements

- nginx 1.25.1 or newer with the HTTP SSL module and a TLS 1.3-capable library
  is required.
- The HTTP/2 module is required by default; enabling HTTP/3 additionally
  requires the HTTP/3 module.
- Application TLS certificates and private keys must already exist on the target
  before enabling a TLS vhost.

## Dependencies

```yaml
collections:
  - name: ansible.posix
    version: '>=2.0.0'
  - name: community.general
    version: '>=12.0.0'
  - name: community.crypto
    version: '>=2.0.0'
```

## Role Variables

### `nginx_extra_packages`

Type: `list`. Required: `false`.

Installs additional packages required by native vhost configuration.

Default:

```yaml
nginx_extra_packages: []
```

### `nginx_sebooleans`

Type: `dict`. Required: `false`.

Manages explicitly selected SELinux booleans persistently on enabled hosts.

Default:

```yaml
nginx_sebooleans: {}
```

### `nginx_listen`

Type: `list`. Required: `false`.

Defines default-deny endpoints; application listen directives must match these
addresses and ports.

Default:

```yaml
nginx_listen:
  - address: 0.0.0.0
    port: 80
    protocol: http
  - address: 0.0.0.0
    port: 443
    protocol: https
```

### `nginx_http2`

Type: `bool`. Required: `false`.

Enables inherited HTTP/2 support for client connections.

Default:

```yaml
nginx_http2: true
```

### `nginx_http3`

Type: `bool`. Required: `false`.

Enables QUIC address validation and default-deny UDP listeners for HTTPS
endpoints; native vhosts opt in separately.

Default:

```yaml
nginx_http3: false
```

### `nginx_worker_processes`

Type: `str`. Required: `false`.

Sets the number of workers or uses automatic CPU detection.

Default:

```yaml
nginx_worker_processes: auto
```

### `nginx_worker_connections`

Type: `int`. Required: `false`.

Limits simultaneous connections per worker, including upstream connections.

Default:

```yaml
nginx_worker_connections: 1024
```

### `nginx_keepalive_timeout`

Type: `str`. Required: `false`.

Sets the idle client keepalive timeout and optional advertised timeout.

Default:

```yaml
nginx_keepalive_timeout: 5 5
```

### `nginx_tcp_nodelay`

Type: `str`. Required: `false`.

Enables TCP_NODELAY for responsive keepalive connections.

Default:

```yaml
nginx_tcp_nodelay: 'on'
```

### `nginx_server_tokens`

Type: `str`. Required: `false`.

Controls disclosure of the nginx version in responses.

Default:

```yaml
nginx_server_tokens: 'off'
```

### `nginx_security_headers`

Type: `dict`. Required: `false`.

Sets edge-owned response headers on all statuses and suppresses matching
upstream headers.

Default:

```yaml
nginx_security_headers:
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  X-Frame-Options: DENY
  X-XSS-Protection: '0'
```

### `nginx_client_max_body_size`

Type: `str`. Required: `false`.

Limits request bodies; applications can explicitly override this limit.

Default:

```yaml
nginx_client_max_body_size: 1m
```

### `nginx_client_header_timeout`

Type: `str`. Required: `false`.

Limits time to read a complete request header.

Default:

```yaml
nginx_client_header_timeout: 10s
```

### `nginx_client_body_timeout`

Type: `str`. Required: `false`.

Limits idle time between reads of a request body.

Default:

```yaml
nginx_client_body_timeout: 10s
```

### `nginx_send_timeout`

Type: `str`. Required: `false`.

Limits idle time between writes to a client.

Default:

```yaml
nginx_send_timeout: 30s
```

### `nginx_resolvers`

Type: `list`. Required: `false`.

Defines DNS servers for runtime name resolution; an empty list leaves resolver
selection unset.

Default:

```yaml
nginx_resolvers: []
```

### `nginx_resolver_timeout`

Type: `str`. Required: `false`.

Limits the time spent resolving an upstream hostname.

Default:

```yaml
nginx_resolver_timeout: 2s
```

### `nginx_resolver_valid`

Type: `str`. Required: `false`.

Overrides DNS cache validity; an empty value preserves the response TTL.

Default:

```yaml
nginx_resolver_valid: ''
```

### `nginx_limit_req_log_level`

Type: `str`. Required: `false`.

Sets the log level for requests rejected by rate limiting.

Default:

```yaml
nginx_limit_req_log_level: warn
```

### `nginx_limit_req_status`

Type: `int`. Required: `false`.

Sets the HTTP status returned when a rate limit rejects a request.

Default:

```yaml
nginx_limit_req_status: 429
```

### `nginx_limit_req_zones`

Type: `list`. Required: `false`.

Defines shared rate-limit zones; native vhost locations activate them with
limit_req.

Default:

```yaml
nginx_limit_req_zones:
  - name: login
    key: $binary_remote_addr
    size: 10m
    rate: 10r/m
  - name: api_limit
    key: $binary_remote_addr
    size: 10m
    rate: 20r/s
```

### `nginx_proxy_buffering`

Type: `str`. Required: `false`.

Buffers upstream responses to isolate backends from slow clients.

Default:

```yaml
nginx_proxy_buffering: 'on'
```

### `nginx_proxy_request_buffering`

Type: `str`. Required: `false`.

Buffers request bodies before forwarding them to a backend.

Default:

```yaml
nginx_proxy_request_buffering: 'on'
```

### `nginx_proxy_connect_timeout`

Type: `str`. Required: `false`.

Limits time to establish an upstream connection.

Default:

```yaml
nginx_proxy_connect_timeout: 5s
```

### `nginx_proxy_send_timeout`

Type: `str`. Required: `false`.

Limits idle time between writes to an upstream.

Default:

```yaml
nginx_proxy_send_timeout: 60s
```

### `nginx_proxy_read_timeout`

Type: `str`. Required: `false`.

Limits idle time between reads from an upstream.

Default:

```yaml
nginx_proxy_read_timeout: 60s
```

### `nginx_proxy_websockets`

Type: `bool`. Required: `false`.

Enables WebSocket upgrade forwarding as a role-wide policy.

Default:

```yaml
nginx_proxy_websockets: false
```

### `nginx_proxy_ssl_trusted_certificate`

Type: `path`. Required: `false`.

Selects the HTTPS upstream CA bundle; an empty value uses the platform trust
store.

Default:

```yaml
nginx_proxy_ssl_trusted_certificate: ''
```

### `nginx_proxy_ssl_verify_depth`

Type: `int`. Required: `false`.

Sets the verification depth for HTTPS upstream certificate chains.

Default:

```yaml
nginx_proxy_ssl_verify_depth: 3
```

### `nginx_protocols`

Type: `list`. Required: `false`.

Selects modern TLS protocols for clients and HTTPS upstreams.

Default:

```yaml
nginx_protocols:
  - TLSv1.3
```

### `nginx_ciphers`

Type: `list`. Required: `false`.

Selects TLS 1.2 ECDHE ciphers; TLS 1.3 uses the TLS library suites.

Default:

```yaml
nginx_ciphers:
  - ECDHE-ECDSA-AES128-GCM-SHA256
  - ECDHE-RSA-AES128-GCM-SHA256
  - ECDHE-ECDSA-AES256-GCM-SHA384
  - ECDHE-RSA-AES256-GCM-SHA384
  - ECDHE-ECDSA-CHACHA20-POLY1305
  - ECDHE-RSA-CHACHA20-POLY1305
```

### `nginx_vhosts`

Type: `list`. Required: `false`.

Defines named native nginx blocks embedded in the http context.

Default:

```yaml
nginx_vhosts: []
```

## Managed Files

- `/etc/nginx/nginx.conf`
- `/var/lib/nginx/tmp`
- `/var/lib/nginx/empty`

## Check Mode

Predicts package, directory, and template changes, including on first
installation.
Native candidate validation is performed by the template module only on real
writes, not in check mode.
Service operations and SELinux boolean changes are skipped in check mode because
prerequisites may only be simulated.

## Service Behavior

Enables and starts nginx after installing validated configuration. Configuration
changes notify a graceful reload.
An unchanged run does not reload nginx. Invalid candidates do not replace the
main configuration or notify a reload.
Package installation may start the distribution service before role
configuration is applied.

### Handlers

- NGINX | Reload service

## Security Notes

- Default-deny listeners prevent requests with unexpected hostnames from
  reaching an application selected by listener order. Rejecting unknown TLS and
  QUIC SNI also avoids presenting an unrelated application's certificate. This
  protection requires a matching default listener for every application address
  and port; it does not replace authentication.
- An unreadable, root-owned document root prevents an omitted proxy handler from
  exposing packaged web content or local files. Disabling directory listings
  adds protection if a readable directory is introduced deliberately. Native
  root and alias directives establish new file-access boundaries that the
  administrator must secure separately.
- Replacing client-supplied forwarding headers prevents forged client identities
  from corrupting audit trails, IP-based access decisions and rate limits.
  Removing the Proxy request header prevents httpoxy-style proxy injection.
  Backends must accept identity headers only from this proxy; real-IP rewriting
  must trust only explicitly selected upstream proxies.
- Verifying the certificate chain and hostname of HTTPS backends prevents an
  intercepted or misdirected connection from silently reaching an impostor.
  Disabling upstream TLS session reuse prevents a session validated under one
  location's CA policy from carrying that trust into another location. New
  backend connections therefore incur a full TLS handshake.
- TLS 1.3 reduces exposure to legacy protocol and cipher constructions. The
  optional TLS 1.2 policy retains ephemeral key exchange and authenticated
  encryption, preserving forward secrecy and integrity instead of enabling
  obsolete compatibility suites. These protocol choices still require maintained
  nginx and TLS-library packages.
- Disabling TLS early data prevents replayable requests from reaching
  authentication or state-changing application endpoints. Limiting
  session-resumption lifetime bounds the reuse window for stolen resumption
  secrets; keeping resumption state server-side avoids dependence on the
  rotation of stateless ticket-encryption keys.
- HTTP/3 remains opt-in to avoid exposing an unnecessary UDP service. QUIC
  address validation makes clients demonstrate source-address reachability
  before nginx proceeds with the handshake, reducing spoofed-source
  amplification and resource-exhaustion opportunities. It does not authenticate
  clients or replace network-level denial-of-service protection.
- Disabling nginx response compression reduces compression side channels when
  responses combine secrets with attacker-controlled input. Compression already
  performed by a backend can preserve that exposure and must be assessed in the
  application as well.
- Body-size limits and idle timeouts constrain memory, disk and connection
  exhaustion by large or slow requests. Buffering helps decouple backend
  processing from slow clients. Disabling response buffering avoids nginx
  response spill files, but increases backend exposure to slow readers; neither
  mode replaces application limits or secret-safe logging.
- MIME sniffing restrictions reduce content-type confusion, framing restrictions
  mitigate clickjacking, and referrer restrictions reduce leakage of sensitive
  URL paths to other origins. Disabling obsolete browser XSS filters avoids
  their unsafe legacy behavior. Suppressing software-identification headers
  reduces passive fingerprinting, but does not conceal an unpatched service from
  active probes.
- A single owner for each response-header policy avoids conflicting edge and
  backend instructions. Applying the policy to error responses prevents failures
  from dropping browser protections. Native add_header, proxy_set_header and
  proxy_hide_header directives can replace their inherited directive sets;
  review the complete effective policy whenever an application overrides them.
- Application-specific CSP can limit script execution and framing without
  permitting broad sources that weaken XSS protection. Preserve the
  application's policy and verify that its frame-ancestors restriction matches
  the intended embedding boundary, because browsers can prefer it over
  X-Frame-Options. Generic permissive CSP is not a substitute for
  application-aware restrictions.
- HSTS makes returning browsers use HTTPS and refuse certificate errors,
  reducing downgrade opportunities after a successful HTTPS visit. Extending it
  to subdomains or preload affects services beyond one vhost and can make them
  inaccessible if HTTPS or renewal fails. COOP, CORP and Permissions-Policy can
  restrict cross-origin interaction and browser capabilities, but must preserve
  required login, embedding, camera and microphone workflows.
- Rate limits can slow brute-force attempts and protect expensive endpoints from
  request floods, but declaring a zone alone enforces nothing: the relevant
  routes must activate it. IP-based quotas group users behind NAT and may span
  several vhosts using one zone. Choose the quota scope and trusted client-IP
  source deliberately; rate limiting does not replace application
  authentication.
- Trusted resolvers reduce exposure to forged DNS answers and avoid sending
  internal backend names to an arbitrary public DNS service. Keep dynamic
  backend hostnames under administrator control so request input cannot turn the
  proxy into an SSRF path. HTTPS hostname and certificate verification remain
  necessary because resolver trust alone does not authenticate a backend.
- Restrictive configuration permissions and redacted rendering output reduce
  disclosure of credentials embedded in native vhosts. Inventory, backups and
  logs are separate disclosure paths; protect inventory secrets with Vault and
  restrict access to retained files. Native vhost text is privileged
  administrator input: a syntactically valid override can still weaken the
  security policy.
- Keeping SELinux enforcement active limits what a compromised nginx worker can
  do beyond its service permissions. Grant outbound-network booleans only when
  required by the application; broad network permission also broadens the
  worker's access to reachable services and should be paired with appropriate
  network restrictions.

## Operational Notes

- Every application listen address and port must have an equivalent entry in
  nginx_listen, including address-specific and IPv6 listeners. Configure TLS
  protocols globally because handshake selection begins in the default server.
  Named HTTPS upstream groups need proxy_ssl_name set to the certificate
  hostname; custom CAs can be selected through
  nginx_proxy_ssl_trusted_certificate.
- HTTP/2 is inherited by native vhosts when nginx_http2 is enabled; backend
  proxy connections continue to use HTTP/1.1. WebSocket upgrades require
  nginx_proxy_websockets. Disabling HTTP/2 omits the global directive; native
  overrides remain possible.
- HTTP/3 needs nginx_http3 enabled globally, a matching listen address:port quic
  in each application vhost, and reachable UDP through firewalls and load
  balancers. The role sets reuseport once per managed QUIC endpoint; omit it
  from application listeners. Advertise the actual external UDP port with
  Alt-Svc only on participating vhosts, alongside their complete response-header
  policy. Remove native QUIC listeners and Alt-Svc together when disabling
  HTTP/3. TLS 1.3 remains required.
- Distribution packages are used without additional repositories or nginx
  builds. The checked openSUSE Leap and Tumbleweed packages provide HTTP/2 but
  lack HTTP/3; use HTTP/3 only with a supporting package. Unsupported requested
  features fail native validation.
- A local response-header policy replaces inherited add_header directives. Most
  application examples repeat the baseline headers; Vaultwarden instead
  preserves backend headers and route-specific exceptions. Local
  proxy_hide_header overrides must repeat X-Powered-By and any security headers
  that the edge continues to own. Ensure that HSTS is emitted by only one layer.
- Rate-limit zones become active through limit_req in the selected server or
  location. A custom zone list replaces the defaults; an empty list declares no
  zones. Zone size is shared-memory capacity, not an upload limit. Use distinct
  zones or a key such as $server_name|$binary_remote_addr for independent
  per-vhost quotas. Changing a deployed key requires a new zone name and
  references for graceful reload. Do not redeclare role-managed zones in native
  configuration.
- Setting nginx_resolvers does not make a static proxy_pass hostname refresh
  dynamically. Variable-based proxy_pass supports runtime resolution on the
  role's baseline; named upstream server resolve requires nginx OSS 1.27.3 or
  newer and a shared upstream zone. Resolver entries support ports and bracketed
  IPv6 addresses; nginx_resolver_valid overrides DNS TTL only when explicitly
  configured.
- Application examples below are host/group variable fragments used with the
  baseline playbook. Combine their nginx_vhosts entries into one list when
  hosting multiple applications; enable WebSockets if any application requires
  them. The login examples share the global login zone; isolate their keys or
  zones if each application needs its own allowance. Each example uses existing
  certificates for an example.com subdomain and a backend reachable only
  locally. The explicit httpd_can_network_connect boolean permits proxy
  connections on enforcing SELinux hosts. Keep the role's default IPv4 listeners
  or add matching IPv6 entries to both the role and each vhost.
- Each nginx_vhosts entry requires a unique name and native config content.
  Removing an entry removes its configuration on the next run. All declared
  blocks are embedded in the main file.
- Native blocks are rendered in list order inside http and may contain upstream
  and map directives. Do not redefine the role-owned connection_upgrade map when
  WebSockets are enabled. Use absolute paths for native includes, certificates,
  and other file references so nginx -t checks the same files at runtime. No
  separate TLS or proxy include is needed: application servers inherit the
  http-level settings automatically.
- The role does not add location blocks for assets, dotfiles, favicon.ico or
  robots.txt; application routes stay with the backend. Native application
  vhosts must declare explicit server_name and listen directives without
  default_server.
- Configuration is validated with nginx -t -c against the exact candidate,
  including all referenced files, before atomic replacement. Changes to
  externally managed certificates or includes require their owner to validate
  and reload nginx.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Default-deny baseline

Install an nginx service that denies all requests until application vhosts are declared.

```yaml
---
- name: Configure nginx
  hosts: nginx
  gather_facts: true
  roles:
    - role: jomrr.nginx
```

### Optional TLS 1.2 compatibility

Enable TLS 1.2 for older Nextcloud WebDAV clients and HTTPS upstreams.

```yaml
nginx_protocols:
  - TLSv1.2
  - TLSv1.3
```

### Runtime DNS policy

Inventory settings for a trusted resolver in the deployment network.
Replace these documentation addresses with reachable DNS servers.
The explicit cache lifetime overrides DNS TTLs; omit it to respect them.
Native routing must use runtime resolution as described above.

```yaml
nginx_resolvers:
  - 192.0.2.53
  - '[2001:db8::53]'
nginx_resolver_valid: 60s
```

### Vaultwarden

Host or group variables for a local Vaultwarden backend on port 8000.
Set `DOMAIN=https://vaultwarden.example.com` and trust only the connecting proxy
for client identity.
WebSockets use the main HTTP port; no separate legacy notification port is
needed.
Vaultwarden owns CSP, framing, referrer, permissions and resource policies.
Preserve its route-specific exceptions for MFA connectors, icons and
WebSockets. The local proxy_hide_header resets inherited hide rules, and
the local add_header set adds only edge-owned HSTS.
See the [Vaultwarden header
implementation](https://github.com/dani-garcia/vaultwarden/blob/main/src/util.rs).
Token requests use the global login zone with a burst of 10; account for
multiple devices and users behind a shared client IP when choosing a rate.
The admin UI is blocked publicly; expose it separately through a restricted
management path if needed.
Access logging is disabled for notification URLs containing tokens; error logs
still need restricted access.
The 525 MiB body limit accommodates attachments; align application limits and
storage capacity.
Response buffering is off to avoid temporary response files; request
buffering remains on.
HTTP/2 is inherited from the global default. For optional HTTP/3, uncomment
nginx_http3 and both marked QUIC/Alt-Svc directives together; UDP/443 must
be reachable and nginx must include the HTTP/3 module. Keep these three
settings in sync when disabling HTTP/3 too.
See the [Vaultwarden proxy
examples](https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples)
and [hardening
guide](https://github.com/dani-garcia/vaultwarden/wiki/Hardening-Guide).

```yaml
nginx_proxy_websockets: true
# Optional HTTP/3: enable together with the marked vhost directives.
# nginx_http3: true
nginx_sebooleans:
  httpd_can_network_connect: true
nginx_vhosts:
  - name: vaultwarden.example.com
    config: |-
      server {
          listen 80;
          server_name vaultwarden.example.com;
          return 308 https://vaultwarden.example.com$request_uri;
      }
      server {
          listen 443 ssl;
          server_name vaultwarden.example.com;
          # Optional HTTP/3:
          # listen 443 quic;
          ssl_certificate /etc/acme/vaultwarden.example.com/fullchain.pem;
          ssl_certificate_key /etc/acme/vaultwarden.example.com/key.pem;
          client_max_body_size 525m;
          proxy_buffering off;
          # Preserve backend headers and their route-specific exceptions.
          proxy_hide_header X-Powered-By;
          add_header Strict-Transport-Security "max-age=31536000" always;
          # Optional HTTP/3:
          # add_header Alt-Svc 'h3=":443"; ma=86400' always;
          location ~ ^/admin(?:/|$) { return 404; }
          location ~ ^/\.(?!well-known(?:/|$)) { return 404; }
          location = /identity/connect/token {
              limit_req zone=login burst=10 nodelay;
              proxy_pass http://127.0.0.1:8000;
          }
          location /notifications/ {
              access_log off;
              proxy_read_timeout 3600s;
              proxy_pass http://127.0.0.1:8000;
          }
          location / {
              proxy_pass http://127.0.0.1:8000;
          }
      }
```

### Forgejo

Host or group variables for Forgejo on loopback port 3000.
Set [server] `ROOT_URL=https://forgejo.example.com/` and [security]
`REVERSE_PROXY_LIMIT=1`,
`REVERSE_PROXY_TRUSTED_PROXIES=127.0.0.1/32` for this topology. Leave
proxy-header authentication disabled.
Preserve encoded repository paths with merge_slashes off and proxy_pass without
a URI suffix.
The example limits login requests to 10/minute per client IP with a burst of 10
and returns 429 on excess traffic.
Increase the 512 MiB upload limit for larger LFS or package objects; Git SSH
needs separate service configuration.
Response buffering is disabled for event streams; request buffering stays
enabled.
See [Forgejo reverse proxy
setup](https://forgejo.org/docs/latest/admin/setup/reverse-proxy/)
and [proxy trust
settings](https://forgejo.org/docs/latest/admin/config-cheat-sheet/).

```yaml
nginx_proxy_websockets: true
nginx_sebooleans:
  httpd_can_network_connect: true
nginx_vhosts:
  - name: forgejo.example.com
    config: |-
      server {
          listen 80;
          server_name forgejo.example.com;
          return 308 https://forgejo.example.com$request_uri;
      }
      server {
          listen 443 ssl;
          server_name forgejo.example.com;
          ssl_certificate /etc/acme/forgejo.example.com/fullchain.pem;
          ssl_certificate_key /etc/acme/forgejo.example.com/key.pem;
          merge_slashes off;
          client_max_body_size 512m;
          proxy_read_timeout 300s;
          proxy_buffering off;
          add_header X-Content-Type-Options nosniff always;
          add_header Referrer-Policy strict-origin-when-cross-origin always;
          add_header X-Frame-Options DENY always;
          add_header X-XSS-Protection "0" always;
          add_header Strict-Transport-Security "max-age=31536000" always;
          add_header Content-Security-Policy "frame-ancestors 'none'" always;
          add_header Permissions-Policy
              "camera=(), microphone=(), geolocation=()" always;
          location ~ ^/\.(?!well-known(?:/|$)) { return 404; }
          location = /user/login {
              limit_req zone=login burst=10 nodelay;
              proxy_pass http://127.0.0.1:3000;
          }
          location / {
              proxy_pass http://127.0.0.1:3000;
          }
      }
```

### NetBox

Host or group variables for local NetBox Gunicorn on port 8001.
Set `ALLOWED_HOSTS=['netbox.example.com']` and configure HTTPS/CSRF origin
settings for that hostname.
Only the collected static directory is exposed through an explicit alias; grant
nginx read access and the required SELinux context.
If NetBox runs on another host, proxy to its complete HTTP frontend that serves
static assets instead of to bare Gunicorn.
A 25 MiB request limit follows the application's example and should match
import/upload requirements.
The added CSP only restricts framing; any backend CSP remains effective as an
additional policy.
See the [NetBox nginx
configuration](https://github.com/netbox-community/netbox/blob/main/contrib/nginx.conf).

```yaml
nginx_sebooleans:
  httpd_can_network_connect: true
nginx_vhosts:
  - name: netbox.example.com
    config: |-
      server {
          listen 80;
          server_name netbox.example.com;
          return 308 https://netbox.example.com$request_uri;
      }
      server {
          listen 443 ssl;
          server_name netbox.example.com;
          ssl_certificate /etc/acme/netbox.example.com/fullchain.pem;
          ssl_certificate_key /etc/acme/netbox.example.com/key.pem;
          client_max_body_size 25m;
          add_header X-Content-Type-Options nosniff always;
          add_header Referrer-Policy strict-origin-when-cross-origin always;
          add_header X-Frame-Options DENY always;
          add_header X-XSS-Protection "0" always;
          add_header Strict-Transport-Security "max-age=31536000" always;
          add_header Content-Security-Policy "frame-ancestors 'none'" always;
          add_header Permissions-Policy
              "camera=(), microphone=(), geolocation=()" always;
          location ~ /\.(?!well-known(?:/|$)) { return 404; }
          location /static/ {
              alias /opt/netbox/netbox/static/;
          }
          location / {
              proxy_pass http://127.0.0.1:8001;
          }
      }
```

### Nextcloud

Host or group variables for a complete Nextcloud HTTP frontend on loopback port
8080, including PHP and static assets.
Set `trusted_domains=['nextcloud.example.com']`,
`trusted_proxies=['127.0.0.1']`, `overwriteprotocol='https'`
and `overwrite.cli.url='https://nextcloud.example.com'` in config.php; trust the
actual peer address in container deployments.
Keep the application's CSP and same-origin framing for app integrations.
Camera/microphone permissions remain application-owned for Talk.
This example streams request bodies up to 10 GiB and allows 300-second idle
periods; align PHP, backend, quota and storage limits.
HTTP discovery redirects cover DAV, WebFinger and NodeInfo; ACME challenges
remain externally managed.
Only root-level hidden paths are blocked: a blanket dotfile restriction would
break hidden files accessed through WebDAV.
All WebDAV methods reach the backend. Talk signaling, notify_push and office
integrations need their own routes when deployed.
See [Nextcloud proxy
configuration](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/reverse_proxy_configuration.html).

```yaml
nginx_sebooleans:
  httpd_can_network_connect: true
nginx_vhosts:
  - name: nextcloud.example.com
    config: |-
      server {
          listen 80;
          server_name nextcloud.example.com;
          return 308 https://nextcloud.example.com$request_uri;
      }
      server {
          listen 443 ssl;
          server_name nextcloud.example.com;
          ssl_certificate /etc/acme/nextcloud.example.com/fullchain.pem;
          ssl_certificate_key /etc/acme/nextcloud.example.com/key.pem;
          client_max_body_size 10g;
          client_body_timeout 300s;
          proxy_request_buffering off;
          proxy_send_timeout 300s;
          proxy_read_timeout 300s;
          add_header X-Content-Type-Options nosniff always;
          add_header Referrer-Policy strict-origin-when-cross-origin always;
          add_header X-Frame-Options SAMEORIGIN always;
          add_header X-XSS-Protection "0" always;
          add_header Strict-Transport-Security "max-age=31536000" always;
          location = /.well-known/carddav {
              return 301 https://nextcloud.example.com/remote.php/dav/;
          }
          location = /.well-known/caldav {
              return 301 https://nextcloud.example.com/remote.php/dav/;
          }
          location = /.well-known/webfinger {
              return 301 https://nextcloud.example.com/index.php/.well-known/webfinger$is_args$args;
          }
          location = /.well-known/nodeinfo {
              return 301 https://nextcloud.example.com/index.php/.well-known/nodeinfo$is_args$args;
          }
          location ~ ^/\.(?!well-known(?:/|$)) { return 404; }
          location / {
              proxy_pass http://127.0.0.1:8080;
          }
      }
```

## References

- [nginx HTTP/2](https://nginx.org/en/docs/http/ngx_http_v2_module.html)
- [nginx HTTP/3](https://nginx.org/en/docs/http/ngx_http_v3_module.html)
- [nginx proxy directives and header inheritance](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)
- [nginx TLS and handshake rejection](https://nginx.org/en/docs/http/ngx_http_ssl_module.html)
- [nginx response header inheritance](https://nginx.org/en/docs/http/ngx_http_headers_module.html)
- [nginx runtime DNS resolver](https://nginx.org/en/docs/http/ngx_http_core_module.html#resolver)
- [nginx dynamic upstream resolution](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#resolve)
- [nginx request rate limiting](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2024-2026 Jonas Mauer.
