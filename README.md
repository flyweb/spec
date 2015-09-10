
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
  Promise<FlyWebServices> discoverNearbyServices(in any type);
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
  // should we do "readonly maplike<DOMString, FlyWebService>" instead?
  readonly attribute unsigned long length;
  getter FlyWebService (unsigned long index);
  FlyWebService? get(DOMString id);

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

### services.get(id)

Returns the `FlyWebService` object with the given identifier, or null if
no service has that identifier.

## services.stopDiscovery()

Tells the user agent to stop listening for services of that type.  After this
method is called, no further `onservicefound` or `onservicelost` events will
be emitted by this FlyWebServices object.

*Should we rename this to 'close()' to match other APIs with similar "stop consuming resources" behavior?*

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
dictionary FlyWebService {
  DOMString      id;
  DOMString      name;
  DOMString      type;
  object         config;
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

# Connecting to nearby services

```language-webidl
interface FlyWebConnectAPI {
  Promise<FlyWebConnection> connectToService(in any type);
  Promise<sequence<FlyWebConnection>> connectToServices(in any type);
}
```

## Methods

### navigator.connectToService()

```
promise = navigator.connectToService(type)
```

Immediately returns a Promise representing a session establishment attempt to
the service.  The user agent then asynchronously does a discovery of all devices
matching the specified type.

Once a list of devices has been established, the user is shown UI containing
this list. The user can then choose which device he/she wants to let the webpage
to connect to.

Once a device has been selected by the user, the user agent then asynchronously
begins a session establishment protocol with the remote service.

This protocol may include a pairing activity, as dictated by the session
construction protocol.  See the `Session Establishment Protocol` which
describes how sessions are established.

If session establishment is successful, a new `FlyWebConnection` object is
created and the promise object is resolved with the object as result.

If the user declines, or an error occurs, the promise object is rejected.

### navigator.connectToServices()

Same as `navigator.connectToService()`, except that it allows the user to select
multiple devices which are returned to the web page.

# Interacting with a connected Fly Web service

```language-webidl
enum BinaryType {
  "blob", "arraybuffer"
};
enum FlyWebConnectionState {
  "connected",
  "disconnected",
  "closed"
};

interface FlyWebConnection {
  readonly attribute DOMString serviceId;
  readonly attribute DOMString sessionId;

  // Base-uri for other party. Null if other party doesn't have http
  // server support.
  readonly attribute DOMString? url;

  // State management
  readonly attribute FlyWebConnectionState state;
  attribute EventHandler ondisconnect;
  attribute EventHandler onreconnect;
  attribute EventHandler onclose;
  void close();

  // Communication
  attribute BinaryType binaryType;
  attribute EventHandler onmessage;
  void send(DOMString message);
  void send(Blob data);
  void send(ArrayBuffer data);
  void send(ArrayBufferView data);

  // Http server
  attribute EventHandler onrequest;
  readonly attribute DOMString userAgent;
}
```

## Attributes

### connection.serviceId

A identifier of the service endpoint. This can be used when reconnecting
to the service.

### connection.sessionId

A unique ID that represents the established session.  This ID will
not be re-used for any other sessions.

### connection.url

A "http://...local/" base URL identifying the connected server.  This URL can be
used to refer to resources on the server, including embedding in tags,
and in XMLHTTPRequests.

### clientHandle.userAgent

The user agent of a connected client.

*Do we need this given that the same information is exposed in Request objects?*

## Methods

### connection.close()

Immediately closes the connection with the other party and transitions the
`connection.state` to `closed`.

## Events

## ondisconnect

This event is fired when the remote endpoint is disconnected from the session
due to a network problem. The event type is `Event`.

NOTE: The disconnection may be due to the remote device going out of range,
or otherwise losing connectivity.

## onreconnect

This event is raised when the remote endpoint is reconnected to the session
after recovering from a network problem. The event type is `Event`.

## onclose

This event is fired when the remote endpoint explicitly closed the connection 
and disconnected from the session. The event type is `Event`.

## onrequest

This event is raised when a request is recieved from a remote client endpoint.
The event type is ['FetchEvent'](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#fetch-event-section).

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
The event type is 'FlyWebConnectEvent'.

```language-webidl
interface FlyWebClientConnectEvent : Event {
  readonly attribute FlyWebConnection connection;
}
```

