- Feature Name: network errors handling
- Start Date: 2017-02-15
- RFC PR: (leave this empty)
- Pony Issue: (leave this empty)

# Summary

Provide API for handling network errors.

# Motivation

Current interfaces, such as `TCPListenNotify`, `TCPConnectionNotify`
and `UDPNotify` do not provide any information why some action has
failed. This make it difficult to figure out software configuration
errors.

# Detailed design

1. Create primivite union of possible network errors, something like
   `FileErrNo`, mapping platform specific errno codes into this
   primive values.

2. Extend interfaces to accept errors. (New API functions)

For example:

```pony
type SocketError is
  ( SocketOK
  | SocketAccess
  | SocketInUse
  | SocketConnRefused
  | SocketNetUnreach
  | SocketTimeout
  ....
  )

interface TCPListenNotify
  ...
  fun ref listen_error(listen: TCPListener ref, error: SocketError) =>
    """
    Called if it wasn't possible to bind the listener to an address.
    """
    not_listening(listen)
  ...

interface UDPNotify
  ...
  fun ref listen_error(sock: UDPSocket ref, error: SocketError) =>
    """
    Called if it wasn't possible to bind the socket to an address.
    Default implementation calls
    """
    not_listening(listen)
  ...

interface TCPConnectionNotify
  ...
  fun ref connect_error(conn: TCPConnection ref, error: SocketError) =>
    """
    Called when we have failed to connect to all possible addresses for the
    server. At this point, the connection will never be established.
    """
    connect_failed(conn)
  ...
```

Network code should use new API functions instead of `not_listening`
and `connect_failed`. Default implementation of new API should
fallbacks to old API calls. This will keep compatibility with existing
network code, but in case you need to handle network error for
yourself, you can override default implementation.

# How We Teach This

We could have some annotation for notifier interface functions, such
as `\deprecated\`, so if programmer overrides `\deprecated\` function
compiler will yield a warning.  In such case developer will know about
API change.

# How We Test This

Existing tests for network code already covers the case, since default
implementation fallbacks to old API.

# Drawbacks

Adds a little overhead to handling failures, however network failures
is not that frequent thing to bother about.  Keeping backward
compatibility is more important.

# Alternatives

We could have network error stored inside `TCPListener`, `UDPSocket`
and `TCPConnection`, so notifier could extract the error value from
caller on network failure.

This won't require any addons not notifiers at all and won't break
existing code as well.

# Unresolved questions

How proposed API conforms with [RFC 23](https://github.com/ponylang/rfcs/blob/master/text/0023-network-dont-provide-default-implementation-for-failures.md)
