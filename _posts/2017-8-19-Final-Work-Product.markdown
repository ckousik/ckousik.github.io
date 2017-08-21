---
layout: post
title: "Livelog Proxy(WebhookTunnel): Final Work Product"
date: 2017-7-19 22:30:00 +530
categories: gsoc
---

<script src="https://bramp.github.io/js-sequence-diagrams/js/webfont.js"></script>
<script src="https://bramp.github.io/js-sequence-diagrams/js/snap.svg-min.js"></script>
<script src="https://bramp.github.io/js-sequence-diagrams/js/underscore-min.js"></script>
<script src="https://bramp.github.io/js-sequence-diagrams/js/sequence-diagram-min.js"></script>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>

The project was initially named Livelog Proxy, but during the community bonding period was renamed as Webhooktunnel, as it
more accurately captured the full scope of the project.The Webhooktunnel repository can be found [here](https://github.com/taskcluster/webhooktunnel).

### Tasks Completed:
- [x] Main webhooktunnel [project](https://github.com/taskcluster/webhooktunnel).
- [x] [Taskcluster Auth integration](https://github.com/taskcluster/taskcluster-auth/pull/107) 
- [x] [taskcluster-worker integration](https://github.com/taskcluster/taskcluster-worker/pull/301)
- [x] [docker-worker integration](https://github.com/taskcluster/docker-worker/pull/304) [Stretch Goal]
- [ ] generic-worker integration [Stretch Goal]
- [ ] Routing DHT [Stretch Goal]

### Webhooktunnel Details:
Webhooktunnel works by multiplexing HTTP requests over a WebSocket connection. This allows clients to connect to the proxy and 
server webhooks over the websocket connection instead of exposing a port to the internet.

The connection process for clients(workers) is explained in the diagram below:
<div id="whtunnel-connection"></div>
The client(worker) needs an ID and JWT to connect to the proxy. These are supplied by [tc-auth](https://github.com/taskcluster/taskcluster-auth).
The proxy(whtunnel) responds by upgrading the HTTP(S) connection to a websocket connection and supplies the client's base URL in a response header.

An example of request forwarding works as follows:
<div id="whtunnel-request-forwarding"></div>

Webhooktunnel can also function as a websocket proxy.
<div id="whtunnel-wsbridge"></div>

Webhooktunnel has already been integrated into Taskcluster worker and is used for serving livelogs from task builds.

The core of Webhooktunnel is the multiplexing library [wsmux](https://github.com/taskcluster/webhooktunnel/tree/master/wsmux).
Wsmux allows creating client and server sessions over a WebSocket connection and creates multiplexed streams over the 
connection. These streams are exposed using a `net.Conn` interface.

Webhooktunnel also consists of a [command line client](https://github.com/taskcluster/webhooktunnel/tree/master/cmd/whclient), 
which can forward incoming connections from the proxy to a local port. 
This is useful as it can be used by servers which are behind a NAT/Firewall.

<script type="text/javascript">
var keys = ['whtunnel-connection', 'whtunnel-request-forwarding', 'whtunnel-wsbridge'];
var diagrams = {};
diagrams['whtunnel-connection'] = `ExampleWorker->WebhookTunnel: WebSocket request
Note right of WebhookTunnel: JWT and ID processed by WebhookTunnel
WebhookTunnel->ExampleWorker: Websocket Connection
Note right of ExampleWorker: Worker is hosted at example.tasks.build`

diagrams['whtunnel-request-forwarding'] = `Internet->WebhookTunnel:Request GET /<some-path> Host: example.tasks.build
Note right of WebhookTunnel: Checks for worker
WebhookTunnel->Example Worker: Request multiplexed over WS using wsmux
Note right of Example Worker: Request processed by worker
Example Worker->WebhookTunnel: Send response over wsmux stream
Note left of WebhookTunnel: Checks if chunked  transfer is possible over request TCP stream
WebhookTunnel->Internet: Response streamed to client`

diagrams['whtunnel-wsbridge'] = `Internet->WebhookTunnel: WS Upgrade request for example worker
Note right of WebhookTunnel: checks for worker
WebhookTunnel->ExampleWorker: Initiate WS upgrade over mux stream
Note left of WebhookTunnel: wsbridge and connects both streams`

var options = {theme: 'hand'};
keys.forEach(function(k){
	Diagram.parse(diagrams[k]).drawSVG(k,options);
});
</script>
