---
title: Explicit Flow Measurements Techniques
abbrev: Delay and Loss bits
docname: draft-mdt-ippm-explicit-flow-measurements-latest
date: {DATE}
category: info

ipr: trust200902
area: Transport
workgroup: IPPM
keyword: Internet-Draft

stand_alone: yes
pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  compact: yes
  inline: yes
  text-list-symbols: -o*+

author:
  -
    ins: M. Cociglio
    name: Mauro Cociglio
    org: Telecom Italia
    street: Via Reiss Romoli, 274
    city: Torino
    code: 10148
    country: Italy
    email: mauro.cociglio@telecomitalia.it
  -
    ins: A. Ferrieux
    name: Alexandre Ferrieux
    org: Orange Labs
    email: alexandre.ferrieux@orange.com
  -
    ins: G. Fioccola
    name: Giuseppe Fioccola
    org: Huawei Technologies
    street: Riesstrasse, 25
    city: Munich
    code: 80992
    country: Germany
    email: giuseppe.fioccola@huawei.com
  -
    ins: I. Lubashev
    name: Igor Lubashev
    org: Akamai Technologies
    email: ilubashe@akamai.com
  -
    ins: F. Bulgarella
    name: Fabio Bulgarella
    org: Telecom Italia
    street: Via Reiss Romoli, 274
    city: Torino
    code: 10148
    country: Italy
    email: fabio.bulgarella@guest.telecomitalia.it
  -
    ins: I. Hamchaoui
    name: Isabelle Hamchaoui
    org: Orange Labs
    email: isabelle.hamchaoui@orange.com
  -
    ins: M. Nilo
    name: Massimo Nilo
    org: Telecom Italia
    email: massimo.nilo@telecomitalia.it
  -
    ins: R.Sisto
    name: Riccardo Sisto
    org: Politecnico di Torino
    email: riccardo.sisto@polito.it
  -
    ins: D. Tikhonov
    name: Dmitri Tikhonov
    org: LiteSpeed Technologies
    email: dtikhonov@litespeedtech.com


normative:
  IP: RFC0791
  IPv6: RFC8200
  TCP: RFC0793
  ECN: RFC3168
  ConEx: RFC7713
  ConEx-TCP: RFC7786
  IPM-Methods: RFC7799

informative:
  QUIC-TRANSPORT: I-D.ietf-quic-transport
  TRANSPORT-ENCRYPT: I-D.ietf-tsvwg-transport-encrypt
  SPIN-BIT: I-D.trammell-quic-spin
  UDP-OPTIONS: I-D.ietf-tsvwg-udp-options
  UDP-SURPLUS: I-D.herbert-udp-space-hdr
  ACCURATE: I-D.ietf-tcpm-accurate-ecn
  IPv6AltMark: I-D.ietf-6man-ipv6-alt-mark
  AltMark: RFC8321
  ANRW19-PM-QUIC: DOI.10.1145/3340301.3341127

--- abstract

This document describes protocol independent methods called Explicit
Flow Measurement Techniques that employ few marking bits, inside the
header of each packet, for loss and delay measurement. The endpoints,
marking the traffic, signal these metrics to intermediate observers
allowing them to measure connection performance, and to locate the
network segment where impairments happen. Different alternatives are
considered within this document. These signaling methods apply to all
protocols but they are especially valuable when applied to protocols
that encrypt transport header and do not allow traditional methods for
delay and loss detection.

--- middle

# Introduction

Packet loss and delay are hard and pervasive problems of day-to-day network
operation.  Proactively detecting, measuring, and locating them is crucial to
maintaining high QoS and timely resolution of crippling end-to-end throughput
issues. To this effect, in a TCP-dominated world, network operators have been
heavily relying on information present in the clear in TCP headers: sequence and
acknowledgment numbers and SACKs when enabled (see {{?RFC8517}}). These allow
for quantitative estimation of packet loss and delay by passive on-path
observation. Additionally, the problem can be quickly identified in the network
path by moving the passive observer around.

With encrypted protocols, the equivalent transport headers are encrypted and
passive packet loss and delay observations are not possible, as described in
{{TRANSPORT-ENCRYPT}}.

Measuring TCP loss and delay between similar endpoints cannot be relied upon to
evaluate encrypted protocol loss and delay. Different protocols could be routed
by the network differently, and the fraction of Internet traffic delivered using
protocols other than TCP is increasing every year. It is imperative to measure
packet loss and delay experienced by encrypted protocol users directly.

This document defines Explicit Flow Measurement Techniques. These hybrid
measurement path signals (see {{IPM-Methods}}) are to be embedded into a
transport layer protocol and are explicitly intended for exposing RTT and loss
rate information to on-path measurement devices.  These measurement mechanisms
are applicable to any transport-layer protocol, and, as an example, the document
describes QUIC and TCP bindings.

The Explicit Flow Measurement Techniques described in this document can be used
alone or in combination with other Explicit Flow Measurement Techniques. Each
technique uses a small number of bits and exposes a specific measurement.

Following the recommendation in {{!RFC8558}} of making path signals explicit,
this document proposes adding a small number of dedicated measurement bits to
the clear portion of the protocol headers. These bits can be added to an
encrypted portion of a header belonging to any protocol layer, e.g. IP (see
{{IP}}) and IPv6 (see {{IPv6}}) headers or extensions, such as {{IPv6AltMark}},
UDP surplus space (see {{UDP-OPTIONS}} and {{UDP-SURPLUS}}), reserved bits in a
QUIC v1 header (see {{QUIC-TRANSPORT}}).

The loss signal is not designed for use in automated control of the network in
environments where loss bits are set by untrusted hosts, Instead, the signal is
to be used for troubleshooting individual flows as well as for monitoring the
network by aggregating information from multiple flows and raising operator
alarms if aggregate statistics indicate a potential problem.

Note that spin bit, delay bit and loss bits explained in this
document are inspired by {{AltMark}}.  This is also mentioned
in {{SPIN-BIT}}, {{?I-D.trammell-tsvwg-spin}} and {{?I-D.trammell-ippm-spin}}.

Additional details about the Performance Measurements for
QUIC are also described in the paper {{ANRW19-PM-QUIC}}.

# Notational Conventions    {#conventions}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in {{!RFC2119}}.

# Delay Bits

This section introduces bits that can be used for delay measurements.  Whenever
this section of the specification refers to packets, it is referring only to
packets with protocol headers that include the delay bits.

{{QUIC-TRANSPORT}} introduces an explicit per-flow transport-layer signal for
hybrid measurement of RTT.  This signal consists of a spin bit that toggles once
per RTT.  {{SPIN-BIT}} discusses an additional two-bit Valid Edge Counter (VEC)
to compensate for loss and reordering of the spin bit and increase fidelity of
the signal in less than ideal network conditions.

This document introduces an additional single-bit delay signal that can be used
together with the spin bit by passive observers to measure the RTT of a network
flow, avoiding the spin bit ambiguities that arise as soon as network conditions
deteriorate.


## Spin Bit   {#spinbit}

This section is a small recap of the spin bit working mechanism. For a
comprehensive explanation of the algorithm, please see {{SPIN-BIT}}.

The spin bit is an alternate marking {{AltMark}} generated signal, where the
size of the alternation changes with the flight size each RTT.

The latency spin bit is a single bit signal that toggles once per RTT,
enabling latency monitoring of a connection-oriented communication
from intermediate observation points.

A "spin period" is a set of packets with the same spin bit value sent during one
RTT time interval. A "spin period value" is the value of the spin bit shared by
all packets in a spin period.

The client and server maintain an internal per-connection spin value (i.e. 0 or
1) used to set the spin bit on outgoing packets. Both endpoints initialize the
spin value to 0 when a new connection starts. Then:

* when the client receives a packet with the packet number larger than any
   number seen so far, it sets the connection spin value to the opposite value
   contained in the received packet;

* when the server receives a packet with the packet number larger than any
   number seen so far, it sets the connection spin value to the same value
   contained in the received packet.

The computed spin value is used by the endpoints for setting the spin
bit on outgoing packets.  This mechanism allows the endpoints
to generate a square wave such that, by measuring the distance in time
between pairs of consecutive edges observed in the same direction, a
passive on-path observer can compute the round trip delay of that
network flow.

Spin bit enables round trip latency measurement by observing a single direction
of the traffic flow.

Note that packet reordering can cause spurious edges that require heuristics to
correct. The spin bit performance deteriorates as soon as network impairments
arise as explained in {{delaybit}}.

## Delay Bit {#delaybit}

The delay bit, different from a two-bit VEC, has been designed to overcome
accuracy limitations experienced by the spin bit under difficult network
conditions:

* packet reordering leads to generation of spurious edges and errors
   in delay estimation;

* loss of edges causes wrong estimation of spin periods and therefore
   wrong RTT measurements;

* application-limited senders cause the spin bit to measure the
   application delays instead of network delays.

If enabled, delay bit has to be used in addition to the spin bit. Unlike the
spin bit, which is set in every packet transmitted on the network, the delay bit
is set only once per round trip.

When the delay bit is used, a single packet with a second marked bit (the delay
bit) bounces between a client and a server during the entire connection
lifetime.  This single packet is called "delay sample".

An observer placed at an intermediate point, observing a single direction of
traffic, tracking the delay sample and the relative timestamp in every spin
period, can measure the round trip delay of the connection.

The delay bit lifetime is comprised of two phases: initialization and
reflection.  The initialization is the generation of the delay sample, while the
reflection realizes the bounce behavior of this single packet between the two
endpoints.

The next figure describes the Delay bit mechanism: the first bit is
the spin bit and the second one is the delay bit.

~~~~
      +--------+   --  --  --  --  --   +--------+
      |        |       ----------->     |        |
      | Client |                        | Server |
      |        |      <-----------      |        |
      +--------+   --  --  --  --  --   +--------+

      (a) No traffic at beginning.

      +--------+   00  00  01  --  --   +--------+
      |        |       ----------->     |        |
      | Client |                        | Server |
      |        |      <-----------      |        |
      +--------+   --  --  --  --  --   +--------+

       (b) The Client starts sending data and
        sets the first packet as Delay Sample.

      +--------+   00  00  00  00  00   +--------+
      |        |       ----------->     |        |
      | Client |                        | Server |
      |        |      <-----------      |        |
      +--------+   --  --  01  00  00   +--------+

       (c) The Server starts sending data
        and reflects the Delay Sample.

      +--------+   10  10  11  00  00   +--------+
      |        |       ----------->     |        |
      | Client |                        | Server |
      |        |      <-----------      |        |
      +--------+   00  00  00  00  00   +--------+

      (d) The Client inverts the spin bit and
       reflects the Delay Sample.

      +--------+   10  10  10  10  10   +--------+
      |        |       ----------->     |        |
      | Client |                        | Server |
      |        |      <-----------      |        |
      +--------+   00  00  11  10  10   +--------+

      (e) The Server reflects the Delay Sample.

      +--------+   00  00  01  10  10   +--------+
      |        |       ----------->     |        |
      | Client |                        | Server |
      |        |      <-----------      |        |
      +--------+   10  10  10  10  10   +--------+

      (f) The client reverts the spin
       bit and reflects the Delay Sample.
~~~~
{: title="Spin bit and Delay bit"}

### Generation phase

Only client is actively involved in the generation phase.

When connection starts and spin bit is set to 0, the client initializes the
delay bit of the first packet to 1, so it becomes the delay sample for that
marking period.  Only this packet is marked with the delay bit set to 1 for this
round trip period; the other ones will carry the spin bit, while the delay bit
will be set to 0.

The server initialized the delay bit to 0 at the beginning of the connection,
and its only only task during the connection is described in
{{reflection-phase}}.

In absence of network impairments, the delay sample should bounce between client
and server continuously, for the entire duration of the connection.  That is
highly unlikely for two reasons:

1. the packet carrying the delay bit might be lost;

2. an endpoint could stop or delay sending packets because the application is
limiting the amount of traffic transmitted;

To deal with these problems, the algorithm provides a procedure named "recovery
process" to regenerate the delay sample and to inform a possible observer of the
problem so the measurement can be restarted.

#### The recovery process

Absent packet loss or reordering, every spin period ends with a delay sample.
If that does not happen and a spin period terminates without a delay sample, the
client waits a further spin period; then, it creates a new delay sample by
setting the delay bit to 1 on the first outgoing packet of the subsequent
period,

The spin period with all delay bits set to 0 informs observers that there was a
problem and delay measurements for this flow should be reset till the next delay
sample is received.

### Reflection phase   {#reflection-phase}

Reflection is the process that enables the bouncing of the delay
sample between a client and a server.  The behavior of the two endpoints
is slightly different.

* Server side reflection: when a delay sample arrives, the server marks the
   first packet in the opposite direction as the delay sample, if the outgoing
   packet has the same spin bit value as the delay sample.  Otherwise, the delay
   sample is ignored.

* Client side reflection: when a delay sample arrives, the client marks the
   first packet in the opposite direction as the delay sample, if the outgoing
   packet has the opposite spin bit value then the delay sample.  Otherwise, the
   delay sample is ignored.

In both cases, if the outgoing delay sample is being transmitted with a delay
greater than a predetermined threshold after the reception of the incoming delay
sample (1ms by default), the delay sample is not reflected, and the outgoing
delay bit is kept at 0.

Note that reflection takes place for the delay sample regardless of its position
within the spin period, as long as it stays within its original spin period.

A time threshold for the retransmission of the delay sample is used to eliminate
measurements that would overestimate the delay due to lack of traffic on the
endpoints.  Hence, the maximum estimation error would amount to twice the
threshold (e.g. 2ms) per measurement.

### Two bits delay measurement: Spin Bit + Delay Bit

When the Delay Bit is used, a passive observer can use delay samples directly
and avoid inherent ambiguities in the calculation of the RTT in spin bit
analysis, such as heuristic validation of the goodness of an edge signal.

#### RTT measurement

The delay sample generation process ensures that only one packet marked with the
delay bit set to 1 runs back and forth between two endpoints per round trip
time.  To determine the RTT measurement of a flow, an on-path passive observer
computes the time difference between two delay samples observed in a single
direction.

To ensure a valid measurement, the observer must identify spin periods in the
packet flow and assign delay samples to spin periods. If a spin period is
missing a delay sample, the measurement needs to be restarted from the
subsequent delay sample.


~~~~
           =======================|======================>
           = **********     -----Obs---->     ********** =
           = * Client *                       * Server * =
           = **********     <------------     ********** =
           <==============================================

                     (a) client-server RTT

           ==============================================>
           = **********     ------------>     ********** =
           = * Client *                       * Server * =
           = **********     <----Obs-----     ********** =
           <======================|=======================

                     (b) server-client RTT
~~~~
{: title="Round-trip time (both direction)"}


#### Half-RTT measurement

An observer that is able to observe both forward and return traffic directions
can use the delay samples to measure "upstream" and "downstream" RTT components,
also known as the half-RTT measurements. It does this by measuring the time
between a delay sample observed in one direction and the reflected delay sample
observed in the opposite direction.

As with RTT measurement, the observer must identify spin periods in the packet
flow and assign delay samples to spin periods. If a spin period is missing a
delay sample, the measurement needs to be restarted from the subsequent delay
sample.

Note that upstream and downstream sections of paths between the endpoints and
the observer, i.e. observer-to-client vs client-to-observer and observer-to-server
vs server-to-observer, may have different delay characteristics due to the
difference in network congestion and other factors.

~~~~
           =======================>
           = **********     ------|----->     **********
           = * Client *          Obs          * Server *
           = **********     <-----|------     **********
           <=======================

                  (a) client-observer half-RTT

                                  =======================>
             **********     ------|----->     ********** =
             * Client *          Obs          * Server * =
             **********     <-----|------     ********** =
                                  <=======================

                  (b) observer-server half-RTT
~~~~
{: title="Half Round-trip time (both direction)"}


#### Intra-domain RTT measurement

Intra-domain RTT is the portion of the entire RTT used by a flow to traverse the
network of a provider.  To measure intra-domain RTT, two observers capable of
observing traffic in both directions must be employed simultaneously at ingress
and egress of the network to be measured.  Intra-domain RTT is difference
between the two computed upstream (or downstream) RTT components.

~~~~
        =========================================>
        = =====================>
        = = **********      ---|-->           ---|-->      **********
        = = * Client *         Obs               Obs       * Server *
        = = **********      <--|---           <--|---      **********
        = <=====================
        <=========================================

                 (a) client-observer RTT components (half-RTTs)

                               ==================>
            **********      ---|-->           ---|-->      **********
            * Client *         Obs               Obs       * Server *
            **********      <--|---           <--|---      **********
                               <==================

                 (b) the intra-domain RTT resulting from the
                 subtraction of the above RTT components
~~~~
{: title="Intra-domain Round-trip time (client-observer: upstream)"}


### Observer's algorithm and Edge Rejection Interval

To provide a formal description of the observer behavior, we define a "matching
delay sample" to be a delay sample with the spin bit value that matched the spin
bit value of then-current spin period.

Upon detecting a matching delay sample, if a matching delay sample was also
detected in the previous period, then the two delay samples can be used to
calculate RTT measurement.

If the observer can observe both forward and return traffic flows, and it is
able to determine, by observing the spin bit, which direction contains the
client and the server:

* If matching delay samples have been detected in both directions in the current
  spin period, they can be used to measure the observer-server half-RTT.

* If a matching delay sample has been detected in client-to-observer direction,
  AND a matching delay sample had been detected in observer-to-client direction
  in the previous spin period, they can be used to measure the observer-client
  half-RTT.

The described observer behavior depends on the ability to accurately identify
current spin periods and to reject spurious spin edges, caused by packet
reordering. Failure to do so will lead to many missed measurement opportunities
and will decrease the amount of usable delay samples available to the observer.

To implement spurious edge rejection, every time a spin bit edge is detected,
the observer starts a new spin period and begins a time interval during which it
rejects spin edges.  This guarantees protection against spurious edges due to
packets that have been reordered by less than the time interval.  The mechanism
only works for intervals smaller than the RTT of the observed connection; if RTT
is smaller than the edge rejection interval, the observer cannot measure the
RTT.

# Loss Bits

This section introduces bits that can be used for loss measurements.
Whenever this section of the specification refers to packets, it is
referring only to packets with protocol headers that include the loss
bits -- the only packets whose loss can be measured.

* T: the "round Trip loss" bit is used in combination with the Spin bit to
  measure round-trip loss. See {{rtlossbit}}.

* Q: the "sQuare signal" bit is used to measure upstream loss. See
  {{squarebit}}.

* L: the "Loss event" bit is used to measure end-to-end loss. See {{lossbit}}.

* R: the "Reflection square signal" bit is used in combination with Q bit to
  measure end-to-end loss. See {{rtlossbit}}.

Loss measurements enabled by T, Q, and L the bits can be implemented by those
loss bits alone (T bit requires a working Spin Bit). Two-bit combinations Q+L
and Q+R enable additional measurement opportunities discussed below.

Each endpoint maintains appropriate counters independently and
separately for each separately identifiable flow (each sub-flow for
multipath connections).

Since loss is reported independently for each flow, all bits (except for L bit)
require a certain minimum number of packets to be exchanged per flow before any
signal can be measured. Therefore, loss measurements work best for flows that
transfer more than a minimal amount of data.

## T Bit -- Round Trip Loss Bit  {#rtlossbit}

The round Trip loss bit is used to mark a variable number of packets exchanged
twice between the endpoints realizing a two round-trip reflection. A passive
on-path observer, observing either direction, can count and compare the number
of marked packets seen during the two reflections, estimating the loss rate
experienced by the connection. The overall exchange comprises:

*  The client selects, generates and consequently transmits
   a first train of packets, by setting the T bit to 1;

*  The server, upon receiving each packet included in the first train, reflects
   to the client a respective second train of packets of the same size as the
   first train received, by setting the T bit to 1;

*  The client, upon receiving each packet included in the second train, reflects
   to the server a respective third train of packets of the same size as the
   second train received, by setting the T bit to 1;

*  The server, upon receiving each packet included in the third train, finally
   reflects to the client a respective fourth train of packets of the same size
   as the third train received, by setting the T bit to 1.

Packets belonging to the first round trip (first and second train)
represent the Generation Phase, while those belonging to the second
round trip (third and fourth train) represent the Reflection Phase.

A passive on-path observer can count and compare the number of marked
packets seen during the two round trips (i.e. the first and third
or the second and the fourth trains of packets, depending on which
direction is observed) and estimate the loss rate experienced by the
connection. This process is repeated continuously to obtain more
measurements as long as the endpoints exchange traffic.  These
measurements can be called Round Trip losses.

Since packet rates in two directions may be different, the number of marked
packets in the train is determined by the direction with the lowest packet rate.
See {{tbit-details}} for details on packet generation and for a mechanism to
allow an observer to distinguish between trains belonging to different phases
(Generation and Reflection).

### Round Trip Packet Loss Measurement

Since the measurements are performed on a portion of the traffic
exchanged between the client and the server, the observer calculates the end-
to-end Round Trip Packet Loss (RTPL) that, statistically, will correspond to
the loss rate experienced by the connection along the entire network path.

~~~~
           =======================|======================>
           = **********     -----Obs---->     ********** =
           = * Client *                       * Server * =
           = **********     <------------     ********** =
           <==============================================

                     (a) client-server RTPL

           ==============================================>
           = **********     ------------>     ********** =
           = * Client *                       * Server * =
           = **********     <----Obs-----     ********** =
           <======================|=======================

                     (b) server-client RTPL
~~~~
{: title="Round-trip packet loss (both direction)"}


This methodology also allows the Half-RTPL measurement and
the Intra-domain RTPL measurement in a way similar to RTT measurement.

~~~~
           =======================>
           = **********     ------|----->     **********
           = * Client *          Obs          * Server *
           = **********     <-----|------     **********
           <=======================

                  (a) client-observer half-RTPL

                                  =======================>
             **********     ------|----->     ********** =
             * Client *          Obs          * Server * =
             **********     <-----|------     ********** =
                                  <=======================

                  (b) observer-server half-RTPL
~~~~
{: title="Half Round-trip packet loss (both direction)"}


~~~~
                           =========================================>
                                             =====================> =
        **********      ---|-->           ---|-->      ********** = =
        * Client *         Obs               Obs       * Server * = =
        **********      <--|---           <--|---      ********** = =
                                             <===================== =
                           <=========================================

             (a) observer-server RTPL components (half-RTPLs)

                           ==================>
        **********      ---|-->           ---|-->      **********
        * Client *         Obs               Obs       * Server *
        **********      <--|---           <--|---      **********
                           <==================

             (b) the intra-domain RTPL resulting from the
             subtraction of the above RTPL components
~~~~
{: title="Intra-domain Round-trip packet loss (observer-server)"}

### Setting the round Trip loss bit on Outgoing Packets  {#tbit-details}

The round Trip loss signal requires a working Spin-bit signal to separate trains
of marked packets (packets with T bit set to 1).  A "pause" of at least one
empty spin-bit period between each phase of the algorithm serves as such
separator for the on-path observer.

The client is in charge of launching trains of marked packets and does so
according to the algorithm:

1.  Generation Phase. The client starts generating marked packets for two
    consecutive spin-bit periods; it maintains a "generation token" count that
    is reset to zero at the beginning of the algorithm phase and is incremented
    every time a packet arrives. When the client transmits a packet and a
    "generation token" is available, the client marks the packet and retires a
    "generation token". If no token is available, the outgoing packet is
    transmitted unmarked.  At the end of the first spin-bit period spent in
    generation, the reflection counter is unlocked to start counting incoming
    marked packets that will be reflected later;

2.  Pause Phase. When the generation is completed, the client pauses till it has
    observed one entire spin bit period with no marked packets.  That spin bit
    period is used by the observer as a separator between generated and
    reflected packets.  During this marking pause, all the outgoing packets are
    transmitted with T bit set to 0.  The reflection counter is still
    incremented every time a marked packet arrives;

3.  Reflection Phase. The client starts transmitting marked packets,
    decrementing the reflection counter for each transmitted marked packet until
    the reflection counter reached zero. The "generation token" method from the
    generation phase is used during this phase as well.  At the end of the first
    spin-period spent in reflection, the reflection counter is locked to avoid
    incoming reflected packets incrementing it;

4.  Pause Phase 2. The pause phase is repeated after the reflection phase and
    serves as a separator between the reflected packet train and a new packet
    train.

A server maintains a "marking counter" that starts at zero and is incremented
every time a marked packet arrives. When the server transmits a packet and the
"marking counter" is positive, the server marks the packet and decrements the
"marking counter". If the "marking counter" is zero, the outgoing packet is
transmitted unmarked.

### Observer's logic for round trip loss signal

The on-path observer counts marked packets and separates different trains by
detecting spin-bit periods (at least one) with no marked packets.  The Round
Trip Packet Loss (RTPL) is the difference between the size of the Generation
train and the Reflection train.

In the following example, packets are represented by two bits (first one
is the spin bit, second one is the loss bit):

~~~~
        Generation          Pause           Reflection       Pause
   ____________________ ______________ ____________________ ________
  |                    |              |                    |        |
   01 01 00 01 11 10 11 00 00 10 10 10 01 00 01 01 10 11 10 00 00 10

~~~~
{: title="Round Trip Loss signal example"}

Note that 5 marked packets have been generated of which 4 have been reflected.

### Loss Coverage and Signal Timing

A cycle of the round Trip loss signaling algorithm contains 2 RTTs of Generation
phase, 2 RTTs of Reflection phase, and two Pause phases at least 1 RTT in
duration each.  Hence, the loss signal is delayed by about 6 RTTs since the loss
events.

The observer can only detect loss of marked packets that occurs after its
initial observation of the Generation phase and before its subsequent
observation of the Reflection phase. Hence, if the loss occurs on the path that
sends packets at a lower rate (typically ACKs in such asymmetric scenarios),
`2/6` (`1/3`) of the packets will be sampled for loss detection (i.e. loss of
any of the other `2/3` of the packets on that path would be missed).

If the loss occurs on the path that sends packets at a higher rate,
`lowPacketRate/(3*highPacketRate)` of the packets will be sampled for loss
detection. For protocols that use ACKs, the portion of packets sampled for loss
in the higher rate direction during unidirectional data transfer is
`1/(3*packetsPerAck)`, where the value of `packetsPerAck` can vary by protocol,
by implementation, and by network conditions.

## Q Bit -- Square Bit {#squarebit}

The sQuare bit (Q bit) takes its name from the square wave generated by its
signal. Every outgoing packet contains the Q bit value, which is initialized to
the 0 and inverted after sending N packets (a sQuare Block or simply Q
Block). Hence, Q Period is 2*N. The Q bit represents "packet color" as defined
by {{AltMark}}.

Observation points can estimate upstream losses by watching a single direction
of the traffic flow and counting the number of packets in each observed Q Block,
as described in {{upstreamloss}}.

### Q Block Length Selection

The length of the block must be known to the on-path network probes.  There are
two alternatives to selecting the Q Block length. The first one requires that
the length is known a priori and therefore set within the protocol
specifications that implements the marking mechanism. The second requires the
sender to select it.

In this latter scenario, the sender is expected to choose N (Q Block
length) based on the expected amount of loss and reordering on the
path.  The choice of N strikes a compromise -- the observation could
become too unreliable in case of packet reordering and/or severe loss
if N is too small, while short flows may not yield a useful upstream
loss measurement if N is too large (see {{upstreamloss}}).

The value of N should be at least 64 and be a power of 2. This requirement allows
an Observer to infer the Q Block length by observing one period of the square
signal. It also allows the Observer to identify flows that set the loss bits to
arbitrary values (see {{ossification}}).

If the sender does not have sufficient information to make an informed decision
about Q Block length, the sender should use N=64, since this value has been
extensively tried in large-scale field tests and yielded good results.
Alternatively, the sender may also choose a random power-of-2 N for each flow,
increasing the chances of using a Q Block length that gives the best signal for
some flows.

The sender must keep the value of N constant for a given flow.

### Upstream loss {#upstreamloss}

Blocks of N (Q Block length) consecutive packets are sent with the same
value of the Q bit, followed by another block of N packets with an
inverted value of the Q bit.  Hence, knowing the value of N, an
on-path observer can estimate the amount of upstream loss after
observing at least N packets.  The upstream loss rate (`uloss`) is one
minus the average number of packets in a block of packets with the
same Q value (`p`) divided by N (`uloss=1-avg(p)/N`).

The observer needs to be able to tolerate packet reordering that can
blur the edges of the square signal, as explained in {{endmarkingblock}}.

~~~~
          =====================>
          **********     -----Obs---->     **********
          * Client *                       * Server *
          **********     <------------     **********

            (a) in client-server channel (uloss_up)

          **********     ------------>     **********
          * Client *                       * Server *
          **********     <----Obs-----     **********
                               <=====================

            (b) in server-client channel (uloss_down)
~~~~
{: title="Upstream loss"}

### Identifying Q Block Boundaries  {#endmarkingblock}

Packet reordering can produce spurious edges in the square signal. To address
this, the observer should look for packets with the current Q bit value up to X
packets past the first packet with a reverse Q bit value. The value of X, a
"Marking Block Threshold", should be less than `N/2` and, in practice, can be
set to `N/2-1`.

## L Bit -- Loss Event Bit {#lossbit}

The Loss Event bit uses an Unreported Loss counter maintained by the protocol
that implements the marking mechanism. To use the Loss Event bit, the protocol
must allow the sender to identify lost packets. This is true of protocols such
as TCP, SCTP, and QUIC and is not true of protocols such as UDP and IP/IPv6.

The Unreported Loss counter is initialized to 0, and L bit of every outgoing
packet indicates whether the Unreported Loss counter is positive (L=1 if the
counter is positive, and L=0 otherwise).

The value of the Unreported Loss counter is decremented every time a packet
with L=1 is sent.

The value of the Unreported Loss counter is incremented for every packet that
the protocol declares lost, using whatever loss detection machinery the protocol
employs. If the protocol is able to rescind the loss determination later, a
positive Unreported Loss counter may be decremented due to the rescission, but
it should NOT become negative due to the rescission.

This loss signaling is similar to loss signaling in {{ConEx}}, except the Loss
Event bit is reporting the exact number of lost packets, whereas Echo Loss bit
in {{ConEx}} is reporting an approximate number of lost bytes.

For protocols, such as TCP ({{TCP}}), that allow network devices to change data
segmentation, it is possible that only a part of the packet is lost. In these
cases, the sender must increment Unreported Loss counter by the fraction of the
packet data lost (so Unreported Loss counter may become negative when a packet
with L=1 is sent after a partial packet has been lost).

Observation points can estimate the end-to-end loss, as determined by the
upstream endpoint, by counting packets in this direction with the L bit equal to
1, as described in {{endtoendloss}}.

### End-To-End Loss    {#endtoendloss}

The Loss Event bit allows an observer to estimate the end-to-end loss rate by
counting packets with L bit value of 0 and 1 for a given flow. The end-to-end
loss rate is the fraction of packets with L=1.

The assumption here is that upstream loss affects packets with L=0 and L=1
equally. If some loss is caused by tail-drop in a network device, this may be a
simplification.  If the sender's congestion controller reduces the packet send
rate after loss, there may be a sufficient delay before sending packets with L=1
that they have a greater chance of arriving at the observer.

### Loss Profile Characterization {#loss-profile}

In addition to measuring the end-to-end loss rate, the Loss Event bit allows an
observer to characterize loss profile, since the distribution of observed
packets with L bit set to 1 roughly corresponds to the distribution of packets
lost between 1 RTT and 1 RTO before (see {{loss-correlation}}).  Hence,
observing random single instances of L bit set to 1 indicates random single
packet loss, while observing blocks of packets with L bit set to 1 indicates
loss affecting entire blocks of packets.

## L+Q Bits -- Upstream, Downstream, End-to-End, and Observer Loss Measurements

Combining L and Q bits allows a passive observer watching a single direction of
traffic to accurately measure:

*  upstream loss: sender-to-observer loss (see {{upstreamloss}})

*  downstream loss: observer-to-receiver loss (see {{downstreamloss}})

*  end-to-end loss: sender-to-receiver loss on the observed path (see
   {{endtoendloss}}) with loss profile characterization (see {{loss-profile}})

*  observer loss: loss in sender's measurement infrastructure (see
   {{observerloss}})

#### Correlating End-to-End and Upstream Loss   {#loss-correlation}

Upstream loss is calculated by observing packets that did not suffer the
upstream loss ({{upstreamloss}}). End-to-end loss, however, is calculated by
observing subsequent packets after the sender's protocol detected the loss.
Hence, end-to-end loss is generally observed with a delay of between 1 RTT (loss
declared due to multiple duplicate acknowledgments) and 1 RTO (loss declared
due to a timeout) relative to the upstream loss.

The flow RTT can sometimes be estimated by timing protocol handshake
messages. This RTT estimate can be greatly improved by observing a dedicated
protocol mechanism for conveying RTT information, such as the Spin bit (see
{{spinbit}}) or Delay bit (see {{delaybit}}).

Whenever the observer needs to perform a computation that uses both upstream and
end-to-end loss rate measurements, it should use upstream loss rate leading the
end-to-end loss rate by approximately 1 RTT. If the observer is unable to
estimate RTT of the flow, it should accumulate loss measurements over time
periods of at least 4 times the typical RTT for the observed flows.

If the calculated upstream loss rate exceeds the end-to-end loss rate calculated
in {{endtoendloss}}, then either the Q Period is too short for the amount of
packet reordering or there is observer loss, described in {{observerloss}}. If
this happens, the observer should adjust the calculated upstream loss rate to
match end-to-end loss rate.

#### Downstream Loss   {#downstreamloss}

Because downstream loss affects only those packets that did not suffer upstream
loss, the end-to-end loss rate (`eloss`) relates to the upstream loss rate
(`uloss`) and downstream loss rate (`dloss`) as `(1-uloss)(1-dloss)=1-eloss`.
Hence, `dloss=(eloss-uloss)/(1-uloss)`.

#### Observer Loss   {#observerloss}

A typical deployment of a passive observation system includes a network tap
device that mirrors network packets of interest to a device that performs
analysis and measurement on the mirrored packets. The observer loss is the loss
that occurs on the mirror path.

Observer loss affects upstream loss rate measurement, since it causes the
observer to account for fewer packets in a block of identical Q bit values (see
{{upstreamloss}}). The end-to-end loss rate measurement, however, is unaffected
by the observer loss, since it is a measurement of the fraction of packets with
the L bit value of 1, and the observer loss would affect all packets equally
(see {{endtoendloss}}).

The need to adjust the upstream loss rate down to match end-to-end loss rate as
described in {{loss-correlation}} is a strong indication of the observer loss,
whose magnitude is between the amount of such adjustment and the entirety of the
upstream loss measured in {{upstreamloss}}. Alternatively, a high apparent
upstream loss rate could be an indication of significant packet reordering,
possibly due to packets belonging to a single flow being multiplexed over
several upstream paths with different latency characteristics.

## R Bit -- Reflection Square Bit {#refsquarebit}

R bit requires a deployment alongside Q bit. Unlike the square signal for which
packets are transmitted into blocks of fixed size, the Reflection square signal
(being an alternate marking signal too) produces blocks of packets whose size
varies according to these rules:

*  when the transmission of a new block starts, its size is set equal
   to the size of the last Q Block whose reception has been completed;

*  if, before transmission of the block is terminated, the reception
   of at least one further Q Block is completed, the size of the block
   is updated to the average size of the further received Q Blocks.
   Implementation details follow.

The Reflection square value is initialized to 0 and is applied to the
R-bit of every outgoing packet.  The Reflection square value is
toggled for the first time when the completion of a Q Block is
detected in the incoming square signal (produced by the opposite node
using the Q-bit).  When this happens, the number of packets (`p`),
detected within this first Q Block, is used to generate a reflection
square signal which toggles every `M=p` packets (at first). This new
signal produces blocks of M packets (marked using the R-bit) and each
of them is called "Reflection Block" (R Block).

The M value is then updated every time a completed Q Block in the
incoming square signal is received, following this formula:
`M=round(avg(p))`.

The parameter `avg(p)` is the average number of packets in a marking
period computed considering all the Q Blocks received since the
beginning of the current R Block.

Looking at the R-bit, observation points have an indication of
losses experienced by the entire unobserved channel plus those occurred
in the path from the sender up to them.

Since the Q Block is send in one direction, and the corresponding reflected R
Block is sent in the opposite direction, the reflected R signal is transmitted
with the packet rate of the slowest direction. Namely, if the observed direction
is the slowest, there can be multiple Q Blocks transmitted in the unobserved
direction before a complete R Block is transmitted in the observed direction. If
the unobserved direction is the slowest, the observed direction can be sending R
Blocks of the same size repeatedly before it can update the signal to account
for a newly-completed Q Block.

### R+Q Bits -- Using R and Q Bits for Passive Loss Measurement

Since both square and Reflection square bits are toggled at most every
N packets (except for the first transition of the R-bit as explained
before), an on-path observer can count the number of packets
of each marking block and, knowing the value of N, can estimate the
amount of loss experienced by the connection.  An observe can calculate
different measurements depending on whether it is able to observe a single
direction of the traffic or both directions.

Single directional observer:

*  upstream loss in the observed direction: the loss between the sender and the
   observation point (see {{upstreamloss}})

*  "three-quarters" connection loss: the loss between the receiver and
   the sender in the unobserved direction plus the loss between the
   sender and the observation point in the observed direction

*  end-to-end loss in the unobserved direction: the loss between the
   receiver and the sender in the opposite direction

Two directions observer (same metrics seen previously applied to both
direction, plus):

*  client-observer half round-trip loss: the loss between the client
   and the observation point in both directions

*  observer-server half round-trip loss: the loss between the
   observation point and the server in both directions

*  downstream loss: the loss between the observation point and the
   receiver (applicable to both directions)


#### Three-quarters connection loss  {#tqloss}

Except for the very first block in which there is nothing to reflect
(a complete Q Block has not been yet received), packets are
continuously R-bit marked into alternate blocks of size lower or equal
than N.  Knowing the value of N, an on-path observer can estimate the
amount of loss occurred in the whole opposite channel plus the loss
from the sender up to it in the observation channel. As for the
previous metric, the `three-quarters` connection loss rate (`tqloss`) is
one minus the average number of packets in a block of packets with the
same R value (`t`) divided by `N` (`tqloss=1-avg(t)/N`).

~~~~
        =======================>
        = **********     -----Obs---->     **********
        = * Client *                       * Server *
        = **********     <------------     **********
        <============================================

            (a) in client-server channel (tqloss_up)

          ============================================>
          **********     ------------>     ********** =
          * Client *                       * Server * =
          **********     <----Obs-----     ********** =
                               <=======================

            (b) in server-client channel (tqloss_down)
~~~~
{: title="Three-quarters connection loss"}

The following metrics derive from this last metric and the upstream
loss produced by the Q Bit.

#### End-To-End loss in the opposite direction

End-to-end loss in the unobserved direction (`eloss_unobserved`) relates to the
"three-quarters" connection loss (`tqloss`) and upstream loss in the observed
direction (`uloss`) as `(1-eloss_unobserved)(1-uloss)=1-tqloss`.  Hence,
`eloss_unobserved=(tqloss-uloss)/(1-uloss)`.

~~~~
          **********     -----Obs---->     **********
          * Client *                       * Server *
          **********     <------------     **********
          <==========================================

            (a) in client-server channel (eloss_down)

          ==========================================>
          **********     ------------>     **********
          * Client *                       * Server *
          **********     <----Obs-----     **********

            (b) in server-client channel (eloss_up)
~~~~
{: title="End-To-End loss in the opposite direction"}


#### Half round-trip loss

If the observer is able to observe both directions of traffic, it is able to
calculate two "half round-trip" loss measurements -- loss from the observer to
the receiver (in a given direction) and then back to the observer in the
opposite direction.  For both directions, "half round-trip" loss (`hrtloss`)
relates to "three-quarters" connection loss (`tqloss_opposite`) measured in the
opposite direction and the upstream loss (`uloss`) measured in the given
direction as `(1-uloss)(1-hrtloss)=1-tqloss_opposite`.  Hence,
`hrtloss=(tqloss_opposite-uloss)/(1-uloss)`.

~~~~
        =======================>
        = **********     ------|----->     **********
        = * Client *          Obs          * Server *
        = **********     <-----|------     **********
        <=======================

      (a) client-observer half round-trip loss (hrtloss_co)

                               =======================>
          **********     ------|----->     ********** =
          * Client *          Obs          * Server * =
          **********     <-----|------     ********** =
                               <=======================

      (b) observer-server half round-trip loss (hrtloss_os)
~~~~
{: title="Half Round-trip loss (both direction)"}


#### Downstream loss

If the observer is able to observe both directions of traffic, it is able to
calculate two downstream loss measurements using either end-to-end loss and
upstream loss, similar to the calculation in {{downstreamloss}} or using "half
round-trip" loss and upstream loss in the opposite direction.

For the latter, `dloss=(hrtloss-uloss_opposite)/(1-uloss_opposite)`.

~~~~
                               =====================>
          **********     ------|----->     **********
          * Client *          Obs          * Server *
          **********     <-----|------     **********

             (a) in client-server channel (dloss_up)

          **********     ------|----->     **********
          * Client *          Obs          * Server *
          **********     <-----|------     **********
          <=====================

             (b) in server-client channel (dloss_down)
~~~~
{: title="Downstream loss"}


### Enhancement of R Block length computation

The use of the rounding function used in the M computation introduces errors
that can be minimized by storing the rounding applied each time M is computed,
and using it during the computation of the M value in the following R Block.

This can be achieved introducing the new `r_avg` parameter in the computation of
M.  The new formula is `Mr=avg(p)+r_avg; M=round(Mr); r_avg=Mr-M` where the
initial value of `r_avg` is equal to 0.

### Improved resilience to packet reordering

When a protocol implementing the marking mechanism is able to detect when
packets are received out of order, it can improve resilience to packet
reordering beyond what is possible using methods described in
{{endmarkingblock}}.

This can be achieved by updating the size of the current R Block while
this is being transmitted.  The reflection block size is then updated
every time an incoming reordered packet of the previous Q Block is
detected.  This can be done if and only if the transmission of the
current reflection block is in progress and no packets of the
following Q Block have been received.

# Summary of Delay and Loss Marking Methods

This section summarizes the marking methods described in this draft.

For the Delay measurement, it is possible to use the spin bit alone or
combined with the delay bit. A unidirectional or bidirectional
observer can be used.

~~~~
 +------------------+----+-------------------------+---------------+
 | Method           |# of|        Available        |               |
 |                  |bits|      Delay Metrics      |  Impairments  |
 |                  |    +------------+------------+  Resiliency   |
 |                  |    |   UNIDIR   |   BIDIR    |               |
 |                  |    |  Observer  |  Observer  |               |
 +------------------+----+------------+------------+---------------+
 |S: Spin Bit       | 1  | RTT        | x2         | lower         |
 |                  |    |            | Half RTT   |               |
 +------------------+----+------------+------------+---------------+
 |SD: Spin Bit +    | 2  | RTT        | x2         | higher        |
 |    Delay Bit     |    |            | Half RTT   |               |
 +------------------+----+------------+------------+---------------+

 x2 Same metric for both directions.
~~~~
{: #fig_summary_D title="Delay Comparison"}

For the Loss measurement, each row in the table of {{fig_summary_L}}
represents a loss marking method. For each method the table specifies
the number of bits required in the header, the available metrics using
an unidirectional or bidirectional observer, applicable protocols,
measurement fidelity and delay.

~~~~
 +-------------+-+-----------------------+-+------------------------+
 | Method      |B|        Available      |P|  Measurement Aspects   |
 |             |i|      Loss Metrics     |r+------------+-----------+
 |             |t|   UNIDIR  |   BIDIR   |t|  Fidelity  |   Delay   |
 |             |s|  Observer |  Observer |o|            |           |
 +-------------+-+-----------+-----------+-+------------+-----------+
 |T: Round Trip|$| RT        | x2        | | Rate by    | ~6 RTT    |
 |   Loss Bit  |1|           | Half RT   |*| sampling   +-----------+
 |             | |           |           | | 1/3 to 1/(3*ppa) of    |
 |             | |           |           | | Pkts over 2 RTT        |
 +-------------+-+-----------+-----------+-+------------+-----------+
 |Q: Square Bit|1| Upstream  | x2        |*| Rate over  | N Pkts    |
 |             | |           |           | | N Pkts     | (e.g. 64) |
 |             | |           |           | | (e.g. 64)  |           |
 +-------------+-+-----------+-----------+-+------------+-----------+
 |L: Loss Event|1| E2E       | x2        |#| Loss shape | Min: RTT  |
 |   Bit       | |           |           | | (and rate) | Max: RTO  |
 +-------------+-+-----------+-----------+-+------------+-----------+
 |QL: Square + |2| Upstream  | x2        | | -> see Q   | Up: see Q |
 |    Loss Ev. | | Downstream| x2        |#| -> see Q|L | Others:   |
 |    Bits     | | E2E       | x2        | | -> see L   |     see L |
 |             | | Observer  | x2        | | -> see Q   |           |
 +-------------+-+-----------+-----------+-+------------+-----------+
 |QR: Square + |2| Upstream  | x2        | | Rate over  | Up: see Q |
 |    Ref. Sq. | | 3/4 RT    | x2        | | N*ppa Pkts | Others:   |
 |    Bits     | | !E2E      | E2E       |*| (see Q bit |  N*ppa pk |
 |             | |           | Downstream| |   for N)   |   (see Q  |
 |             | |           | Half RT   | |            |    for N) |
 +-------------+-+-----------+-----------+-+------------+-----------+

 *   All protocols
 \#   Protocols employing loss detection
 $   Require a working Spin Bit.
 !   Metric relative to the opposite channel.
 x2  Same metric for both directions.
 ppa Packets-Per-Ack
 Q|L See Q if Upstream loss is significant; L otherwise
~~~~
{: #fig_summary_L title="Loss Comparison"}

# ECN-Echo Event Bit

While the primary focus of the draft is on exposing packet loss and
delay, modern networks can report congestion before they are forced to
drop packets, as described in [ECN].  When transport protocols keep
ECN-Echo feedback under encryption, this signal cannot be observed by
the network operators.  When tasked with diagnosing network
performance problems, knowledge of a congestion downstream of an
observation point can be instrumental.

If downstream congestion information is desired, this information can be
signaled with an additional bit.

* E: The "ECN-Echo Event" bit is set to 0 or 1 according to the Unreported ECN
  Echo counter, as explained below in {{ecnbit}}.

## Setting the ECN-Echo Event Bit on Outgoing Packets {#ecnbit}

The Unreported ECN-Echo counter operates identically to Unreported Loss counter
({{lossbit}}), except it counts packets delivered by the network with CE
markings, according to the ECN-Echo feedback from the receiver.

This ECN-Echo signaling is similar to ECN signaling in {{ConEx}}. ECN-Echo
mechanism in QUIC provides the number of packets received with CE marks. For
protocols like TCP, the method described in {{ConEx-TCP}} can be employed. As
stated in {{ConEx-TCP}}, such feedback can be further improved using a method
described in {{ACCURATE}}.

## Using E Bit for Passive ECN-Reported Congestion Measurement {#ech-usage}

A network observer can count packets with CE codepoint and determine the
upstream CE-marking rate directly.

Observation points can also estimate ECN-reported end-to-end congestion by
counting packets in this direction with a E bit equal to 1.

The upstream CE-marking rate and end-to-end ECN-reported congestion can provide
information about downstream CE-marking rate. Presence of E bits along with L
bits, however, can somewhat confound precise estimates of upstream and
downstream CE-markings in case the flow contains packets that are not
ECN-capable.


# Protocol Ossification Considerations  {#ossification}

Accurate loss and delay information is not critical to the operation of any protocol,
though its presence for a sufficient number of flows is important for the
operation of networks.

The delay and loss bits are amenable to "greasing" described in {{?RFC8701}}, if
the protocol designers are not ready to dedicate (and ossify) bits used for loss
reporting to this function. The greasing could be accomplished similarly to the
Latency Spin bit greasing in {{QUIC-TRANSPORT}}. Namely, implementations could
decide that a fraction of flows should not encode loss and delay information and,
instead, the bits would be set to arbitrary values. The observers would need to be
ready to ignore flows with delay and loss information more resembling noise
than the expected signal.


# Examples of application

## QUIC

The binding of the delay bit signal to QUIC is partially described in
{{QUIC-TRANSPORT}}, which adds the spin bit to the first byte
of the short packet header, leaving two reserved bits for future
experiments.

To implement the additional signals discussed in this document, the
first byte of the short packet header can be modified as follows:

*  the delay bit (D) can be placed in the first reserved bit (i.e.
   the fourth most significant bit _0x10_) while the loss bit (L) in the
   second reserved bit (i.e. the fifth most significant bit _0x08_);
   the proposed scheme is:

~~~~
          0 1 2 3 4 5 6 7
         +-+-+-+-+-+-+-+-+
         |0|1|S|D|L|K|P|P|
         +-+-+-+-+-+-+-+-+
~~~~
{: title="Scheme 1"}

*  alternatively, a two bits loss signal (QL or QR) can be placed in both
   reserved bits; the proposed schemes, in this case, are:

~~~~
          0 1 2 3 4 5 6 7
         +-+-+-+-+-+-+-+-+
         |0|1|S|Q|L|K|P|P|
         +-+-+-+-+-+-+-+-+
~~~~
{: title="Scheme 2A"}

~~~~
          0 1 2 3 4 5 6 7
         +-+-+-+-+-+-+-+-+
         |0|1|S|Q|R|K|P|P|
         +-+-+-+-+-+-+-+-+
~~~~
{: title="Scheme 2B"}

## TCP

The signals can be added to TCP by defining bit 4 of byte 13 of the
TCP header to carry the spin bit, and eventually bits 5 and 6 to carry
additional information, like the delay bit and the round-trip loss
bit, or a two bits loss signal (QL or QR).

# Security Considerations

Passive loss and delay observations have been a part of the network operations
for a long time, so exposing loss and delay information to the network does not
add new security concerns for protocols that are currently observable.

In the absence of packet loss, Q and R bits signals do not provide any
information that cannot be observed by simply counting packets transiting a
network path. In the presence of packet loss, Q and R bits will disclose
the loss, but this is information about the environment and not the endpoint
state. The L bit signal discloses internal state of the protocol's loss
detection machinery, but this state can often be gleamed by timing packets and
observing congestion controller response.

Hence, loss bits do not provide a viable new mechanism to attack data integrity
and secrecy.

## Optimistic ACK Attack

A defense against an Optimistic ACK Attack, described in {{QUIC-TRANSPORT}},
involves a sender randomly skipping packet numbers to detect a receiver
acknowledging packet numbers that have never been received. The Q bit signal may
inform the attacker which packet numbers were skipped on purpose and which had
been actually lost (and are, therefore, safe for the attacker to
acknowledge). To use the Q bit for this purpose, the attacker must first receive
at least an entire Q Block of packets, which renders the attack ineffective
against a delay-sensitive congestion controller.

A protocol that is more susceptible to an Optimistic ACK Attack with the loss
signal provided by Q bit and uses a loss-based congestion controller, should
shorten the current Q Block by the number of skipped packets numbers. For example,
skipping a single packet number will invert the square signal one outgoing
packet sooner.

Similar considerations apply to the R Bit, although a shortened R Block along
with a matching skip in packet numbers does not necessarily imply a lost
packet, since it could be due to a lost packet on the reverse path along with a
deliberately skipped packet by the sender.

# Privacy Considerations

To minimize unintentional exposure of information, loss bits provide an explicit
loss signal -- a preferred way to share information per {{!RFC8558}}.

New protocols commonly have specific privacy goals, and loss reporting must
ensure that loss information does not compromise those privacy goals. For
example, {{QUIC-TRANSPORT}} allows changing Connection IDs in the middle of a
connection to reduce the likelihood of a passive observer linking old and new
sub-flows to the same device. A QUIC implementation would need to reset all
counters when it changes the destination (IP address or UDP port) or the
Connection ID used for outgoing packets. It would also need to avoid
incrementing Unreported Loss counter for loss of packets sent to a different
destination or with a different Connection ID.


# IANA Considerations

This document makes no request of IANA.

# Change Log

TBD

# Contributors
The following people provided valuable contributions to this document:

* Marcus Ihlar, Ericsson, marcus.ihlar@ericsson.com

* Jari Arkko, Ericsson, jari.arkko@ericsson.com

* Emile Stephan, Orange, emile.stephan@orange.com


# Acknowledgements

TBD
