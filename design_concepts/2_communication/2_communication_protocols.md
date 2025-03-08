## Communication Protocols
- Communication protocols are used for communication
  - Between a server and a client (like web browser)
- Defines rules, syntax, semantics and synchronization of communication
  - And possible error recovery methods
- Types of communication: synchronous, aynchronous

## Polling (or Short Polling)
- Client repeatedly polls (requests) server for data, at regular intervals using HTTP
- It waits for the server to respond with data
- If no data is available, an empty request is returned
  - Which may accumulate to create HTTP overhead

## Long Polling
- Also known as hanging GET
- Variation of polling which allows server to push information to client
  - Only when the data is available
- Client sends request over HTTP to server and waits
- Server holds request instead of sending empty response
  - Sends response when data is available
- Client re-requests server (polls) after getting response
  - So that server will almost always have an available waiting request
  - That it can use to deliver data in response to an event
- Each long-poll request has a timeout
  - Client has to reconnect periodically after the connection is closed due to timeouts

## Web Socket
- Full duplex communication (send & receive) over a single TCP connection
  - Client establishes connection through 3-way handshake
  - Client sends request, server sends handshake, websocket connection is established
- Stateful protocol
  - Persistent connection between client and server to send data any time
  - The connection is not terminated until either the client of server disconnects
- Real-time communication without waiting for a response
  - Allows message passing back and forth while keeping connection open
  - Server sends content to client without being asked

## SSE (Server Sent Events)
- Only server sends data on this connection
- Client establishes a persistent and long-term connection
- If client wants to send data, it would require another protocol
- Usage: Get real-time traffic, server is generating data in loop
