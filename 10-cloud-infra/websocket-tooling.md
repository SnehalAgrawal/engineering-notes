# Cloud Architecture: WebSocket Libraries & Testing Tools

### 1. Overview
A WebSocket is a long-lived, two-way connection between client and server (RFC 6455). It starts as a normal HTTP/1.1 request with an `Upgrade: websocket` header. This note compares the three common JavaScript options and shows how to test an endpoint locally. For proxying through NGINX, see [nginx.md](nginx.md).

### 2. WebSocket API vs. `ws` vs. Socket.IO

| | Browser `WebSocket` API | `ws` package | Socket.IO |
|---|---|---|---|
| What it is | Client built into browsers | WebSocket server and client for Node.js | Its own protocol and library on top of WebSocket |
| Fallback | None | None | HTTP long-polling if WebSocket fails |
| Extras | None | None | Reconnection, rooms, acknowledgements, broadcasting |
| Talks to | Any WebSocket server | Any WebSocket client | Only Socket.IO clients and servers |

*   **No mixing**: A Socket.IO client cannot connect to a plain WebSocket server, and a plain WebSocket client cannot use a Socket.IO server.
*   **Browser limits**: The browser API cannot set custom headers, so auth uses a cookie, a short-lived token in the URL, or a first "auth" message. Reconnection is up to you.
*   **Node.js client**: Node.js 22 and later have a built-in `WebSocket` client.
*   **Which to pick**: Use plain WebSocket (with `ws` on the server) when you want a thin protocol or have non-JavaScript clients. Use Socket.IO when you want rooms and reconnection built in and can use its libraries on both ends.
*   **Scaling Socket.IO**: With several servers, add an adapter (such as the Redis adapter) so a broadcast reaches clients on every server. Long-polling also needs sticky sessions.

### 3. Testing Tools
`wscat` is a command-line WebSocket client from the `ws` maintainers.

```bash
npm install -g wscat
wscat -c ws://localhost:8080                       # type messages, see replies
wscat -c ws://localhost:8080 -x '{"event":"ping"}' -w 1   # send one message, wait 1s, exit
wscat -c wss://example.com/socket -H "Authorization: Bearer <token>"
wscat -l 8080                                      # act as a server for a client you are testing
```

A local echo server with `ws`, for testing a client or a proxy:

```js
// echo-server.mjs   (npm install ws, then: node echo-server.mjs)
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ port: 8080 });
wss.on('connection', (socket) => {
  socket.on('message', (data, isBinary) => socket.send(data, { binary: isBinary }));
});
```

```text
$ wscat -c ws://localhost:8080
Connected (press CTRL+C to quit)
> {"event":"ping"}
< {"event":"ping"}
```

*   **Browser DevTools**: The Network tab (filter "WS") shows the handshake and every message.

### 4. Common Mistakes
*   **Testing Socket.IO with `wscat`**: You get rejected or see raw Engine.IO packets. Use a Socket.IO client.
*   **Idle timeouts**: Proxies and load balancers close silent connections. Send pings more often than the timeout.
*   **No reconnect**: Every deploy drops open connections. Clients should reconnect with random backoff so they do not all return at once.
*   **`ws://` in production**: Use `wss://` so traffic is encrypted.

### Further Reading
*   [wscat](https://github.com/websockets/wscat)
*   [ws](https://github.com/websockets/ws)
*   [Socket.IO documentation](https://socket.io/docs/v4/)
*   [RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455)
