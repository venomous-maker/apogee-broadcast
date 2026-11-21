# apogee-broadcast

Events sent to browsers as they happen, for [Apogee](https://github.com/venomous-maker)
apps: an event the app dispatches is also pushed over WebSockets to the pages
listening on its channels, through
[apogee-sockets](https://github.com/venomous-maker/apogee-sockets) or another
server speaking the Pusher Channels protocol. Private and presence channels
are authorized by rules in the app.

```grav
import "std"
import "apogee-broadcast"
use apogee.*

class OrderShipped extends BroadcastEvent {
    public order: Order
    constructor(o: Order) { self.order = o }
    fn broadcastOn() -> Array<string> { return ["private-orders." + str(self.order.userId)] }
    fn broadcastWith() -> HashMap<string, any> { return {"id": self.order.id, "status": "shipped"} }
}

Broadcast.use(new SocketsBroadcaster("http://127.0.0.1:6001", "app-id", "app-key", "app-secret"))
Broadcast.listenTo(events)                 // dispatching a BroadcastEvent broadcasts it
Broadcast.channel("orders.{userId}", fn(user: any, p: HashMap<string, string>) -> any {
    return (user as User).id == Integer.parseOr(p.get("userId"), 0)
})
router.post("/broadcasting/auth", fn(req: Request) -> Response { return Broadcast.auth(req) }).middleware(["auth"])

events.dispatch(new OrderShipped(order))
```

```js
// the browser (Laravel Echo or pusher-js)
const pusher = new Pusher("app-key", { wsHost: location.hostname, wsPort: 6001, forceTLS: false,
                                       enabledTransports: ["ws", "wss"], cluster: "mt1",
                                       authEndpoint: "/broadcasting/auth" })
pusher.subscribe("private-orders.7").bind("OrderShipped", (e) => show(e.status))
```

## Events

Extend `BroadcastEvent` and say:

| | |
|---|---|
| `broadcastOn()` | the channels: `"orders"` (public), `"private-orders.7"`, `"presence-chat.1"` |
| `broadcastWith()` | the data the browsers get |
| `broadcastAs()` | the event name they listen for (the class name by default) |
| `broadcastWhen()` | whether to send it at all |
| `toOthers(req)` | leave out the connection whose request caused it (its `X-Socket-ID` header) |

`Broadcast.listenTo(events)` makes every `BroadcastEvent` dispatched through
apogee-events a broadcast too; `Broadcast.event(e)` sends one directly, and
`Broadcast.send(channels, name, data)` sends one that is not a class.

## Channels

`Broadcast.channel(pattern, authorize)`: `orders.{userId}` matches
`private-orders.7` and `presence-orders.7` (`*` matches any one segment).
`authorize(user, params)` gets the signed-in user and the pattern's
parameters. For a private channel it returns a bool; for a presence channel,
the member's info as a `HashMap<string, any>` with an `"id"` (or null to
refuse).

`Broadcast.auth(req)` is the `/broadcasting/auth` endpoint the browser
clients call (`socket_id` and `channel_name` in the request, the user from
`req.user`): it answers with the signature, and the presence `channel_data`,
or 403.

## Broadcasters

| | |
|---|---|
| `SocketsBroadcaster(baseUrl, appId, key, secret, background = true)` | the Pusher HTTP API: apogee-sockets, or another server that speaks it; also gives `Broadcast.auth` its key and secret. Events go out from a background thread, in order, so dispatching does not wait for HTTP (failures go to its `onError`); `Broadcast.flush()` waits until they are sent (call it before a short-lived process exits). With `background` false, each is sent before `broadcast` returns. |
| `LogBroadcaster(fn(string))` | each event as a line |
| `NullBroadcaster` | nothing |
| `Broadcast.fake()` | a `FakeBroadcaster` that keeps what it is sent: `has(event, channel)`, `broadcasted(event)`, `count()` |

`Broadcast.credentials(key, secret)` sets the signing key and secret when the
broadcaster is not a `SocketsBroadcaster`.

## Tests

```bash
grav test tests/broadcast.grav
```

They run an apogee-sockets server and a WebSocket client in the same process.
