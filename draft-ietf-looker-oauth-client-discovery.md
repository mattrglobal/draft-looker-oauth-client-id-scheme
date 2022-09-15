---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "OAuth 2.0 Client Discovery"
category: info

docname: draft-ietf-looker-oauth-client-discovery-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: OAuth Working Group
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

This specification defines a mechanism for an OAuth 2.0 authorization server to obtain the metadata of an OAuth 2.0 client, including its endpoint locations and capabilities without the need for a prior registration step. Once the client metadata is accepted by OAuth 2.0 authorization server, the client can interact with the authorisaiton server like any other OAuth 2.0 client registered with the OAuth 2.0 authorization server. This specificaiton also defines a new request parameter 'client_discovery' to indicate that the interacting OAuth 2.0 client has no prior registration with authorization server and expects the authorization server to resolve the metadata from the specified URL.

--- middle

# Introduction

In the current OAuth 2.0 authorisation model [RFC6749], predominantly, the OAuth 2.0 authorization server is expected to assign client ids to clients through an out of band registration process to access the server. This greatly reduces dynamic relationship between clients and authorization server. To address this issue, mechanisms such as dynamic client registration [RFC7591] was introduced. To dynamically register clients, a client need to make a pre-flight request to authorizaiton server by including a set of client metadata and post it to the authorization server. If successfull, the authorization server responds with a client id (and secret, if applicable) and a registration confirmation returning the registered metadata (including any applicable defaults)

The metadata for a client is retrieved from a well-known location as a JSON [RFC8259] document, which declares its endpoint locations and client capabilities. This process is described in Section 3 (TODO).

TODO link to client metadata make it clear we aren't defining the metadata format just a resolution ..

Define how in conventional OAuth2.0 registration of the client is required and this negates the need for registration.

## Conventions and Terminology

{::boilerplate bcp14-tagged}

This specification uses the terms "access token", "refresh token", "authorization server", "resource server", "authorization endpoint", "authorization request", "authorization response", "token endpoint", "grant type", "access token request", "access token response", "client", "public client", and "confidential client" defined by The OAuth 2.0 Authorization Framework [RFC6749].

The terms "request", "response", "header field", and "target URI" are imported from [RFC9110].

# Client Discovery Process

## Client Metadata

## Authorizaiton Request

## AS response / Error response 





# Operational Consideration

## Caching

## Error codes / retry

## Proposed Client Authentication methods (to avoid storing client credentials)

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
