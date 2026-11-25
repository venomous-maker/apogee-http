# Changelog

Every release of apogee-http, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

- `Request.matched`: the route the router matched (`MatchedRoute`: its pattern,
  name, methods, action and middleware), and `routeUri()`, its pattern
  ("/users/{id}"), for metrics and logs to group requests by.

## 0.2.0 (2026-11-20)

### Requirements

- Needs grav 1.9.0 or later, and uses the 0.2.0 releases of the packages it
  depends on.

## 0.1.0 (2026-11-19)

The first tagged release. Needs grav 1.7.0 or newer.

The HTTP layer of [Apogee](https://github.com/venomous-maker): `Request`, `Response`, HTTP errors, the middleware contract and pipeline, API resources with pagination, signed cookies, uploads, and a JSON writer. It sits on the standard library's HTTP server (`std/net`), which keeps the transport: event loops, keep-alive, TLS, compression.
