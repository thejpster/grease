# cuslip
## copper slip makes threads go smoothly when you have Rust

Cuslip is designed to encourage a message-passing based approach to stack development.

In a stack based system, consider two layers, `N` and `N+1`. Layer `N` is the *provider* of a *service* and `N+1` is the *user* of that *service*. *Users* and *providers* exist in different threads and speak to each other using *primitives*. There are four sorts of *primitive*.

1. Request (or REQ). A *user* sends a *request* to a *provider* to request that it do something. Examples might include, open a new connection, or transmit some data on an existing connection.
2. Confirmation (or CFM). A *provider* sends a *confirmation* back to a *user*. Each *request* solicits exactly one *confirmation* in reply - it usually tells the *user* whether the *request* was successful or not and includes any relevant results from the *request*.
3. Indication (or IND). Sometimes a *provider* may wish to indicate to the *user* than an event has occured - this is an *indication*. For example, a *provider* may wish to *indicate* that data has arrived on a connection, or that a connection is no longer in existence.
4. Response (or RSP). Rarely, it is necessary for a *user* to respond to an *indication* in such a fashion that it would not make sense to use a *request*/*confirmation* pair. In this case, the *user* may send a *response* to a service.

```
+-------------------------+
|     Layer N + 1         |
+-------------------------+
     |    ^     |     ^
  REQ| CFM|  IND|  RSP|
     |    |     |     |
     v    |     v     |
+-------------------------+
|     Layer N             |
+-------------------------+
```

Note that in all four cases, the primitives are defined by the *provider* and the definitions are used by the *user*.

In some cases, the *confirmation* for a *request* is fast - user requests provider sends some data, confirmation provider has received data for sending, time passes, provider indicates that sending is now complete. In other cases, the *confirmation* for a *request* is slow - user requests provider sends some data, time passes, provider confirms that data has now been sent. Deciding between these two models is something of an artform.

Often it is useful to describe these interaction via the medium of the Message Sequence Diagram.

```
    Layer N +1         Layer N
       |                 |
       |      REQ        |
       |---------------->|
       |                 |
       |      CFM        |
       |<----------------|
       |                 |
       |      IND        |
       |<----------------|
       |                 |
       |      RSP        |
       |---------------->|
       |                 |
```

During very early development, the code is built as a socket based data server. It will receive connections on a socket and answer queries given to it. Later on I'll work out how to split the generic stuff from the implementation specific stuff.

```
+---------------------------+
|        Application        |
+-------------+-------------+
| ProtoServer | Database    |
+-------------+-------------+
| Socket      |
+-------------+

```

* Application. All the application logic - the top level module. It is a *user*, but never a *provider*. It is also known as an *inflexion point* - a point where messages stop going up, turn around and start going down again. A big system may have multiple *inflexion points* but we only have one.
* ProtoServer. Decodes a custom protocol sent over a socket.
* SocketServer. Handles TCP sockets.
* Database. Stores information which Application needs to service requests from remote systems (which speak Proto).

Proto is a noddy protocol I've invented which you can speak over any sort of stream. In this system, you could replace Socket with Serial and it would still work. Whether you can generically implement a Stream in both the Socket and Serial tasks such that ProtoServer didn't care which you used (yet where they still maintain their unique connection properties) is something I tend to explore later.

PS: I know there's an AT command implementation in the source code at the moment. That was a first draft and I'll probably change it.
