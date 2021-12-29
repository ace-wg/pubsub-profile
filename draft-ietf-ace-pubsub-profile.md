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
  RFC8949:
  I-D.ietf-cose-rfc8152bis-algs:
  I-D.ietf-cose-rfc8152bis-struct:
  I-D.ietf-ace-oauth-authz:
  I-D.ietf-core-coap-pubsub:
  I-D.ietf-ace-key-groupcomm:
  I-D.ietf-core-groupcomm-bis:
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

informative:

  RFC8259:
  I-D.ietf-ace-actors:
  I-D.ietf-ace-dtls-authorize:
  I-D.ietf-ace-oscore-profile:
  I-D.ietf-ace-mqtt-tls-profile:

entity:
        SELF: "[RFC-XXXX]"

--- abstract

This specification defines an application profile for authentication and authorization for publishers and subscribers in a constrained pub-sub scenario, using the ACE framework. This profile relies on transport layer or application layer security to authorize the pub-sub clients to the broker. Moreover, it describes the use of application layer security to protect the content of the pub-sub client message exchange through the broker. The profile covers pub-sub scenarios using
either the Constrained Application Protocol (CoAP) {{I-D.ietf-core-coap-pubsub}} or the
Message Queue Telemetry Transport (MQTT) {{MQTT-OASIS-Standard-v5}} protocol.

--- middle

# Introduction

In the publish-subscribe (pub-sub) scenario, devices with limited reachability communicate via a broker, which enables store-and-forward messaging between the devices. This document defines a way to authorize pub-sub clients using the ACE framework {{I-D.ietf-ace-oauth-authz}} to obtain the keys for protecting the content
of their pub-sub messages when communicating through the broker. The pub-sub communication using the Constrained Application Protocol (CoAP) {{RFC7252}} is specified in {{I-D.ietf-core-coap-pubsub}}, while the one using MQTT is specified in {{MQTT-OASIS-Standard-v5}}.  This document gives detailed specifications for MQTT and CoAP pub-sub, but can easily be adapted for other transport protocols as well.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 {{RFC2119}}.

Readers are expected to be familiar with the terms and concepts
described in {{I-D.ietf-ace-oauth-authz}}, {{I-D.ietf-ace-key-groupcomm}}. In particular, analogously to {{I-D.ietf-ace-oauth-authz}}, terminology for entities in the architecture such as Client (C), Resource Server (RS), and Authorization Server (AS) is defined in OAuth 2.0 {{RFC6749}} and {{I-D.ietf-ace-actors}}, and terminology for entities such as the Key Distribution Center (KDC) and Dispatcher in {{I-D.ietf-ace-key-groupcomm}}.

Readers are expected to be familiar with terms and concepts of pub-sub group communication, as described in {{I-D.ietf-core-coap-pubsub}}, or MQTT {{MQTT-OASIS-Standard-v5}}.

# Application Profile Overview {#overview}

The objective of this document is to specify how to authorize nodes, provide keys, and protect a pub-sub communication, using {{I-D.ietf-ace-key-groupcomm}}, which expands from the ACE framework ({{I-D.ietf-ace-oauth-authz}}), and transport profiles ({{I-D.ietf-ace-dtls-authorize}}, {{I-D.ietf-ace-oscore-profile}}, {{I-D.ietf-ace-mqtt-tls-profile}}). The pub-sub communication protocol can be based on CoAP, as described in {{I-D.ietf-core-coap-pubsub}}, MQTT {{MQTT-OASIS-Standard-v5}} , or other transport. Note that both publishers and subscribers use the same pub-sub communication protocol and the same transport profile of ACE in their interaction with the broker. However, all clients need to use CoAP when communicating to the KDC.

The architecture of the scenario is shown in {{archi}}.

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

Publisher or Subscriber Clients is referred to as Client in short. 
The Broker acts as the ACE RS, and also corresponds to the Dispatcher in {{I-D.ietf-ace-key-groupcomm}}).

This profile specifies:

1. The establishment of a secure connection between a Client and Broker, using an ACE transport profile such as DTLS {{I-D.ietf-ace-dtls-authorize}}, OSCORE {{I-D.ietf-ace-oscore-profile}}, or MQTT-TLS {{I-D.ietf-ace-mqtt-tls-profile}} (A and C).
2. The Clients retrieval of keying material for the Publisher Client to publish protected publications to the Broker, and for the Subscriber Client to read protected publications (B).

These exchanges aim at setting up two different security associations. 
On the one hand, the Publisher and the Subscriber clients have a security association with the Broker, so that, as the ACE RS, it can verify that the Clients are authorized (Security Association 1). On the other hand, the Publisher has a security association with the Subscriber, to protect the publication content (Security Association 2) while sending it through the broker. The Security Association 1 is set up using AS and a transport profile of {{I-D.ietf-ace-oauth-authz}}, the Security Association 2 is set up using AS, KDC and {{I-D.ietf-ace-key-groupcomm}}. Note that, given that the publication content is protected, the Broker MAY accept unauthorised Subscribers. In this case, the Subscriber client can skip setting up Security Association 1 with the Broker and connect to it as an anonymous client to subscribe to topics of interest at the Broker.

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

# PubSub Authorisation {#authorisation}

 Since {{I-D.ietf-ace-oauth-authz}} recommends the use of CoAP and CBOR, this document describes the exchanges assuming CoAP and CBOR are used. However, using HTTP instead of CoAP is possible, using the corresponding parameters and methods. Analogously, JSON
 {{RFC8259}} can be used instead of CBOR, using the conversion method specified in Sections 6.1 and 6.2 of {{RFC8949}}. In case JSON is used, the Content Format or Media Type of the message has to be changed accordingly. Exact definition of these exchanges are considered out of scope for this document.

 {{authorisation-flow}} shows the message flow for authorisation purposes.

~~~~~~~~~~~
   Client                                       Broker   AS      KDC
      | [--Resource Request (CoAP/MQTT/other)-->] |       |       |
      |                                           |       |       |
      | [<----AS Information (CoAP/MQTT/other)--] |       |       |
      |                                                   |       |
      | ----- Authorisation Request (CoAP/HTTP/other)---->|       |
      |                                                   |       |
      | <------Authorisation Response (CoAP/HTTP/other) --|       |
      |                                                           |
      |----------------------Token Post (CoAP)------------------->|
      |                                                           |
      |------------------- Joining Request (CoAP) --------------->|
      |                                                           |
      |------------------ Joining Response (CoAP) --------------->|

~~~~~~~~~~~
{: #authorisation-flow title="Authorisation Flow"}
{: artwork-align="center"}

## AS Discovery (Optional) {#AS-discovery}
Complementary to what is defined in {{I-D.ietf-ace-oauth-authz}} (Section 5.1) for AS discovery, the Broker MAY send the address of the AS to the Client in the 'AS' parameter in the AS Information as a response to an Unauthorized Resource Request (Section 5.2).  An example using CBOR diagnostic notation and CoAP is given below:

~~~~~~~~~~~
    4.01 Unauthorized
    Content-Format: application/ace-groupcomm+cbor
    {"AS": "coaps://as.example.com/token"}
~~~~~~~~~~~
{: #AS-info-ex title="AS Information example"}
{: artwork-align="center"}

Authorisation Server (AS) Discovery is also defined in Section 2.2.6.1 of 
{{I-D.ietf-ace-mqtt-tls-profile}} for MQTT v5 clients (and not supported for MQTT v3 clients).  

##  Authorising to the KDC and the Broker {#auth-request} 

After retrieving the AS address, the Client sends two Authorisation Requests to the AS for the KDC and the Broker, respectively.

Note that the AS authorises:

1. What endpoints are allowed to Publish or Subscribe to the Broker.
2. What endpoints are allowed to access to which topic(s).

Both requests include the following fields from the Authorization Request
(Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}):

* 'scope', containing the topic identifiers, that the Client wishes to access
* 'audience', an identifier, corresponding to either the KDC or the Broker.

Other additional parameters can be included if necessary, as defined in
{{I-D.ietf-ace-oauth-authz}}.

The 'scope' parameter is encoded as follows, where 'gname' is treated as topic identifier or filter.

~~~~~~~~~~~
   gname = tstr

   role = tstr

   scope_entry = [ gname , ? ( role / [ 2*role ] ) ]

   scope = << [ + scope_entry ] >>
~~~~~~~~~~~
{: #scope-aif title="CDLL definition of scope, using as example group name encoded as tstr and role as tstr."}
{: artwork-align="center"}

Other scope representations are also possible and are described in (Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}). Note that in the AIF-MQTT data model described in Section 3 of the {{I-D.ietf-ace-mqtt-tls-profile}}, the role values have been further constrained to "pub" and "sub".

It must be noted that for pub-sub brokers, the scope represents pub-sub topics i.e., the application group. On the other hand, for the KDC, the scope represents
the security group. If there is a one-to-one mapping between the application group
and the security group, the client uses the same scope for both requests.
If there is not a one-to-one mapping, the correct policies regarding both
sets of scopes MUST be available to the AS.   
To be able to join the right security group associated with requested application groups (i.e., pub-sub topics), the client The client MUST ask for the correct
scopes in its Authorization Requests. How the client discovers the (application group, security group) association is out of scope of this document.

The AS responds with an Authorization Response to each request as defined in Section 5.8.2 of {{I-D.ietf-ace-oauth-authz}} and Section 3.2 of {{I-D.ietf-ace-key-groupcomm}}. 
The client needs to keep track of which response corresponds to which entity to
use the right token for the right audience, i.e., the KDC or the Broker.
In case CoAP PubSub is used as communication protocol, 'profile' claim is set to "coap_pubsub_app" as defined in {{iana-coap-profile}}. In case MQTT PubSub is used as communication protocol, 'profile' claim is set to "mqtt_pubsub_app" as defined in {{iana-mqtt-profile}}.

# Key Distribution for PubSub Content Protection {#retr-cosekey}

## Token POST {#token-post}
After receiving a token from the AS, the Client posts the token to the KDC 
(Section 3.3 {{I-D.ietf-ace-key-groupcomm}}).  In addition to the token post, 
a Subscriber Client MAY ask for the format of the public keys in the group, used for source authentication, as well as any other group parameters. In this case, the message MUST have Content-Format set to "application/ace+cbor" defined in Section 8.16 of {{I-D.ietf-ace-oauth-authz}}.  The message payload MUST be formatted as a CBOR map, which MUST include the access token and
the 'sign_info' parameter. The details for the 'sign_info' parameter can be found in Section 3.3 of {{I-D.ietf-ace-key-groupcomm}}. Alternatively, the joining node may retrieve this information by other means as described in {{I-D.ietf-ace-key-groupcomm}}.

The KDC verifies the token to check of the Client is authorized to access the topic with the requested role. After successful verification, the Client is authorized to receive the group keying material from the KDC and join the group. The KDC replies to the Client with a 2.01 (Created) response, using Content-Format "application/ace+cbor". The payload of the 2.01 response is a CBOR map.  

A Publisher Client MUST send its own public key to the KDC when joining the group.
Since the access token from a Publisher Client will have "pub" role, the KDC MUST include
'kdcchallenge' in the CBOR map, specifying a dedicated challenge N_S generated by the KDC.  The Client uses this challenge to prove possession of its own private key (see {{I-D.ietf-ace-key-groupcomm}} for details).

## Join Request and Join Response {#join-request}
In the next step, a joining node MUST have a secure communication association
established with the KDC, before starting to join a group under that KDC.  Possible ways to provide a secure communication association are described in the DTLS transport profile
{{I-D.ietf-ace-dtls-authorize}} and OSCORE transport profile {{I-D.ietf-ace-oscore-profile}} of ACE. 

After establishing a secure communication, the Client sends a Joining Request to the KDC as described in Section 4.3 of {{I-D.ietf-ace-key-groupcomm}}. More specifically, the Client sends a POST request to the /ace-group/GROUPNAME endpoint on KDC, with Content-Format "application/ace-groupcomm+cbor" that MUST contain in the payload (formatted as a CBOR map, Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}):

* 'scope' parameter set to the specific group that the Client is attempting to join, i.e., the group name, and the roles it wishes to have in the group. This value corresponds to one scope entry, as defined in {{auth-request}}.
* 'get_pub_keys' parameter set to the empty array if the Client needs to retrieve the public keys of the other pubsub members,
* 'client\_cred' parameter containing the Client's public key formatted as a COSE_Key (as defined in {{iana-ace-groupcomm-key}}), if the Client is a Publisher,
* 'cnonce', encoded as a CBOR byte string, and including a dedicated nonce N_C generated by the Client, if 'client\_cred' is present,
* 'client\_cred\_verify', set to a signature computed over the 'rsnonce' concatenated with cnonce, if 'client\_cred' is present,
* OPTIONALLY, if needed, the 'pub_keys_repos' parameter

TODO: Check 'cnonce'

Note that for a Subscriber-only Client,  the Joining Request MUST NOT contain the 'client\_cred parameter', the role element in the 'scope' parameter MUST be set to "sub". The Subscriber MUST have access to the public keys of all the Publishers; this MAY be achieved in the Joining Request by using the parameter 'get\_pub\_keys' 
encoding the CBOR simple value 'null' (0xf6) (as described in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}) to retrieve the public keys of all the publishers.

If the 'client\_cred' parameter is present, KDC stores the public key of the Client.
Note that the alg parameter in the 'client\_cred' COSE\_Key MUST be a signing algorithm, as defined in {{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}, and that it is the same algorithm used to compute the signature sent in 'client\_cred\_verify'.

The KDC responds with a Joining Response, which has the Content-Format "application/ace-groupcomm+cbor". The payload (formatted as a CBOR map) MUST contain the following fields from the Joining Response (Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}):

- 'kty' identifies a key type "COSE\_Key".
- 'key', which contains a "COSE\_Key" object (defined in {{I-D.ietf-cose-rfc8152bis-algs}}{{I-D.ietf-cose-rfc8152bis-struct}}, containing:
  * 'kty' with value 4 (symmetric)
  * 'alg' with value defined by the AS (Content Encryption Algorithm)
  * 'Base IV' with value defined by the AS
  * 'k' with value the symmetric key value
  * OPTIONALLY, 'kid' with an identifier for the key value
- OPTIONALLY, 'exp' with the expiration time of the key
- 'pub\_keys', containing the public keys of all publishers, formatted as COSE_Keys, if the 'get\_pub\_keys' parameter was present and set to the empty array in the Key Distribution Request. For Subscriber Clients, the Joining Response MUST contain the 'pub\_keys' parameter.

An example of the Joining Request and corresponding Response for a CoAP Publisher using CoAP and CBOR is specified in {{fig-req-pub-kdc}} and {{fig-resp-pub-kdc}}, where SIG is a signature computed using the private key associated to the public key and the algorithm in 'client\_cred'.

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

~~~~~~~~~~~~
{
  "kty" : "COSE_Key",
  "key" : {1: 4, 2: h'1234', 3: 12, 5: h'1f389d14d17dc7',
          -1: h'02e2cc3a9b92855220f255fff1c615bc'}
}
~~~~~~~~~~~~
{: #fig-resp-pub-kdc title="Joining Response payload for a Publisher"}
{: artwork-align="center"}


An example of the payload of a Joining Request and corresponding Response for a Subscriber using CoAP and CBOR is specified in {{fig-req-sub-kdc}} and {{fig-resp-sub-kdc}}.

~~~~~~~~~~~~
{
  "scope" : ["Broker1/Temp", "sub"],
  "get_pub_keys" : null
}
~~~~~~~~~~~~
{: #fig-req-sub-kdc title="Joining Request payload for a Subscriber"}
{: artwork-align="center"}

~~~~~~~~~~~~
{
  "scope" : ["Broker1/Temp", "sub"],
  "kty" : "COSE_Key"
  "key" : {1: 4, 2: h'1234', 3: 12, 5: h'1f389d14d17dc7',
          -1: h'02e2cc3a9b92855220f255fff1c615bc'},
  "pub_keys" : [
   {
      1 : 2, / type EC2 /
      2 : h'11', / kid /
      3 : -7, / alg ECDSA with SHA-256 /
      -1 : 1 , / crv P-256 /
      -2 : h'65eda5a12577c2bae829437fe338701a10aaa375e1bb5b5de108de43
      9c08551d', / x /
      -3 : h'1e52ed75701163f7f9e40ddf9f341b3dc9ba860af7e0ca7ca7e9eecd
      0084d19c' / y /
    }
  ]
}
~~~~~~~~~~~~
{: #fig-resp-sub-kdc title="Joining Response payload for a Subscriber"}
{: artwork-align="center"}

# PubSub Protected Communication {#protected_communication}

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
The publication (the resource representation) is protected with COSE  ({{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}) by the publisher.
The (E) message is the subscription of the Subscriber. The subscription MAY be unprotected.
The (F) message is the response from the Broker, where the publication is protected with COSE by the publisher.


~~~~~~~~~~~
  Publisher                Broker               Subscriber
      | --- PUT /topic ----> |                       |
      |  protected with COSE |                       |
      |                      | <--- GET /topic ----- |
      |                      |                       |
      |                      | ---- response ------> |
      |                      |  protected with COSE  |
~~~~~~~~~~~
{: #flow title="(E), (F), (G): Example of protected communication for CoAP"}
{: artwork-align="center"}

The flow graph is presented below for CoAP. The message flow is similar for MQTT, where PUT corresponds to a PUBLISH message, and GET corresponds to a SUBSCRIBE message. 
Whenever a Client publishes a new message, the Broker sends this message to all valid subscribers. 

## Using COSE Objects To Protect The Resource Representation {#oscon}

The Publisher uses the symmetric COSE Key received from the KDC ({{retr-cosekey}}) to protect the payload of the PUBLISH operation (Section 4.3 of {{I-D.ietf-core-coap-pubsub}} and {{MQTT-OASIS-Standard-v5}}). Specifically, the COSE Key is used to create a COSE\_Encrypt0 object with algorithm specified by KDC. The Publisher uses the private key corresponding to the public key sent to the KDC in exchange B ({{retr-cosekey}}) to countersign the COSE Object as specified in {{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}. The payload is replaced by the COSE object before the publication is sent to the Broker.

The Subscriber uses the 'kid' in the 'countersignature' field in the COSE object to retrieve the right public key to verify the countersignature. It then uses the symmetric key received from KDC to verify and decrypt the publication received in the payload from the Broker (in the case of CoAP the publication is received by the CoAP Notification and for MQTT, it is received as a PUBLISH message from the Broker to the subscribing client).

The COSE object is constructed in the following way:

* The protected Headers (as described in {{I-D.ietf-cose-rfc8152bis-algs}} {{I-D.ietf-cose-rfc8152bis-struct}}) MUST contain the kid parameter if it was provided in the
Joining Response, with value the kid of the symmetric COSE Key received in {{retr-cosekey}} and MUST contain the content encryption algorithm.
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

# Profile-specific Considerations
This section summarises the CoAP and MQTT specific pub-sub communications, and considerations respectively. 

## CoAP PubSub Application Profile {#coap_profile}

A CoAP Pub-Sub Client and Broker use an ACE transport profile such as DTLS {{I-D.ietf-ace-dtls-authorize}}, OSCORE {{I-D.ietf-ace-oscore-profile}}. 

As shown in {{archi}}, (A) is an Access Token Request and Response exchange between Publisher and Authorization Server to retrieve the Access Token and RS (Broker) Information. As specified, the Client has the role of a CoAP client, the Broker has the role of the CoAP server. 

(B) corresponds to the retrieval of the keying material to protect the publication end-to-end (see {{oscon}}), and uses {{I-D.ietf-ace-key-groupcomm}}. The details are defined in {{retr-cosekey}}.

(C) corresponds to the exchange between the Client and  the Broker, where the Client sends its access token to the Broker and establishes a secure connection with the Broker. Depending on the Information received in (A), this can be for example DTLS handshake, or other protocols. Depending on the application, there may not be the need for this set up phase: for example, if OSCORE is used directly. Note that, in line with what defined in the ACE transport profile used, the access token includes the scope (i.e. pubsub topics on the Broker) the Publisher is allowed to publish to. For implementation simplicity, it is RECOMMENDED that the ACE transport profile used.

After the previous phases have taken place, the pub-sub communication can commence. The operations of publishing and subscribing are defined in {{I-D.ietf-core-coap-pubsub}}.

## MQTT PubSub Application Profile {#mqtt-pubsub}

The steps MQTT clients go through are similar to the CoAP clients as described in {{coap_profile}}. 
 The payload that is carried in MQTT messages will be protected using COSE.

In MQTT, topics are organised as a tree, and in the {{I-D.ietf-ace-mqtt-tls-profile}} 
'scope' captures permissions for not a single topic but a topic filter. Therefore, topic names (i.e., group names) may include wildcards spanning several levels of the topic tree.
Hence, it is important to distinguish application groups and security groups defined in {{I-D.ietf-core-groupcomm-bis}}. An application group has relevance at the application level - for example, in MQTT an application group could denote all topics stored under "“home/lights/". On the other hand, a security group is a group of endpoints that each store group security material to exchange secure communication within the group. The group communication in {{I-D.ietf-ace-key-groupcomm}} refers to security groups.
ToDo: Give a more complete example

For an MQTT client we envision the following steps to take place:

1. Client sends a token request to AS for the requested topics (application groups) using the broker as the audience.
2. Client sends a token request to AS for the corresponding security groups for its application groups using the KDC as the audience.
3. Client sends join requests to KDC to gets the keys for these security groups.
4. Client authorises to the Broker with the token (described in {{I-D.ietf-ace-mqtt-tls-profile}}).
5. A Publisher Client sends PUBLISH messages for a given topic and protects the payload with the corresponding key for the associated security group. RS validates the PUBLISH message by checking the topic's security group association and the stored token.
6. A Subscriber Client may send SUBSCRIBE messages with one or multiple topic filters. 
A topic filter may correspond to multiple topics but MUST belong to a single security group. If requested topics are in multiple security groups, then these topics SHOULD be separated into the corresponding topic filters in the SUBSCRIBE message. 

# Security Considerations

In the profile described above, the Publisher and Subscriber use asymmetric crypto, which would make the message exchange quite heavy for small constrained devices. Moreover, all Subscribers must be able to access the public keys of all the Publishers to a specific topic to be able to verify the publications. Such a database could be set up and managed by the same entity having control of the
key material for that topic, i.e. KDC.

An application where it is not critical that only authorized Publishers can publish on a topic may decide not to make use of the asymmetric crypto and only use symmetric encryption/MAC to confidentiality and integrity protection of the publication.
However, this is not recommended since, as a result, any authorized Subscribers with access to the Broker may forge unauthorized publications without being detected. In this symmetric case the Subscribers would only need one symmetric key per topic, and would not need to know any information about the Publishers, that can be anonymous to it and the Broker.

Subscribers can be excluded from future publications through re-keying for a certain topic. This could be set up to happen on a regular basis, for certain applications. How this could be done is out of scope for this work.

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

--- back


# Requirements on Application Profiles

This section lists the specifications on this profile based on the requirements defined in Appendix A of {{I-D.ietf-ace-key-groupcomm}}

* REQ1: Specify the encoding and value of the identifier of group or topic of 'scope': see {{retr-cosekey}}).

* REQ2: Specify the encoding and value of roles of 'scope': see {{retr-cosekey}}).

* REQ3: Optionally, specify the acceptable values for 'sign_alg': TODO

* REQ4: Optionally, specify the acceptable values for 'sign_parameters': TODO

* REQ5: Optionally, specify the acceptable values for 'sign_key_parameters': TODO

* REQ6: Optionally, specify the acceptable values for 'pub_key_enc': TODO

* REQ7: Specify the exact format of the 'key' value: COSE_Key, see {{retr-cosekey}}.

* REQ8: Specify the acceptable values of 'kty' : "COSE_Key", see {{retr-cosekey}}.

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
