# Changelog

Every release of apogee-broadcast, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.2.1 (2026-11-27)

### Fixes

- Database connections that the server dropped while idle (MySQL's `wait_timeout`, a restart, a proxy's idle limit) reconnect on their next use instead of failing with "server has gone away" (grav-db 0.2.1). Uses the new releases of apogee-package, apogee-sockets.

## 0.2.0 (2026-11-25)

Needs grav 1.12.0 or newer.

- `BroadcastPackage`: the module connects itself to an apogee-framework app (apogee-package):
  its settings, wiring moved here from the framework.

### Dependencies

- Uses apogee-package 0.1.0, apogee-routing 0.3.0, apogee-http 0.3.0, apogee-sockets 0.2.0 (from their GitHub tags).

## 0.1.1 (2026-11-21)

### Performance

- `SocketsBroadcaster` sends from a background thread by default: dispatching
  an event waits about 4 µs instead of the HTTP request's 100 µs (in order;
  failures go to `onError`). `flush()` and `Broadcast.flush()` wait until
  everything queued is sent; `new SocketsBroadcaster(..., background: false)`
  sends before returning, as before.
- Requests from several threads no longer queue behind one HTTP client: a small
  pool of keep-alive clients serves them.

## 0.1.0 (2026-11-20)

The first release. Needs grav 1.10.0 or newer.

- `BroadcastEvent` and `Broadcast.listenTo(events)`: dispatched events also go
  to browsers; `toOthers(req)` leaves out the sender's connection.
- Channel rules with parameters, and `Broadcast.auth`, the endpoint that signs
  private and presence subscriptions.
- `SocketsBroadcaster` (the Pusher HTTP API: apogee-sockets and compatible
  servers), `LogBroadcaster`, `NullBroadcaster`, and `Broadcast.fake()`.
