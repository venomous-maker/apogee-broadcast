# Changelog

Every release of apogee-broadcast, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

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
