---
title: "OAuth 2.0 Client ID Scheme"
category: info

docname: draft-looker-oauth-client-id-scheme-latest
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

 -
    fullname: Karthik Sivasamy
    organization: MATTR
    email: karthik.sivasamy@mattr.global

normative:

informative:

--- abstract

This specification defines a new extensibility point to OAuth2 which allows clients to identify through different methods to an authorization server using an identifier not assigned by the authorization server. Furthermore, the specification defines one client ID scheme "urn:ietf:params:oauth:client-id-scheme:oauth-discoverable-client", including the nessary rules around how the authorization server can obtain the endpoint locations and capabilities of the client without the need for a registration process.

--- middle

# Introduction

In the traditional OAuth 2.0 model {{!RFC6749}}, the authorization server (AS) registers and assigns an identifier to a client through a registration process, during which the authorization server records certain characteristics about the client, commonly known as its metadata.

This requirement for registration greatly reduces how dynamic the relationship between a client and authorization server can be. For instance, a client that is updating the capabilities it supports must update its registration(s) with affected authorization servers for this change to be recognized.

To enable a more dynamic relationship between a client and an authorization server, dynamic client registration via {{!RFC7591}} was introduced. This model allows a client to register dynamically with a supporting authorization server by sending a registration request. Although this mechanism does provide some benefits it also introduces new operational challenges for both the client and AS. For instance clients that interface with many authorization servers are burdened with having to manage a client identifier per authorization server and in some cases forced to re-register the same client instance multiple times due to local storage limitations. Protecting the authorization servers client registration endpoint can also force other design tradeoffs, typically either the authorization server requires some form of authentication (e.g a "registration_token") for registration requests, which is often problematic for public clients to obtain and or manage. Or the authorization server permits any registration request and has to mitigate potential spam/malicious registration requests via some other mechanism.

Instead of being limited to approaches which requiring a registration process for the client, this specification defines an extensibility point to allow clients to identify with the authorization server using an identifier not assigned by the authorization server.

## Conventions and Terminology

{::boilerplate bcp14-tagged}

This specification uses the terms "access token", "refresh token", "authorization server", "resource server", "authorization endpoint", "authorization request", "authorization response", "token endpoint", "grant type", "access token request", "access token response", "client", "public client", and "confidential client" defined by The OAuth 2.0 Authorization Framework {{!RFC6749}}.

The terms "request" and "response" are imported from {{!RFC9110}}.

# Client ID Scheme

A client can indicate to an authorization server that it is using a client identifier not assigned by the authorization server in an authorization request and or token request via the "client_id_scheme" request parameter. The value of the client_id_scheme request parameter indicates to the authorization server how it should process the "client_id" parameter which may include how to obtain the endpoint locations and capabilities of the client and validate that the party making the authorization request is infact the client represented by the reported "client_id".

The following are non-normative example requests of a client making an authorization and a token request to an authorization server with the "client_id_scheme" parameter present and set to a value of "urn:ietf:params:oauth:client-id-scheme:example-value":

~~~ http
GET /authorize?response_type=code
    &client_id=https%3A%2F%2Fclient.example.com
    &client_id_scheme=urn%3Aietf%3Aparams%3Aoauth%3Aclient-id-scheme%3Aexample-value
    &state=af0ifjsldkj
    &redirect_uri=https%3A%2F%2Fclient.example.com%2Fcb
HOST: server.example.com
~~~

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
&client_id_scheme=urn%3Aietf%3Aparams%3Aoauth%3Aclient-id-scheme%3Aexample-value
~~~

## OAuth Discoverable Client ID Scheme

The following section defines the the "client_id_scheme" of "urn:ietf:params:oauth:client-id-scheme:oauth-discoverable-client" including the requirements placed on both the client and authorization server when using this scheme.

In essense when the "client_id_scheme" parameter is present and is set to "urn:ietf:params:oauth:client-id-scheme:oauth-discoverable-client" in an authorization and or token request, it indicates to the receiving authorization server that the value of the "client_id" parameter in the request is an HTTPS based URL corresponding to the "client_uri" for the client and if the authorization server does not already have the metadata for the identified client, it can retrieve the metadata by following the procedure outlined in [Obtaining Client Metadata](#client-metadata).

### Client Metadata

Clients can have metadata described in their configuration. Examples of existing registered metadata fields that a client can make use of can be found at the <eref target= "https://www.rfc-editor.org/rfc/rfc7591.html#section-4.1"> OAuth 2.0 dynamic client registration metadata IANA registry [RFC 7591]</eref>.

The client's published metadata MUST include the client_uri field as defined in section 2 of RFC7591 {{!RFC7591}}. The value of this field MUST be a URI as defined in RFC3986 {{!RFC3986}} with a scheme component that MUST be https, a host component, and optionally, port and path components and no query or fragment components. Additionally, host names MUST be domain names and MUST NOT be IPv4 or IPv6 addresses.

### Obtaining Client Metadata

A client supporting the "client_id_scheme" of "urn:ietf:params:oauth:client-id-scheme:oauth-discoverable-client" MUST make a JSON document containing metadata as specified in RFC7591 {{!RFC7591}} available at a path formed by inserting a well-known URI string into the client_uri between the host component and the path component, if any. By default, the well-known URI string used is "/.well-known/oauth-client". This path MUST use the "https" scheme. The syntax and semantics of ".well-known" are defined in RFC 5785 {{!RFC5785}}. The well-known URI suffix used MUST be registered in the IANA <eref target= "https://www.iana.org/assignments/well-known-uris">"Well-Known URIs"</eref> registry.

Different clients utilizing OAuth 2.0 in application-specific ways may define and register different well-known URI suffixes used to publish client metadata as used by those applications, for example using a well-known URI string such as "/.well-known/example-configuration". Alternatively, many such clients will use the default well-known URI string "/.well-known/oauth-client", which is the right choice for general-purpose OAuth 2.0 applications.

An OAuth 2.0 client using this specification MUST specify what well-known URI suffix it will use for this purpose. The same client MAY choose to publish its metadata at multiple well-known locations derived from its client_uri, for example, publishing metadata at both "/.well-known/example-configuration" and
"/.well-known/oauth-client".

Some OAuth 2.0 applications will choose to use the well-known URI suffix "openid-federation", as described in [Compatibility Notes](#compatibility-notes).

#### Client Metadata Request

A client metadata document MUST be queried using an HTTP "GET" request at the previously specified path. The OAuth 2.0 authorization server would make the following request when the client_uri is "https://client.example.com" and the well-known URI suffix is "oauth-client" to obtain the metadata, since the client_uri contains no path component:

~~~ http
GET /.well-known/oauth-client HTTP/1.1
Host: client.example.com
~~~

If the client_uri value contains a path component, any terminating "/" MUST be removed before inserting "/.well-known/" and the well-known URI suffix between the host component and the path component. The OAuth 2.0 authorization server would make the following request when the client_uri is "https://client.example.com/client1" and the well-known URI suffix is "oauth-client" to obtain the metadata, since the client_uri contains a path component:

~~~ http
GET /.well-known/oauth-client/client1 HTTP/1.1
Host: client.example.com
~~~

Using path components enables supporting multiple clients per host. This is required in some complex client configurations. This use of ".well-known" is for supporting multiple clients per host; unlike its use in RFC 5785 {{!RFC5785}}, it does not provide general information about the host.

#### Client Metadata Response

The response is a set of metadata values describing client's configuration, including all valid redirection URIs and features supported by the client. A successful response MUST use the 200 OK HTTP status code and return a JSON object using the "application/json" content type that contains a set of metadata fields and values as defined in [Client Metadata](#client-metadata). Other metadata fields MAY also be returned.

Metadata fields that return multiple values are represented as JSON arrays. Metadata fields with no values MUST be omitted from the response.

An error response uses the applicable HTTP status code value.

The following is a non-normative example response:

~~~ http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "client_uri": "https://client.example.com",
    "client_name": "My Example Client",
    "redirect_uris": [
        "https://client.example.com/cb",
        "https://client.example.com/cb2"
    ],
    "logo_uri": "https://client.example.com/logo.png",
    "jwks_uri": "https://client.example.com/my_public_keys.jwks",
    "tos_uri": "https://client.example.com/tos",
    "policy_uri": "https://client.example.com/policy",
    "example_extension_parameter": "example_value"
}
~~~

#### Client Metadata Validation

The client_uri value returned in the client metadata response MUST be identical to the client_uri value into which the well-known URI string was inserted to create the URL used to retrieve the metadata. If these values are not identical, the data contained in the response MUST NOT be used.

## Authorization Request Using Client ID Scheme

The following is a non-normative example request of a client making an authorization request to an authorization server with the "client_id_scheme" parameter set to "urn:ietf:params:oauth:client-id-scheme:oauth-discoverable-client":

~~~ http
GET /authorize?response_type=code
    &client_id=a-non-as-assigned-client-id
    &client_id_scheme=urn%3Aietf%3Aparams%3Aoauth%3Aclient-id-scheme%3Aoauth-discoverable-client
    &state=af0ifjsldkj
    &redirect_uri=https%3A%2F%2Fclient.example.com%2Fcb
HOST: server.example.com
~~~

The value of the "client_id" parameter in the authorization request MUST represent the URL encoded form of the "client_uri" value for the corresponding client. The "client_id" value MUST be URL decoded by the authorization server to obtain the "client_uri" value which can be used to resolve the client metadata as described in the [Obtaining Client Metadata](#obtaining-client-metadata) section.

**TODO stipulate new error responses**

## Token Request Using Client ID Scheme

The following is a non-normative example request of a client making a token request using the "client_id_scheme" parameter set to "urn:ietf:params:oauth:client-id-scheme:oauth-discoverable-client":

~~~ http
POST /token
Host: server.example.com
Content-type: application/x-www-form-urlencoded
Accept: application/json

grant_type=authorization_code
&code=xxxxxxxx
&client_id=a-non-as-assigned-client-id
&redirect_uri=https://client.example.com/redirect
&code_verifier=a6128783714cfda1d388e2e98b6ae8221ac31aca31959e59512c59f5
&client_id_scheme=urn%3Aietf%3Aparams%3Aoauth%3Aclient-id-scheme%3Aoauth-discoverable-client
~~~

The "client_id" parameter is passed to the token request during client authentication (<eref target="https://www.rfc-editor.org/rfc/rfc6749#section-3.2.1">as described in the Section 3.2.1 of [RFC6749]</eref>).

**TODO stipulate on other possible methods of client authentication**

In case of any errors, error response is returned (<eref target="https://www.rfc-editor.org/rfc/rfc6749#section-5.2">as described in the Section 5.2 of [RFC6749]</eref>).

# String Operations

Processing some OAuth 2.0 messages requires comparing values in the messages to known values.  For example, the member names in the metadata response might be compared to specific member names such as "client_uri". Comparing Unicode [UNICODE] strings, however, has significant security implications.

Therefore, comparisons between JSON strings and other Unicode strings MUST be performed as specified below:

1.  Remove any JSON-applied escaping to produce an array of Unicode
    code points.

2.  Unicode Normalization [USA15] MUST NOT be applied at any point to
    either the JSON string or the string it is to be compared
    against.

3.  Comparisons between the two strings MUST be performed as a
    Unicode code-point-to-code-point equality comparison.

Note that this is the same equality comparison procedure described in (<eref target= "https://www.rfc-editor.org/rfc/rfc8259#section-8.3"> Section 8.3 of [RFC8259]</eref>).

# Security Considerations

## TLS Requirements

Implementations MUST support TLS.  Which version(s) ought to be implemented will vary over time and depend on the widespread
deployment and known security vulnerabilities at the time of implementation.  The client MUST support TLS version 1.2 {{!RFC5246}} and MAY support additional TLS mechanisms meeting its security requirements.  When using TLS, the authorization server MUST perform a TLS/SSL server certificate check, per RFC 6125 {{!RFC6125}}.
Implementation security considerations can be found in "Recommendations for Secure Use of Transport Layer Security (TLS) and Datagram Transport Layer Security (DTLS)" {{!BCP195}}.

To protect against information disclosure and tampering, confidentiality protection MUST be applied using TLS with a ciphersuite that provides confidentiality and integrity protection.

## Impersonation Attacks

TLS certificate checking MUST be performed by the authorization server, as described in [](#tls-requirements), when making a client metadata request. Checking that the server certificate is valid for the "client_uri" URL prevents man-in-middle and DNS-based attacks. These attacks could cause a authorization server to be tricked into using an attacker's keys and endpoints, which would enable impersonation of the legitimate client.  If an attacker can accomplish this, they can access the resources that the affected client has access to by impersonating their profile.

An attacker may also attempt to impersonate a client by publishing a metadata document that contains a "client_uri" claim using the "client_uri" URL of the client being impersonated, but with its own endpoints and signing keys. This would enable it to impersonate that client, if accepted
by the authorization server.  To prevent this, the authorization server MUST ensure that the "client_uri" URL it is using as the prefix for the metadata request exactly matches the value of the "client_uri" metadata value in the client's metadata document received by the authorization server.

## Server Side Request Forgery (SSRF) Attacks

Authorization servers resolving metadata of a client and resolving URLs located in the metadata document should be aware of possible SSRF attacks. Authorization servers should pay attention to the possibility of these URLs using private or loopback based addresses and consider network policies or other measures to prevent making requests to these addresses. Authorization servers should also be aware of the possibility of some URLs featuring non-http based URI schemes which can lead to other possible SSRF attack vectors.

# Compatibility Notes

**TODO**

# IANA Considerations

The following IANA registration requests are made by this document.

## OAuth Parameters Registry

This specification registers the following parameters in the IANA "OAuth Parameters" registry defined in OAuth 2.0 RFC 6749 {{!RFC6749}}

client_id_scheme - Authorization request

- Parameter name: client_id_scheme
- Parameter usage location: authorization request
- Change controller: IESG
- Specification document(s): RFC XXXX (this document)

client_id_scheme - Token request

- Parameter name: client_id_scheme
- Parameter usage location: token request
- Change controller: IESG
- Specification document(s): RFC XXXX (this document)

<< TODO registering the OAuth urn parameter value>>

## Well-Known URI Registry

This specification registers the well-known URI defined in [Obtaining Client Metadata](#obtaining-client-metadata) in the (<eref target="https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml">IANA "Well-Known URIs" registry</eref>) established by RFC 5785 {{!RFC5785}}

### Registry Contents

* URI suffix: oauth-client
* Change controller: IESG
* Specification document: [](#obtaining-client-metadata) of RFC 8414
* Related information: (none)

<< TODO - IANA registration - https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml >>

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
