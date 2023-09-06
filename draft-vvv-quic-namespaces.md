---
title: "Stream Namespaces for QUIC"
category: info

docname: draft-vvv-quic-namespaces-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: tsv
workgroup: QUIC Working Group
keyword:
 - QUIC
 - namespaces
venue:
  group: quic
  type: Working Group
  mail: quic@ietf.org
  arch: https://example.com/WG
  github: vasilvv/draft-vvv-quic-namespaces
  #latest: https://example.com/LATEST

author:
 -
    fullname: Victor Vasiliev
    organization: Google
    email: vasilvv@google.com

normative:

informative:


--- abstract

QUIC Stream Namespaces provide an extension to the QUIC protocol that enables
multiplexing multiple logical groups of streams within the same connection,
while providing flow control isolation.


--- middle

# Introduction

QUIC {{!RFC9000}} provides an ordered bytestream abstraction called streams.
Streams are subject to various flow control mechanisms that allow a network
endpoint to control how much resources a peer is allowed to consume.  Some of
the flow control mechanisms are scoped to a single stream; others are global to
the entire connection.  The connection-level flow control mechanisms are a good
fit in cases when all of the streams originate from the same entity; however,
in cases when multiple logical entities share the same connection, a single
global limit may lead to one entity starving another.  This document provides a
mechanism by which a single QUIC connection can have multiple namespaces, each
with its own resource limits for streams.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Namespaces

A QUIC namespace is a 62-bit unique ID number. In the initial state, every
namespace ID is assumed to exist, but have a MAX_STREAMS number associated with
it set to 0 for all types of streams, and a MAX_DATA value of 0 in both
directions. A peer opens a namespace by sending a combination of MAX_DATA and
MAX_STREAMS frames for that namespace. The recepient may response with either
its own MAX_DATA and MAX_STREAMS, confirming the response, or it may close the
namespace. Frames that do not have a namespace ID associated with them are said
to be a part of _the default namespace_.

Note that there is no way to set a namespace-specific initial_max_stream_data
parameters; those remain connection-global.

# Frames

## NS frame

An NS frame (frame type=0x29c5) is a frame that alters the meaning of the frame
that comes immediately after it.  If the subsequent frame has a stream ID in it,
that ID refers to the stream with the corresponding ID in the specified
namespace. If the subsequent frame alters connection-global flow control
limits, those limits are altered for the namespace in question, instead of the
default namespace.

~~~
NS Frame {
  Type (i) = 0x29c5,
  Namespace ID (i),
}
~~~
{: #fig-NS title="NS Frame Format"}

The following frames are allowed to follow the NS frame: STREAM, RESET_STREAM,
STOP_SENDING, MAX_DATA, MAX_STREAM_DATA, MAX_STREAMS, DATA_BLOCKED,
STREAM_DATA_BLOCKED, STREAMS_BLOCKED.  Extensions that define their own frames
can define their own semantics of interacting with namespaces.  If a frame that
is not listed above and does not have extension semantics defined for it is
prefixed with an NS frame, the recepient MUST close the connection with a
PROTOCOL_VIOLATION error code. Same applies to an NS frame that is not followed
by anything.

Note that this intentionally does not define NS prefix for the DATAGRAM frames
{{!RFC9221}}, as datagrams already have pre-defined mechanisms for multiplexing
(such as {{?RFC9297}}) that may conflict with QUIC stream namespaces, and there
is no technical advantage of using an NS frame with datagrams over doing
multiplexing within the datagram payload.

## CLOSE_NAMESPACE frame

A CLOSE_NAMESPACE frame indicates to the peer that the sender will not process
any further data received for a given namespace.  The sender can discard all of
the state related to the namespace after sending this frame.

~~~
CLOSE_NAMESPACE Frame {
  Type (i) = 0x29c6,
  Namespace ID (i),
}
~~~
{: #fig-close-namespace title="CLOSE_NAMESPACE Frame Format"}

# Security Considerations

TODO Security

TODO: discuss the issue where the peer has to remember flow control limits for arbitrary unexpected namespaces.


# IANA Considerations

TODO: add a transport parameter to negotiate this feature.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
