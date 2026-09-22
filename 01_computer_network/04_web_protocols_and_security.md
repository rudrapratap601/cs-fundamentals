# 4. Web Protocols and Security

[Index](README.md) · [Previous](03_transport_protocols.md) · [Next](05_interview_practice.md)

## HTTP semantics — core

HTTP defines requests and responses. Requests have a method, target, headers, and sometimes content; responses have status, headers, and possibly content. HTTP is stateless in that request semantics can be understood independently; applications add sessions and other state.

| Method | Typical use | Safe | Idempotent by semantics |
|---|---|---|---|
| GET | Retrieve representation | Yes | Yes |
| HEAD | Retrieve metadata without response content | Yes | Yes |
| POST | Submit for processing | No | Not guaranteed |
| PUT | Create/replace target state | No | Yes |
| PATCH | Apply partial modification | No | Not guaranteed |
| DELETE | Remove target association | No | Yes |

Safe means the client does not request state change; incidental logging may occur. Idempotent means repeated identical requests have the same intended effect as one request, not identical responses. A repeated DELETE may return a different status.

| Code | Meaning |
|---|---|
| 200 / 201 / 204 | Success / created / success without response content |
| 301 / 308 | Permanent redirect; 308 preserves method |
| 302 / 307 | Temporary redirect; 307 preserves method |
| 304 | Reuse cached representation after validation |
| 400 | Invalid request |
| 401 / 403 | Missing/invalid credentials / request refused |
| 404 / 409 / 429 | Not found / state conflict / too many requests |
| 500 | Internal server error |
| 502 / 503 / 504 | Bad upstream response / unavailable / upstream timeout |

## HTTP versions

| Version | Key distinction |
|---|---|
| HTTP/1.1 | Persistent connections; conventional textual message format; pipelined responses remain ordered |
| HTTP/2 | Binary framing, multiplexed streams over TCP, header compression |
| HTTP/3 | HTTP over QUIC, which uses UDP and integrates TLS security |

HTTP/2 multiplexes requests, but TCP loss can block all streams on that connection. QUIC offers per-stream ordering, so missing stream data need not block delivery on unrelated streams; shared congestion/resources can still affect them.

## HTTPS and TLS

TLS provides confidentiality, integrity, and peer authentication when validation is performed correctly. In a simplified certificate-based handshake, peers negotiate parameters, exchange key-agreement information, the server presents a certificate and proves private-key possession, the client validates its chain/hostname/validity, and both derive symmetric traffic keys.

Details depend on TLS version and resumption. Modern TLS does not simply encrypt every byte with the server's public key. A certificate binds identity to a public key; it does not prove the business is trustworthy. HTTPS does not fix application authorization bugs or SQL injection.

## Cookies, sessions, and browser security

- **Cookie:** browser-managed data sent with matching requests according to cookie rules.
- **Session:** application state associated with a client, often stored server-side and referenced through a cookie.
- **Token:** credential or claims container. Signing a JWT does not encrypt its contents.
- **Secure:** cookie restricted to secure transport, subject to browser rules.
- **HttpOnly:** prevents ordinary client-side JavaScript access to a cookie.
- **SameSite:** constrains cross-site cookie sending and helps mitigate some CSRF risks.

Authentication asks who the caller is; authorization asks what they may do. CORS controls browser-script access to cross-origin responses. It is not authentication and does not block arbitrary non-browser clients.

## Caches and intermediaries

`Cache-Control: max-age` gives a freshness lifetime. `no-cache` permits storage but requires validation before reuse. `no-store` instructs caches not to store the response. With an `ETag`, a client can send `If-None-Match`; an unchanged representation can produce `304`.

| Component | Purpose |
|---|---|
| Forward proxy | Makes outbound requests for clients |
| Reverse proxy | Fronts servers; can route, cache, or terminate TLS |
| CDN | Distributed content delivery, often closer to users |
| Load balancer | Distributes traffic across healthy backends |

## What happens when you enter an HTTPS URL?

1. Browser parses the URL and checks relevant caches/local state.
2. DNS resolves the host if necessary.
3. Host selects a route and may resolve the next-hop MAC with ARP on Ethernet/IPv4.
4. A connection is established or reused: commonly TCP plus TLS, or QUIC for HTTP/3.
5. Browser sends an HTTP request, including applicable headers/cookies.
6. Intermediaries route to an application, which may query a database.
7. The response returns; the browser processes caching, parses content, fetches subresources, and renders.

Caches and reused connections can skip steps; a fresh DNS lookup/handshake is not necessary on every request.

## Interview answers

**HTTP versus HTTPS?** HTTPS adds TLS-based transport security while retaining HTTP application semantics.

**Cookie versus session?** A cookie is a browser storage/transport mechanism; a session is application state. A cookie often holds a session identifier.

**Why can retries create duplicates?** A server may commit an operation but lose the response. An application idempotency key can help recognize repeated requests.
