---
title: "BGP over TLS/TCP" 
abbrev: bgp-tls
docname: draft-wirtgen-bgp-tls
category: exp

submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "IDR"
keyword:
 - tcp
 - tls
 - bgp

venue:
  group: "IDR"
  type: "Working Group"
  mail: "idr@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/idr/"
  github: "obonaventure/draft-bgp-tls"


author:
 -
    name: Thomas Wirtgen
    organization: UCLouvain
    email: thomas.wirtgen@uclouvain.be
 -
    name: Olivier Bonaventure 
    organization: UCLouvain 
    email: olivier.bonaventure@uclouvain.be



normative:
  RFC7301:
  RFC4271:
  RFC4272:
  RFC7301:

informative:
  I-D.draft-retana-idr-bgp-quic:
  RFC5082:
  RFC5246:
  RFC8446:
  RFC9000:

  

--- abstract

This document specifies the utilization of TCP/TLS to support BGP. 

--- middle

# Introduction


The Border Gateway Protocol (BGP) {{RFC4271}} relies on the TCP protocol
to establish BGP sessions between routers. A recent draft
{{I-D.draft-retana-idr-bgp-quic}} has proposed to replace TCP with
the QUIC protocol {{RFC9000}}. QUIC brings many features compared to
TCP including security, the support of multiple streams or datagrams.

From a security viewpoint, an important benefit of QUIC compared to TCP is
that QUIC by design prevents injection attacks that are possible when
TCP is used by BGP {{RFC4272}}. Several techniques can be used by BGP routers
to counter this attacks {{RFC5082}} {{RFC5925}}. TCP-AO {{RFC5925}}
authenticates the packets exchanged over a BGP session provides similar
features as QUIC. However, it is notoriously difficult to configure the
keys used to protect BGP sessions.

The widespread deployment of TLS combined with the possibility of
deriving TCP-AO keys from the TLS handshake {{draft-bonaventure-tcp-tls}}
creates an interest in using TLS to secure BGP sessions. This document
describes how BGP can operate over TCP/TLS.



# Conventions and Definitions

{::boilerplate bcp14-tagged}



This document uses network byte order (that is, big endian) values.
Fields are placed starting from the high-order bits of each byte.

# Summary of operation

A BGP over TLS/TCP session is established in two phases:

 - establish a transport layer connection using TCP 
 - establish a TLS session over the TCP connection

The TCP connection SHOULD be established on port TBD1.

During the establishment of the TLS session, the router that initiates the
connection MUST use the "botls" token in the Application Layer Protocol
Negotiation (ALPN) extension {{RFC7301}}. The support for other ALPN MUST
NOT be proposed during the TLS handshake. This document supports both
TLS 1.2 {{RFC5246}} and TLS 1.3 {{RFC8446}}.

Once the TLS handshake is established and finished, the BGP session is
initiated as defined in {{RFC4271}} and the protocol operates in the
same way as a normal BGP over TCP session. The difference is that the
BGP session is now encrypted using the TLS layer.

TODO: How can TLS failures be handled? For non-critical TLS errors,
a new BGP notification option could be requested to indicate to
the other router that the TLS session has failed. For critical errors,
the TCP session will be reset without any BGP notification...
If TCP-AO is enabled, this question does not arise.

# Security Considerations

This document improves the security of BGP sessions since the information exchanged over the
session is now protected by using TLS. When combined with TCP-AO, it also counters
TCP injection attacks listed in {{RFC5082}}. 

If the BGP router supports TCP-AO, we recommend an opportunistic
TCP-AO approach as suggested in {{draft-bonaventure-tcp-ao-tls}}. The
router will attempt to connect using TCP-AO with a default key. When the TLS
handshake is finished, the routers will derive a new TCP-AO key using the TLS key.

TODO: how do you distinguish a BGP router that supports TCP-AO from one that does not?
Try using TCP first. Establish a BGP session over TLS. When the BGP session is
initiated, both routers must indicate that they support TCP-AO by adding a new BGP
capability to the BGP OPEN message. If both routers agree to use TCP-AO, close the
BGP/TLS/TCP session, then restart the session with TCP-AO enabled.  


# IANA Considerations

IANA is requested to assign a TCP port (TBD1) from the "Service Name and Transport
Protocol Port Number Registry" as follows:

- Service Name: botls
- Port Number: TBD1
- Transport Protocol: TCP
- Description: BGP over TLS/TCP
- Assignee: IETF
- Contact: IDR WG
- Registration Data: TBD
- Reference: this document
- Unauthorized Use Reported: idr@ietf.org


It is suggested to use the same port as the one selected for BGP over QUIC
{{I-D.draft-retana-idr-bgp-quic}}.

# Acknowledgments
{:numbered="false"}

The authors thank xx for their comments on the first version of this draft and
Dimitri Safonov for the TCP-AO implementation in Linux. 

# Change log
{:numbered="false"}



