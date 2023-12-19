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
 RFC8428:

informative:

 

--- abstract

This document describes how a Session Initiation Protocol (SIP) Conference Server can be used to realize a Publish/Subscribe (PubSub) broker to distribute non-audiovisual data.


--- middle

# Introduction

This document describes how a Session Initiation Protocol (SIP) {{!RFC3261}} Conference Server can be used to realize a Publish/Subscribe (PubSub) broker to distribute non-audiovisual data.

One main advantage of the solution is the possibility to use existing RTP-based audiovisual conferencing infrastructure and protocols to realize data distributing
using the Publish/Subscribe traffic pattern, instead of using dedicated Publish/Subscribe infrastructure and protocols.

NOTE: SIP Conference servers might behave differently depending on configuration, profiles etc. The procedures in this document are based on a generic understanding of how conference servers behave.

NOTE: The examples in this document use the RTP T.140 real-time text (RTT) payload format to transport the payload data, and SenML to structure and encode the payload data. Other mechanisms can also be used.


While RTP is a generic transport protocol, the main usage has been for transport of real-time AudioVisual data. Non-AudioVisual data has typically been data associated with AudioVisual data, e.g., real-time text (rtt), DTMF signals. However, at the time of writing this document, IETF is working on a number of specifications where RTP is used to transport other types of non-AV data.

~~~~ aasvg

   .-----------.          .----------.  observe  .-----------.
   |           | publish  |          |<----------+           |
   | publisher +--------->+          +---------->| subscribe |
   |           |          |          +---------->|           |
   '-----------'          |          |           '-----------'
        ...               |  broker  |                ...
        ...               |          |                ...
   .-----------.          |          |  observe  .-----------.
   |           | publish  |          |<----------+           |
   | publisher +--------->|          +---------->| subscribe |
   |           |          |          +---------->|           |
   '-----------'          '----------'           '-----------'
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

NOTE: There are more and more use-cases where both AudioVisual data and non-AudioVisual data is exchanged within the same conference. The description of a mixed AudioVisual/PubSub
conference is outside the scope of this document.

AudioVisual Data:
: Data that contains encoded audio or video.

# SIP Considerations {#sec-sip-considerations}

# SIP Subject Header Field {#sec-sip-considerations-subject}

The SIP Subject header field can be used to indicate the PubSub topic associated with the conference.

NOTE: If a participant joins an existing conference, and uses a 

# SDP Considerations {#sec-sdp-considerations}

# SDP Direaction Attribute {#sec-sdp-considerations-dir-attr}

These attributes can be used to indicate the PubSub role of a participant.

A participant can use SDP 'sendonly' attribute to indicate that it is acting as a Publisher.

A participant can use SDP 'recvonly' attribute to indicate that it is acting as a Subscriber.

A participant can use SDP 'sendrecv' attribute to indicate that it is acting as both a Publisher and a Subscriber.

A participant can use SDP 'inactive' attribute to indicate that it is not acting as a Publisher nor a Subscriber. A participant can use the attribute e.g., to temporary indicate to
the conference server that it does not want to receive data associated with the conference, but also to indicate that it will not send data associated with the conference.

# SIP Conference Considerations

## Conference lifecycle

An AV conference


## Join existing conference

Having multiple conferences associated with the same Topic is not practical, unless there is a specific reason why

When a conference participants joins a conference by sending a SIP INVITE request to the 



# SIP Event Package for PubSub

{{!RFC4575}} defines a SIP Event Package {{!RFC3265}} for Conference State. A conference participant can subscribe to the event package, and retrieve conferance state information, including
information about the conference itsel, and information about other conference participants. For a PubSub conference, the <subject> child element of the <conference-description> element can
be used to indicate the Topic associated with the conference.

## Extensions for PubSub

### Maximum number of conference participants

The <maximum-user-count> element is used to indicate the maximum number of conference participants. For an AudioVisual conference, where participants typcially both send and receive media
a single element will be enough. However, in a PubSub conference, a majority of the conference participants might be either subscribers or publishers. There might be a large variation in
how many publishers and how mnay subscribers a conference server is able to handle. Therefore it could be useful to have separate elements to indicate that, e.g., <maximum-user-count-publisher>
and <maximum-user-count-subscriber>.


This document describes a new SIP Event Package, the SIP PublishSubscribe E


# RTP Considerations

## RTP Payload Type (PT) for PubSub


## RTP Extensions for PubSub

A large number of RTP extensions have been specified for RTP. Many of the extensions have been specified with an AudioVisual use-case in mind. However, many of them can also
be applied when RTP is used to transport non-AudioVisual data. While an extensive study of the usage of RTP extensions for transport of non-AudioVisual data is outside the scope of
this document, this chapter describes a few extensions that have been studied in the work that lead to this document.

### Simulcast and Resolution




A publisher can publish the same data using different "resolutions". 



# Security Considerations

The Security Considerations for SIP Conferencing apply to this document.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

This document is based on the Master Thesis of Trung Van.
