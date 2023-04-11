---
title: "Amplification Attacks Using the Constrained Application Protocol (CoAP)"
abbrev: "CoAP Amplification Attacks"
category: info

docname: draft-irtf-t2trg-amplification-attacks-latest
submissiontype: IRTF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "IRTF"
workgroup: "Thing-to-Thing"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "Thing-to-Thing"
  type: "Research Group"
  mail: "t2trg@irtf.org"
  arch: "https://mailarchive.ietf.org/arch/search/?email_list=t2trg"
  github: "t2trg/t2trg-amplification-attacks"
  latest: "https://t2trg.github.io/t2trg-amplification-attacks/draft-irtf-t2trg-amplification-attacks.html"

author:
- name: John Preuß Mattsson
  initials: J.
  surname: Preuß Mattsson
  org: Ericsson AB
  abbrev: Ericsson
  street: SE-164 80 Stockholm
  country: Sweden
  email: john.mattsson@ericsson.com
- name: Göran Selander
  surname: Selander
  org: Ericsson AB
  abbrev: Ericsson
  street: SE-164 80 Stockholm
  country: Sweden
  email: goran.selander@ericsson.com
- name: Christian Amsüss
  surname: Amsüss
  org: Energy Harvesting Solutions
  email: c.amsuess@energyharvesting.at

normative:

informative:

  RFC7252:
  RFC7641:
  RFC8152:
  RFC8323:
  RFC8446:
  RFC8613:
  RFC9000:
  RFC9146:
  RFC9147:
  RFC9175:
  I-D.ietf-core-conditional-attributes:
  I-D.ietf-core-groupcomm-bis:
  I-D.ietf-core-oscore-groupcomm:

  DDoS-Infra:
    target: https://www.darkreading.com/attacks-breaches/critical-infrastructure-under-attack-/a/d-id/1340960
    title: "Critical Infrastructure Under Attack"
    seriesinfo:
      "Dark Reading"
    date: November 2021

--- abstract

Protecting Internet of Things (IoT) devices against attacks is not enough.
IoT deployments need to make sure that they are not used for
Distributed Denial-of-Service (DDoS) attacks. DDoS attacks are
typically done with compromised devices or with amplification attacks
using a spoofed source address. This document gives examples of different
theoretical amplification attacks using the Constrained Application Protocol
(CoAP). The goal with this document is to raise awareness and
to motivate generic and protocol-specific recommendations on the usage of
CoAP. Some of the discussed attacks can be mitigated by not using
NoSec or by using the Echo option.

--- middle

# Introduction

One important protocol used to interact with Internet of Things (IoT)
sensors and actuators is the Constrained Application Protocol (CoAP) {{RFC7252}}.
CoAP can be used without security in the so called NoSec mode but any
Internet-of-Things (IoT) deployment valuing security and privacy would use a
security protocol such as DTLS {{RFC9147}}, TLS {{RFC8446}}, or OSCORE {{RFC8613}}
to protect CoAP, where the choice of security protocol depends on the transport
protocol and the presence of intermediaries. The use of CoAP over UDP and DTLS is
specified in {{RFC7252}} and the use of CoAP over TCP and TLS is specified in {{RFC8323}}.
OSCORE protects CoAP end-to-end with the use of COSE {{RFC8152}} and the CoAP
Object-Security option {{RFC8613}} and can therefore be used over any
transport. Group OSCORE {{ I-D.ietf-core-oscore-groupcomm}} can be used to
protect CoAP Group Communication {{I-D.ietf-core-groupcomm-bis}}.

Protecting Internet of Things (IoT) devices against attacks is not enough.
IoT deployments need to make sure that they are not used for
Distributed Denial-of-Service (DDoS) attacks. DDoS attacks are
typically done with compromised devices or with amplification attacks
using a spoofed source address. DDoS attacks is a huge and
growing problem for services and critical infrastructure {{DDoS-Infra}}
and mitigations are costly.

The document gives examples of different theoretical amplification attacks using CoAP.
When transported over UDP, the CoAP NoSec mode is susceptible to source
IP address spoofing and as a single request can result in multiple responses
from multiple servers, CoAP can have very large amplification factors.
The goal with this document is to raise awareness and understanding of amplification
attacks and to motivate mitigations suitable for constrained devices and networks. The
intent is not to suggest that CoAP is more vulnerable to amplification attacks than
other protocols.

Some of the discussed attacks can be mitigated by not using
NoSec or by using the Echo option {{RFC9175}}.

# Amplification Attacks using CoAP {#dos}

In a Denial-of-Service (DoS) attack, an attacker sends a large number of requests
or responses to a target endpoint. The denial-of-service might be caused by
the target endpoint receiving a large amount of data, sending a large amount
of data, doing heavy processing, or using too much memory, etc. In a Distributed
Denial-of-Service (DDoS) attack, the request or responses come from a large
number of sources.

In an amplification attack, the amplification factor is the ratio between the
total size of the data sent to the target and the total size of the data
received from the attacker. Note that in the presence of intermediaries, the
size of the data received by the target might be different than the size of
the data sent to the target and the size of the data received from the
attacker might be different than the size of the data sent from the attacker.

In the attacks described in this section, the attacker sends one or more requests,
and the target receives one or more responses. By spoofing the source IP address of
the targeted victim the and requesting as much information as possible from several
servers an attacker can multiply the amount of traffic and create a distributed
denial-of-service attack on the target. When transported over UDP, the CoAP NoSec
mode is susceptible to source IP address spoofing.

The amplification factor and the bandwidth depend on the layer in the protocol stack that
is used for the calculation. The amplification factor and bandwidth can e.g., be calculated
using whole IP packets, UPD payloads, or CoAP payloads. The bandwidth decreases and the
amplification factor typically increases higher up in the protocol stack. The bandwidth
should be calculated using the layer that is considered to be under attack.

The following sections give examples of different theoretical amplification attacks using CoAP.

## Simple Amplification Attacks

An amplification attack using a single response is illustrated in {{ampsingle}}.
If the response is c times larger than the request, the amplification factor is c.

~~~~ aasvg
Victim   Foe   Server
   |      |      |
   |      +----->|      Code: 0.01 (GET)
   |      | GET  |  Uri-Path: random quote
   |      |      |
   |<------------+      Code: 2.05 (Content)
   |      | 2.05 |   Payload: "just because you own half the county
   |      |      |             doesn't mean that you have the power
   |      |      |             to run the rest of us. For twenty-
   |      |      |             three years, I've been dying to tell
   |      |      |             you what I thought of you! And now...
   |      |      |             well, being a Christian woman, I can't
   |      |      |             say it!"
~~~~
{: #ampsingle title='Amplification attack using a single response' artwork-align="center"}

An attacker can increase the bandwidth by sending several GET requests. If the server supports
PUT/POST and doesn't limit the payload size, an attacker may be able to increase the amplification factor
by creating or updating a resource. By creating new resources, an attacker may also increase the
size of /.well-known/core. An amplification attack where the attacker influences the amplification factor
is illustrated in {{ampmulti_post}}.


~~~~ aasvg
Victim   Foe   Server
   |      |      |
   |      +----->|      Code: 0.02 (POST)
   |      | POST |  Uri-Path: member
   |      |      |   Payload: hampsterdance.hevc
   |      |      |
     ....   ....
   |      |      |
   |      +----->|      Code: 0.02 (GET)
   |      | GET  |  Uri-Path: member
   |      |      |
   |<------------+      Code: 2.05 (Content)
   |      | 2.05 |   Payload: hampsterdance.hevc
   |      |      |
   |      +----->|      Code: 0.02 (GET)
   |      | GET  |  Uri-Path: member
   |      |      |
   |<------------+      Code: 2.05 (Content)
   |      | 2.05 |   Payload: hampsterdance.hevc
     ....   ....
~~~~
{: #ampmulti_post title='Amplification attack using several requests and a chosen amplification factor' artwork-align="center"}

## Amplification Attacks using Observe

Amplification factors can be significantly worse when combined with observe {{RFC7641}}. As a single request can result in multiple responses from the server, the amplification factors can be very large.

An amplification attack using observe is illustrated in
{{ampmulti_obs}}. If each notification response is c times larger than the registration
request and each request results in n notifications, the amplification factor is c {{{⋅}}} n.

~~~~ aasvg
Victim   Foe   Server
   |      |      |
   |      +----->|       Code: 0.01 (GET)
   |      | GET  |      Token: 0x83
   |      |      |    Observe: 0
   |      |      |   Uri-Path: ozone
   |      |      |
     ....   ....
   |      |      |
   |<------------+       Code: 2.05 (Content)
   |      | 2.05 |      Token: 0x83
   |      |      |    Observe: 80085
   |      |      |    Payload: "21.4 ppbv"
   |      |      |
     ....   ....
   |      |      |
   |<------------+       Code: 2.05 (Content)
   |      | 2.05 |      Token: 0x84
   |      |      |    Observe: 80086
   |      |      |    Payload: "21.5 ppbv"
     ....   ....
~~~~
{: #ampmulti_obs title='Amplification attack using observe' artwork-align="center"}

A more advanced amplification attack using observe is illustrated in
{{ampmulti_nk}}. By registering the same client several times using different Tokens or port numbers,
the bandwidth can be increased. By updating the observed resource, the attacker
may trigger notifications and increase the size of the notifications. By using
conditional attributes {{I-D.ietf-core-conditional-attributes}} an attacker may increase the frequency of
notifications and therefore the amplification factor. The maximum period attribute pmax
indicates the maximum time, in seconds, between two consecutive notifications (whether or not the
resource state has changed). If it is predictable when notifications
are sent as confirmable and which Message ID are used the acknowledgements may be spoofed.

~~~~ aasvg
Victim   Foe   Server
   |      |      |
   |      +----->|       Code: 0.01 (GET)
   |      | GET  |      Token: 0x83
   |      |      |    Observe: 0
   |      |      |   Uri-Path: temperature
   |      |      |  Uri-Query: pmax="0.1"
   |      |      |
   |      +----->|       Code: 0.01 (GET)
   |      | GET  |      Token: 0x84
   |      |      |    Observe: 0
   |      |      |   Uri-Path: temperature
   |      |      |  Uri-Query: pmax="0.1"
   |      |      |
     ....   ....
   |      |      |
   |<------------+       Code: 2.05 (Content)
   |      | 2.05 |      Token: 0x83
   |      |      |    Observe: 217362
   |      |      |    Payload: "299.7 K"
   |      |      |
   |<------------+       Code: 2.05 (Content)
   |      | 2.05 |      Token: 0x84
   |      |      |    Observe: 217362
   |      |      |    Payload: "299.7 K"
   |      |      |
     ....   ....
   |      |      |
   |<------------+       Code: 2.05 (Content)
   |      | 2.05 |      Token: 0x83
   |      |      |    Observe: 217363
   |      |      |    Payload: "299.7 K"
   |      |      |
   |<------------+       Code: 2.05 (Content)
   |      | 2.05 |      Token: 0x84
   |      |      |    Observe: 217363
   |      |      |    Payload: "299.7 K"
     ....   ....
~~~~
{: #ampmulti_nk title='Amplification attack using observe, registering the same client several times, and requesting notifications at least 10 times every second' artwork-align="center"}

## Amplification Attacks using Group Requests

Amplification factors can be significantly worse when combined with observe {{RFC7641}}. As a single request can result in responses from multiple servers, the amplification factors can be very large.

As the group request is sent over multicast or broadcast the request has to be sent by a node on the local network. An attacker on a local network (e.g., a compromised node) might use local CoAP servers to attack targets on the Internet or on the local network. The servers might also be accessible through a gateway (GW) that transforms unicast requests into multicast request. In such architechtures, amplification attacks using group requests might be possible to launch from the Internet.

An amplification attack using a group request is illustrated in
{{ampmulti_m}}. A single unicast request results is tranformed into
a multicast group request by the gateway and results in m responses
from m different servers. If each response is c times larger than the request,
the amplification factor is c {{{⋅}}} m. Note that the servers usually do not know
the variable m.

~~~~ aasvg
Victim   Foe    GW    Servers
   |      |      |      |  |
   |      +--------+--->|  |      Code: 0.01 (GET)
   |      |      |  '----->|     Token: 0x69
   |      |      | GET  |  |  Uri-Path: </c>
   |      |      |      |  |
   |<-------------------+  |      Code: 2.05 (Content)
   |      |      | 2.05 |  |     Token: 0x69
   |      |      |      |  |   Payload: { 1721 : { ...
   |      |      |      |  |
   |<----------------------+      Code: 2.05 (Content)
   |      |      | 2.05 |  |     Token: 0x69
   |      |      |      |  |   Payload: { 1721 : { ...
   |      |      |      |  |
     ....   ....
~~~~
{: #ampmulti_m title='Amplification attack using multicast' artwork-align="center"}

Even higher amplification factors can be achieved when combining group requests
{{I-D.ietf-core-groupcomm-bis}} and observe {{RFC7641}}. In this case a single
request can result in multiple responses from multiple servers.

An amplification attack using a multicast request and observe is
illustrated in {{ampmulti_mn}}. In this case a single request results
in n responses each from m different servers giving a total of n {{{⋅}}} m
responses. If each response is c times larger than the request,
the amplification factor is c {{{⋅}}} n {{{⋅}}} m.

~~~~ aasvg
Victim   Foe    GW    Servers
   |      |      |      |  |
   |      +--------+--->|  |      Code: 0.01 (GET)
   |      |      |  '----->|     Token: 0x44
   |      |      | GET  |  |  Uri-Path: temperature
   |      |      |      |  |
   |<-------------------+  |       Code: 2.05 (Content)
   |      |      | 2.05 |  |      Token: 0x44
   |      |      |      |  |    Observe: 217
   |      |      |      |  |    Payload: "301.2 K"
   |      |      |      |  |
   |<----------------------+       Code: 2.05 (Content)
   |      |      | 2.05 |  |      Token: 0x44
   |      |      |      |  |    Observe: 363
   |      |      |      |  |    Payload: "293.4 K"
   |      |      |      |  |
   | ....          ....
   |      |      |      |  |
   |<-------------------+  |       Code: 2.05 (Content)
   |      |      | 2.05 |  |      Token: 0x44
   |      |      |      |  |    Observe: 218
   |      |      |      |  |    Payload: "301.3 K"
   |      |      |      |  |
   |<----------------------+       Code: 2.05 (Content)
   |      |      | 2.05 |  |      Token: 0x44
   |      |      |      |  |    Observe: 364
   |      |      |      |  |    Payload: "293.3 K"
   |      |      |      |  |
     ....   ....
~~~~
{: #ampmulti_mn title='Amplification attack using multicast and observe' artwork-align="center"}

An attacker can use the same techniques as in {{ampmulti_nk}} to increase the number of notifications.

## MITM Amplification Attacks

TLS and DTLS without Connection ID {{RFC9146}}{{RFC9147}} validate the IP address and port of the other peer, binds them to the connection, and do not allow them to change. DTLS with Connection ID allows the IP address and port to change at any time. As the source address is not protected, an MITM attacker can change the address. Note that an MITM attacker is a more capable attacker then an attacker just spoofing the source address. It can be discussed if and how much such an attack is reasonable for DDoS, but DTLS 1.3 states that "This attack is of concern when there is a large asymmetry of request/response message sizes." {{RFC9147}}.

DTLS 1.2 with Connection ID {{RFC9146}} requires that "the receiver MUST NOT replace the address" unless “there is a strategy for ensuring that the new peer address is able to receive and process DTLS records” but does not give more details than that. It seems like the receiver can start using the new peer address and test that it is able to receive and process DTLS records at some later point. DTLS 1.3 with Connection ID {{RFC9147}} requires that "implementations MUST NOT update the address" unless “they first perform some reachability test” but does not give more details than that. OSCORE {{RFC8613}} does not discuss address updates, but it can be assumed that most servers send responses to the address it received the request from without any reachability test. A difference between (D)TLS and OSCORE is that in DTLS the updated address is used for all future records, while in OSCORE a new address is only used for responses to a specific request.

An MITM amplification attack updating the client's source address in an observe registration is illustrated in {{amp_mitm_client}}. This attack is possible in OSCORE and DTLS with Connection ID. The server will send notifications to the Victim until it at some unspecified point requires an acknowledgement {{RFC7641}}. In DTLS 1.2 the reachability test might be done at a later point. In OSCORE a reachability test is likely not done.

~~~~ aasvg
Client  Victim  Foe   Server
   |      |      |      |
   +-----------> S----->|      Code: 0.01 (GET)
   | GET  |      |      |   Observe: 0
   |      |      |      |  Uri-Path: humidity
   |      |      |      |
   |<------------D <----+  Reachability test (DTLS)
   +-----------> S----->|
   |      |      |      |
     ....   ....   ....
   |      |      |      |
   |      |<------------+      Code: 2.05 (Content)
   |      |      | 2.05 |   Observe: 263712
   |      |      |      |   Payload: "68 %"
   |      |      |      |
   |      |<------------+      Code: 2.05 (Content)
   |      |      | 2.05 |   Observe: 263713
   |      |      |      |   Payload: "69 %"
     ....   ....   ....
~~~~
{: #amp_mitm_client title='MITM Amplification attack by updating the client's source address in a observe registration request' artwork-align="center"}

Where 'S' means the MITM attacker is changing the source address of the message and 'D' means the MITM attacker is changing the destination address of the message.

An MITM amplification attack updating the server's source address is illustrated in {{amp_mitm_server}}. This attack is possible in DTLS with Connection ID. In DTLS 1.2 the reachability test might be done at a later point.

~~~~ aasvg
Client   Foe  Victim  Server
   |      |      |      |
   +------------------->|      Code: 0.01 (POST)
   | POST |      |      |  Uri-Path: video
   |      |      |      |
   |<-----S <-----------|      Code: 2.01 (Created)
   |      |      | 2.01 |
   |      |      |      |
   +----> D------------>|  Reachability test (DTLS)
   |<-----S <-----------+
   |      |      |      |
     ....   ....   ....
   |      |      |      |
   +------------>|      |      Code: 0.01 (POST)
   | POST |      |      |  Uri-Path: video
   |      |      |      |   Payload: survailance_1139.hevc
   |      |      |      |
   +------------>|      |      Code: 0.01 (POST)
   | POST |      |      |  Uri-Path: video
   |      |      |      |   Payload: survailance_1140.hevc
     ....   ....   ....
~~~~
{: #amp_mitm_server title='MITM Amplification attack by updating the server's source address in a response' artwork-align="center"}

# Summary

CoAP has always considered amplification attacks, but most of the requirements in
{{RFC7252}}, {{RFC7641}}, {{RFC9175}}, and
{{I-D.ietf-core-groupcomm-bis}} are "SHOULD" instead of "MUST", it is
undefined what a "large amplification factor" is, {{RFC7641}} does not specify
how many notifications that can be sent before a potentially spoofable
acknowledgement must be sent, and in several cases the "SHOULD" level is
further softened by “If possible" and "generally". {{I-D.ietf-core-conditional-attributes}}
does not have any amplification attack considerations.

QUIC {{RFC9000}} mandates that ”an endpoint MUST limit the amount of data it sends
to the unvalidated address to three times the amount of data received from that
address” without any exceptions. This approach should be seen as current best practice
for non-constrained devices.

While it is clear when a QUIC implementation violates the requirement in {{RFC9000}}, it
is not clear when a CoAP implementation violates the requirement in {{RFC7252}},
{{RFC7641}}, {{RFC9175}}, and {{I-D.ietf-core-groupcomm-bis}}.

In CoAP, an address can be validated with a security protocol or by using the Echo Option {{RFC9175}}. Restricting the bandwidth per server is not enough as the number of servers the attacker can use is typically unknown. For multicast requests, anti-amplification limits and the Echo Option do not really work unless the number of servers sending responses is known. Even if the responses have the same size as the request, the amplification factor from m servers is m, where m is typically unknown. While DoS attacks from CoAP servers accessible over the Internet pose the largest threat, an attacker on a local network (e.g., a compromised node) might use local CoAP servers to attack targets on the Internet or on the local network.

# Security Considerations

The whole document can be seen as security considerations for CoAP.


# IANA Considerations

This document has no actions for IANA.

--- back

# Acknowledgements
{: numbered="false"}

The authors would like to thank
{{{Sultan Alshehri}}},
{{{Carsten Bormann}}},
{{{Klaus Hartke}}},
{{{Jaime Jiménez}}},
{{{Ari Keränen}}},
{{{Matthias Kovatsch}}},
{{{Achim Kraus}}},
{{{Sandeep Kumar}}},
and
{{{András Méhes}}}
for their valuable comments and feedback.
