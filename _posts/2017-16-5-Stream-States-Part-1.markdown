---
layout: post
title: "Stream States Part 1"
date: 2017-5-16 22:30:00 +530
categories: gsoc
---
Streams can be modelled as an FSM by determining the different states a stream can be in and all valid state transistions. Initially, a stream is in the `created` state.
This state signifies that the stream has been created. This is possible in two different ways: a) The stream was created by the session, added to the stream map, and a SYN
packet with the stream's ID was sent to the remote session, or b) A SYN packet was received from the remote session and a new stream was created and added to the stream map.
In case of (a) the stream waits for an ACK packet from the remote session and as soon as the ACK packet arrives, it transistions to the `accepted` state. In case of (b) the 
session sends an ACK packet, and the the stream transitions to the `accepted` state. 

Once in the `accepted` state the stream can read and write from the stream. When a DAT packet arrives, the data is push to the stream's buffer. When data is read out of the
buffer using a `Read()` call, an ACK packet is sent to the remote stream with the number of bytes read. When an ACK packet is received in the accepted state, the number of 
bytes unblocked (the number of bytes the remote session is willing to accept), is updated. If the stream is closed by a call to `Close()`, then the stream transitions to the 
`closed` state and sends a FIN packet to the remote stream. When a FIN packet is received, the stream transitions to the `remoteClosed` state.

In the `closed` state, the stream can not write any data to the remote connection. All `Write()` calls return an ErrBrokenPipe error. The stream can still receive data, and canread data from the buffer.

The `remoteClosed` state signifies that the remote stream will not send any more data to the stream. `Read()` calls can still read data from the buffer. If the buffer is empty
then the `Read()` calls return EOF. The stream can write data to the remote session.

If a FIN packet is received when in the `closed` state, or `Close()` is called in the `remoteClosed` state, the stream transitions to the `dead` state. All `Write()` calls fail
in the `dead` state, but `Read()` can retreive data from the buffer. If the stream is in the `dead` state, and the buffer is empty, then the stream is removed by its Session.

The state transitions can be summed up in the following diagram:

![stream states](/img/streamstates.jpg)
