---
layout: post
title: "WebSocket Multiplexer Overview"
date: 2017-6-16 20:00:00 +530
categories: gsoc
---
### General Idea
WebSocket multpilexer enables creation of multiple tcp-like streams over a WebSocket connection. Since each stream can be treated as a separate `net.Conn` instance, it is used by other components to proxy HTTP requests. A new stream can be opened for each request, and they can be handled in a manner identical to tcp streams. Wsmux contains two components: Sessions and Streams. Sessions wrap WebSocket connections and allow management of streams over the connection. Session implements `net.Listener`, and can be used by an `http.Server` instance to serve requests. Streams are the interface which allow users to send and receive multiplexed data. Streams implement `net.Conn`. Streams have internal mechanisms for buffering and congestion control.

### Why WebSocket?
The decision to use WebSocket (`github.com/gorilla/websocket`) instead of supporting a `net.Conn` was made for the following reasons:
- WebSocket handshakes can be used for intitiating a connection instead of writing a custom handshake. Wsmux can be used as a subprotocol in the WebSocket handshake. This greatly simplifies the implementation of wsmux.
- WebSocket convenience mathods (`ReadMessage` and `WriteMessage`) simplify sending and receiving of data and control frames.
- Control messages such as ping, pong, and close need not be implemented separately in wsmux. WebSocket control frames can be used for this purpose.
- Adding another layer of abstraction over WebSocket enables connections to be half-closed. WebSocket does not allow for half closed connections, but wsmux streams can be half closed, thus simplifying usage.
- Since WebSocket frames already contain the length of the message, the length field can be dropped from wsmux frames. This reduces the size of the wsmux header to 5 bytes.

### Framing
WebSocket multiplexer implements a very simple framing technique. The total header size is 5 bytes. 
```
[ message type - 8 bits ][ stream ID - 32 bits ]
```
Messages can have the following types:
- msgSYN: Used to initiate a connection.
- msgACK: Used to acknowledge bytes read on a connection.
- msgDAT: Signals that data is being sent.
- msgFIN: Signals stream closure.

### Sessions
A Session wraps a WebSocket connection and enables usage of the wsmux subprotocol. A Session is typically used to manage the various wsmux streams. Sessions are of two types:Server, Client. The only difference between a Server Session and a Client Session is that the ID of a stream created by a Server Session will be even numbered while the ID of a stream created by a Client will be odd numbered. A connection must have only one Server and one Client. Sessions read messages from the WebSocket connection and forward the data to the appropriate stream. Streams are responsible for buffering and framing of data. Streams must send data by invoking the `send` method of their Session.

### Streams
Streams allow users to interface with data tagged with a particular ID, called the stream ID. Streams contain circular buffers for congestion control and are also responsible for generating and sending msgACK frames to the remote session whenever data is read. Streams handle framing of data when data is being sent, and also allow setting of deadlines for `Read` and `Write` calls. Internally, streams are represented using a Finite State Machine, which has been described in a [previous blog post](https://ckousik.github.io/gsoc/2017/05/16/Stream-States-Part-1.html). Performance metrics for streams have also been measured and are availabe [here](https://ckousik.github.io/gsoc/2017/05/24/Stream-Metrics.html).

### Conclusion
Wsmux is being used by the two major components of Webhooktunnel: Webhook Proxy, and Webhook Client. It has been demonstrated that wsmux can be used to multiplex HTTP requests reliably, and can also support WebSocket over wsmux streams.

The repository can be found [here](https://github.com/taskcluster/webhooktunnel/tree/master/wsmux).
