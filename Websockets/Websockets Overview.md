## Overview
WebSockets provide full-duplex, bidirectional communication between client and server over a single TCP connection.

## Key Concepts
- Persistent connection
- Real-time data transfer
- Reduced latency compared to HTTP polling
- Supported by modern browsers

## WebSocket Handshake
1. Client sends HTTP upgrade request
2. Server responds with HTTP 101 (Switching Protocols)
3. WebSocket connection established

## JavaScript Client-side API

### Creating a WebSocket
```javascript
const socket = new WebSocket('ws://example.com/socketserver');
```

### Event Listeners
```javascript
socket.onopen = (event) => {
  console.log('Connection opened');
};

socket.onmessage = (event) => {
  console.log('Message received:', event.data);
};

socket.onerror = (error) => {
  console.error('WebSocket error:', error);
};

socket.onclose = (event) => {
  console.log('Connection closed. Code:', event.code, 'Reason:', event.reason);
};
```

### Sending Messages
```javascript
socket.send('Hello, server!');
```

### Closing the Connection
```javascript
socket.close();
```

## Server-side Implementation (Node.js with 'ws' library)

### Installation
```bash
npm install ws
```

### Basic Server Setup
```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    console.log('Received:', message);
    ws.send('Server received: ' + message);
  });

  ws.send('Welcome to the WebSocket server!');
});
```
Certainly! I'll add a section comparing WebSocket (ws) and WebSocket Secure (wss) to the cheatsheet. Here's the updated version with the new section:
## WebSocket (ws) vs WebSocket Secure (wss)

### WebSocket (ws)
- Uses unencrypted connection (similar to HTTP)
- Default port: 80
- URL scheme: `ws://`
- Faster, but less secure
- Suitable for development or non-sensitive data

### WebSocket Secure (wss)
- Uses encrypted connection over TLS/SSL (similar to HTTPS)
- Default port: 443
- URL scheme: `wss://`
- More secure, but slightly higher latency
- Recommended for production and sensitive data transmission

### Key Differences
```
Feature        | ws                | wss
---------------|-------------------|------------------
Encryption     | None              | TLS/SSL
Security       | Low               | High
Performance    | Slightly faster   | Slight overhead
Production use | Not recommended   | Recommended
Firewall       | May be blocked    | Usually allowed
```

### Switching from ws to wss
Client-side:
```javascript
// Change this:
const socket = new WebSocket('ws://example.com/socketserver');

// To this:
const socket = new WebSocket('wss://example.com/socketserver');
```

Server-side (Node.js with 'ws' and 'https' modules):
```javascript
const https = require('https');
const WebSocket = require('ws');
const fs = require('fs');

const server = https.createServer({
  cert: fs.readFileSync('/path/to/cert.pem'),
  key: fs.readFileSync('/path/to/key.pem')
});

const wss = new WebSocket.Server({ server });

server.listen(443);
```

### Considerations
- Always use wss:// in production environments
- Ensure proper SSL/TLS certificate configuration on the server
- Update firewall rules if necessary when switching to wss
- Some legacy systems may not support wss, but this is rare in modern environments
## WebSocket Protocol

### Frame Format
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127  |
+ - - - - - - - - - - - - - - - +-------------------------------+
|                               |Masking-key, if MASK set to 1  |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------- - - - - - - - - - - - - - - - +
:                     Payload Data continued ...                :
+ - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
|                     Payload Data continued ...                |
+---------------------------------------------------------------+
```

## Key Points and Gotchas
- Always handle connection errors and closures gracefully
- Implement reconnection logic for robustness
- Be aware of browser limitations (e.g., max connections)
- Secure WebSockets (wss://) are recommended for production
- Large messages may need to be fragmented
- Consider heartbeat mechanisms to keep connections alive
- Be cautious with shared/global variables in multi-client scenarios

## Related Topics
- Socket.IO (abstraction layer over WebSockets)
- HTTP/2 Server Push
- Server-Sent Events (SSE)

Remember, this cheatsheet provides a quick overview. For production use, always refer to the official documentation and consider security best practices.