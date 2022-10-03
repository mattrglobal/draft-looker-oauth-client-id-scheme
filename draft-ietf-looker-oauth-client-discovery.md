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

This specification defines a mechanism for an OAuth 2.0 authorization server to obtain the metadata of an OAuth 2.0 client, including its endpoint locations and capabilities without the need for a prior registration step.

--- middle

# Introduction

In the traditional OAuth 2.0 model {{!RFC6749}}, the authorization server registers and assigns an identifier to a client through a registration process, whether it be dynamically or out of band, during this registration process the authorization server records certain characteristics about the client known as metadata.

The requirement for client registration greatly reduces how dynamic the relationship between a client and authorization server can be. For instance, a client that is updating the capabilities it supports must update its registration with affected authorization servers for this change to be recognized. The limitation of registration also constrains distributed deployments that feature many clients and authorization servers whereby requiring the client to register is costly.

To improve the dynamic relationship between client and authorization server, mechanisms such as dynamic client registration {{!RFC7591}} was introduced. In dynamic client registration model, to register clients in real-time, a client will make a pre-flight request to authorizaiton server by including a set of client metadata and post it to the authorization server. If successful, the authorization server responds with a client id (and secret, if applicable) and a registration confirmation returning the registered client metadata (including any applicable defaults).

Although dynamic client registration enables Just-In-Time (JIT) provisioning of client IDs, management of client IDs and secrets is an operational challenge experienced by both clients and server that is not addressed with the dynamic client registration specification. Short-lived clients that runs on a browser may need to register for new client ID everytime it is instantiated due to the lack of long-term storage. This causes lot of dead client registration in the authorization server. To effeciently manage the registration storage, the authorization server needs to implement a mechanism to periodically prune the dead client entities. Also, when dynamic client registration is enabled for public clients, malicious actors can target the authorization server to overwhelm its resources by registering massive number of fake client entries. Appropriate security mechanisms should be considered by authorization servers to prevent these types of attacks. With the current client registration model, clients that needs to communicate with multiple authorization servers has to maintain multiple client identifiers to interact with them. This forces state management at client side and can be avoided if a client can use a single client identifier across multiple authorization servers.

Instead of requiring a registration process, this specification describes a model where a client can make itself discoverable to an authorization server in a similar way an authorization server makes itself discoverable to a client today with OAuth 2.0 Authorization Server Metadata {{!RFC8414}}.

The metadata for a client is retrieved from a .well-known location as a JSON {{!RFC8259}} document, which declares its endpoint locations and client capabilities (This process is described in [Client Discovery Flow](#client-discovery-flow)). This removes the need to send a pre-flight request to register the client metadata. Also, In this model a client can interact with mulitple authorization servers without the need to maintain state information (such as client ids and secrets). Once the client metadata is accepted by OAuth 2.0 authorization server, the client can interact with the authorisaiton server like any other OAuth 2.0 client registered with the OAuth 2.0 authorization server.

This specification defines a new request parameter 'client_discovery' to indicate that the interacting OAuth 2.0 client has no prior registration with authorization server and expects the authorization server to resolve the metadata from the specified URL. This specification uses the same metadata format defined in the client registration specification {{!RFC7591}} and no additional metadata elements or formats are defined in this specification.

## Conventions and Terminology

{::boilerplate bcp14-tagged}

This specification uses the terms "access token", "refresh token", "authorization server", "resource server", "authorization endpoint", "authorization request", "authorization response", "token endpoint", "grant type", "access token request", "access token response", "client", "public client", and "confidential client" defined by The OAuth 2.0 Authorization Framework {{!RFC6749}}.

The terms "request", "response", "header field", and "target URI" are imported from {{!RFC9110}}.

# Client Discovery Flow

The client discovery is performed by an authorization server once the authorisation server has the knowledge of client's url. One such way in which this url is obtained by the authorization server is via an authorization request as outlined in [Authorization request using Client Discovery](#authorization-request-using-client-discovery), where the client's metadata url is derived from the client_id. The client_id is expected to be a URL and the path derived from the URL must point to client's metadata.

Clients supporting discovery MUST make a JSON document available at the path formed by concatenating the string /.well-known/client-configuration to the client's url supplied as client_id. The syntax and semantics of .well-known are defined in RFC 5785 {{!RFC5785}} and apply to the client_id when it contains no path component. Client configuration MUST point to a JSON document compliant with OAuth 2.0 dynamic client registration protocol {{!RFC7591}} and MUST be returned using the application/json content type.

## Client Discovery Request

The flow begins by the authorization server making an HTTP GET request to retrieve the metadata of the client from the previously specified path (.well-known/client-configuration).

The following is a non-normative example request of an authorization server making a get request to client's .well-known endpoint to retrieve client metadata:

~~~ http
GET /.well-known/client-configuration HTTP/1.1
Host: client.example.org
~~~

## Client Discovery Response

The response is a set of client's metadata configuration as a JSON object. A successful response MUST use the 200 OK HTTP status code and return a JSON object using the application/json content type that contains a set of client's metadata configuration as defined in OAuth 2.0 dynamic client registration protocol {{!RFC7591}}.

<< TBC - Claims that return multiple values are represented as JSON arrays. Claims with zero elements MUST be omitted from the response >>

The following is a non-normative example response:

~~~ http
HTTP/1.1 200 OK
Content-Type: application/json
    {
      "redirect_uris": [
        "https://client.example.org/callback",
        "https://client.example.org/callback2"],
      "client_name": "My Example Client",
      "token_endpoint_auth_method": "client_secret_basic",
      "logo_uri": "https://client.example.org/logo.png",
      "jwks_uri": "https://client.example.org/my_public_keys.jwks",
      "example_extension_parameter": "example_value"
     }
~~~

An error response uses the applicable HTTP status code value.

In the following sections, we describe the mechanism through which a client communicates its metadata discovery url.

# Authorization Request Using Client Discovery

For a client to advertise itself as a discoverable client, a new request parameter "client_discovery" is defined and used during an authorization request. This authorization request, processed by a supporting authorization server, would indicate that the client_id value supplied is infact a URL that should be resolved to obtain the client's metadata, instead of trying to make sense of the value amongst existing registered clients.

## Authorization Request

The following is a non-normative example request of a client making an authorization request to an authorization server with the "client_discovery" parameter:

~~~ http
GET /authorize?response_type=code
    &client_id=https%3A%2F%2Fclient.example.org
    &client_discovery=true
    &state=af0ifjsldkj
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
HOST: server.example.com
~~~

The client metadata is discovered using the URL supplied in the "client_id" parameter of the request. The supplied URL MUST be a URI RFC 3986 {{!RFC3986}} with a scheme component that MUST be https, a host component, and optionally, port and path components and no query or fragment components. Additionally, host names MUST be domain names or a loopback interface and MUST NOT be IPv4 or IPv6 addresses except for IPv4 127.0.0.1 or IPv6 [::1].

After extracting the "client_id" URL from the authorization request, the authorization server MAY execute the [Client Discovery Flow](#client-discovery-flow) in order to obtain the client's metadata. Once obtained, it can perform checks based on this metadata in order to decide whether to proceed with the authorization request.

Following is a non-normative check that an authorization server can perform to validate the clients:

:   If the URL scheme, host or port of the redirect_uri in the request do not match that of the client_id, then the authorization endpoint SHOULD verify that the requested redirect_uri matches one of the redirect URLs published by the client, and SHOULD block the request from proceeding if not.
:   Since domain names are case insensitive, the host component of the URL MUST be compared case insensitively. Implementations SHOULD convert the host to lowercase when storing and using URLs.

Once the authorization request is successfully validated and processed, the authorization server issues an authorization code and delivers it to the client by
adding it as query parmeter (<eref target="https://www.rfc-editor.org/rfc/rfc6749#section-4.1.1">as described in the Section 4.1.2 of [RFC6749]</eref>) to the redirection URI using the "application/x-www-form-urlencoded" format.

~~~ http
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
&state=xyz
~~~

In case of any errors, error response is returned (<eref target="https://www.rfc-editor.org/rfc/rfc6749#section-4.1.2.1">as described in the Section 4.1.2.1 of [RFC6749]</eref>).

# Token Request Using Client Discovery

If an authorization server requires a client id during token request, the token request must include the new request parameter "client_discovery" to advertise itself as a discoverable client. This token request, processed by a supporting authorization server, would indicate that the client_id value supplied is infact a URL that should be resolved to obtain the client's metadata, instead of trying to make sense of the value amongst existing registered clients.

## Token Request

The following is a non-normative example request of a client making an token request using "client_discovery" parameter:

~~~ http
POST /token
Host: server.example.com
Content-type: application/x-www-form-urlencoded
Accept: application/json

grant_type=authorization_code
&code=xxxxxxxx
&client_id=https://client.example.com/
&redirect_uri=https://client.example.com/redirect
&code_verifier=a6128783714cfda1d388e2e98b6ae8221ac31aca31959e59512c59f5
&client_discovery=true
~~~

After extracting the "client_id" URL from the token request, the authorization server MAY execute the [Client Discovery Flow](#client-discovery-flow) in order to obtain the client's metadata. Once obtained, it can perform checks based on this metadata in order to decide whether to proceed with the token request.

Once the token request is successfully validated, the token endpoint MUST continue processing as normal (as defined by OAuth 2.0 [RFC6749])

In case of any errors, error response is returned (<eref target="https://www.rfc-editor.org/rfc/rfc6749#section-5.2">as described in the Section 5.2 of [RFC6749]</eref>).

# Client Metadata

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
