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

In the traditional OAuth 2.0 authorisation model {{!RFC6749}}, predominantly, the OAuth 2.0 authorization server is expected to assign client ids to clients through an out of band registration process to access the server. This greatly reduces dynamic relationship between clients and authorization server.

To improve the dynamic relationship between client and authorization server, mechanisms such as dynamic client registration {{!RFC7591}} was introduced. In dynamic client registration model, to register clients in real-time, a client will make a pre-flight request to authorizaiton server by including a set of client metadata and post it to the authorization server. If successfull, the authorization server responds with a client id (and secret, if applicable) and a registration confirmation returning the registered client metadata (including any applicable defaults).

To further enhance the decoupling between client and authorization server, this specification describes a model where a client can make it self discoverable to an authorization server in the same way an authorization server makes it self discoverable to a client today with openid discovery (https://openid.net/specs/openid-connect-discovery-1_0.html).

The metadata for a client is retrieved from a .well-known location as a JSON {{!RFC8259}} document, which declares its endpoint locations and client capabilities (This process is described in Section 3 (TODO)). This removes the need to send a pre-flight request to register the client metadata. Also, In this model a client can interact with mulitple authorization servers without the need to maintain state information (such as client ids and secrets).

This specification defines a new request parameter 'client_discovery' to indicate that the interacting OAuth 2.0 client has no prior registration with authorization server and expects the authorization server to resolve the metadata from the specified URL. This specification uses the same metadata format defined in the client registration specification {{!RFC7591}} and no additional metadata elements or formats are defined in this specification.

## Conventions and Terminology

{::boilerplate bcp14-tagged}

This specification uses the terms "access token", "refresh token", "authorization server", "resource server", "authorization endpoint", "authorization request", "authorization response", "token endpoint", "grant type", "access token request", "access token response", "client", "public client", and "confidential client" defined by The OAuth 2.0 Authorization Framework {{!RFC6749}}.

The terms "request", "response", "header field", and "target URI" are imported from {{!RFC9110}}.

# Client Discovery Flow

The client discovery is performed by an authorization server once the authorisation server has the knowledge of client's metadata url.

Authorization server makes a GET request to retrieve the metadata of a client from a .well-known location as a JSON document.
The client declares its endpoint locations and client capabilities in the client metadata document.

## Client Discovery Request

The following is a non-normative example request of an authorization server making a get request to client's .well-known endpoint to retrieve client metadata:

~~~ http
GET /.well-known/client-metadata HTTP/1.1
  Host: client.example.org
~~~

## Client Discovery Response

~~~ http
200 OK
~~~

## Client Discovery Error Response

When an error condition occurs during discovery, the authorization server returns an HTTP 400 status code (unless otherwise specified) with content type "application/json" consisting of a JSON object {{!RFC7159}} describing the error in the response body.

~~~ http
400 Bad Request
~~~

In the following sections, we describe the mechanism through which a client communicates its metadata discovery url.


## Authorization request using Client Discovery

For a client to advertise itself as a discoverable client, a new request parameter "client_discovery" is defined and used during an authorization request. This authorization request, processed by a supporting authorization server, would indicate that the client_id value supplied is infact a URL that should be resolved to obtain the clients metadata, instead of trying to make sense of the value amongst existing registered clients.

### Authorization Request

The following is a non-normative example request of a client making an authorization request to an authorization server with the "client_discovery" parameter:

~~~ http
 HTTP/1.1 302 Found
  Location: https://server.example.com/authorize?
    response_type=code
    &scope=openid%20profile%20email
    &client_id=https%3A%2F%2Fclient.example.org
    &client_discovery=true
    &state=af0ifjsldkj
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
~~~

<< TODO - Define URL restrictions (like query parameters, fragments, special characters etc - probably reference spec) and encoding rules for the url value >>

### Authorization Request - Client Response / Error codes

<< TODO - Define response format and error codes >>

## Token request using Client Discovery

<< TODO Similar to authorization request, define token request example and restrictions >>

### Token Request

### Token Request - Client Response / Error codes

## Client Metadata

<< TODO Expand on client metadata formats. describe briefly about using sub-domain for multiple clients under the same domain >>


# Operational Consideration

## Caching

<< TODO - Expand on caching considerations for the client metadata that could be added (e.g HTTP request caching) to limit how often an AS/OP needs to actually resolve the clients ID. >>

## Proposed Client Authentication methods (to avoid storing client credentials)

<< TODO - Explain different methods for client authenticaiton (attestation, JWTs for client authentication {{!RFC7523}} >>

# Security Considerations

<< TODO - HTTPS url , checking the redirect url matches the url associated in the client ID field>>

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
