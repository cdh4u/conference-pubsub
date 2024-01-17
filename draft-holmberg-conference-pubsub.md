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
 RFC3550:
 RFC4353:
 RFC4575:
 RFC4585
 RFC8428:

informative:

 

--- abstract

This document describes how a Session Initiation Protocol (SIP) Conference Server can be used to realize a Publish/Subscribe (PubSub) broker to distribute non-audiovisual data.


--- middle

# Introduction

This document describes how a Session Initiation Protocol (SIP) {{!RFC3261}} Conference Server can be used to realize a Publish/Subscribe (PubSub) broker to distribute non-audiovisual data.

One main advantage of the solution is the possibility to use existing RTP-based audiovisual conferencing infrastructure and protocols to realize data distributing
using the Publish/Subscribe traffic pattern, instead of using dedicated Publish/Subscribe frameworks and protocols.

NOTE: SIP Conference servers might behave differently depending on configuration, profiles etc. The procedures in this document are based on a generic understanding of how conference servers behave.

NOTE: The examples in this document use the RTP T.140 real-time text (RTT) payload format to transport the payload data, and SenML to structure and encode the payload data. Other payload formats and
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
{: #fig-arch title='Publish/Subscribe Architecture' artwork-align="center"}


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
: A xxx

Subscribe:
: A xxx

Topic:
: A xxx

Broker:
: A xxx

Conference:
: A xxx

AudioVisual Conference/AV Conference:
: A "traditional" SIP conference, where human participants send and receive speech audio, and in some cases video. The conference server has different
policies for distributing audio and video. In a typcial sceanio it mixes and forwards the audio of all participants, while forwarding the
video of the currently speaking conference participant (referred to as "current speaker").

PubSub Conference:
: A SIP conference used to realize data distribution using the Publish/Subscribe traffic pattern, following the procedures in this document.

PubSub Conference Participant/PubSub Participant:
: An endpoint that has joined (or is about to join) a PubSub Conference. Within the PubSub Conference, the PubSub Participant will take the role as a Publisher, Subscriber or both.


NOTE: There are more and more use-cases where both AudioVisual data and non-AudioVisual data is exchanged within the same conference. The description of a mixed AudioVisual/PubSub
conference is outside the scope of this document.

AudioVisual Data:
: Data that contains encoded audio or video.

# SIP Considerations {#sec-sip-considerations}

# SIP Subject Header Field {#sec-sip-considerations-subject}

The SIP Subject header field can be used to indicate the PubSub Topic associated with the PubSub Conference.
 
# SDP Considerations {#sec-sdp-considerations}

## SDP Direaction Attribute {#sec-sdp-considerations-dir-attr}

These attributes can be used to indicate the PubSub role of a participant.

A participant can use SDP 'sendonly' attribute to indicate that it is acting as a Publisher.

A participant can use SDP 'recvonly' attribute to indicate that it is acting as a Subscriber.

A participant can use SDP 'sendrecv' attribute to indicate that it is acting as both a Publisher and a Subscriber.

A participant can use SDP 'inactive' attribute to indicate that it is not acting as a Publisher nor a Subscriber. A participant can use the attribute e.g., to temporary indicate to
the conference server that it does not want to receive data associated with the conference, but also to indicate that it will not send data associated with the conference.

# SIP Conference Considerations

## Conference lifecycle

### Conference creation

An AV conference is typically created either by one of the participants, or by a contralized function tool. 

### Conference duration and termination

The duration of an AV conference may vary, but is typcially measured in minutes or hours. An AV conference is typically terminated when the last participant has left the conference.

The interest for a PubSub topic might last for a very long time. Because of that, a PubSub conference associated with the topc might last for days, months or "infinite". While there might be times when there are no PubSub participants within a conference, the conference server might still keep the PubSub conference "alive", as new participants are expected to join in the near future. One advantage of keeping the conference "alive" is that participants can use the same conference URI whenever they are re-joining the conference.

## Join existing conference

A conference server might host multiple conferences that share the same conference name (Subject). Since the name is just a human readable string value, it is not uncommon that multiple AV conferences share the same name. Each conference will obviously have a unique conference URI. 

It is not practical to host multiple PubSub conferences that share the same Topic. Because of that, if a PubSub participant tries to create a PubSub conference with a Topic for which there already exist a conference, the conference server might choose to either reject the conference creation request (and inform the endpoint about the existing conference), redirect the participant to the existing conference (using a SIP 3xx response code{{!RFC3261}}) or simply add the endpoint to the existing conference.

# SIP Event Package Considerations

## SIP Event Package for Conference State

{{!RFC4575}} defines a SIP Event Package {{!RFC3265}} for Conference State. A conference participant can subscribe to the event package, and retrieve conferance state information, including
information about the conference itsel, and information about other conference participants. For a PubSub conference, the "subject" child element of the "conference-description" element can
be used to indicate the Topic associated with the conference.

### Extensions for PubSub

#### Maximum Number of Conference Participants

The "maximum-user-count" element is used to indicate the maximum number of conference participants. For an AudioVisual conference, where participants typcially both send and receive media
a single element will be enough. However, in a PubSub conference, a majority of the conference participants might be either subscribers or publishers. There might be a large variation in
how many publishers and how mnay subscribers a conference server is able to handle. Therefore it could be useful to have separate elements to indicate that, e.g., "maximum-user-count-publisher"
and "maximum-user-count-subscriber".

## SIP Event Package for PublishSubscribe
 
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
{: #fig-arch title='SIP Event Package for PublishSubscribe' artwork-align="center"}

## XML Schema

TBD

## Example

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
{: #fig-arch title='Example: SIP Event Package for PublishSubscribe' artwork-align="center"}


# RTP Considerations

## RTP Payload Type (PT) for PubSub



## RTP Header Marker Bit

This document does not define usage of the RTP header Marker bit.

## RTP Extensions for PubSub

A large number of RTP extensions have been specified for RTP. Many of the extensions have been specified with an AudioVisual use-case in mind. However, many of them can also
be applied when RTP is used to transport non-AudioVisual data. While an extensive study of the usage of RTP extensions for transport of non-AudioVisual data is outside the scope of
this document, this chapter describes a few extensions that have been studied in the work that lead to this document.

### Simulcast and Resolution

While it is quite clear what "resolution" means for audio and video, there is no unique definition for non-AV data.

For example, resolution can refer to the number of digits are included when the payload contains numeric values.

For example, resolution can refer to how often the publisher is sending data. The more freuquent, the higher the resolution.

For example, resolution can refer to the amount of data (e.g., netadata) that a publisher in sending.


A publisher can publish the same data using different "resolutions". 

### Retransmission

The RTP header carries both a sequence number and a timestamp to allow a receiver to distinguish between lost packets and periods of time when no data was transmitted.

By default RTP provides unreliable transport. The same applies to different PubSub frameworks that use UDP as transport protocol. However, some PubSub
frameworks use TCP transport, or use other mechanisms in order to provide reliable data delivery. There are RTP extension that have been used to provide
reliable data delivery, or to simply inform the sender that data has been lost.

XXX specifies an RTP extension where RTP packets are re-transmitted by default.

#### Redundant Audio Data

{{!RFC2198}} defines an RTP payload format for encoding redundant audio data. If a data packet is lost, it might be possible to reconstruct the information of the lost packet from the redundant data
that is included in the subsequent packets. The mechanism can also be used for non-AV data. However, in case of time-critical data, where the lost data would be considered "expired" it it would
arrive as redundant data in a subsequent packet, the mechanism might not be useful (unless for logging purpose etc).

#### Forward Error Detection (FEC)

{{!RFC2733}}

Multiple data packets are used to create a single FEC packet. The FEC payload contains information which data packets have been used to create the FEC packet.


# RTCP Considerations

## RTCP FB

The RTCP Feedback (FB) {{!RFC4585}} 




# Standardization Considerations

This document does not formally standardize any new protocol extensions, SIP event packages etc. Each extension, event package etc described in the document
would need to be standardized following the normal standardization procedures. The protocol extensions and event packages described in this document are collated
and listed below.


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
