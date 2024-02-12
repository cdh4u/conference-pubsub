---
title: "Session Initiation Protocol (SIP) Conference Server for Publish/Subscribe"
abbrev: "SIP PubSub"
docname: draft-holmberg-conference-pubsub-latest
category: info

ipr: trust200902
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - Session Initiation Protocol
 - SIP
 - Publish
 - Subscribe
 - PubSub
 - Conference
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "cdh4u/conference-pubsub"
  latest: "https://cdh4u.github.io/conference-pubsub/draft-holmberg-conference-pubsub.html"

author:
 -
    ins: C. Holmberg
    fullname: Christer Holmberg
    organization: Ericsson
    email: "christer.holmberg@ericsson.com"

normative:

 RFC3261:
 RFC3264:
 RFC3265:
 RFC3389:
 RFC3550:
 RFC4353:
 RFC4575:
 RFC4579:
 RFC4585:
 RFC6263:
 RFC8428:
 RFC9071:

informative:


--- abstract

This document describes how a Session Initiation Protocol (SIP) Conference Server can be used to realize a Publish/Subscribe (PubSub) broker to distribute non-audiovisual data.


--- middle

# Terminology

Synchronization source (SSRC): The source of a stream of RTP packets, identified by a 32-bit numeric SSRC identifier carried in the RTP header so as not to be dependent upon the network address. All packets from a synchronization source form part of the same timing and sequence number space, so a receiver groups packets by synchronization source for playback. Examples of synchronization sources include the sender of a stream of packets derived from a signal source such as a microphone or a camera, or an RTP mixer. A synchronization source may change its data format, e.g., audio encoding, over time.  The SSRC identifier is a randomly chosen value meant to be globally unique within a particular RTP session. A participant need not use the same SSRC identifier for all the RTP sessions in a multimedia session; the binding of the SSRC identifiers is provided through RTCP. If a participantgenerates multiple streams in one RTP session, for example from separate video cameras, each MUST be identified as a different SSRC.

Contributing source (CSRC): A source of a stream of RTP packetsthat has contributed to the combined stream produced by an RTP mixer. The mixer inserts a list of the SSRC identifiers of the sources that contributed to the generation of a particular packet into the RTP header of that packet. This list is called the CSRC list. An example application is audio conferencing where a mixer indicates all the talkers whose speech was combined to produce the outgoing packet, allowing the receiver to indicate the current talker, even though all the audio packets contain the same SSRC identifier (that of the mixer).

Mixer: An intermediate system that receives RTP packets from one or more sources, possibly changes the data format, combines the packets in some manner and then forwards a new RTP packet. Since the timing among multiple input sources will not generally be synchronized, the mixer will make timing adjustments among the streams and generate its own timing for the combined stream. Thus, all data packets originating from a mixer will be identified as having the mixer as their synchronization source.

Translator: An intermediate system that forwards RTP packets with their synchronization source identifier intact. Examples of translators include devices that convert encodings without mixing, replicators from multicast to unicast, and application-level filters in firewalls.

~~~~ aasvg

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |V=2|P|X|  CC   |M|     PT      |       sequence number         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                           timestamp                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           synchronization source (SSRC) identifier            |
   +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
   |            contributing source (CSRC) identifiers             |
   |                             ....                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

~~~~
{: #fig-rtp-packet title='RTP Packet Header' artwork-align="center"}


# Introduction

This document describes how a Session Initiation Protocol (SIP) {{!RFC3261}} Conference Server can be used to realize a Publish/Subscribe (PubSub) broker to distribute non-audiovisual data, e.g., IoT sensor readings.

One main advantage of the solution is the possibility to use existing SIP- and RTP-based audiovisual conferencing infrastructure and protocols to realize data distributing using the Publish/Subscribe traffic pattern, instead of using dedicated Publish/Subscribe frameworks and protocols.

NOTE: Conference servers might behave differently depending on configuration, profiles etc. The procedures in this document are based on a generic assumptions on how conference servers behave.

The examples in this document use the RTP T.140 real-time text (RTT) payload format {{!RFC4103}} to transport the payload data, as it will allow to use a conference server that supports the payload format to act as a PubSub broker. Sen



and SenML to structure and encode the payload data. Other payload formats and
encodings can also be used.

While RTP is a generic transport protocol, the main usage has been for transport of real-time AudioVisual data. Non-AudioVisual data has typically been data associated with AudioVisual data, e.g., real-time text (rtt), DTMF signals. However, at the time of writing this document, IETF is working on a number of specifications where RTP is used to transport other types of non-AV data.


## Publish/Subscribe

Publish/Subscribe (PubSub)


When a publisher publishes data it associates it with a topic. The topic typcially describes the semantics of the data (e.g., "water-temperature-data") or identifies the publsiher (e.g., "water-pump-123"). The structure and syntaxof the topic depends on the Pub/Sub framework. Some PubSub frameworks define tree-structured topics (e.g. "factory/temperature/sensor-123"), and allow topic wildcarding (e.g., "factory/temperature/*"). This document does not define topic syntax or structure. Topics are simply seen as token or string values.

Subscribers that are interested in data assocoated with the topic will subscribe to the topic. The Pub/Sub framework will then route published
data to each subscriber that has subscribed to the topic. The subscriptions and routing of published data is often handled by an intermediary function, often called a broker. Publishers will publish data to the broker, and the broker will forward the data to each subscriber that has subscribed to the topic associated with the data.


Many Pub/Sub frameworks.

Some Pub/Sub frameworks do not use a broker (broker-less Pub/Sub), but rather relies on other mechanisms (e.g., IP multicast) to route published messages from publishers to subscribers. Broker-less Pub/Sub is outside the scope of this document.



~~~~ aasvg

   .-----------.          .----------.  subscribe  .------------.
   |           |   data   |          |<------------+            |
   | Publisher +--------->+          |    data     | Subscriber |
   |           |          |          +------------>|            |
   '-----------'          |          |             '------------'
        ...               |  Broker  |                ...
        ...               |          |                ...
   .-----------.          |          |  subscribe  .------------.
   |           |   data   |          |<------------+            |
   | Publisher +--------->+          |    data     | Subscriber |
   |           |          |          +------------>|            |
   '-----------'          '----------'             '------------'
~~~~
{: #fig-arch-pubsub title='Publish/Subscribe Architecture' artwork-align="center"}


~~~~
=> 0.01 GET
   Uri-Path: ps
   Uri-Path: h9392

<= 2.05 Content
   Content-Format: TBD2 (application/core-pubsub+cbor)
   {
      "topic-name" : "living-room-sensor",
      "topic-data" : "ps/data/1bd0d6d",
      "resource-type": "core.ps.conf",
      "media-type": "application/senml-cbor",
      "topic-type": "temperature",
      "expiration-date": "2023-04-00T23:59:59Z",
      "max-subscribers": 100
   }
~~~~


# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the SIP conference terminology defined in {{!RFC4353}}.

Publish/Subscribe (PubSub):
: A message communication model where messages associated with specific topics are sent to a broker. Interested parties, i.e. subscribers, receive these topic-based messages from the broker without the original sender knowing the recipients. The broker handles matching and delivering these messages to the appropriate subscribers.

Publisher:
: A xxx. Within the scope of this document, a Publisher is a conference participant that sends data towards a conference server using RTP.

Subscribe:
: A xxx. Within the scope of this document, a Subscriber is a conference participant that receives data from a conference server using RTP.

NOTE: A conference participant might act both as Publisher and Subscriber.

Topic:
: A xxx. A Topic typically refers to the semantic of the published data (e.g., 'water-temperature'), but it might also refer to the Publisher (e.g., 'water-pump-123') or a combination (e.g., 'water-pump-123/water-temperature'). Topics might also have a tree structure (e.g., 'factory/line-2/water-pump-123/water-temperature'), or allow wildcarding (e.g., 'factory/line-2/water-pump-123/*'). Some systems might use a standardized naming scheme and structure for the Topics, while other systems allow applications to define their own Topic namings and structures. This document does not mandate or define a specific naming or structure for Topcis, and the Topics shown examples are only examples.

Broker:
: An intermediary function that receives published data from Publishers, and forwards it to Subscribers. Within the scope of this document, the Broker is a conference server that supports SIP/SDP signalling and RTP data transport. Note that even if other signalling protocols than SIP/SDP are used, the RTP and data handling considerations within this document might still apply.

Published data
: Data that a Publisher sends (publishes), and a Subscriber receives, for a Topic. Within the scope of this document, the published data is the RTP payload {{!RFC3550}}, i.e., the data transported in an RTP packet.

Conference:
: A xxx

AudioVisual Conference/AV Conference:
: A "traditional" SIP conference, where human participants send and receive speech audio, and in some cases video. The conference server has different policies for distributing audio and video. In a typcial sceanio it mixes and forwards the audio of all participants, while forwarding the video of the currently speaking conference participant (referred to as "current speaker").

PubSub Conference:
: A SIP conference used to realize data distribution using the Publish/Subscribe traffic pattern, following the procedures in this document.

PubSub Conference Participant/PubSub Participant:
: An endpoint that has joined (or is about to join) a PubSub Conference. Within the PubSub Conference, the PubSub Participant will take the role as a Publisher, Subscriber or both.


NOTE: There are more and more use-cases where both AudioVisual data and non-AudioVisual data is exchanged within the same conference. The description of a mixed AudioVisual/PubSub conference is outside the scope of this document.

AudioVisual Data:
: Data that contains encoded audio or video.

Network Time Protocol (NTP): The absolute time in seconds relative to midnight UTC on 1 January 1900.



# Generic Conference Considerations {#sec-conf-considerations-generic}

This Section discusses the major differences between an AV Conference and a PubSub Conference.

## Join Conference

A conference server might host multiple conferences that share the same conference name (Subject). Since the name is just a human readable string value, it is not uncommon that multiple AV conferences share the same name. Each conference will obviously have a unique conference URI.

It is not practical to host multiple PubSub conferences that share the same Topic. Because of that, if a PubSub participant tries to create a PubSub conference with a Topic for which there already exist a conference, the conference server might choose to either reject the conference creation request (and inform the endpoint about the existing conference), redirect the participant to the existing conference (using a SIP 3xx response code {{!RFC3261}}) or simply add the endpoint to the existing conference.

### Conference Duration

The duration of an AV conference may vary, but is typcially measured in minutes or hours. An AV conference is typically terminated when the last participant has left the conference.

The interest for a PubSub topic might last for a very long time. Because of that, a PubSub conference associated with the topc might last for days, months or "infinite". While there might be times when there are no PubSub participants within a conference, the conference server might still keep the PubSub conference "alive", as new participants are expected to join in the near future. One advantage of keeping the conference "alive" is that participants can use the same conference URI whenever they are re-joining the conference.

### Conference Termination

An AV conference is typically terminated once the last participant leaves the conference, when the conference createer leaves the conference, or at a pre-configured clock time.

Within a PubSub conference, participants might join the conference only when they want to send or receive data. In between they might leave the conference. Because of this there might be periods when there are no participants wihtin the conference. However, as participants might re-join later, and new participants might join the conference. Therefore, as long as it can be assumed that there is an interest in the Topic associated with the conference, the conference might be kept alive even if there are no participants.

### Sending and Receiving Data

Within an AV Conference, participants typically both send and receive media. This can also be the case in a PubSub Conference, if the Publish/Subscribe traffic pattern is used to realize bi-directional data exchange between conference participants. However, typically participants within a PubSub Conference will either send (Publish) or receive (Subscribe) data. For example, sensors will typically only publish data, while analyticsetc applications will only subscribe to data.

Note that a PubSub Conference participant might publish data to one Topic, while subscribing to another Topic.

### Simultanous Senders

Within an AV Conference typically only one participant sends speech audio at any given time. Within a PubSub Conference, multiple Publishers might publish data simultaneously, as data is often published as soon as it becomes available. In addition, a Publisher typically has no idea when other Publishers are publishing data. A Publishers might not even be aware of the other Publishers (if any). Because of this, the conference server might receive published data from multiple Publishers simultaneously. Depending on how quickly the conference server can forward the data to the subscribers it might have to buffer the data, which can cause delays.

### Number of Conference Participants

Within an AV Conference, the number of participants is relatively constant throughout the lifetime of the conference. The number of participants might also be known before the conference begins, e.g., based on the number of participants that have accepted an invitation to the conference.

Within a PubSub Conference, the number of participants might vary widely throughout the lifetime of the conference. A Publisher might join a conference only when it is publishing data, then leave the conference and re-join later again. If a Subscriber is only interested in receiving data at specific times, it might also join the conference only for those time.

### Data Sending Frequency

In an AV Conference, audio and video data is typically sent constantly, eventhough there are ways to temporarily stop the sending of data (e.g., by turning off the camera, muting the microphone etc).

In a PubSub Conference, the data publishing frequency can vary widely. In some cases, a Publisher will publish data very frequently (measured in milliseconds). In other cases, a Publisher might publish data more seldom: once a minute, once an hour, once a day, etc.

### Data Synchronization

Within an AV Conference, if the conference server is mixing the audio from all participants, the conference server needs to be able to ensure that the audio packets that are mixed together have been generated at the same time. The conference server does not necessarily need to know that real clock value, only that the packets have been generated at the same time.

Within a PubSub Conference, the conference server will not mix data from different Publishers. In some case, for network optimization purpose, the conference server might forward data from multiple Publishers in a single packet towards the Subscribers.


# SIP Signalling Considerations {#sec-conf-considerations-sip-signalling}

The discussions within this Section are based on the procedures and concepts defined in {{!RFC4353}} and {{!RFC4579}}.

## SIP Subject Header Field {#sec-conf-considerations-sip-subject}

The SIP Subject header field can be used to indicate the Topic associated with the PubSub Conference. When a new new conference is created (using SIP signalling) the conference creator uses the Subject header field to indicate the Topic of the conference.

~~~~ aasvg

pubsub-conferences-info
     |
     |-- pubsub-conferences
          |-- pubsub-conference
          |    |-- conference-URI
          |    |-- topic
          |-- pubsub-conference
          .    |-- conference-URI
          .    |-- topic
          .

~~~~
{: #fig-sip-subject title='SIP Subject header field for Topic' artwork-align="center"}


# SDP Considerations {#sec-conf-considerations-sdp}

## SDP Direaction Attribute {#sec-sdp-considerations-dir-attr}

These attributes can be used to indicate the PubSub role of a participant.

A participant can use SDP 'sendonly' attribute to indicate that it is acting as a Publisher.

A participant can use SDP 'recvonly' attribute to indicate that it is acting as a Subscriber.

A participant can use SDP 'sendrecv' attribute to indicate that it is acting as both a Publisher and a Subscriber.

A participant can use SDP 'inactive' attribute to indicate that it is not acting as a Publisher nor a Subscriber. A participant can use the attribute e.g., to temporary indicate to the conference server that it does not want to receive data associated with the conference, but also to indicate that it will not send data associated with the conference.

# SIP Event Package Considerations {#sec-conf-considerations-sip-event}

This section describes extensions for the SIP Event Package for Conference State {{!RFC4575}} that are useful for a PubSub Conference. In addition, this section describes a new SIP Event Package, SIP Event Package for PublishSubscribe, that can be used to inform participants about the PubSub Conferences hosted by a conference server, e.g., the Topic and conference-uri associated with each conference.
 
## SIP Event Package for Conference State {#sec-conf-considerations-sip-event-conf-state}

{{!RFC4575}} defines a SIP Event Package {{!RFC3265}} for Conference State. A conference participant can subscribe to the event package, and retrieve conferance state information, including information about the conference itsel, and information about other conference participants. For a PubSub conference, the "subject" child element of the "conference-description" element can be used to indicate the Topic associated with the conference.

### Extensions for PubSub

#### Maximum Number of Conference Participants

The "maximum-user-count" element is used to indicate the maximum number of conference participants. For an AudioVisual conference, where participants typcially both send and receive media a single element will be enough. However, in a PubSub conference, a majority of the conference participants might be either subscribers or publishers. There might be a large variation in how many publishers and how mnay subscribers a conference server is able to handle. Therefore it could be useful to have separate elements to indicate that, e.g., "maximum-user-count-publisher" and "maximum-user-count-subscriber".

## SIP Event Package for PublishSubscribe {#sec-conf-considerations-sip-event-pubsub}

While the SIP Event Package for Conference State provides information and state information for a given conference, it does not provide information about other conferences that are hosted by the conference server.

This section suggests a new SIP Event Package, SIP Event Package for PublishSubscribe. The event package is not assoiciated with a specific PubSub conference, but provides information about the PubSub conferences hosted by the conference server. For each PubSub conference hosted by conference server, the event package contains the conference URI and the associatd topic. It might also contain additional information about each PubSub conference. In addition, if the topic is associated with some namespace or dictionary, there might be information about that.

~~~~ aasvg

pubsub-conferences-info
     |
     |-- pubsub-conferences
          |-- pubsub-conference
          |    |-- conference-URI
          |    |-- topic
          |-- pubsub-conference
          .    |-- conference-URI
          .    |-- topic
          .

~~~~
{: #fig-sip-eventpackage-pubsub title='SIP Event Package for PublishSubscribe' artwork-align="center"}

### Example

~~~~ aasvg

   <?xml version="1.0" encoding="UTF-8"?>
   <pubsub-conferences-info
    xmlns="urn:ietf:params:xml:ns:pubsub-conferences-info"
    entity="sips:confserver@example.com"
    state="full" version="1">
   <!--
     PUBSUB CONFERENCES
   -->
    <pubsub-conferences>
     <pubsub-conference entity="sip:pubsubconf123@example.com" state="full">
      <topic>water temperature</topic>
     </pubsub-conference>
     <pubsub-conference entity="sip:pubsubconf456@example.com" state="full">
      <topic>air temperature</topic>
     </pubsub-conference>
    </pubsub-conferences>
   </pubsub-conferences-info>

NOTE: The conference-uri is defined as an pubsub-conference element attribute. As an option, it could be defined as a separate element.

NOTE: As an option, the topic could also be defined as an pubsub-conference element attribute.

~~~~
{: #fig-sip-pubsup-event title='Example: SIP Event Package for PublishSubscribe' artwork-align="center"}



## RTP Considerations {#sec-conf-considerations-rtp}

### RTP Timestamp

The RTP Timestamp, together with the RTCP SR (Sending Report) can be used by Conference servers and participants to synchronize audio and video received from multiple participants.

Note that the RTCP SR messages might be terminated by the Conference server.

Google RTP extension

#### Payload

In this case, the sampling timestamp is carried in the payload data, instead of the RTP/RTCP packets. The disadvantage of this mechanim is that one needs to ensure that the data payload format always supports the transport of the sampling timestamp.



### Keep-alive and Heartbeat

In a non-AV PubSub Conference, Publishers might stay within a PubSub conference for a long periods without publishing any data. Subscribers will not publish any data at all. There are a couple of possible implications that must be considered: NAT binding keep-alives and heartbeats (i.e., inform other PubSub Participants that a PubSub Participant is still 'alive'). NAT binding keep-alives are needed in cases where a PubSub Participants needs to send periodic data in order to maintain NAT bindings between itself and the PubSub Conference servers. This is important for Subscribers, but also for Publishers that publish data with very long intervals.

{{!RFC6263}} describes different RTP/RTCP mechanisms to send NAT keep-alives.

The 'Empty (0-Byte) Transport Packet' mechanism does not use RTP/RTCP. Instead, a participant will send an empty transport packet (e.g., UDP packet).  Note that this mechanism is useful mainly for NAT traversal purpose. The conference server application will typcially not be informed about these packets, and will not forward the packets to other conference particiapants. This mechanism is applicable to non-AV data in PubSub Conferences.

The 'RTP Packet with Comfort Noise Payload' mechanism uses a specific RTP payload format for comfort noise {{!RFC3389}}. The payload format has been defined for audio data, and must be supported by both senders and receivers. Is not applicable for non-AV data in PubSub conferences.

The 'RTCP Packets Multiplexed with RTP Packets' mechanism uses RTP/RTCP multiplexing, where the same 5-typle is used for the RTP and RTCP packets. When there is no data to be sent, an RTCP packet can be sent as a keep-alive. Note that Subscribers that do not send RTP packets can still send RTCP packets.

The 'RTP Packet with Incorrect Version Number' and 'RTP Packet with Unknown Payload Type' mechanisms uses invalid with an unvalid RTP version number respectively a RTP packet with a non-negotiated payload type. Receivers are expected to ignore and discard these types of RTP packets. This mechanism is applicable to non-AV data in PubSub Conferences.



In an AV conference, most participants will typically both send and receive data. However, in a PubSub Conference many of the PubSub Participants will only send data (Publihser) or receive data (Subscriber). However, eventhough a Subsriber does not publish any data, it might still use the mechanisms above if needed, and a PubSub Conference Server needs to be prepared to receive such RTP/RTCP packets from both Publishers and Subscribers.

NOTE: One of the ideas behind the Publish/Subsribe traffic pattern is that Publishers and Subscribers do not need to be aware of each other. In such cases there is no need to use an end-to-end heartbeat mechanism between Publishers and Subscribers. Note that there might still be a heartbeat mechanism used between Publishers/Subsribers and the Broker. However, there might be cases where Publishers and Subscribers are tightly coupled, and where a heartbeat mechanism is required.

NOTE: Some data formats might define their own methods for sending heartbeats. For example, there might a way to indicate that the payload is only used for heartbeat purpose, and does not contain any additional data.






### Data Aggregation



NOTE: SenML supports aggregation. Mutli

## RFC 4103 Consierations {#sec-conf-considerations-4103}

https://datatracker.ietf.org/doc/html/rfc4103

{{!RFC4103}} defines an RTP payload format ('text/t140') to transport T.140 real-time text. Within this document, it is assumed that the t140 payload format is used in the RTP packets to transport the published data. The reason for this is the possibility to re-use existing conference servers that support the payload format to realize the Broker.

### Redundancy

https://datatracker.ietf.org/doc/html/rfc2198

By default, when no other redundancy mechanism is supported, the usage of the redundancy mechanism defined in {{!RFC2198}} is required, unless the network conditions can guarantee that all text will always be delivered from the sender to the receiver. Additional mechanisms, e.g., Forward Error Correction (FEC) {{!RFC2733}} might also be used.

In a Publish/Subscribe network, the data delivery requirements will determine whether redundancy or error correction mechanism are needed. In some cases, data loss might be toleraded, while in other cases there might be requirements that all published data reaches each Subscriber that has subscribed to the Topic of the data.

### Packet Loss Detection

As t140 packets are only sent when there is new text to send. Because of that, as the time between sent packets may vary, the Timestamp cannot be used by a receiver to detect packet loss. Instead, the Sequence Number (SN) is used to detect packet loss.

Note that, if there is no new text to send, an RTP packet that only contains redundant data might be sent. This can be useful to e.g., maintain NAT bindings, and as a generic heartbeat mechanism to indicate that the sender is still alive.






### RFC 9071 Considerations {#sec-conf-considerations-9071}

{{!RFC9071}} provides guidance on mixing of real-time text (RTT) by a conference server. This sections discusses considerations to take into account when T.140 is used to transport Publish/Subscribe data. Note that some of the considerations have also been addressed from a generic conference perspective.

#### Simultanous Senders

As described in Section 3.1 of {{!RFC9071}}, in RTT conferences typically only one participant writes and sends text (at least long pieces of text) at any given time. Within a Publish/Subscribe network, multiple Publishers might publish data simultaneously, as data might be published as soon as it has been sampled, and as Publishers are now aware of when other Publsihers are publishing data. Because of this, the conference server might receive published data from multiple Publishers simultaneously. If the conference server is not able to simultaneously forward all published data, it will have to buffer the data.

### Idle Period



{{!RFC9071}} gives guidance on how to process real-time text in a conference server.


#### Mode

Section 1.2 of {{!RFC9071}} describes different "modes".



#### Sending Frequency


Within a Publish/Subscribe network, the data publishing interval can vary widely, depending on the use-case. In some constrained environements, a Publisher may publish data e.g., once a day, while in an industrial environment a Publisher may publish data all the time, with a very short publishing interval.






Received Real-time text is often read by humans. Because of that, it is important that text that was sent simultaneously by different senders is also received at the simultanelusly by the receivers.

 RTP-mixer-based method for multiparty-aware endpoints:

{{!RFC9071}} makes assumptions regarding how participant are sending text. It assumes that in a typcial scenario only one participant will send text at any given time. In a PubSub Conference, the same assumption cannot be done, as multiple publisher might simulateneously publish data to the same topic. In addition, unless a publisher also acts as a subscriber, it does not even know when and if other publishers are publishing data.

{{!RFC9071}} focuses on two mixing solutions: 'The RTP-Mixer-Based Solution for Multiparty-Aware Endpoints' (Section 2.2) and 'Mixing for Multiparty-Unaware Endpoints (Section 2.3)'.

In the 'The RTP-Mixer-Based Solution for Multiparty-Aware Endpoints' solution, the receivers will receive the text from all senders within a single RTP stream from the conference server.






## Time Considerations

In a Publish/Subscribe network, as publishers publish data independently from each other, there is typically no need for subscribers to syncrhonize or "lip-synch" the data.




## RTCP Considerations

The Real-time Control Protocol (RTCP) is used in conjunction with RTP. While RTP carries the actual data, RTCP carries information (e.g., statistics, control information), within an RTP session. 

NOTE: As RTCP messages sent by a Publisher might be terminated by a conference server (performing data mixing), essential information might not reach the Subscribers. For example, an RTPC Sender Report (SR) that provides mapping between the absolute time and the RTP Timestamp might not reach the Subscribers. 

Note that if is a large number of participants within a PubSub Conference, and if there is a need to send RTCP messages frequently, the RTCP messages might consume a large portion of network- and conference server capacity.



## RTP Mixing or Translating

In an AV conference, the actual time when the AV data was created is typcially not that important to the receiver. It is more important that the conference server is able to mix and lip-synch AV data that has been created at the same time by the senders.


### Translating

By default, when a Broker receives published data to a Topic, it forwards the data to each Subscriber that has subscribed to the data, without any processing of the data.

Aggregation: instead of forwarding each published data directly to a Subscriber, the PubSub Conference server might choose to store the published data in an aggregated manner, and forward all stored data in a single RTP packet towards the Subscriber based on different policies, e.g., depen


NOTE: The mechansims used by a PubSub Conference server to determine the constraints (network bandwidth etc) of a Subscriber are outside the scope of this document.





In case of translation, the original SSRC and the Timestamp will not be replaced by the translator.


### Mixing



As a mixer is consdiered a source by itself, it will often terminate received RTCP packets. Because of this, subscribers might not receive the RTCP SR packets that contain the mapping between the RTP Timestamp time and the real clock.





In an RTP packet, the Timestamp value typcially indicates when the payload data has been sampled. The exact details depends on the media type and payload format.

In case of mixing, the conference server will insert its own SSRC and Timestamp in the outgoing RTP packets with the mixed media. While the CSRC field can provide the SSRCs of the RTP packets used to create the mix, the Timestamp values will be lost. In a Publish/Subscribe scenarios, if the Subscribers need to know when data has been published, they cannot rely on getting that information from RTP.


POTENTIAL STANDARDIZATION WORK: In addition to contributing SSRCs, also include contributing Timestamps.

POTENTIAL STANDARDIZATION WORK: Add data sampling timestamp to RTP


## Single vs multiple RTP sessions

An RTP Stream is identified by the data source (SSRC).


Within a PubSub Conference, the number of publishers might be very large. In addition, the number of publsihers might vary quite frequently, as publishers join and leave the PubSub Conference. For that reason, it is not convenient for a subscriber to negotiate a separate RTP session for each publisher with the conference server. In addtion, one of the ideas behind the Publish/Subscribe pattern is that a Subscriber does not need to know, or be impacted, based on the number of Publsihers.


In the case of real-time text, receivers need to be able to identify the sender of each RTP packet, so that the text from all senders is not mixed togheter.

In some PubSub Conferences, Subscribers might not need to know which Publisher has published a specific set of data.

However, the SSRC


## Sender Timestamp

RTP does not provide a mechanism to indicate the time when a packet is sent, or when the RTP payload was sampled.

The following experimental RTP header extension to include the absolute packet send time in an RTP packet:

https://webrtc.googlesource.com/src/+/refs/heads/main/docs/native-code/rtp-hdrext/abs-send-time

https://webrtc.googlesource.com/src/+/refs/heads/main/docs/native-code/rtp-hdrext/abs-capture-time/







NOTE: In addition to the RTP SSRC value, the data format used in the RTP packet payload might have a source indicator that tells Subscrbiers





## Data Mixing




The way a conference server depends on the media type. In an AV conference, the audio ... For example, all incoming audio data is typically mixed together, so that everyone can hear everyone else. In case of video, if the con the conference server typcially forwards video stream of the participants currently spekakinh.

In the case of non-AV data, the default behavior within a PubSub Conference is to forward the data from a PubSub Publisher to each PubSub Subscriber, without performing any mixing or selection of what data is forwarded.

~~~~ aasvg

   .-----------.          .----------.             .------------.
   |           |  data X  |          |             |            |
   | Publisher +--------->+          |    data X   | Subscriber |
   |           |          |          +------------>|            |
   '-----------'          |          |             '------------'
                          |  Broker  |                ...
                          |          |                ...
                          |          |             .------------.
                          |          |             |            |
                          |          |    data X   | Subscriber |
                          |          +------------>|            |
                          '----------'             '------------'
~~~~
{: #fig-broker-forward title='PubSub Conference Data Forwarding' artwork-align="center"}

As an optimization, if the Broker receives data from multiple Publishers, it may forward data from multiple Publishers in a single RTP payload towards the Subscribers.


For example, if the Broker receives data in a SenML Record from Publisher A and Publisher B at the same time it might choose to place and forward the SenML Records in a SenML Pack.


~~~~ aasvg

SenML Record from Publisher A:

   [
     {"n":"urn:dev:ow:10e2073a01080063","u":"Cel","v":23.1}
   ]

SenML Record from Publisher B:

   [
     {"n":"urn:dev:ow:F0ea673a01000036","u":"Cel","v":25.7}
   ]


SenML Pack forwarded by the Broker towards the Subscribers:

   [
     {"n":"urn:dev:ow:10e2073a01080063","u":"Cel","v":23.1},
     {"n":"urn:dev:ow:F0ea673a01000036","u":"Cel","v":25.7}
   ]

~~~~
{: #fig-senml-pack title='SenML Pack created by broker' artwork-align="center"}









# RTP Considerations

## RTP Payload Type (PT) for PubSub



## RTP Header Marker Bit

This document does not define usage of the RTP header Marker bit.

## RTP Extensions for PubSub

A large number of RTP extensions have been specified for RTP. Many of the extensions have been specified with an AudioVisual use-case in mind. However, many of them can also be applied when RTP is used to transport non-AudioVisual data. While an extensive study of the usage of RTP extensions for transport of non-AudioVisual data is outside the scope of this document, this chapter describes a few extensions that have been studied in the work that lead to this document.

### Simulcast and Resolution

While it is quite clear what "resolution" means for audio and video, there is no unique definition for non-AV data.

For example, resolution can refer to the number of digits are included when the payload contains numeric values.

For example, resolution can refer to how often the publisher is sending data. The more freuquent, the higher the resolution.

For example, resolution can refer to the amount of data (e.g., netadata) that a publisher in sending.


A publisher can publish the same data using different "resolutions".

### Retransmission

The RTP header carries both a sequence number and a timestamp to allow a receiver to distinguish between lost packets and periods of time when no data was transmitted.

By default RTP provides unreliable transport. The same applies to different PubSub frameworks that use UDP as transport protocol. However, some PubSub frameworks use TCP transport, or use other mechanisms in order to provide reliable data delivery. There are RTP extension that have been used to provide reliable data delivery, or to simply inform the sender that data has been lost.

XXX specifies an RTP extension where RTP packets are re-transmitted by default.

#### Redundant Audio Data

{{!RFC2198}} defines an RTP payload format for encoding redundant audio data. If a data packet is lost, it might be possible to reconstruct the information of the lost packet from the redundant data that is included in the subsequent packets. The mechanism can also be used for non-AV data. However, in case of time-critical data, where the lost data would be considered "expired" it it would arrive as redundant data in a subsequent packet, the mechanism might not be useful (unless for logging purpose etc).

#### Forward Error Detection (FEC)

{{!RFC2733}}

Multiple data packets are used to create a single FEC packet. The FEC payload contains information which data packets have been used to create the FEC packet.


# RTCP Considerations

## RTCP FB

The RTCP Feedback (FB) {{!RFC4585}}




# Standardization Considerations

This document does not formally standardize any new protocol extensions, SIP event packages etc. Each extension, event package etc described in the document would need to be standardized following the normal standardization procedures. The protocol extensions and event packages described in this document are collated and listed below.


SIP Event Package for Conference State          Section XXX        Extension to the event package for providing separate information about publishers and subscribers.

SIP Event Package for PublishSubscribe          Section XXX        New SIP event package for providing information about PubSub Conferences (conference URI and topic) hosted by a Conference Server.




# Security Considerations

The Security Considerations for SIP Conferencing apply to this document.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

This document is based on the Master Thesis of Trung Van.
