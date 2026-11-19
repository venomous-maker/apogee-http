# Changelog

Every release of apogee-http, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

The first tagged release. Needs grav 1.7.0 or newer.

The HTTP layer of [Apogee](https://github.com/venomous-maker): `Request`, `Response`, HTTP errors, the middleware contract and pipeline, API resources with pagination, signed cookies, uploads, and a JSON writer. It sits on the standard library's HTTP server (`std/net`), which keeps the transport: event loops, keep-alive, TLS, compression.
