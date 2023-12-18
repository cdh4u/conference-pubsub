---
title: "Session Initiation Protocol (SIP) Conference Server for Publish/Subscribe"
abbrev: "SIP PubSub"
category: info

docname: draft-holmberg-conference-pubsub-latest
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
    fullname: "Christer Holmberg"
    organization: Ericsson
    email: "christer.holmberg@ericsson.com"

normative:

informative:


--- abstract

This document describes how a Session Initiation Protocol (SIP) Conference Server can be used to realize a Publish/Subscribe (PubSub) broker to distribute non-audiovisual data.


--- middle

# Introduction

This document describes how a Session Initiation Protocol (SIP) Conference Server can be used to realize a Publish/Subscribe (PubSub) broker to distribute non-audiovisual data.

One main advantage of the solution is the possibility to use existing RTP-based audiovisual conferencing infrastructure and protocols to realize data distributing using the Publish/Subscribe traffic pattern, instead of using dedicated Publish/Subscribe infrastructure and protocols. 

NOTE: SIP Conference servers might behave differently depending on configuration, profiles etc. The procedures in this document are based on a generic understanding of how conference servers behave.

NOTE: The examples in this document use the RTP T.140 real-time text (RTT) payload format to transport the payload data, and SenML to structure and encode the payload data. Other mechanisms can also be used.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

The Security Considerations for SIP Conferencing apply to this document.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

This document is based on the Master Thesis of Trung Van.
