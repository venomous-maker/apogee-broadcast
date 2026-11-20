# Changelog

Every release of apogee-broadcast, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.1.0 (2026-11-20)

The first release. Needs grav 1.10.0 or newer.

- `BroadcastEvent` and `Broadcast.listenTo(events)`: dispatched events also go
  to browsers; `toOthers(req)` leaves out the sender's connection.
- Channel rules with parameters, and `Broadcast.auth`, the endpoint that signs
  private and presence subscriptions.
- `SocketsBroadcaster` (the Pusher HTTP API: apogee-sockets and compatible
  servers), `LogBroadcaster`, `NullBroadcaster`, and `Broadcast.fake()`.
