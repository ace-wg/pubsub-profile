---
title: Pub-Sub Profile for Authentication and Authorization for Constrained Environments (ACE)
abbrev: pubsub-profile
docname: draft-ietf-ace-pubsub-profile-latest
category: std

coding: utf-8

ipr: trust200902
area: Security
workgroup: ACE Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: F. Palombini
    name: Francesca Palombini
    org: Ericsson
    email: francesca.palombini@ericsson.com
  -
    ins: C. Sengul
    name: Cigdem Sengul
    org: Brunel University
    email: csengul@acm.org

normative:
  RFC2119:
  RFC6749:
  RFC7252:
  RFC7925:
  RFC8392:
  RFC8949:
  RFC9237:
  I-D.draft-ietf-rats-uccs-01:
  I-D.ietf-cose-cbor-encoded-cert:
  I-D.ietf-cose-rfc8152bis-algs:
  I-D.ietf-cose-rfc8152bis-struct:
  I-D.ietf-ace-oauth-authz:
  I-D.ietf-core-coap-pubsub:
  I-D.ietf-ace-key-groupcomm:
  I-D.ietf-core-groupcomm-bis:

informative:

  RFC8259:
  I-D.ietf-ace-actors:
  I-D.ietf-ace-dtls-authorize:
  I-D.ietf-ace-oscore-profile:
  I-D.ietf-ace-mqtt-tls-profile:
  I-D.draft-ietf-ace-revoked-token-notification-02:
  MQTT-OASIS-Standard-v5:
    title: "OASIS Standard MQTT Version 5.0"
    date: "2017"
    target: http://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html
    author:
      -
        ins: A. Banks
      -
        ins: E. Briggs
      -
        ins: K. Borgendale
      -
        ins: R. Gupta

entity:
        SELF: "[RFC-XXXX]"

--- abstract

This specification defines an application profile for authentication and authorization for Publishers and Subscribers in a constrained pub-sub scenario, using the ACE framework. This profile relies on transport layer or application layer security to authorize the pub-sub clients to the broker. Moreover, it describes the use of application layer security to protect the content of the pub-sub client message exchange through the broker. The profile mainly focuses on the pub-sub scenarios using
the Constrained Application Protocol (CoAP) {{I-D.ietf-core-coap-pubsub}}.

--- middle

# Introduction

In the publish-subscribe (pub-sub) scenario, devices with limited reachability communicate via a broker, which enables store-and-forward messaging between the devices. This document defines a way to authorize pub-sub clients using the ACE framework {{I-D.ietf-ace-oauth-authz}} to obtain the keys for protecting the content
of their pub-sub messages when communicating through the broker. The pub-sub communication using the Constrained Application Protocol (CoAP) {{RFC7252}} is specified in {{I-D.ietf-core-coap-pubsub}}.This document gives detailed specifications for CoAP pub-sub, and describes how it can be adapted for MQTT {{MQTT-OASIS-Standard-v5}}; similar adaptations can extend to other transport protocols as well.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 {{RFC2119}}.

Readers are expected to be familiar with:

* The terms and concepts
described in {{I-D.ietf-ace-oauth-authz}}. In particular, analogously to {{I-D.ietf-ace-oauth-authz}}, terminology for entities in the architecture such as Client (C), Resource Server (RS), and Authorization Server (AS) is defined in OAuth 2.0 {{RFC6749}}.
* The terms and concept related to the message formats and processing specified in {{I-D.ietf-ace-key-groupcomm}}, for
provisioning and renewing keying material in group communication
scenarios. and terminology for entities such as the Key Distribution Center (KDC) and Dispatcher in {{I-D.ietf-ace-key-groupcomm}}.
* The terms and concepts of pub-sub group communication, as described in {{I-D.ietf-core-coap-pubsub}}.

# Application Profile Overview {#overview}

The objective of this document is to specify how to request, distribute and renew keying material and configuration parameters to protect message exchanges for pub-sub communication, using {{I-D.ietf-ace-key-groupcomm}}, which expands from the ACE framework ({{I-D.ietf-ace-oauth-authz}}). The pub-sub communication protocol can be based on CoAP, as described in {{I-D.ietf-core-coap-pubsub}}, MQTT {{MQTT-OASIS-Standard-v5}} , or other transport. This document focuses on CoAP and expands on the transport profiles {{I-D.ietf-ace-dtls-authorize}} and {{I-D.ietf-ace-oscore-profile}}.

The architecture of the scenario is shown in {{archi}}. Publisher or Subscriber Clients is referred to as Client in short. A Client can act both as a publisher and a subscriber, publishing to some topics, and subscribing to others. However, for the simplicity of presentation, this profile describes Publisher and Subscriber clients separately.
The Broker acts as the ACE RS, and also corresponds to the Dispatcher in {{I-D.ietf-ace-key-groupcomm}}).Both Publishers and Subscribers use the same pub-sub communication protocol and the same transport profile of ACE in their interaction with the broker. All clients need to use CoAP when communicating to the KDC.

~~~~~~~~~~~~
             +----------------+   +----------------+
             |                |   |      Key       |
             | Authorization  |   |  Distribution  |
             |    Server      |   |     Center     |
             |      (AS)      |   |     (KDC)      |
             +----------------+   +----------------+
                      ^                  ^
                      |                  |
     +---------(A)----+                  |
     |   +--------------------(B)--------+
     v   v
+------------+             +------------+
|            |             |            |
| Pub-Sub    | <-- (C)---> |   Broker   |
| Client     |             |            |
|            |             |            |
+------------+             +------------+
~~~~~~~~~~~~~
{: #archi title="Architecture for Pub-Sub with Authorization Server and Key Distribution Center"}
{: artwork-align="center"}

This profile expects the establishment of a secure connection between a Client and Broker, using an ACE transport profile such as DTLS {{I-D.ietf-ace-dtls-authorize}} or OSCORE {{I-D.ietf-ace-oscore-profile}} (A and C). Once the client establishes a secure association with KDC with the help of AS, it can request to join the security groups of its pub-sub topics (A and B), and  can communicate securely with the other group members, using the keying material provided by the KDC. (C) corresponds to the exchange between the Client and  the Broker, where the Client sends its access token to the Broker and establishes a secure connection with the Broker. Depending on the Information received in (A), this can be for example DTLS handshake, or other protocols. Depending on the application, there may not be the need for this set up phase: for example, if OSCORE is used directly. Note that, in line with what's defined in the ACE transport profile used, the access token includes the scope (i.e. pubsub topics on the Broker) the Publisher is allowed to publish to. After the previous phases have taken place, the pub-sub communication can commence. The operations of publishing and subscribing are defined in {{I-D.ietf-core-coap-pubsub}}.

It must be noted that Clients maintain two different security associations. On the one hand, the Publisher and the Subscriber clients have a security association with the Broker, so that, as the ACE RS, it can verify that the Clients are authorized (Security Association 1). On the other hand, the Publisher has a security association with the Subscriber, to protect the publication content (Security Association 2) while sending it through the broker. The Security Association 1 is set up using AS and a transport profile of {{I-D.ietf-ace-oauth-authz}}, the Security Association 2 is set up using AS, KDC and {{I-D.ietf-ace-key-groupcomm}}. Note that, given that the publication content is protected, the Broker MAY accept unauthorised Subscribers. In this case, the Subscriber client can skip setting up Security Association 1 with the Broker and connect to it as an anonymous client to subscribe to topics of interest at the Broker.

~~~~~~~~~~~~
+------------+             +------------+              +------------+
|            |             |            |              |            |
| Publisher  |             |   Broker   |              | Subscriber |
|            |             |            |              |            |
|            |             |            |              |            |
+------------+             +------------+              +------------+
      :   :                       : :                       : :
      :   '------ Security -------' '-----------------------' :
      :         Association 1                                 :
      '------------------------------- Security --------------'
                                     Association 2
~~~~~~~~~~~~~
{: #associations title="Security Associations between Publisher, Broker, Subscriber pairs."}
{: artwork-align="center"}

# Interfacing the KDC {#kdc-interface}

This profile builds on the mechanisms defined in {{I-D.ietf-ace-key-groupcomm}} 
to enable the following for pub-sub communication:

1. Authorizing a Client to join a topic's security group(s), and providing it with the group keying material to communicate with other group members.
2. Allowing a Client to retrieve group keying material for the Publisher Client to publish protected publications to the Broker, and for the Subscriber Client to read protected publications.
3. Allowing a group member to retrieve authentication credentials of other group members and to provide and updated authentication credential.
4. Allowing a group member to leave the group.
5. Evicting a group member from the group.
6. Renewing and redistributing the group keying (rekeying) material due to membership change in the group.

To this end, the Clients uses the following KDC resources:

* '/ace-group':  A Client can access this resource in order to retrieve a set of
group names, each corresponding to one of the specified group identifiers.
* '/ace-group/GROUPNAME': Corresponds to the resource associated
with group GROUPNAME, and contains its symmetric group keying material.
* '/ace-group/GROUPNAME/creds':  Contains the authentication credentials of all the Publisher members of the group with name GROUPNAME.
* '/ace-group/GROUPNAME/num': Contains the current version number for the symmetric group keying material of the group with name GROUPNAME.
* '/ace-group/GROUPNAME/nodes/NODENAME': Contains the group keying material and the individual keying material for that group member NODENAME in GROUPNAME. All Clients can send a DELETE request to leave the group with name GROUPNAME. GET operation is supported for all Clients. PUT is not supported.
* '/ace-group/GROUPNAME/nodes/NODENAME/cred':  A Publisher Client can POST to this resource to upload at the KDC a new authentication credential to use in the group GROUPNAME. The KDC returns 4.01 (Unauthorized) error response for requests from the Subscriber Clients.
* 'ace-group/GROUPNAME/kdc-cred': MUST be hosted if a group re-keying mechanism is used. Contains the authentication credential of the KDC for the group with name GROUPNAME.

The following resource may be optionally hosted: '/ace-group/GROUPNAME/policies' (KDC doesn't host policies for GROUPNAME).
ToDo:  REQUIRED of the application profiles of this specification to register a Resource Type for the root url-path (REQ10)

# Joining a pub-sub security group (A-B) {#authorisation}

Figure {{message-flow}} provides a high level overview of the message flow for a
   node joining a group. This message flow is expanded in the subsequent sections.

~~~~~~~~~~~
   Client                                       Broker   AS      KDC
      | [--Resource Request (CoAP/MQTT or other)-->] |       |       |
      |                                           |       |       |
      | [<----AS Information (CoAP/MQTT or other)--] |       |       |
      |                                                   |       |
      | ----- Authorisation Request (CoAP/HTTP or other)---->|       |
      |                                                   |       |
      | <------Authorisation Response (CoAP/HTTP or other) --|       |
      |                                                           |
      |----------------------Token Post (CoAP)------------------->|
      |                                                           |
      |------------------- Joining Request (CoAP) --------------->|
      |                                                           |
      |------------------ Joining Response (CoAP) --------------->|

~~~~~~~~~~~
{: #message-flow title="Authorisation Flow"}
{: artwork-align="center"}

 Since {{I-D.ietf-ace-oauth-authz}} recommends the use of CoAP and CBOR, this document describes the exchanges assuming CoAP and CBOR are used. However, using HTTP instead of CoAP is possible, using the corresponding parameters and methods. Analogously, JSON {{RFC8259}} can be used instead of CBOR, using the conversion method specified in Sections 6.1 and 6.2 of {{RFC8949}}. In case JSON is used, the Content Format or Media Type of the message has to be changed accordingly. Exact definition of these exchanges are considered out of scope for this document.


## AS Discovery (Optional) {#AS-discovery}
Complementary to what is defined in {{I-D.ietf-ace-oauth-authz}} (Section 5.1) for AS discovery, the Broker MAY send the address of the AS to the Client in the 'AS' parameter in the AS Information as a response to an Unauthorized Resource Request (Section 5.2).  An example using CBOR diagnostic notation and CoAP is given below:

~~~~~~~~~~~
    4.01 Unauthorized
    Content-Format: application/ace-groupcomm+cbor
    {"AS": "coaps://as.example.com/token"}
~~~~~~~~~~~
{: #AS-info-ex title="AS Information example"}
{: artwork-align="center"}

## Authorisation Request/Response for the KDC and the Broker {#auth-request}

After retrieving the AS address, the Client sends two Authorisation Requests to the AS for the KDC and the Broker, respectively. Note that the AS authorises what topics a Client is allowed to Publish or Subscribe to the Broker, which means authorising which application and security groups a Client can join. This is because being able to publish or subscribe to a topic at the Broker requires authorisation to join an application group. To secure the message contents, the client needs to request to be part of the security group(s) for the selected application groups.

Communications between the Client and the AS MUST be secured,
   according to what is defined by the used transport profile of ACE. Both Authorisation Requests include the following fields
(Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}):

* 'scope', specifying the name of the groups, that the Client requests to access. This parameter is encoded as a CBOR byte
string according to the AIF format (ToDo: Add AIF reference, check for CoAP transport profiles scope format)
* 'audience', an identifier, corresponding to either the KDC or the Broker.

Other additional parameters can be included if necessary, as defined in {{I-D.ietf-ace-oauth-authz}}.

For the Broker, the scope represents pub-sub topics i.e., the application group, and for the KDC, the scope represents the security group. If there is a one-to-one mapping between the application group and the security group, the client uses the same scope for both requests. If there is not a one-to-one mapping, the correct policies regarding both sets of scopes MUST be available to the AS.

The client MUST ask for the correct scopes in its Authorization Requests. How the client discovers the (application group, security group) association is out of scope of this document.
ToDo: Check OSCORE Groups with the CoRE Resource Directory to see if it applies.
ToDo: Should the Client ask with the topic names, and KDC does the security group mapping?

The 'scope' parameter used for the KDC follows the AIF format
(ToDo: Check for CoAP pub-sub), and is encoded as follows, where 'gname' is treated as topic identifier.

~~~~~~~~~~~
   gname = tstr

   permissions = uint . bits roles

   role = &(
    Pub: 1,
    Sub: 2
   )

   scope_entry = AIF_Generic<gname , permissions>

   scope = << [ + scope_entry ] >>
~~~~~~~~~~~
{: #scope-aif title="Pub-Sub scope using the AIF format"}
{: artwork-align="center"}

## Authorisation response

The AS responds with an Authorization Response to each request, 
containing claims, as defined in Section 5.8.2 of {{I-D.ietf-ace-oauth-authz}} and Section 3.2 of {{I-D.ietf-ace-key-groupcomm}}.
On receiving the Authorisation Response, the Client needs to manage which token to use for which audience, i.e., the KDC or the Broker. The 'profile' claim is set to "coap_pubsub_app" as defined in {{iana-coap-profile}}. 

## Token Transfer to KDC {#token-post}
After receiving a token from the AS, the Client transfers the token to the KDC using one of the methods defined Section 3.3 {{I-D.ietf-ace-key-groupcomm}}). This includes sending a POST request to the authz-info endpoint,  and if using the DTLS transport profile of ACE,
the Client MAY provide the access token to the KDC during the secure session establishment.

When using the authz-info endpoint, a Subscriber Client MAY ask in addition for the format of the public keys in the group, used for source authentication, as well as any other group parameters. In this case, the message MUST have Content-Format set to "application/ace+cbor" defined in Section 8.16 of {{I-D.ietf-ace-oauth-authz}}.
The message payload MUST be formatted as a CBOR map, which MUST include the access token and MAY include the 'sign_info' parameter, specifying the CBOR simple value "null" (0xf6) to request information about the signature algorithm, signature algorithm parameters, signature key parameters and about the exact format of authentication credentials used in the groups that the Client has been authorized to join. Alternatively, the joining node MAY retrieve this information by other means as described in {{I-D.ietf-ace-key-groupcomm}}.

The KDC verifies the token to check of the Client is authorized to access the topic with the requested role. After successful verification, the Client is authorized to receive the group keying material from the KDC and join the group. The KDC replies to the Client with a 2.01 (Created) response, using Content-Format "application/ace+cbor". The payload of the 2.01 response is a CBOR map.

For Publisher Clients, i.e., the Clients whose scope of the access token includes the "Pub" role, the KDC MUST include the parameter 'kdcchallenge' in the CBOR map, specifying a dedicated challenge N_S generated by the KDC. Later when joining the group, the Publisher Client MUST send its own public key to the KDC and use the 'kdcchallenge' as part of proving possession of the corresponding private key (see {{I-D.ietf-ace-key-groupcomm}}).

If 'sign_info' is included in the Token Transfer Request, the KDC SHOULD include the 'sign_info' parameter in the Token Transfer Response.

ToDo: Specify sign_alg, sign_parameters, sign_key_parameters, cred_fmt if sign_info present
ToDo: Need to specify N_S generation

## Join Request {#join-request}

In the next step, a node MUST have established a secure communication association
established before attempting to join a group.  Possible ways to provide a secure communication association are described in the DTLS transport profile {{I-D.ietf-ace-dtls-authorize}} and OSCORE transport profile {{I-D.ietf-ace-oscore-profile}} of ACE.

After establishing a secure communication, the Client sends a Join Request to the KDC as described in Section 4.3 of {{I-D.ietf-ace-key-groupcomm}}. More specifically, the Client sends a POST request to the /ace-group/GROUPNAME endpoint, with Content-Format "application/ace-groupcomm+cbor". The payload MUST contain the following information formatted as a CBOR map, which MUST be encoded as defined in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}:

* 'scope' parameter set to the specific group that the Client is attempting to join, i.e., the group name, and the roles it wishes to have in the group. This value corresponds to one scope entry, as defined in {{auth-request}}.
* 'get_creds' parameter, if the Client needs to retrieve the public keys of the other members. The Subscriber Clients MUST have access to the public keys of all the Publisher Clients. This may be achieved by requesting the public keys of all the Publisher Clients, this parameter MUST encode a non-empty CBOR array, with three elements: '\["true", "Pub", \[\]\]'.
* 'client\_cred' parameter MUST be included if the Client is a Publisher and contains the Client's public key formatted according to the encoding of the public keys used in the group. For a Subscriber-only Client,  the Joining Request MUST NOT contain the 'client\_cred parameter'. The alg parameter in the 'client\_cred' COSE\_Key MUST be a signing algorithm, as defined in {{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}, and that it is the same algorithm used to compute the signature sent in 'client\_cred\_verify'.
TODO: Check the specific format for authentication credentials (REQ6)
ToDo: We say MUST for publishers, but the key-groupcomm allows this field to be empty, and KDC to have stored a public key through another method.
Do we allow this or not?
* 'cnonce', includes a dedicated nonce N_C generated by the Client, if 'client\_cred' is present.
* 'client\_cred\_verify', if 'client\_cred' is present. This parameter contains the proof-of-possession evidence. Client signs the scope, concatenated with N\_S and concatenated with N\_C using the private key corresponding to the public key in the 'client_cred' paramater. N\_S is the challenge received from the KDC in the 'kdcchallenge' parameter of the 2.01 (Created) response to the Token Transfer Request (see {{token-post}}).
* OPTIONALLY, the 'creds\_repo', if the format of the Client's authentication credential in the 'client_cred' parameter is a certificate.
ToDo: Do we define this option, do we accept 'client_cred" as a certificate?
* 'control_uri', using the default url-path is /ace-group/GROUPNAME/node

An example of the Join Request for a CoAP Publisher using CoAP and CBOR is specified in {{fig-req-pub-kdc}}, where SIG is a signature computed using the private key associated to the public key and the algorithm in 'client\_cred'.

~~~~~~~~~~~~
{
  "scope" : ["Broker1/Temp", "pub"],
  "client_cred" :
    { / COSE_Key /
      / type / 1 : 2, / EC2 /
      / kid / 2 : h'11',
      / alg / 3 : -7, / ECDSA with SHA-256 /
      / crv / -1 : 1 , / P-256 /
      / x / -2 : h'65eda5a12577c2bae829437fe338701a10aaa375e1bb5b5de1
      08de439c08551d',
      / y /-3 : h'1e52ed75701163f7f9e40ddf9f341b3dc9ba860af7e0ca7ca7e
      9eecd0084d19c',
  "cnonce" : h'd36b581d1eef9c7c,
  "client_cred_verify" : SIG
    }
}
~~~~~~~~~~~~
{: #fig-req-pub-kdc title="Joining Request payload for a Publisher"}
{: artwork-align="center"}

ToDo: Correct scope in artwork

An example of the payload of a Join Request for a Subscriber using CoAP and CBOR is specified in {{fig-req-sub-kdc}}.

~~~~~~~~~~~~
{
  "scope" : ["Broker1/Temp", "sub"],
  "get_creds" : null
}
~~~~~~~~~~~~
{: #fig-req-sub-kdc title="Joining Request payload for a Subscriber"}
{: artwork-align="center"}

ToDo: Correct scope in artwork

## Join Response
On receiving the Join Request, the KDC processes the request as defined in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}, and may return a success or
error response.

If 'client_cred' field is present, the KDC verifies signature in the the 'client_cred_verify'.

In the case of success,the Client is added to the list
of current members, if not already a member. The Client is assigned a NODENAME and assigned a sub-resource /ace-group/GROUPNAME/nodes/NODENAME. NODENAME is associated to the access token and secure session of the Client. For Publishers, their public key is also associated with tuple containing NODENAME, GROUPNAME and access token. The public key of the Publisher is stored.

Then, the KDC responds with a Join Response with response code 2.01 (Created) if the Client has been added to the list of group members, and 2.04 (Changed) otherwise (e.g., if the Client is re-joining).  The Content-Format  is "application/ace-groupcomm+cbor". The payload (formatted as a CBOR map) MUST contain the following fields from the Join Response  and encode them as defined in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}:
- 'gkty', they key type for the 'key' parameter. ToDo: Check ACE Groupcomm Key Types registry and gkty accepted by the 
application
- 'key', the keying material for group communication. This is a "COSE\_Key" object (defined in {{I-D.ietf-cose-rfc8152bis-algs}}{{I-D.ietf-cose-rfc8152bis-struct}}, containing:
    * 'kty' with value 4 (symmetric)
    * 'alg' with value defined by the AS (Content Encryption Algorithm)
    * 'Base IV' with value defined by the AS
    * 'k' with value the symmetric key value
    * OPTIONALLY, 'kid' with an identifier for the key value ToDo: Check the format of the 'key' value (REQ17)
    ToDo: Additionally, documents specifying the
   key format MUST register it in the "ACE Groupcomm Key Types" registry defined in Section 11.7, including its name, type and application profile to be used with.
- 'num' containing the version number for the 'key'. The initial version number
is set to 1.
- 'exp', the value of the expiration time of the 'key'
- 'creds', MUST be present when responding to a join request from a Subscriber Client and contain the public keys of all Publisher Clients, formatted according to the public key encoding for the group, if the 'get\_creds' parameter was present. Otherwise, it MUST NOT be present. The encoding accepted for this document is UCCS (Unprotected CWT Claims Set) {{I-D.draft-ietf-rats-uccs-01}}. ToDo: Consider allowing other public key formats with the following text. If CBOR Web Tokens (CWTs) or CWT Claims Sets (CCSs) {{RFC8392}} are used as public key format, the public key algorithm is fully described by a COSE key type and its "kty" and "crv" parameters. If X.509 certificates {{RFC7925}} or C509 certificates {{I-D.ietf-cose-cbor-encoded-cert}} are used as public key format, the public key algorithm is fully described by the "algorithm" field of the "SubjectPublicKeyInfo" structure, and by the "subjectPublicKeyAlgorithm" element, respectively. ToDo: This encoding information needs to come earlier, and we need to reduce the set of options considered above.
- 'peer\_roles' MUST be present if 'creds' is also present. Otherwise, it MUST NOT be present.  ToDo: Requested a change for
this, and see how the Groupcomm draft is updated
- 'peer\_identifiers' MUST be present if 'creds' is also present. Otherwise, it MUST NOT be present.
- 'kdc\_cred', MUST be present if group re-keying is used, and encoded as a CBOR byte string, with value the original binary representation of the KDC's authentication credential.
- 'kdc\_nonce', MUST be present, if 'kdc\_cred' is present and encoded as a CBOR byte string, and including a dedicated nonce N_KDC generated by the KDC.
- 'kdc_cred_verify' MUST be present, if 'kdc\_cred' is present and encoded as a CBOR byte string.
ToDo: Application profiles of this specification MUST specify the exact approaches used by the KDC to compute the PoP evidence to include in 'kdc_cred_verify', and MUST specify which of those approaches is used in which case (REQ21).

An example of the Join Response for a CoAP Publisher using CoAP and CBOR
(corresponding to Join Request in {{fig-req-pub-kdc}}) is shown in.  {{fig-resp-pub-kdc}}.

~~~~~~~~~~~~
{
  "gkty" : "COSE_Key",
  "key" : {1: 4, 2: h'1234', 3: 12, 5: h'1f389d14d17dc7',
          -1: h'02e2cc3a9b92855220f255fff1c615bc'}
}
~~~~~~~~~~~~
{: #fig-resp-pub-kdc title="Join Response payload for a Publisher"}
{: artwork-align="center"}

ToDo: Add "num"

An example of the payload of a Join Response for a Subscriber using CoAP and CBOR (corresponding to the request in {{fig-req-sub-kdc}}) is shown in {{fig-resp-sub-kdc}}.

~~~~~~~~~~~~
{
  "scope" : ["Broker1/Temp", "sub"],
  "gkty" : "COSE_Key"
  "key" : {1: 4, 2: h'1234', 3: 12, 5: h'1f389d14d17dc7',
          -1: h'02e2cc3a9b92855220f255fff1c615bc'},
  "creds" : [
   {/UCCS/
      2:  "42-50-31-FF-EF-37-32-39", /sub/
      8: {/cnf/
      1: {/COSE_Key/
      1 : 1, /alg/
      3 : -8 /kty/
      -1 : 6 , /crv/
      -2 : h'C6EC665E817BD064340E7C24BB93A11E /x/
8EC0735CE48790F9C458F7FA340B8CA3', / x /
    }
    }
   }
  ]
}
~~~~~~~~~~~~
{: #fig-resp-sub-kdc title="Joining Response payload for a Subscriber"}
{: artwork-align="center"}
ToDo: Fix Example for COSE_Key for public key
ToDo: Add peer roles and peer ids.


# PubSub Protected Communication (C) {#protected_communication}

~~~~~~~~~~~~
+------------+             +------------+              +------------+
|            |             |            |              |            |
| Publisher  | ----(D)---> |   Broker   |              | Subscriber |
|            |             |            | <----(E)---- |            |
|            |             |            | -----(F)---> |            |
+------------+             +------------+              +------------+
~~~~~~~~~~~~~
{: #pubsub-3 title="Secure communication between Publisher and Subscriber"}
{: artwork-align="center"}


(D) corresponds to the publication of a topic on the Broker.
The publication (the resource representation) is protected with COSE  ({{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}) by the Publisher.
The (E) message is the subscription of the Subscriber. The subscription MAY be unprotected.
The (F) message is the response from the Broker, where the publication is protected with COSE by the Publisher.

~~~~~~~~~~~
  Publisher                Broker               Subscriber
      | --- PUT /topic ----> |                       |
      |  protected with COSE |                       |
      |                      | <--- GET /topic ----- |
      |                      |                       |
      |                      | ---- response ------> |
      |                      |  protected with COSE  |
~~~~~~~~~~~
{: #flow title="Example of protected communication for CoAP"}
{: artwork-align="center"}


## Using COSE Objects To Protect The Resource Representation {#oscon}

The Publisher uses the symmetric COSE Key received from the KDC to protect the payload of the PUBLISH operation (Section 4.3 of {{I-D.ietf-core-coap-pubsub}}). Specifically, the COSE Key is used to create a COSE\_Encrypt0 object with algorithm specified by the KDC. The Publisher uses the private key corresponding to the public key sent to the KDC in exchange B  to countersign the COSE Object as specified in {{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}. The payload is replaced by the COSE object before the publication is sent to the Broker.

The Subscriber uses the 'kid' in the 'countersignature' field in the COSE object to retrieve the right public key to verify the countersignature. It then uses the symmetric key received from KDC to verify and decrypt the publication received in the payload from the Broker (in the case of CoAP the publication is received by the CoAP Notification).

The COSE object is constructed in the following way:
* The protected Headers (as described in {{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}) MUST contain the kid parameter if it was provided in the Joining Response, with value the kid of the symmetric COSE Key received and MUST contain the content encryption algorithm.
* The unprotected Headers MUST contain the Partial IV, with value a sequence number that is incremented for every message sent, and the counter signature that includes:
  - the algorithm (same value as in the asymmetric COSE Key received in (B)) in the protected header;
  - the kid (same value as the kid of the asymmetric COSE Key received in (B)) in the unprotected header;
  - the signature computed as specified in {{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}.
* The ciphertext, computed over the plaintext that MUST contain the message payload.

The 'external\_aad' is an empty string.

An example is given in {{fig-cose-ex}}:

~~~~~~~~~~~~
16(
  [
    / protected / h'a2010c04421234' / {
        \ alg \ 1:12, \ AES-CCM-64-64-128 \
        \ kid \ 4: h'1234'
      } / ,
    / unprotected / {
      / iv / 5:h'89f52f65a1c580',
      / countersign / 7:[
        / protected / h'a10126' / {
          \ alg \ 1:-7
        } / ,
        / unprotected / {
          / kid / 4:h'11'
        },
        / signature / SIG / 64 bytes signature /
      ]
    },
    / ciphertext / h'8df0a3b62fccff37aa313c8020e971f8aC8d'
  ]
)
~~~~~~~~~~~~
{: #fig-cose-ex title="Example of COSE Object sent in the payload of a PUBLISH operation"}
{: artwork-align="center"}

The encryption and decryption operations are described in  {{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}.

# Other Group Operations 

## Querying for Group Information

* '/ace-group': All Clients use FETCH requests to retrieve a set of group names associated with their group identifiers. ToDo: Encoding of gid
* '/ace-group/GROUPNAME':  All Clients can use GET requests to retrieve the symmetric group keying material of the group with the name GROUPNAME. The value of the GROUPNAME URI path and the group name in the access token scope ('gname') MUST coincide.
* '/ace-group/GROUPNAME/creds': KDC acts as a repository of authentication credentials for Publisher Clients. The Subscriber Clients of the group use GET/FETCH requests to retrieve the authentication credentials of all or subset of the group members of the group with name GROUPNAME.
* '/ace-group/GROUPNAME/num': All group member Clients use GET requests to retrieve the current version number for the symmetric group keying material of the group with name GROUPNAME.
* '/ace-group/GROUPNAME/kdc-cred': All group member Clients use GET requests to retrieve the current authentication credential of the KDC. 

## Updating Authentication Credentials

A Publisher Client can contact the KDC to upload a new authentication credential to use in the group, and replace the
currently stored one. To this end, it sends a sends a CoAP POST
   request to the /ace-group/GROUPNAME/nodes/NODENAME/cred.
The KDC replaces the stored authentication credential of this Client (identified by NODENAME) with the one specified in the request at the KDC, for the group identified by GROUPNAME.

## Removal from a Group
A Client can actively request to leave the group.  In this case, the Client sends a CoAP DELETE request to the endpoint /ace- group/GROUPNAME/nodes/NODENAME at the KDC, where GROUPNAME is the group name and NODENAME is its node name.
KDC can also remove a group member due to any of the reasons described in Section 5 of {{I-D.ietf-ace-key-groupcomm}}.

## Rekeying a Group 
KDC MAY trigger a group rekeying as described in Section 6 
of {{I-D.ietf-ace-key-groupcomm}} due to a change in the group membership, the current group keying material approaching its expiration time, or a regularly scheduled update of the group keying material.
ToDo: Do we require a backward security? Anything above
turned to a MUST?

Default rekeying scheme is Point-to-point (Section 6.1 of {{I-D.ietf-ace-key-groupcomm}}), where KDC individually targets ach node to rekey, using the pairwise secure communication association with that node.

If the group rekeying is performed due to one or multiple
Publisher Clients that have joined the group, then a rekeying
message MUST also include the authentication credentials that those Clients use in the group, together with the roles and node identifier that the corresponding Client has in the group.  This information is specified by means of the parameters 'creds',
'peer_roles' and 'peer_identifiers', like done in the Join Response message.

ToDo: Any additional rekeying mechanisms?
The pub-sub model would make sense. In this case, the KDC acts
as publisher (KDC authentication credential??) and publishes each rekeying message to a specific "rekeying topic", which is associated with the group and is hosted at a broker server.  Following their group joining, the group members subscribe to the rekeying topic at the broker, thus receiving the group rekeying messages as they are published by the KDC.

# Considerations for Supporting MQTT PubSub Profile {#mqtt-pubsub}

The steps MQTT clients go through would be similar to the CoAP clients, where PUT corresponds to a PUBLISH message, and GET corresponds to a SUBSCRIBE message. Whenever a Client publishes a new message, the Broker sends this message to all valid subscribers. The payload that is carried in MQTT messages will be protected using COSE.

In MQTT, topics are organised as a tree, and in the {{I-D.ietf-ace-mqtt-tls-profile}},
'scope' captures permissions for not a single topic but a topic filter. Therefore, topic names (i.e., group names) may include wildcards spanning several levels of the topic tree.
Hence, it is important to distinguish application groups and security groups defined in {{I-D.ietf-core-groupcomm-bis}}. An application group has relevance at the application level - for example, in MQTT an application group could denote all topics stored under "home/lights/". On the other hand, a security group is a group of endpoints that each store group security material to exchange secure communication within the group. The group communication in {{I-D.ietf-ace-key-groupcomm}} refers to security groups.
ToDo: Give a more complete example;
ToDo: How does a client figure out the mapping?

In summary, for an MQTT client we envision the following steps to take place:
1. Client sends a token request to AS for the requested topics (application groups) using the broker as the audience. AIF-MQTT data model for representing the requested scopes is described in Section 3 of the {{I-D.ietf-ace-mqtt-tls-profile}}.
2. Client sends a token request to AS for the corresponding security groups for its application groups using the KDC as the audience.
3. Client received Authorisation responses for its requests. In the authorisation response, the 'profile' claim is set to "mqtt_pubsub_app" as defined in {{iana-mqtt-profile}}.
4. Client sends join requests to KDC to gets the keys for these security groups.
5. Client authorises to the Broker with the token (described in {{I-D.ietf-ace-mqtt-tls-profile}}).
6. A Publisher Client sends PUBLISH messages for a given topic and protects the payload with the corresponding key for the associated security group. RS validates the PUBLISH message by checking the topic stored token.
7. A Subscriber Client may send SUBSCRIBE messages with one or multiple topic filters.
A topic filter may correspond to multiple topics. RS validates the SUBSCRIBE message by checking the stored token for the Client.

Authorisation Server (AS) Discovery is defined in Section 2.2.6.1 of {{I-D.ietf-ace-mqtt-tls-profile}} for MQTT v5 clients (and not supported for MQTT v3 clients).

# Security Considerations

All the security considerations in {{I-D.ietf-ace-key-groupcomm}}
apply. 

In the profile described above, the Publisher and Subscriber use asymmetric crypto, which would make the message exchange quite heavy for small constrained devices. Moreover, all Subscribers must be able to access the public keys of all the Publishers to a specific topic to be able to verify the publications. Such a database could be set up and managed by the same entity having control of the key material for that topic, i.e. KDC.


 Even though Access Tokens have expiration times, an Access Token may need to be revoked before its expiration time (see {{I-D.draft-ietf-ace-revoked-token-notification-02}} for a list of possible circumstances). Subscribers can be excluded from future publications through re-keying for a certain topic. This could be set up to happen on a regular basis, for certain applications. How this could be done is out of scope for this work. In the case when publishers, for CoAP-supporting brokers and clients, the method described in {{I-D.draft-ietf-ace-revoked-token-notification-02}} MAY be used to allow an Authorization Server to notify Clients and Resource Servers (i.e., registered devices) about revoked Access Tokens.

The Broker is only trusted with verifying that the Publisher is authorized to publish, but is not trusted with the publications itself, which it cannot read nor modify. In this setting, caching of publications on the Broker is still allowed.

TODO: expand on security and privacy considerations

# IANA Considerations

## ACE Groupcomm Profile Registry {#iana-profile}

The following registrations are done for the "ACE Groupcomm Profile" Registry following the procedure specified in {{I-D.ietf-ace-key-groupcomm}}.

Note to RFC Editor: Please replace all occurrences of "\[\[This document\]\]"
with the RFC number of this specification and delete this paragraph.

### CoAP Profile Registration {#iana-coap-profile}

Name: coap_pubsub_app

Description: Profile for delegating client authentication and authorization for publishers and subscribers in a CoAP pub-sub setting scenario in a constrained environment.

CBOR Key: TBD

Reference: \[\[This document\]\]

### MQTT Profile Registration {#iana-mqtt-profile}

Name: mqtt_pubsub_app

Description: Profile for delegating client authentication and authorization for publishers and subscribers in a MQTT pub-sub setting scenario in a constrained environment.

CBOR Key: TBD

Reference: \[\[This document\]\]

## ACE Groupcomm Key Registry {#iana-ace-groupcomm-key}

The following registrations are done for the ACE Groupcomm Key Registry following the procedure specified in {{I-D.ietf-ace-key-groupcomm}}.

Note to RFC Editor: Please replace all occurrences of "\[\[This document\]\]"
with the RFC number of this specification and delete this paragraph.

Name: COSE_Key

Key Type Value: TBD

Profile: coap_pubsub_app, mqtt_pubsub_app

Description: COSE_Key object

References: {{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}, \[\[This document\]\]

## AIF {#aif}

For the media-types application/aif+cbor and application/aif+json defined in Section 5.1 of {{RFC9237}}, IANA is requested to register the following entries for the two media-type parameters Toid and Tperm, in the respective sub-registry defined in Section 5.2 of {{RFC9237}} within the "MIME Media Type Sub-Parameter" registry group.

For Toid:
Name: 
Description/Specification:
Reference: [[This document]]

For Tperm:
Name: 
Description/Specification: 
Reference: [[This document]]

--- back

# Requirements on Application Profiles

This section lists the specifications on this profile based on the requirements defined in Appendix A of {{I-D.ietf-ace-key-groupcomm}}.

* REQ1: Specify the encoding and value of the identifier of group or topic of 'scope': TODO.

* REQ2: Specify the encoding and value of roles of 'scope': TODO

* REQ3: Optionally, specify the acceptable values for 'sign_alg': TODO

* REQ4: Optionally, specify the acceptable values for 'sign_parameters': TODO

* REQ5: Optionally, specify the acceptable values for 'sign_key_parameters': TODO

* REQ6: Optionally, specify the acceptable values for 'pub_key_enc': TODO

* REQ7: Specify the exact format of the 'key' value: COSE_Key, TODO

* REQ8: Specify the acceptable values of 'kty' : "COSE_Key", TODO

* REQ9: Specify the format of the identifiers of group members: TODO

* REQ10: Optionally, specify the format and content of 'group\_policies' entries: not defined

* REQ11: Specify the communication protocol the members of the group must use: CoAP pub/sub.

* REQ12: Specify the security protocol the group members must use to protect their communication. This must provide encryption, integrity and replay protection: Object Security of Content using COSE, see {{oscon}}.

* REQ13: Specify and register the application profile identifier : "coap_pubsub_app", see {{iana-profile}}.

* REQ14: Optionally, specify the encoding of public keys, of 'client\_cred', and of 'pub\_keys' if COSE_Keys are not used: NA.

* REQ15: Specify policies at the KDC to handle id that are not included in get_pub_keys: TODO

* REQ16: Specify the format and content of 'group_policies': TODO

* REQ17: Specify the format of newly-generated individual keying material for group members, or of the information to derive it, and corresponding CBOR label : not defined

* REQ18: Specify how the communication is secured between Client and KDC. Optionally, specify transport profile of ACE {{I-D.ietf-ace-oauth-authz}} to use between Client and KDC: pre-set, as KDC is AS.

* OPT1: Optionally, specify the encoding of public keys, of 'client\_cred', and of 'pub\_keys' if COSE_Keys are not used: NA

* OPT2: Optionally, specify the negotiation of parameter values for signature algorithm and signature keys, if 'sign_info' and 'pub_key_enc' are not used: NA

* OPT3: Optionally, specify the format and content of 'mgt\_key\_material': not defined

* OPT4: Optionally, specify policies that instruct clients to retain unsuccessfully decrypted messages and for how long, so that they can be decrypted after getting updated keying material: not defined

# Acknowledgments
{: numbered="no"}

The author wishes to thank Ari Keränen, John Mattsson, Ludwig Seitz, Göran Selander, Jim Schaad and Marco Tiloca for the useful discussion and reviews that helped shape this document.

--- fluff

<!-- Local Words: -->
<!-- Local Variables: -->
<!-- coding: utf-8 -->
<!-- ispell-local-dictionary: "american" -->
<!-- End: -->
