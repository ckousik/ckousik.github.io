---
layout: post
title: "GSOC Project: Webhook Tunnel"
date: 2017-5-8 21:31:00 +530
categories: gsoc
---

I got accepted to Google Summer of Code (GSoC) 2017. I will be working with Mozilla Taskcluster, and my project is Webhook Tunnel (we changed the name from livelog proxy).
TaskCluster workers are hosted on services such as EC2 and currently expose ports to the internet and allows clients to call API endpoints. 
This may not be feasible in a data center setup. Webhook proxy aims to mitigate this problem by allowing workers to connect to a proxy (part of webhook tunnel) over an 
outgoing WebSocket connection and the proxy in turn exposes API endpoints to the internet. This is implemented as a distributed system for handling high loads.

This is similar to ngrok, or localtunnel, but a key difference is that instead of providing a port that clients can connect to, webhook tunnel exposes APIs as
"\<worker-id\>.taskcluster-proxy.net/\<endpoint\>". This is a much more secure way of exposing endpoints. 

The initial plan is to deploy this on Docker Cloud. Details will follow in further posts.
