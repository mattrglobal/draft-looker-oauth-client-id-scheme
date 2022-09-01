---
title: "OAuth 2.0 Client Discovery"
category: info

docname: draft-ietf-looker-oauth-client-discovery-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: OAuth Working Group
keyword:
 - oauth

author:
 -
    fullname: Tobias Looker
    organization: MATTR
    email: tobias.looker@mattr.global

normative:

informative:


--- abstract

This specification defines a mechanism for an OAuth 2.0 authorization server to obtain the metadata of an OAuth 2.0 client, including its endpoint locations and capabilities without the need for a prior registration step.

--- middle

# Introduction

The metadata for a client is retrieved from a well-known location as a JSON [RFC8259] document, which declares its endpoint locations and client capabilities. This process is described in Section 3 (TODO).

TODO link to client metadata make it clear we aren't defining the metadata format just a resolution ..

Define how in conventional OAuth2.0 registration of the client is required and this negates the need for registration.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
