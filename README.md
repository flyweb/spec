
# Introduction.

This specification aims to allow web-based applications to connect with and
communicate to each other over local-area transport protocols.

In particular, this specification aims to bring the web's client/server
application model to inter-device communication.

The web's application architecture enables an application running on a server
to dynamically and incrementally send application state and logic to an
intermittently connected client.  This model enables a powerful multi-homed
application architecture.

In contrast to this power, the web architecture is also constrained.  A
server must present itself on a TCP-based network, and must have an IP address
on this network.  A client can only identify a server via a URL it is given.
Only clients may initiate connections to servers - servers cannot identify
and initiate connections to clients.  A client/server session must be
established over a TCP socket, and no other underlying transport protocol
may be used.

This specification aims to lift the above restrictions, and allow the web's
application architecture to work between devices which are proximally located.
In FlyWeb, both web clients and servers may advertise themselves for
local-area consumption, over local-area transport protocols such as BlueTooth
or Wi-Fi direct.  Clients may discover and connect to servers, and vice-versa,
servers may discover and connect to locally available web clients.

In addition, FlyWeb aims to extend the web API to allow web-pages to host
a `server endpoint`.  This mechanism is designed as a way to enable
cross-device communication using no infrastructure other than web-browsers.

# Conformance

TODO

# Dependencies

TODO

# Terminology

This API is structured around a process for establishing a web session between
two computational endpoints, where one endpoint serves as a web client, and
a web server.

A `client endpoint` is a device or program on that device which plays the role
of a web client in a FlyWeb session.  This endpoint is able to receive and
execute web applications written using web standards such as HTML, Javascript,
and CSS.  The canonical a client endpoint is a web-browser.

A `server endpoint` is a device or program on that device which plays the role
of a web server in a FlyWeb session.  This endpoint is able to serve web
applications using the HTTP protocol.

To establish a FlyWeb session, the `client endpoint` performs a local-area
discovery (at the direction of the user) over supported underlying transports
(e.g. mDNS over Wi-Fi).  The discovery reveals the presence of any local-area
`server endpoint`s, which are presentd to the user.  The user then selects
one of the `server endpoint`s to connect to, at which point the client
performs whatever transport-specific steps are required to set up a web-app
session with the server.  It then navigates to a landing page on the server,
which beings the application session between the user and the service exposed
by the server.

# Security and privacy considerations

The API defined in this specification can be used to find and connect to
devices and services near the user.  This specification uses two mechanisms
to ensure that services are not inadvertantly exposed to potentially
malicious code executing on web-pages.

Firstly, a server endpoint advertises its services with explicit metadata
that indicates its willingness to serve as a FlyWeb endpoint.  Other,
non-FlyWeb services which are advertised using similar mechanisms cannot
thus be mistaken for a FlyWeb service.

Secondly, an endpoint may force a pairing protocol at the point
of session establishment.  This protocol may take various forms, including
entering a secret code, scanning a QR code displayed on a screen, or a
synchronized button press.  These pairing protocols are similar to
bluetooth device pairing protocols, and already familiar to users.

# Advertising a server to nearby consumers

FlyWeb allows web-pages to expose a `server endpoint` using a web API.

```language-webidl
interface FlyWebPublishingAPI {
  Promise<FlyWebPublishedServer> publishServer(DOMString name,
                                               FlyWebPublishOptions options);
};

dictionary FlyWebPublishOptions {
  DOMString? uiUrl = null; // URL to user interface. Can be different server. Makes
                           // endpoint show up in browser's "local services" UI.
                           // If relative, resolves against the root of the server.
};
```

*Do we need something to hide server from discovery?*

*Rather than have a separate `uiUrl` property, we could make discoverable
servers use a specific cetegory and then indicate the url through a property
in `data`. This might depend on mDNS capabilities since we want it to be
efficient to query for all servers with a user UI.*

## Methods

### navigator.publishServer

```
promise = navigator.publishServer(name, {...options...});
```

Immediately returns a new Promise object.  The user agent asyncronously
begins the process of establishing a published service under the given
name.  The user agent SHOULD prompt the user before advertising
a service from a web page, allowing the user to prevent the page from
advertising services to nearby clients.

If the publishing is successful, the promise is resolved with a
FlyWebPublishedServer object.

If the user rejected the publishing, or an error occurs, the promise
is rejected.

# Interacting with a remote FlyWeb client

```language-webidl
interface FlyWebPublishedServer : EventTarget {
  readonly attribute DOMString name;
  readonly attribute DOMString? uiUrl;

  void close();

  attribute EventHandler onclose;
  attribute EventHandler onfetch;
  attribute EventHandler onwebsocket;
};
```

## Attributes

### publishedServer.name

The display name the server is published under.

### publishedServer.uiUrl

The server provides a user UI at the given URL.

## Events

### onclose

Raised when a the published server is closed, for example because
the ```close()``` method being called on the PublishedServer object.

### onfetch

Raised when a client of the published server makes a new HTTP fetch request.
The event type is 'FlyWebFetchEvent'.

```language-webidl
interface FlyWebFetchEvent : Event {
  [SameObject] readonly attribute Request request;

  [Throws]
  void respondWith(Promise<Response> r);
};
```

The fetch event's ```request``` attribute holds a DOM Request object
describing the details of the fetch request.

The ```respondWith``` method on the event can be used to send a response
for the request as a promise.

### onwebsocket

Raised when a client of the published server makes a new websocket connection
request.  The event type is 'FlyWebFetchEvent'.

```language-webidl
interface FlyWebWebSocketEvent : Event {
  [SameObject] readonly attribute Request request;

  [Throws]
  WebSocket accept(optional DOMString protocol);

  [Throws]
  void respondWith(Promise<Response> r);
};
```

The fetch event's ```request``` attribute holds a DOM Request object
describing the details of the request.

The ```accept``` method on the event can be used to accept the websocket
request.  It immediately returns a WebSocket instance which can be used
to communicate with the client.

The ```respondWith``` method on the event can be used to send a response
for the request as a promise.
