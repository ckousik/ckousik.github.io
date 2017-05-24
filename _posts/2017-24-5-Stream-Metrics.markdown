---
layout: post
title: "Stream Metrics"
date: 2017-5-24 20:00:00 +530
categories: gsoc
---

In the previous post, I gave a brief explanation of how the stream has been refactored to resemble a finite state machine. This post elaborates on the performance metrics of streams based on buffer size and number of concurrent streams.
### Buffers
Each stream has an internal buffer which is used to store data which is sent to it from the remote side. The default buffer size is 1024 bytes as of now. The buffer size is immutable and cannot be changed once the stream has been created. The buffer is internally implemented as a circular queue of bytes, and implements `io.ReadWriter`. When a stream is created, the stream assumes the remote buffer capacity to be zero. When the stream is accepted, the remote connection informs the stream of its buffer size and the remote buffer capacity is updated. Streams are setup to track remote capacity, unblocking bytes when an ACK frame arrives, and reducing remote capacity when a certain number of bytes are written. Streams can only write as many bytes as remote capacity, and will block writes until further bytes are unblocked. Thus, buffer size has a significant effect on performance.<br>
The following plot shows the time taken for a `Write()` call over 100 concurrent streams as a function of buffer size.

<iframe width="900" height="800" frameborder="0" scrolling="no" src="//plot.ly/~ckousik/1.embed"></iframe>

1500 bytes are sent and echoed back over each stream. It is clear that the time taken reduces exponentially as a function of buffer size. This is because smaller buffers require more messages to be sent over the websocket connection. A stream with a 1024 byte buffer needs to exchange a minimum of 3 messages for the data to be completely sent to the remote connection: write 1024 bytes, receive ACK >= 476 bytes, write 476 bytes. A stream with a large enough buffer can write data using a single message. The intended buffer size is 4k.

### Concurrent Streams
Each session is capabale of handling concurrent streams. This test keeps the buffer size constant as 1024 bytes and varies the number of concurrent streams.<br>
The following plot describes the time taken to echo 1500 bytes over each stream with a buffer size of 1k as a function of number of concurrent streams:

<iframe width="900" height="800" frameborder="0" scrolling="no" src="//plot.ly/~ckousik/3.embed"></iframe>

It is simple to fit a quadratic curve to this curve. A reason for this could be a limit on the throughput of the websocket connection.
