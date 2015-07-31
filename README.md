
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
In Fly Web, both web clients and servers may advertise themselves for
local-area consumption, over local-area transport protocols such as BlueTooth
or Wi-Fi direct.  Clients may discover and connect to servers, and vice-versa,
servers may discover and connect to locally available web clients.

# Conformance

TODO

# Dependencies

TODO

# Terminology

This API is structured around a process for establishing a web session between
two computational endpoints, where one endpoint serves as a web client, and
a web server.

A `client endpoint` is a device or program on that device which plays the role
of a web client in a Fly Web session.

A `server endpoint` is a device or program on that device which plays the role
of a web server in a Fly Web session.

When a Fly Web session is established, it is established from a device or
program which initiates the session establishment, and to a device or program
which passively advertises itself as available for session establishment.
This concept is orthogonal to the notion of server and client.  Both servers
and clients may advertise themselves.

A `passive endpoint` is a device or program on that device which passively
advertises its services for consumption.

An `active endpoint` is a device or program on that device which discovers
and initiates a connection with a passive endpoint.

# Security and privacy considerations

The API defined in this specification can be used to find and connect to
devices and services near the user.  This specification uses three mechanisms
to ensure that services are not inadvertantly exposed to potentially
malicious code executing on web-pages.

Firstly, a passive endpoint advertises its services with explicit metadata
that indicates its willingness to serve as a Fly Web endpoint.  Other,
non-Fly Web services which are advertised using similar mechanisms cannot
thus be mistaken for a Fly Web service.

Secondly, a passive endpoint may force a pairing protocol at the point
of session establishment.  This protocol may take various forms, including
entering a secret code, scanning a QR code displayed on a screen, or a
synchronized button press.  These pairing protocols are similar to
bluetooth device pairing protocols, and already familiar to users.

Thirdly, when a web-page uses these APIs to discover nearby services,
or to advertise its own services, the user agent SHOULD notify the user
of the activity, and SHOULD prompt the user to allow the activity to
proceed.

# Discovering nearby services

```language-webidl
interface FlyWebDiscoveryAPI {
  Promise discoverNearbyServices(in any type);
}
```

## Methods

### navigator.discoverNearbyServices

```
promise = navigator.discoverNearbyServices(type)
```

Immediately returns a new Promise object.  The user agent asynchronously
begins discovering network services that have advertised support for the
requested service type(s).  The type argument contains one or more
valid service type tokens that the web page would like to interact with.

If the user accepts, the promise object is resolved, with a `FlyWebServices`
object as its argument.

If the user declines, or an error occurs, the promise object is rejected.

The `type` parameter is either a DOMString, or array of DOMStrings, with
the prefix 'flyweb:'.  See the 'Service Query Type Strings' section for
information on type strings, their format and interpretation.

TODO: Specify formal steps.


# Inspecting nearby Fly Web services

The `FlyWebServices` interface represents a collection of zero or more
indexed properties that are each a user-authorized FlyWebService object.

A `FlyWebServices` is the promise result from a call to
`discoverNearbyServices()`.

```language-webidl
interface FlyWebServices {
  readonly attribute unsigned long length;
  getter FlyWebService (unsigned long index);
  FlyWebService? getServiceById(DOMString id);

  void stopDiscovery();

  attribute EventHandler           onservicefound;
  attribute EventHandler           onservicelost;
}

FlyWebServices implements EventTarget
```

## Attributes

### services.length

Returns the current number of indexed properties in the current object's
collection.

## Methods

### services[index]

Returns the specified `FlyWebService`.

### services.getServiceById(id)

Returns the `FlyWebService` object with the given identifier, or null if
no service has that identifier.

## services.stopDiscovery()

Tells the user agent to stop listening for services of that type.  After this
method is called, no further `onservicefound` or `onservicelost` events will
be emitted by this FlyWebServices object.

## Events

## onservicefound

Raised when a new service is discovered which matches the requested service
type.  The event type is `FlyWebServiceFoundEvent`.

```language-webidl
interface FlyWebServiceFoundEvent : Event {
  readonly attribute DOMString serviceId;
}
```

The `serviceId` represents a unique ID for the newly discovered service.
This id can be passed to `services.getServiceById()` to retrieve the
associated `FlyWebService` instance.

## onservicelost

Raised when an existing service listed in the services collection is lost.
The event type is `FlyWebServiceLostEvent`.

```language-webidl
interface FlyWebServiceLostEvent : Event {
  readonly attribute DOMString serviceId;
}
```

The `serviceId` represents a unique ID for the lost service.

# Connecting to a specific Fly Web service

```language-webidl
interface FlyWebService {
  readonly attribute DOMString      id;
  readonly attribute DOMString      name;
  readonly attribute DOMString      type;
  readonly attribute object         config;

  readonly attribute boolean        online;

  Promise establishSession();

  attribute EventHandler            onavailable;
  attribute EventHandler            onunavailable;
}
```

## Attributes

### service.id

A unique identifier for the given service instance.

### service.name

A descriptive, service-selected name for the service.

### service.type

One of "client" or "server", identifying whether the service advertises a web
client or a web server.

### service.config

A dictionary object containing the configuration options associated with the
service.

### service.online

The current state of the service, indicating whether it is available or not.
This state changes with the `onavailable` and `onunavailable` events.

## Methods

### service.establishSession()

```language-js
promise = service.connect();
```

Immediately returns a Promise representing a session establishment attempt to
the service.  The user agent then asynchronously begins a session establishment
protocol with the remote service.

This protocol may include a pairing activity, as dictated by the session
construction protocol.  See the `Session Establishment Protocol` which
describes how sessions are established.

If session establishment is successful, the promise object is resolved.
The argument type of the resolved promise depends on the type of
service.  If the service is a HTTP Server, then the promise object is
resolved with a `FlyWebServerHandle` argument.  If the service is
an HTTP Client, then the promise object is resolved with a
`FlyWebClientHandle` argument.

If the user declines, or an error occurs, the promise object is rejected.

## Events

## onavailable

Raised when this service re-registers itself or otherwise becomes available
for connection.  When this event is raised, the `service.online` property is
set to true.  The event type is `FlyWebServiceAvailableEvent`.

```language-webidl
interface FlyWebServiceAvailableEvent : Event {
  readonly attribute DOMString serviceId;
}
```

The `serviceId` attribute is the id of the service that became available.

NOTE: The `onavailable` event being raised on a `FlyWebService` may coincide
with a `onservicefound` event being raised on a `FlyWebServices` object, if
`stopDiscovery()` has not yet been called.

## onunavailable


Raised when this service becomes unavailable or otherwise lost.  When this
event is raised, the `service.online` property is set to false.  The event
type is `FlyWebServiceUnavailableEvent`.

```language-webidl
interface FlyWebServiceUnavailableEvent : Event {
  readonly attribute DOMString serviceId;
}
```

The `serviceId` attribute is the id of the service that became unavailable.

NOTE: The `onunavailable` event being raised on a `FlyWebService` may coincide
with a `onservicelost` event being raised on a `FlyWebServices` object, if
`stopDiscovery()` has not yet been called.


# Interacting with a connected Fly Web service

```language-webidl
interface FlyWebServerHandle {
  readonly attribute DOMString sessionId;
  readonly attribute DOMString serverURL;

  Promise disconnect();

  attribute EventHandler            ondisconnect;
}

interface FlyWebClientHandle {
  readonly attribute DOMString sessionId;
  readonly attribute DOMString userAgent;

  Promise disconnect();

  attribute EventHandler            ondisconnect;
  attribute EventHandler            onrequest;
}
```

## Attributes

### handle.sessionId

A unique ID that represents the established session.  This ID will
not be re-used for any other sessions.

### serverHandle.serverURL

A "flyweb://" base URL identifying the connected server.  This URL can be
used to refer to resources on the server, including embedding in tags,
and in XMLHTTPRequests.

### clientHandle.userAgent

The user agent of a connected client.

## Methods

### handle.disconnect()

Immediately returns a Promise representing a disconnection attempt from the
service.  The user agent then asynchronously ends the session.

When the session has been ended, the promise is resolved.  If there is
an error, the promise is rejected.

## Events

## ondisconnect

This event is raised when the remote endpoint disconnects from the session.
The event type is `FlyWebDisconnectEvent`.

```language-webidl
interface FlyWebDisconnectEvent : Event {
  readonly attribute DOMString sessionId;
}
```

The `sessionId` is the id of the session that became disconnected.

NOTE: The disconnection may be due to the remote device going out of range,
or otherwise losing connectivity.

## onrequest

This event is raised when a request is recieved from a remote client endpoint.
The event type is 'FlyWebClientRequestEvent'.

```language-webidl
interface FlyWebClientRequestEvent : Event {
  readonly attribute DOMString    sessionId;
  readonly attribute HTTPRequest  request;
  readonly attribute HTTPResponse response;
}
```

The `sessionId` is the id of the session that made the request.  The
`request` field holds the HTTPRequest object which can be used to
obtain request data and metadata.  The `response` field holds an
HTTPResponse object which can be used to send a response to the client.


# Advertising a server to nearby consumers

```language-webidl
interface FlyWebPublishingAPI {
  Promise publishServer(DOMString name, object config);
}
```

## Methods

### navigator.publishServer

```
promise = navigator.publishServer(category, name, config);
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

# Interacting with a remote Fly Web client

```language-webidl
interface FlyWebPublishedServer {
  readonly attribute DOMString category;
  readonly attribute DOMString name;
  readonly attribute object config;

  attribute EventHandler            onconnect;
  attribute EventHandler            ondisconnect;
  attribute EventHandler            onrequest;
}
```

## Attributes

### publishedServer.category

The category the server is published under.

### publishedServer.name

The display name the server is published under.

### publishedServer.config

The configuration parameters for the published server.

## Events

### onconnect

Raised when a new client connects to the published server.
The event type is 'FlyWebClientConnectEvent'.

```language-webidl
interface FlyWebClientConnectEvent : Event {
  readonly attribute DOMString sessionId;
}
```

The `sessionId` field uniquely identifies the web session established by the
connection.

## ondisconnect

This event is raised when the remote client disconnects from the session.
The event type is `FlyWebDisconnectEvent`.

```language-webidl
interface FlyWebDisconnectEvent : Event {
  readonly attribute DOMString sessionId;
}
```

The `sessionId` is the id of the session that became disconnected.

NOTE: The disconnection may be due to the remote client going out of range,
or otherwise losing connectivity.

## onrequest

This event is raised when a request is recieved from a remote client endpoint.
The event type is 'FlyWebClientRequestEvent'.

```language-webidl
interface FlyWebClientRequestEvent : Event {
  readonly attribute DOMString    sessionId;
  readonly attribute HTTPRequest  request;
  readonly attribute HTTPResponse response;
}
```

The `sessionId` is the id of the session that made the request.  The
`request` field holds the HTTPRequest object which can be used to
obtain request data and metadata.  The `response` field holds an
HTTPResponse object which can be used to send a response to the client.
