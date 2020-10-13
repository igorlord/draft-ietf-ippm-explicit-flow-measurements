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
    role: 
    name: Mauro Cociglio
    org: Telecom Italia
    street: Via Reiss Romoli, 274
    city: Torino
    code: 10148
    country: Italy
    email: mauro.cociglio@telecomitalia.it
  -
    ins: A. Ferrieux
    role: 
    name: Alexandre Ferrieux
    org: Orange Labs
    email: alexandre.ferrieux@orange.com
  -
    ins: G. Fioccola
    role: 
    name: Giuseppe Fioccola
    org: Huawei Technologies
    street: Riesstrasse, 25
    city: Munich
    code: 80992
    country: Germany
    email: giuseppe.fioccola@huawei.com
  -
    ins: I. Lubashev
    role: 
    name: Igor Lubashev
    org: Akamai Technologies
    email: ilubashe@akamai.com
  -
    ins: F. Bulgarella
    role: 
    name: Fabio Bulgarella
    org: Telecom Italia
    street: Via Reiss Romoli, 274
    city: Torino
    code: 10148
    country: Italy
    email: fabio.bulgarella@guest.telecomitalia.it
  -
    ins: I. Hamchaoui
    role: 
    name: Isabelle Hamchaoui
    org: Orange Labs
    email: isabelle.hamchaoui@orange.com
  -
    ins: M. Nilo
    role: 
    name: Massimo Nilo
    org: Telecom Italia
    email: massimo.nilo@telecomitalia.it
  -
    ins: R.Sisto
    role: 
    name: Riccardo Sisto
    org: Politecnico di Torino
    email: riccardo.sisto@polito.it
  -
    ins: D. Tikhonov
    role: 
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

Packet loss and delay are hard and pervasive problems of day-to-day network operation.
Proactively detecting, measuring, and locating them is crucial to maintaining high
QoS and timely resolution of crippling end-to-end throughput issues. To this
effect, in a TCP-dominated world, network operators have been heavily relying on
information present in the clear in TCP headers: sequence and acknowledgement
numbers and SACKs when enabled (see {{?RFC8517}}). These allow for quantitative
estimation of packet loss and delay by passive on-path observation. Additionally, the
problem can be quickly identified in the network path by moving the passive observer around.

With encrypted protocols, the equivalent transport headers are encrypted and
passive packet loss and delay observations are not possible, as described in
{{TRANSPORT-ENCRYPT}}.

Measuring loss and delay between similar endpoints cannot be relied upon to evaluate
encrypted protocol loss and  delay. Different protocols could be routed by the network
differently and the fraction of Internet traffic delivered using protocols other
than TCP is increasing every year.  So it is imperative to measure packet loss and delay
experienced by encrypted protocol users directly.

This document defines Explicit Flow Measurement Techniques, that are hybrid measurement 
path signals to be embedded into a transport layer protocol, explicitly intended 
for exposing RTT and loss rate information to measurement devices on-path. 
The hybrid measurements described here are aligned with the definition in {{IPM-Methods}}.
These measurement mechanisms are applicable to any transport-layer protocol, 
and, as an example it is reported how to bind the signals to a variety of 
IETF transport protocols, in particular to QUIC and TCP.

The Explicit Flow Measurement represent all the methods that make possible 
the measurements otherwise difficult with encrypted protocols and
therefore explicitly expose the information for on-path observers. 
The application of the spin bit to QUIC is described in
{{QUIC-TRANSPORT}} which adds the spin bit to QUIC for
experimentation purposes. The spin bit can be considered as 
the first example of Explicit Flow Measurement.

Following the recommendation in {{!RFC8558}} of making path signals explicit,
this document proposes adding a delay bit and/or one or two explicit loss bits 
to the clear portion of the protocol headers to restore network operators' ability 
to maintain high QoS. These bits can be added to an unencrypted portion of a header 
belonging to any protocol layer, e.g. IP (see {{IP}}) and IPv6 (see {{IPv6}}) headers or
extensions, such as {{IPv6AltMark}}, UDP surplus space (see {{UDP-OPTIONS}} and
{{UDP-SURPLUS}}), reserved bits in a QUIC v1 header (see {{QUIC-TRANSPORT}}).

The loss signal is not designed for use in automated control of the network in
environments where loss bits are set by untrusted hosts, Instead, the signal is
to be used for troubleshooting individual flows as well as for monitoring the
network by aggregating information from multiple flows and raising operator
alarms if aggregate statistics indicate a potential problem.

Note that spin bit, delay bit and loss bits explained in this
document are inspired by {{AltMark}}.  This is also mentioned
in {{?I-D.trammell-quic-spin}}, {{?I-D.trammell-tsvwg-spin}} 
and {{?I-D.trammell-ippm-spin}}.

Additional details about the Performance Measurements for
QUIC are also described in the paper {{ANRW19-PM-QUIC}}.

# Notational Conventions    {#conventions}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in {{!RFC2119}}.

# Delay Bits

This draft introduces two new bits that are to be present in packets
capable of delay reporting.  These are packets that include protocol
headers with support for the delay bits.  Whenever this specification
refers to packets, it is referring only to packets capable of delay
reporting.

{{QUIC-TRANSPORT}} introduces an explicit per-flow transport-layer
signal for hybrid measurement of RTT.  This signal consists of a 
spin bit which oscillates once per RTT.  It was also discussed 
the addition of a two-bit Valid Edge Counter (VEC), which
compensates for loss and reordering of the spin bit to increase
fidelity of the signal in less than ideal network conditions.

In this document it is introduced the delay bit, that is a single bit
signal that can be used together with the spin bit by passive
observers to measure the RTT of a network flow, avoiding the spin bit
ambiguities that arise as soon as network conditions deteriorate.


## Spin Bit

This section is a small recap of the spin bit working mechanism. For a
comprehensive explanation of the algorithm, please see the references
in bilbiography.

The latency spin bit is a single bit signal that toggles once per RTT,
enabling latency monitoring of a connection-oriented communication
from intermediate observation points.  Basically, client and server,
maintain an internal per-connection spin value (i.e. 0 or 1) used to
set the spin bit on outgoing packets for that connection.

The spin value is set to the appropriate value in accordance with the
following rules.  Both endpoints initialize the spin bit to 0 when a
new connection starts. Then:

*  when the client receives a packet that increases the higher packet
   number seen so far, it sets the connection spin value to the
   opposite value contained in the received packet;

*  when the server receives a packet that increases the higher packet
   number seen so far, it sets the connection spin value to the same
   value contained in the received packet;

The computed spin value is used by the endpoints for setting the spin
bit on outgoing packets.  This simple mechanism allows the endpoints
to generate a square wave such that, by measuring the distance in time
between pairs of consecutive edges observed in the same direction, a
passive on-path observer can compute the round trip delay of that
network flow.  Note that the packet number control is performed to
prevent reordered edges from being reflected by the endpoints.  The
spin bit performance deteriorate as soon as network impairments arise
as explained in {{delaybit}}.

## Delay Bit {#delaybit}

The delay bit has been designed to overcome the performance
limitations experienced by the spin bit under difficult network
conditions:

*  packet reordering leads to the generation of spurious edges which
   causes wrong estimation of delay;

*  loss of edges causes wrong estimation of spin periods width and
   therefore wrong RTT measurements;

*  application-limited senders cause the spin bit to measure no longer
   the delay of the network but the period of the application if the
   latter is larger than the former.

If enabled, it has to be used in addition to the spin bit. Unlike the
spin bit, which is actually set in every packet transmitted on the
network, the delay bit is set only once per round trip.

The main idea is to have a single packet, with a second marked bit
(the delay bit), that bounces between client and server during the
entire connection life.  This single packet is called Delay Sample.

A simple observer placed in an intermediate point, tracking the delay
sample and the relative timestamp in every spin bit period, can
measure the round trip delay of the connection.  In the same way 
as seen with the spin bit, it is possible to carry out other types
of measurements using this additional bit.  The next paragraphs
give an overview of the observer capabilities.

In order to describe the delay sample working mechanism in detail, we
have to distinguish two different phases which take part in the delay
bit lifetime: initialization and reflection.  The initialization is
the generation of the delay sample, while the reflection realizes the
bounce behavior of this single packet between the two endpoints.

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

During this first phase, endpoints play different roles.  First of
all a single delay sample must be bouncing per round trip period (and
so per spin bit period).  According to that statement and in order to
simplify the general algorithm, the delay sample generation is in
charge of just one of the two endpoints:

*  the client, when connection starts and spin bit is set to 0,
   initializes the delay bit of the first packet to 1, so it becomes
   the delay sample for that marking period.  Only this packet is
   marked with the delay bit set to 1 for this round trip period; the
   other ones will carry only the spin bit;

*  the server never initializes the delay bit to 1; its only task is
   to reflect the incoming delay bit into the next outgoing packet
   only if certain conditions occur.

Theoretically, in absence of network impairments, the delay sample
should bounce between client and server continuously, for the entire
duration of the connection.  Actually, that is highly unlikely mainly
for two different reasons:

1. the packet carrying the delay bit might be lost during its journey
on the network which is unreliable by definition;

2. one of the two endpoints could stop or delay sending data because
the application is limiting the amount of traffic transmitted;

To deal with these problems, the algorithm provides a procedure to
regenerate the delay sample and to inform a possible observer that a
problem has occurred, and then the measurement has to be restarted.

#### The recovery process

In order to relieve the server from tasks that go beyond the mere
reflection of the sample, even in this case the recovery process
belongs to the client.  A fundamental assumption is that a delay
sample is strictly related to its spin bit period.  Considering this
rule, the client verifies that every spin bit period ends with its
delay sample.  If that does not happen and a marking period
terminates without a delay sample, the client waits a further empty
period; then, in the following period, it reinitializes the mechanism
by setting the delay bit of the first outgoing packet to 1, making it
the new delay sample.  The empty period is needed to inform the
intermediate points that there was an issue and a new delay
measurement session is starting.

### Reflection phase

The reflection is the process that enables the bouncing of the delay
sample between client and server.  The behavior of the two endpoints
is slightly different.  With the exception of the client that, as
previously exposed, generates a new delay sample, by default the
delay bit is set to 0.

*  Server side reflection: when a packet with the delay bit set to 1
   arrives, the server marks the first packet in the opposite
   direction as the delay sample, if it has the same spin bit value.
   While if it has the opposite spin bit value this sample is
   considered lost.

*  Client side reflection: when a packet with delay bit set to 1
   arrives, the client marks the first packet in the opposite
   direction as the delay sample, if it has the opposite spin bit
   value.  While if it has the same spin bit value this sample is
   considered lost.

In both cases, if the outgoing marked packet is transmitted with a
delay greater than a predetermined threshold after the reception of
the incoming delay sample (1ms by default), reflection is aborted and
this sample is considered lost.

Note that reflection takes place for the packet that is carrying the
delay bit regardless of its position within the period.  For this
reason it is necessary to introduce that condition of validation in
order to identify and discard those samples that, due to reordering,
might move to a contiguous period.  Furthermore, by introducing a
threshold for the retransmission delay of the sample, it is possible
to eliminate all those measurements which, due to lack of traffic on
the endpoints, would be overestimated and not true.  Thus, the
maximum estimation error, without considering any other delays due to
flow control, would amount to twice the threshold (e.g. 2ms) per
measurement, in the worst case.

### Two bits delay measurement: Spin Bit + Delay Bit

Unlike what happens with the spin bit for which it is necessary to
validate or at least heuristically evaluate the goodness of an edge,
the delay sample can be used by an intermediate observer as a simple
demarcation between a period and the following one eliminating the
ambiguities on the calculation of the RTT found with the analysis of
the spin-bit only.  The measurement types, that can be done from the
observation of the delay sample, are exactly the same achievable with
the spin bit only.

#### RTT measurement

The delay sample generation process ensures that only one packet
marked with the delay bit set to 1 runs back and forth on the wire
between two endpoints per round trip time.  Therefore, in order to
determine the RTT measurement of a flow, an on-path passive 
observer can simply compute the time difference between two
delay samples observed in a single direction.  Note that a
measurement, to be valid, must take into account the difference 
in time between the timestamps of two consecutive delay samples
belonging to adjacent spin-bit periods.  For this reason, an
observer, in addition to intercepting and analysing the packets
containing the delay bit set to 1, must maintain awareness of each
spin period in such a way as to be able to assign each delay sample
to its period and, at the same time, identifying those periods that
do not contain it.

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

An on-path passive observer that is sniffing traffic in both
directions -- from client to server and from server to client -- can
also use the delay sample to measure "upstream" and "downstream" RTT
components.  Also known as the half-RTT measurement, it represents
the components of the RTT concerning the paths between the client
and the observer (upstream), and the observer and the server
(downstream).  It does this by measuring the delay between a delay
sample observed in the download direction and the one observed in
the upload direction, and vice versa.  Also in this case, it should
verify that the two delay samples belong to two adjacent periods, for
the upstream component, or to the same period for the downstream
component.

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

Taking advantage of the half-RTT measurements it is also possible to
calculate the intra-domain RTT which is the portion of the entire RTT
used by a flow to traverse the network of a provider (or part of
it).  To achieve this result two observers, able to watch traffic in
both directions, must be employed simultaneously at ingress and
egress of the network to be measured.  At this point, to determine
the delay between the two observers, it is enough to subtract the two
computed upstream (or downstream) RTT components.

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


The spin bit is an alternate marking generated signal and the only
difference than {{AltMark}} is the size of the alternation
that will change with the flight size each RTT.  So it can be useful
to segment the RTT and deduce the contribution to the RTT of the
portion of the network between two on-path observers and it can be
easily performed by calculating the delay between two or more
measurement points on a single direction by applying {{AltMark}}.

### Observer's algorithm and Waiting Interval

Given below is a formal summary of the functioning of the observer
every time a delay sample is detected.  A packet containing the delay
bit set to 1:

*  if it has the same spin bit value of the current period and no
   delay sample was detected in the previous period, then it can be
   used as a left edge (i.e. to start measuring an RTT sample), but
   not as a right edge (i.e. to complete and RTT measurement since the
   last edge).  If the observation point is symmetric (i.e. it can see
   both upload and download flows) and in the current period a delay
   sample was detected in the opposite direction (i.e. in the upload
   direction), the packet can also be used to compute the downstream
   RTT component.

*  if it has the same spin bit value of the current period and a
   delay sample was detected in the previous period, then it can be
   used at the same time as a left or right edge, and to compute RTT
   component in both directions.

Like stated previously, every time an empty period is detected, the
observer must restart the measurement process and consider the next
delay sample that will come as the beginning of a new measure, then
as a left edge.  As a result, being able to assign the delay sample
to the corresponding spin period becomes a crucial factor for the
proper functioning of the entire algorithm.

Considering that the division into periods is realized by exploiting
the spin bit square wave, it is easy to understand that the presence
of spurious spin edges -- caused by packet reordering -- would
inevitably lead the observer to overestimate the amount of periods
actually present in the transmission.  This results in a greater
number of empty periods detected and the consequent decrease of the
actual RTT samples achievable.  Therefore, in order to maximize the
performance of the whole algorithm, the observer must implement a
mechanism to filter out spurious spin edges.

To face this problem the waiting interval has to be introduced.
Basically, every time a spin bit edge is detected, the observer sets
a time interval during which it rejects every potential spurious
edges observed on the wire.  While, at the end of the interval it
starts again to accept changes in the spin bit value.  This
guarantees a proper protection against the spurious edges in relation
to the size of the interval itself.  For instance, an interval of 5ms
is able to filter out edges that have been reordered by a maximum of
5ms.  Clearly, the mechanism does its job for intervals smaller than
the RTT of the observed connection (if RTT is smaller than the
waiting interval the observer can't measure the RTT).

# Loss Bits

The draft introduces four bits that are to be present, in different
configurations, in packets capable of loss reporting.  These are
packets that include protocol headers with the loss bits.  Only loss
of packets capable of loss reporting is reported using loss bits.

Whenever this specification refers to packets, it is referring only to
packets capable of loss reporting.

* T: the "round Trip loss" bit, which depends on the Spin Bit, is set
  to 0 or 1 according to the algorithm discusses in {{rtlossbit}}.

* Q: the "sQuare signal" bit is toggled every N outgoing packets as
  explained below in {{squarebit}}.

* L: the "Loss event" bit is set to 0 or 1 according to the Unreported
  Loss counter, as explained below in {{lossbit}}.

* R: the "Reflection square signal" bit, which depends on the Q Bit,
  is used to reflect the incoming sQuare signal (the one generated by
  the opposite endpoint) according to the algorithm explained in
  {{rtlossbit}}; in a nutshell, it is used to report the losses found
  in the opposite transmission channel.

These bits can be implemented inside network protocol in different
configurations:

1. T Bit enables round-trip loss rate measurement using a single
   loss bit signal coupled with the Spin Bit.

2. Q Bit enables upstream loss measurement.
   
3. L Bit enables end-to-end loss signaling using an event bit
   to signal protocol losses.

4. Q + L, this option uses a two bits signal with one bit toggled
   every N outgoing packets and an event bit to signal protocol
   losses.

5. Q + R, uses a double square signal and {{AltMark}} to mark the
   whole traffic exchanged between endpoints.

The fourth and fifth solutions enable different types of measurements
providing a more complete picture of connection loss events.

Each endpoint maintains appropriate counters independently and
separately for each separately identifiable flow (each subflow for
multipath connections).

## Round Trip Loss Bit  {#rtlossbit}

The round trip loss bit is a single bit signal whose purpose is to
mark a variable number of packets which are exchanged two times
between the endpoints realizing a two round-trip reflection. A passive
on-path observer, placed on whatever direction, can trivially count
and compare the number of marked packets seen during the two
reflections estimating statistically the loss rate experienced by the
connection. The overall exchange comprises:

*  The client first selects, generates and consequent transmits to
   the server a first train of packets, by marking the loss bit to 1;

*  The server, upon reception from the client of each one of the
   packets included in the first train, reflects to the client a
   respective second train of packets of the same size as the first
   train received, by marking the loss bit to 1;

*  The client, upon reception from the server of each one of the
   packets included in the second train, reflects to the server a
   respective third train of packets of the same size as the second
   train received, by marking the loss bit to 1;

*  The server, upon reception from the client of each one of the
   packets included in the third train, finally reflects to the
   client a respective fourth train of packets of the same size as
   the third train received, by marking the loss bit to 1.

Packets belonging to the first round (first and second train)
represent the Generation Phase while those belonging to the second
round (third and fourth train) represent the Reflection Phase.

A passive on-path observer can count and compare the number of marked
packets seen during the two mentioned phases (i.e. the first and third
or the second and the fourth trains of packets, depending on which
direction is observed) and estimate the loss rate experienced by the
connection. This process is repeated continuously to obtain more
measurements as long as the endpoints exchange traffic.  These
measurements can be called Round Trip (RT) losses.

The general algorithm shown above gives an idea of its underlying
principles but is not enough to make the whole process working
properly.

Firstly, there is the issue that packet rates in the two directions
may be different.  Therefore, the right number of packets to be
marked has to be chosen in order to avoid their congestion on the
slowest traffic direction.  As a consequence, this number is
inevitably equal to the amount of packets transited, indeed, on the
slowest direction.  This problem can be easily addressed by a method
wherein the two endpoints of a communication exchange marked packets
interleaved with unmarked packets.  From an implementation point of
view, this result can be achieved by introducing a single token
system that adjusts the number of outgoing marked packets.
Basically, the token is enabled every time a packet arrives and
disabled when a marked packet is transmitted.  Since the creation of
the initial train of marked packets is carried out by the client, the
management and use of this single token is also assigned to it, which
in fact "calculates" the correct number of packets to be marked each
time.

Secondly, a mechanism to individually identify each train of packets
must be provided to enable the observer to distinguish between trains
belonging to different phases (Generation and Reflection).

### Round Trip Packet Loss Measurement

Since the measurements are performed on a portion of the traffic
exchanged between client and server, the observer calculates the end-
to-end Round Trip Packet Loss that, statistically, will be equal to
the loss rate experienced by the connection along the entire network
path.  So this measurement can be simply referred as the Round Trip
Packet Loss (RTPL).

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


In addition, this methodology allows the Half-RTPL measurement and
the Intra-domain RTPL measurement, in the same way as described in
the previous sections for RTT measurement.

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

### Setting the round Trip loss bit on Outgoing Packets

The round trip loss signal, whose basic mechanism was generalized in
the previous section, is implemented using just one bit: marked
packets have this bit set to 1, whereas unmarked ones have it set to
0.  This solution requires a working spin-bit signal used to separate
different trains of packets.  In particular, a "pause" of at least
one empty spin-bit period is introduced between each phase of the
algorithm.  An on-path observer can determine in this way if a phase
(and therefore a train of packets) is ended and a new one is
starting.

The client is in charge of almost the entire complexity of the
algorithm.  Its task can be summarized in 4 different points:

1.  The client starts generating marked packets for two consecutive
    spin-bit periods; it maintains a generation token that is enabled
    every time a packet arrives and disabled when another one is
    forwarded.  When this token is disabled, the generation process
    is paused (i.e. outgoing packets are transmitted unmarked) and
    resumes as soon as its value returns true, and that happens as
    soon as a packet is received.  In addition, at the end of the
    first spin-bit period spent in generation, the reflection counter
    is unlocked to start counting incoming marked packets which will
    be later reflected;

2.  When the generation is completed, the client waits to see in
    input an empty spin-bit period so as to be sure that everyone has
    seen at least that empty period.  This one will be used by the
    observer as a divider between generated and reflected packets.
    During this phase, all the outgoing packets are forwarded with
    the loss bit set to 0.  The reflection counter is still
    incremented every time a marked packet arrives;

3.  The client starts reflecting marked packets until the reflection
    counter is zeroed; the generation token is also used (in the same
    way) during this phase to avoid congestion on the slowest traffic
    direction.  In addition, at the end of the first spin-period
    spent in reflection, the reflection counter is locked to avoid
    incoming reflected packets incrementing it;

4.  When the reflection is completed, the client waits to see in
    input an empty spin-bit period so as to be sure that everyone has
    seen at least that empty period.  This one will be used by the
    observer as a divider between reflected and newly generated
    packets.  During this phase, all the outgoing packets are
    forwarded with the loss bit set to 0.  The whole process restarts
    going back to the first point.

As previously anticipated, the server simply reflects each incoming
marked packet sent by the client.  It maintains a simple counter that
is incremented every time a marked packet arrives and decremented
when a marked one is sent in the opposite direction.


### Observer's logic for round trip loss signal

The on-path observer, placed in any direction, counts marked packets
and separates different trains detecting empty spin-bit periods
between them (one or more).  Then, it simply computes the difference
between a Generation train and a Reflection train to produce a
statistical measurement of the Round Trip Packet Loss (RTPL) and of
the connection loss rate.

Here is an example.  Packets are represented by two digits (first one
is the spin bit, second one is the loss bit):

~~~~
        Generation          Pause           Reflection       Pause
   ____________________ ______________ ____________________ ________
  |                    |              |                    |        |
   01 01 00 01 11 10 11 00 00 10 10 10 01 00 01 01 10 11 10 00 00 10

~~~~
{: title="Round Trip Loss signal example"}

Note that 5 marked packets have been generated of which 4 reflected.

## Square Bit {#squarebit}

The sQuare bit (Q bit) takes its name from the square wave generated
by its signal. The square Value is initialized to the Initial Q Value
(0) and is reflected in the Q bit of every outgoing packet. The square
value is inverted after sending N packets (a sQuare Block or simply Q
Block). Hence, Q Period is 2*N. The Q bit represents "packet color" as
defined by {{AltMark}}. The square Bit can also be called an Alernate
Marking bit.

Observation points can estimate the upstream losses by counting the
number of packets during a half period of the square signal, as
described in {{upstreamloss}}.

### Q Block Length Selection

The length of the block must be known to the on-path network probes.
There are two possible solutions. The first one requires that the
length is known a priori and therefore set within the protocol
specifications that implements the marking mechanism. The second
instead requires the sender to select it.

In this last scenario, the sender is expected to choose N (Q Block
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
Alternatively, the sender may also choose a random N for each flow,
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
blur the edges of the square signal. A simple but effective solution
is explained in {{endmarkingblock}}.

The observer needs to differentiate packets as belonging to different
flows, since they use independent counters.

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

## Loss Event Bit {#lossbit}

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

The Loss Event bit allows an observer to calculate the end-to-end loss rate by
counting packets with L bit value of 0 and 1 for a given flow. The end-to-end
loss rate is the fraction of packets with L=1.

The assumption here is that upstream loss affects packets with L=0 and L=1
equally. If some loss is caused by tail-drop in a network device, this may be a
simplification.  If the sender's congestion controller reduces the packet send
rate after loss, there may be a sufficient delay before sending packets with L=1
that they have a greater chance of arriving at the observer.

### Using the L Bit in addition to the Q Bit for enhanced Passive Loss Measurement {#usage}

#### Correlating End-to-End and Upstream Loss   {#losscorrelation}

Upstream loss is calculated by observing packets that did not suffer the
upstream loss. End-to-end loss, however, is calculated by observing subsequent
packets after the sender's protocol detected the loss.  Hence, end-to-end loss
is generally observed with a delay of between 1 RTT (loss declared due to
multiple duplicate acknowledgements) and 1 RTO (loss declared due to a timeout)
relative to the upstream loss.

The flow RTT can sometimes be estimated by timing protocol handshake
messages. This RTT estimate can be greatly improved by observing a dedicated
protocol mechanism for conveying RTT information, such as the Latency Spin bit
of {{QUIC-TRANSPORT}}.

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

Observer loss affects upstream loss rate measurement since it causes the
observer to account for fewer packets in a block of identical Q bit values (see
{{upstreamloss}}). The end-to-end loss rate measurement, however, is unaffected
by the observer loss, since it is a measurement of the fraction of packets with
the set L bit value, and the observer loss would affect all packets equally
(see {{endtoendloss}}).

The need to adjust the upstream loss rate down to match end-to-end loss rate as
described in {{losscorrelation}} is a strong indication of the observer loss,
whose magnitude is between the amount of such adjustment and the entirety of the
upstream loss measured in {{upstreamloss}}. Alternatively, a high apparent
upstream loss rate could be an indication of significant reordering, possibly
due to packets belonging to a single flow being multiplexed over several
upstream paths with different latency characteristics.

## Reflection Square Bit {#refsquarebit}

Unlike the square signal for which packets are transmitted into blocks
of fixed size, the Reflection square signal (being an alternate
marking signal too) produces blocks of packets whose size varies
according to these simple rules:

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

Looking at the R-bit, observation points have clear indication of
losses experienced by the entire opposite channel plus those occurred
in the path from the sender up to them (if losses occur in this latter
portion of path).

### Determining the completion of an incoming Q Block   {#endmarkingblock}

A simple square bit transition cannot be used to determine the
completion of a Q Block.  Indeed, packet reordering can lead to the
generation of spurious edges in the square signal.  To address this
problem, a Q Block is considered ended when at least X packets (e.g.
5) with reverse marking (i.e. belonging to the following Q Block) have
been received. We can refer to this mechanism as "Marking Block
Threshold".

This same approach can be used by observation points to clean both
sQuare and Reflection square signals.

### Using the R Bit in addition to the Q Bit for enhanced Passive Loss Measurement

Since both square and Reflection square bits are toggled at most every
N packets (except for the first transition of the R-bit as explained
before), an on-path observer can trivially count the number of packets
of each marking block and, knowing the value of N, can estimate the
amount of loss experienced by the connection.  Different metrics can
be measured depending on which direction the observer is looking to.

One direction observer:

*  upstream loss: the loss between the sender and the observation
   point (see {{upstreamloss}})

*  "three-quarters" connection loss: the loss between the receiver and
   the sender in the opposite direction plus the loss between the
   sender and the observation point in the observed direction

*  end-to-end loss in the opposite direction: the loss between the
   receiver and the sender in the opposite direction

Two directions observer (same metrics seen previously applied to both
direction, plus):

*  client-observer half round-trip loss: the loss between the client
   and the observation point in both directions

*  observer-server half round-trip loss: the loss between the
   observation point and the server in both directions

*  downstream loss: the loss between the observation point and the
   receiver (applicable to both directions)


#### Three-quarters connection loss

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

Using the previous discussed metrics, end-to-end loss can be computed:

`eloss_down = tqloss_up - uloss_up`

`eloss_up = tqloss_down - uloss_down`

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

Using the previous metrics, the two half round-trip loss measurements
can be computed:

`hrtloss_co = tqloss_up - uloss_down`

`hrtloss_os = tqloss_down - uloss_up`

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

Using the previous metrics, downstream loss can be computed:

`dloss_up = hrtloss_os - uloss_down`

`dloss_down = hrtloss_co - uloss_up`

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

The use of the rounding function used in the M computation introduces
errors.  However, these errors can be minimized by storing the
rounding applied each time M is computed, and using it during the
computation of the M value in the following R Block.

This can be achieved introducing the new `r_avg` parameter in the
previous M formula.  The new formula is `M=round(avg(p)+r_avg)` where
`r_avg` is computed as not rounded M minus rounded M; its initial value
is equal to 0.

### Improvement of the resilience to out of sequence

Since endpoints have clear indication about reordered packets, we can
use this information to absorb out of sequences in the incoming square
wave, even when the marking block threshold (see {{endmarkingblock}})
has been reached.

This can be achieved by updating the size of the current R Block while
this is being transmitted.  The reflection block size is then updated
every time an incoming reordered packet of the previous Q Block is
detected.  This can be done if and only if the transmission of the
current reflection block is in progress and no packets of the
following Q Block have been received.

# Summary of Delay and Loss Marking Methods

This section summarizes the marking methods described in this draft.

For the Delay measurement, it is possible to use the spin bit alone or
combined with the delay bit. An unidirectional or bidirectional
observer can be used.

~~~~
 +------------------+----+-------------------------+---------------+
 | Method           |# of|        Available        |               |
 |                  |bits|      Delay Metrics      |  Impairments  |
 |                  |    +------------+------------+  Resiliency   |
 |                  |    |   UNIDIR   |   BIDIR    |               |
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
an unidirectional or bidirectional observer, and some notes on
measurement timing.

~~~~
 +------------------+----+-------------------------+---------------+
 | Method           |# of|        Available        |               |
 |                  |bits|      Loss Metrics       |    Timing     |
 |                  |    +------------+------------+    Aspects    |
 |                  |    |   UNIDIR   |   BIDIR    |               |
 +------------------+----+------------+------------+---------------+
 |T: Round Trip     | 1* | RT         | x2         | Every         |
 |   Loss Bit       |    |            | Half RT    | >= 6 RTT      |
 +------------------+----+------------+------------+---------------+
 |Q: Square Bit     | 1  | Upstream   | x2         | Every N Pkts. |
 |                  |    |            |            | (e.g. 64)     |
 +------------------+----+------------+------------+---------------+
 |L: Loss Event Bit | 1  | E2E        | x2         | Delayed of    |
 |                  |    |            |            | RTT > t > RTO |
 +------------------+----+------------+------------+---------------+
 |QL: Square +      | 2  | Upstream   | x2         | Down. and E2E |
 |    Loss Ev. Bits |    | Downstream | x2         | delayed as    |
 |                  |    | E2E        | x2         | above         |
 +------------------+----+------------+------------+---------------+
 |QR: Square +      | 2  | Upstream   | x2         | Reflect. Bit  |
 |    Ref. Sq. Bits |    | 3/4 RT     | x2         | delayed of a  |
 |                  |    | E2E^       | x2         | Q Block in    |
 |                  |    |            | Downstream | the opposite  |
 |                  |    |            | Half RT    | direction     |
 +------------------+----+------------+------------+---------------+

 *  Require a working Spin Bit.
 ^  Metric relative to the opposite channel.
 x2 Same metric for both directions.
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
   the fourth most significant bit _0x10_) while the loss bit in the
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

Passive loss and delay observations have been a part of the network operations for a long
time, so exposing loss and delay information to the network does not add new security
concerns for protocols that are currently observable.

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

A defence against an Optimistic ACK Attack, described in {{QUIC-TRANSPORT}},
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

The same considerations can be applied to the R Bit which however
mitigates the problem considering the fact that the R Block length is
also affected by the losses of the opposite side.


# Privacy Considerations

To minimize unintentional exposure of information, loss bits provide an explicit
loss signal -- a preferred way to share information per {{!RFC8558}}.

New protocols commonly have specific privacy goals, and loss reporting must
ensure that loss information does not compromise those privacy goals. For
example, {{QUIC-TRANSPORT}} allows changing Connection IDs in the middle of a
connection to reduce the likelihood of a passive observer linking old and new
subflows to the same device. A QUIC implementation would need to reset all
counters when it changes the destination (IP address or UDP port) or the
Connection ID used for outgoing packets. It would also need to avoid
incrementing Unreported Loss counter for loss of packets sent to a different
destination or with a different Connection ID.


# IANA Considerations

This document makes no request of IANA.

# Change Log

TBD

#Contributors
The following people provided relevant contributions to this document:

* Marcus Ihlar, Ericsson, marcus.ihlar@ericsson.com

* Jari Arkko, Ericsson, jari.arkko@ericsson.com

* Emile Stephan, Orange, emile.stephan@orange.com


# Acknowledgements

TBD