---
title: Pub-Sub Profile for Authentication and Authorization for Constrained Environments (ACE)
abbrev: pubsub-profile
docname: draft-ietf-ace-pubsub-profile-latest
#date: 2017-03-13
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

  RFC8152:
  RFC2119:
  RFC6749:
  RFC7049:
  I-D.ietf-ace-oauth-authz:
  I-D.ietf-core-coap-pubsub:
  I-D.ietf-ace-key-groupcomm:
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

This specification defines an application profile for authentication and authorization for publishers and subscribers in a constrained pub-sub scenario, using the ACE framework. This profile relies on transport layer or application layer security to authorize the publisher to the broker. Moreover, it describes application layer security for publisher-broker and subscriber-broker communication.


--- middle

# Introduction

In the publish-subscribe (pub-sub) scenario, devices with limited reachability communicate via a broker, which enables store-and-forward messaging between the devices. This document defines a way to authorize pub-sub nodes using the ACE framework {{I-D.ietf-ace-oauth-authz}}, and to provide the keys for protecting the communication between them. The pub-sub communication using the Constrained Application Protocol (CoAP) is specified in {{I-D.ietf-core-coap-pubsub}}, while the one using MQTT is specified in {{MQTT-OASIS-Standard-v5}}.  This document gives detailed specifications for MQTT and CoAP pub-sub, but can easily be adapted for other transport protocols as well.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 {{RFC2119}}.

Readers are expected to be familiar with the terms and concepts
described in {{I-D.ietf-ace-oauth-authz}}, {{I-D.ietf-ace-key-groupcomm}}. In particular, analogously to {{I-D.ietf-ace-oauth-authz}}, terminology for entities in the architecture such as Client (C), Resource Server (RS), and Authorization Server (AS) is defined in OAuth 2.0 {{RFC6749}} and {{I-D.ietf-ace-actors}}, and terminology for entities such as the Key Distribution Center (KDC) and Dispatcher in {{I-D.ietf-ace-key-groupcomm}}.

Readers are expected to be familiar with terms and concepts of pub-sub group communication, as described in {{I-D.ietf-core-coap-pubsub}}, or MQTT {{MQTT-OASIS-Standard-v5}}.

# Application Profile Overview {#overview}

The objective of this document is to specify how to authorize nodes, provide keys, and protect a pub-sub communication, using {{I-D.ietf-ace-key-groupcomm}}, which expands from the ACE framework ({{I-D.ietf-ace-oauth-authz}}), and transport profiles ({{I-D.ietf-ace-dtls-authorize}}, {{I-D.ietf-ace-oscore-profile}}, {{I-D.ietf-ace-mqtt-tls-profile}}). The pub-sub communication protocol can be based on CoAP, as described in {{I-D.ietf-core-coap-pubsub}}, MQTT {{MQTT-OASIS-Standard-v5}} , or other transport.

The architecture of the scenario is shown in {{archi}}.

~~~~~~~~~~~~
             +----------------+   +----------------+
             |                |   |                |
             | Authorization  |   | Authorization  |
             |    Server 1    |   |    Server 2    |
             |                |   |                |
             +----------------+   +----------------+
                      ^                  ^  ^
                      |                  |  |
     +---------(A)----+                  |  +-----(D)------+
     |   +--------------------(B)--------+                 |
     v   v                                                 v
+------------+             +------------+              +------------+
|            |             |            |              |            |
| Publisher  | ----(E)---> |   Broker   |              | Subscriber |
|            |             |            | <----(F)---- |            |
|            |             |            | -----(G)---> |            |
+------------+             +------------+              +------------+
~~~~~~~~~~~~~
{: #archi title="Architecture for Pub-Sub with Authorization Servers"}
{: artwork-align="center"}

The RS is the broker and contains the topics that clients can publish or subscribe to. Therefore, RS corresponds to the Dispatcher in {{I-D.ietf-ace-key-groupcomm}}.
The AS1 hosts the policies about the Broker: what endpoints are allowed to Publish on the Broker. The Clients access AS1 to get write access to the Broker.
The AS2 hosts the policies about the topic: what endpoints are allowed to access what topic. This node represents both the AS and Key Distribution Center roles from {{I-D.ietf-ace-key-groupcomm}}.

There are four phases, the first three can be done in parallel.

<!-- Jim
 One of the things that I am not currently happy with is that you are looking at AS1 and AS2 as being independent appliers of access control logic without any communication between them.  I think that AS1 needs the ability to give policy to AS2 on a topic after it has been created and before any subscribers get keys.  In the case they are co-resident this is trivial, in other cases it may not be.

 FP: AS1 and AS2 have in my mind clearly separated functions. There is some coordination involved of course (to gain knowledge of the policies), but I think that how this is dealt with is application specific. For example, there could be some node distributing those (they do not need to talk to each other directly). Added some generic considerations at the end of the section.
 CS: Agree with Jim that this can be dealt better. Will present a different architecture in the IETF 110 meeting.
-->

1. The Publisher requests publishing access to the Broker at the AS1, and communicates with the Broker to set up security.
2. The Publisher requests access to a specific topic at the AS2.
3. The Subscriber requests access to a specific topic at the AS2.
4. The Publisher and the Subscriber securely post and get publications from the Broker.

This exchange aims at setting up two different security associations: on the one hand, the Publisher has a security association with the Broker, to protect the communication and securely authorize the Publisher to publish on a topic (Security Association 1). On the other hand, the Publisher has a security association with the Subscriber, to protect the publication content itself (Security Association 2).
The Security Association 1 set up using AS1 and a transport profile of {{I-D.ietf-ace-oauth-authz}}, the Security Association 2 is set up using AS2 and {{I-D.ietf-ace-key-groupcomm}}.

Note that, analogously to the Publisher, the Subscriber can also set up an additional security association with the Broker, using an AS, in the same way the Publisher does with AS1. In this case, only authorized Subscribers would be able to get notifications from the Broker. The overhead would be that each Subscriber should access the AS and get all the information to start a secure exchange with the Broker.

<!-- Jim
 It is not clear to me that your allocation of roles to AS1 and
AS2 I correct.  If you have a second publisher, does it need to talk to both
AS1 and AS2 or just to AS2?  Is this really an AS1 controls creation of topics and AS2 controls publishing and subscribing to topics?  If the publisher loses its membership in the group for any reason, should it be able to publish willy-nilly anyway?  I.e. should AS2 be able to "revoke" the publishers right to publish?

FP: A second publisher would need to talk to both AS1 and AS2. As I intended, AS1 controls who can publish to (or create) a topic on a broker, AS2 more generally controls who can decrypt the content of the publication.
"Losing the membership" can mean "not being able to access (read or write) the content of the publication", in which case AS2 should revoke the node's rights or it can mean "not allowed to publish on the broker" (maybe it is still allowed to subscribe to the topic), in which case AS1 should revoke the node's right. Both revocations are not specified for now.

CS: I think AS controls who can encrypt as well as decrypt. So, if pub1 got revoked, is 
it revoked in AS1 or in AS2? Who needs to send which information to whom? 

-->

~~~~~~~~~~~~
+------------+             +------------+              +------------+
|            |             |            |              |            |
| Publisher  |             |   Broker   |              | Subscriber |
|            |             |            |              |            |
|            |             |            |              |            |
+------------+             +------------+              +------------+
      :   :                       :                           :
      :   '------ Security -------'                           :
      :         Association 1                                 :
      '------------------------------- Security --------------'
                                     Association 2
~~~~~~~~~~~~~

<!-- Jim
  I don't think the picture is correct at the bottom of the section.  You have a Publisher-Subscriber client/client association

  FP: Both publisher and subscriber are CoAP client, as specified in the pub-sub doc. The association is done via the sec context that is shared between pubs and subs.
-->
<!-- Jim
Is there any expectation that the broker should be notified
on a "revocation" of a publisher's right to publish?  (As opposed to the right just expiring.)  There is no need to enforce subscribers right to subscribe since a key roll over means that they are getting gibberish.

FP: Yes, the broker should be notified of revocation. This is not specified here, and I think this is a general topic that the framework should address: no profile deals with revocations so far, as far as I can tell. Some additional content on revocation is in the ace-key-groupcomm doc.
-->

Note that AS1 and AS2 might either be co-resident or be 2 separate physical entities, in which case access control policies must be exchanged between AS1 and AS2, so that they agree on rights for joining nodes about specific topics. How the policies are exchanged is out of scope for this specification.
<!-- Cigdem: I think this should be handled differently. 
-->

# PubSub Application Profiles {#profile}

Each profile defined in this document uses {{I-D.ietf-ace-key-groupcomm}}, which expands from the ACE framework. This section defines which exact parameters from {{I-D.ietf-ace-key-groupcomm}} have to be used, and the values for each parameter. Since {{I-D.ietf-ace-oauth-authz}} recommends the use of CoAP anc CBOR, this document describes the exchanges assuming CoAP and CBOR are used. However, using HTTP instead of CoAP is possible, using the corresponding parameters and methods. Analogously, JSON {{RFC8259}} can be used instead of CBOR, using the conversion method specified in Sections 4.1 and 4.2 of {{RFC7049}}. In case JSON is used, the Content Format or Media Type of the message has to be changed accordingly.

The Publisher and the Subscriber map to the Client in {{I-D.ietf-ace-key-groupcomm}}, the AS2 maps to the AS and to the KDC, the Broker maps to the Dispatcher.

Note that both publishers and subscribers use the same profile. <!--, called "coap_pubsub_app". -->

## Retrieval of COSE Key for protection of content {#retr-cosekey}

This phase is common to both Publisher and Subscriber. To maintain generality, the Publisher or Subscriber is referred to as Client in this section.

~~~~~~~~~~~
   Client                            Broker             AS2
      | [----- Resource Request ---->] |                 |
      |                                |                 |
      | [<-- AS1, AS2 Information ---] |                 |
      |                                                  |
      | [------ Pub Key Format Negotiation Request --->] |
      |                                                  |
      | [<---- Pub Key Format Negotiation Response ----] |
      |                                                  |
      | -- Authorization + Key Distribution Request ---> |
      |                                                  |
      | <-- Authorization + Key Distribution Response -- |
      |                                                  |
~~~~~~~~~~~
{: #B title="B: Access request - response"}
{: artwork-align="center"}

<!-- CS: Removed "in charge of the topic back" from the paragraph below, as in MQTT, in a connection request, the Broker won't know which topics the client will request access to.
-->
Complementary to what is defined in {{I-D.ietf-ace-oauth-authz}} (Section 5.1.1), for AS discovery, the Broker MAY send the address of both ASes to the Client in the 'AS' parameter in the AS Information as a response to an Unauthorized Resource Request (Section 5.1.2). The uri of AS2 is concatenated to the uri of AS1, and separated by a comma. An example using CBOR diagnostic notation and CoAP is given below:

~~~~~~~~~~~
    4.01 Unauthorized
    Content-Format: application/ace+cbor
    {"AS": "coaps://as1.example.com/token,
    coaps://as2.example.com/pubsubkey"}
~~~~~~~~~~~
{: #AS-info-ex title="AS1, AS2 Information example"}
{: artwork-align="center"}

Authorisation Server (AS) Discovery is also possible for MQTT v5 clients (and not supported for MQTT v3 clients).  AS Discovery defined in {{I-D.ietf-ace-mqtt-tls-profile}} (Section 2.2.6.1) and is implemented by including a User Property to the CONNACK (Connection Acknowledgement) message returned by the Broker. The User Property can be used multiple times to represent multiple name, value pairs. The same name is allowed to appear more than once. So, the Broker returns two "ace_as_hint" fields corresponding to two ASes.  

<!-- Jim
 I don't' think that the returned info on the first request is going to be the same for publishers and subscribers.  Not sure what this should really look like.

 The broker _may_ send this info to both pub and sub, and then the subscriber could just discard the AS it does not need (AS1). Or the sub could know what AS to contact from a different exchange.
-->

After retrieving the AS2 address, the Client MAY send a request to the AS2, to retrieve necessary information concerning the public keys in the group, as well as the algorithm and related parameters for computing signatures in the group. This request is a subset of the Token POST request defined in Section 3.3 of {{I-D.ietf-ace-key-groupcomm}}, specifically a CoAP POST request to a specific resource at the AS2, including only the parameters 'sign_info' and 'pub_key_enc' in the CBOR map in the payload. The default url-path for this resource is /ace-group/gid/cs-info, where "gid" is the topic identifier, but implementations are not required to use this name, and can use their own instead. The AS MUST respond with the response defined in Section 3.3 of {{I-D.ietf-ace-key-groupcomm}}, specifically including the parameters 'sign_info', 'pub_key_enc', and 'rsnonce' (8 bytes pseudo-random nonce generated by the AS).

After that, the Client sends an Authorization + Joining Request, which is an Authorization Request merged with a Joining Request, as described in {{I-D.ietf-ace-key-groupcomm}}, Sections 3.1 and 4.2. The reason for merging these two messages is that the AS2 is both the AS and the KDC, in this setting, so the Authorization Response and the Post Token message are not necessary.

More specifically, the Client sends a POST request to the /ace-group/gid endpoint on AS2, with Content-Format = "application/ace+cbor" that MUST contain in the payload (formatted as a CBOR map):

- the following fields from the Joining Request (Section 4.2 of {{I-D.ietf-ace-key-groupcomm}}):
  * 'scope' parameter set to a CBOR array containing:
    - the broker's topic as first element, and
    - the text string "publisher" if the client request to be a publisher, "subscriber" if the client request to be a subscriber, or a CBOR array containing both, if the client request to be both.
  * 'get_pub_keys' parameter set to the empty array if the Client needs to retrieve the public keys of the other pubsub members,
  * 'client\_cred' parameter containing the Client's public key formatted as a COSE_Key, if the Client needs to directly send that to the AS2,
  * 'cnonce', set to a 8 bytes long pseudo-random nonce, if 'client\_cred' is present,
  * 'client\_cred\_verify', set to a singature computed over the rsnonce concatenated with cnonce, if 'client\_cred' is present,
  * OPTIONALLY, if needed, the 'pub_keys_repos' parameter
<!-- Cigdem
Should scope explanation be changed as scope is now using the AIF and is defined differently MQTT.
-->

- the following fields from the Authorization Request (Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}):
  * OPTIONALLY, if needed, additional parameters such as 'client_id'

TODO: 'cnonce' might change name.
TODO: register media type ace+json for HTTP?
<!-- I think this needs to be registered with MQTT-TLS Profile?
-->

Note that the alg parameter in the 'client_cred' COSE_Key MUST be a signing algorithm, as defined in section 8 of {{RFC8152}}, and that it is the same algorithm used to compute the signature sent in 'client\_cred\_verify'.

Examples of the payload of a Authorization + Joining Request are specified in {{fig-post-as2}} and {{fig-post2-as2}}.

The AS2 verifies that the Client is authorized to access the topic and, if the 'client_cred' parameter is present, stores the public key of the Client.

The AS2 response is an Authorization + Joining Response, with Content-Format = "application/ace+cbor". The payload (formatted as a CBOR map) MUST contain:

<!-- Jim
 why not use the cnf return value for the key?  Also there is no reason to make it a bstr rather than a map.

 I did not use the cnf because of the following reasoning: the key is not used to authenticate the client (pub or sub) to the rs (broker), it is not a pop-key related to a token (no token). For subs, there are both cnf and key parameter (see {{fig-resp2-as2}}). Also, see the example on https://tools.ietf.org/html/draft-seitz-ace-oauth-authz-00#section-6.5 (token-less exchange).
 OK, Changed to map.
-->
<!-- Jim
  need to define a signers_keys element which returns all of the signing keys.  Defined as an array of keys.  Return other signers for multiple publishers

  Are you sure this comment should be in this section? To a subscriber, yes, the set of all signers keys are returned (see {{subs-profile}} section: "The AS2 response contains a "cnf" parameter whose value is set to a COSE Key Set, (Section 7 of {{RFC8152}}) i.e. an array of COSE Keys, which contains the public keys of all authorized Publishers..."). If you did mean it for publishers, I don't see why.
-->
- the following fields from the Joining Response (Section 4.2 of {{I-D.ietf-ace-key-groupcomm}}):
  * 'kty' identifies a key type "COSE_Key", as defined in {{iana-ace-groupcomm-key}}.
  * 'key', which contains a "COSE_Key" object (defined in {{RFC8152}}, containing:
    * 'kty' with value 4 (symmetric)
    * 'alg' with value defined by the AS2 (Content Encryption Algorithm)
    * 'Base IV' with value defined by the AS2
    * 'k' with value the symmetric key value
    * OPTIONALLY, 'kid' with an identifier for the key value
  * OPTIONALLY, 'exp' with the expiration time of the key
  * 'pub\_keys', containing the public keys of all authorized signing members formatted as COSE_Keys, if the 'get\_pub\_keys' parameter was present and set to the empty array in the Authorization + Key Distribution Request

- the following fields from the Authorization Response (Section 3.2 of {{I-D.ietf-ace-key-groupcomm}}):
  * 'profile' set to the corresponding value, see {{coap}} or {{mqtt}}
  * OPTIONALLY 'scope', set to a CBOR array containing:
    - the broker's topic as first element, and
    - the string "publisher" if the client is an authorized publisher, "subscriber" if the client is an authorized subscriber, or a CBOR array containing both, if the client is authorized to be both.
<!-- Cigdem
Scope should be changed to use AIF and is defined differently MQTT.
Scope parameter itself can be an array. 
-->

Examples for the response payload are detailed in {{fig-resp-as2}} and {{fig-resp2-as2}}.

## coap_pubsub_app Application Profile {#coap}

In case CoAP PubSub is used as communication protocol:

  * 'profile' set to "coap_pubsub_app", as specified in {{iana-coap-profile}}.

## mqtt_pubsub_app Application Profile {#mqtt}

In case mQTT PubSub is used as communication protocol:

  * 'profile' set to "mqtt_pubsub_app", as specified in {{iana-mqtt-profile}}.

# CoAP PubSub Application Profile {#coap_profile}

## Publisher

In this section, it is specified how the Publisher requests, obtains and communicates to the Broker the access token, as well as the retrieval of the keying material to protect the publication.

~~~~~~~~~~~
             +----------------+   +----------------+
             |                |   |                |
             | Authorization  |   | Authorization  |
             |    Server 1    |   |    Server 2    |
             |                |   |                |
             +----------------+   +----------------+
                      ^                  ^
                      |                  |
     +---------(A)----+                  |
     |   +--------------------(B)--------+
     v   v
+------------+             +------------+
|            | ----(C)---> |            |
| Publisher  |             |   Broker   |
|            |             |            |
|            |             |            |
+------------+             +------------+
~~~~~~~~~~~
{: #pubsub-1 title="Phase 1: Publisher side"}
{: artwork-align="center"}

This is a combination of two independent phases:

* one is the establishment of a secure connection between Publisher and Broker, using an ACE transport profile such as DTLS {{I-D.ietf-ace-dtls-authorize}},  OSCORE {{I-D.ietf-ace-oscore-profile}}. (A)(C)
* the other is the Publisher's retrieval of keying material to protect the publication. (B)

In detail:

(A) corresponds to the Access Token Request and Response between Publisher and Authorization Server to retrieve the Access Token and RS (Broker) Information.
As specified, the Publisher has the role of a CoAP client, the Broker has the role of the CoAP server.

(C) corresponds to the exchange between Publisher and Broker, where the Publisher sends its access token to the Broker and establishes a secure connection with the Broker. Depending on the Information received in (A), this can be for example DTLS handshake, or other protocols. Depending on the application, there may not be the need for this set up phase: for example, if OSCORE is used directly. Note that, in line with what defined in the ACE transport profile used, the access token includes the scope (i.e. pubsub topics on the Broker) the Publisher is allowed to publish to. For implementation semplicity, it is RECOMMENDED that the ACE transport profile used and this specification use the same format of "scope".

(A) and (C) details are specified in the profile used.

(B) corresponds to the retrieval of the keying material to protect the publication end-to-end with the subscribers (see {{oscon}}), and uses {{I-D.ietf-ace-key-groupcomm}}. The details are defined in {{retr-cosekey}}.

An example of the payload of an Authorization + Joining Request and corresponding Response for a CoAP Publisher using CoAP and CBOR is specified in {{fig-post-as2}} and {{fig-resp-as2}}, where SIG is a signature computed using the private key associated to the public key and the algorithm in "client_cred".

~~~~~~~~~~~~
{
  "scope" : ["Broker1/Temp", "publisher"],
  "client_id" : "publisher1",
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
{: #fig-post-as2 title="Authorization + Joining Request payload for a Publisher"}
{: artwork-align="center"}

~~~~~~~~~~~~
{
  "profile" : "coap_pubsub_app",
  "kty" : "COSE_Key",
  "key" : {1: 4, 2: h'1234', 3: 12, 5: h'1f389d14d17dc7',
          -1: h'02e2cc3a9b92855220f255fff1c615bc'}
}
~~~~~~~~~~~~
{: #fig-resp-as2 title="Authorization + Joining Response payload for a Publisher"}
{: artwork-align="center"}

## Subscriber {#subs-profile}

In this section, it is specified how the Subscriber retrieves the keying material to protect the publication.

~~~~~~~~~~~
                                  +----------------+
                                  |                |
                                  | Authorization  |
                                  |    Server 2    |
                                  |                |
                                  +----------------+
                                            ^
                                            |
                                            +-----(D)------+
                                                           |
                                                           v
                                                       +------------+
                                                       |            |
                                                       | Subscriber |
                                                       |            |
                                                       |            |
                                                       +------------+
~~~~~~~~~~~~~
{: #pubsub-2 title="Phase 2: Subscriber side"}
{: artwork-align="center"}

Step (D) between Subscriber and AS2 corresponds to the retrieval of the keying material to verify the publication end-to-end with the publishers (see {{oscon}}).  The details are defined in {{retr-cosekey}}

This step is the same as (B) between Publisher and AS2 ({{retr-cosekey}}), with the following differences:

* The Authorization + Joining Request MUST NOT contain the 'client\_cred parameter', the role element in the 'scope' parameter MUST be set to "subscriber". The Subscriber MUST have access to the public keys of all the Publishers; this MAY be achieved in the Authorization + Joining Request by using the parameter 'get_pub_keys' set to empty array.

<!-- CS: This looks like a client cannot join both a publisher and subscriber -->
<!--  I think 
the joining request should just have the scope (in MQTT this has permissions for
topics for both pub and sub), and use that to decide which information to send
to the clients.) 
-->

* The Authorization + Key Distribution Response MUST contain the 'pub_keys' parameter.

An example of the payload of an Authorization + Joining Request and corresponding Response for a CoAP Subscriber using CoAP and CBOR is specified in {{fig-post2-as2}} and {{fig-resp2-as2}}.

~~~~~~~~~~~~
{
  "scope" : ["Broker1/Temp", "subscriber"],
  "get_pub_keys" : [ ]
}
~~~~~~~~~~~~
{: #fig-post2-as2 title="Authorization + Joining Request payload for a Subscriber"}
{: artwork-align="center"}

~~~~~~~~~~~~
{
  "profile" : "coap_pubsub_app",
  "scope" : ["Broker1/Temp", "subscriber"],
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
{: #fig-resp2-as2 title="Authorization + Joining Response payload for a Subscriber"}
{: artwork-align="center"}

# MQTT PubSub Application Profile {#mqtt-pubsub}

The steps MQTT clients go through are similar to the CoAP clients as described in {{#coap_profile}}.

~~~~~~~~~~~
             +----------------+   +----------------+
             |                |   |                |
             | Authorization  |   | Authorization  |
             |    Server 1    |   |    Server 2    |
             |                |   |                |
             +----------------+   +----------------+
                      ^                  ^
                      |                  |
     +---------(A)----+                  |
     |   +--------------------(B)--------+
     v   v
+------------+             +------------+
|            | ----(C)---> |            |
| Publisher/ |             |   Broker   |
| Subscriber |             |            |
|            |             |            |
+------------+             +------------+
~~~~~~~~~~~
{: #pubsub-mqtt-1 title="Phase 1: Publisher side"}
{: artwork-align="center"}

The main difference is that the clients go through the two phases regardless of whether they act only as a Publisher or a Subscriber:

*The establishment of a secure connection between the Client and Broker, using the ACE transport profile for MQTT {{I-D.ietf-ace-mqtt-tls-profile}} ((A)(C) in {{#pubsub-mqtt-1}})
* The Client's retrieval of keying material to protect the publication ((B) in {{#pubsub-mqtt-1}}).

This document describes the exchanges between the Client and AS2 using CoAP and CBOR. However, 
the same exchanges can be implemented in HTTP using the corresponding parameters and methods. 
However, the payload that is carried in MQTT messages will be protected using COSE. 

To retrieve the keying material, the Client sends an Authorization + Joining Request. 
If the client will also act as a Subscriber, it MUST have access to the public keys of all the Publishers; 
this MAY be achieved in the Authorization + Joining Request by using the parameter 'get_pub_keys' set to empty array.  An example of the payload of an Authorization + Joining Request is specified in {{fig-post-mqtt2-as2}}.

<!-- Multiple topics support???-->
~~~~~~~~~~~~
{
  "scope" : [["topic1", ["pub","sub"]]],
  "client_id" : "client1",
  "get_pub_keys" : [ ]
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
{: #fig-post-mqtt-as2 title="Authorization + Joining Request payload for a Client"}
{: artwork-align="center"}

If the client does not have any Publisher role (e.g., scope does not contain "pub" keyword for any topic), 
the request MAY not contain the 'client\_cred parameter'. If the Client request includes Subscriber role (e.g.,
scope contains the "sub" keyword for at least one topic), The Authorization + Key Distribution Response MUST contain the 'pub_keys' parameter corresponding to the Publishers of the requested topics.
An example of the payload of an Authorization + Joining Response is presented in {{fig-resp-mqtt-as2}}.

~~~~~~~~~~~~
{
  "profile" : "mqtt_pubsub_app",
  "scope": [["topic1", ["pub","sub"]]],
  "kty" : "COSE_Key",
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
{: #fig-resp-mqtt-as2 title="Authorization + Joining Response payload for a Client"}
{: artwork-align="center"}

As seen in the above examples, in MQTT, topics are organised as a tree, and subscription requests may include wildcards spanning several levels of the topic tree. Therefore, depending on how topics are grouped, the KDC may have different sections of the tree keyed differently. In this case, an MQTT node may be returned keys for a wider set of topics that their token permits them. However, since the Broker authorises all Clients (regardless of their role is only Publisher or Subscriber), the Clients cannot access any messages sent for a topic beyond their token's scope. 

# Pub-Sub Protected Communication

<!-- Jim
 Need to talk about how to deal with multiple publishers - are you assigning different keys or are you using different IV sections?
Need to ensure that you don't have an error from using the same key/iv pair.

Right, the key is the same ("key" in previous sections), but the IV is different. Added Base IV in the COSE_Key in previous section, and partial IV in the COSE_Key. Added TODO for sending Partial IV range for each publisher.
-->

<!-- Jim
 Do you want to talk about coordination of the observer number and the iv of a message?

 What do you mean by "observer number"?
-->

This section specifies the communication Publisher-Broker and Subscriber-Broker, after the previous phases have taken place. The operations of publishing and subscribing are defined in {{I-D.ietf-core-coap-pubsub}}.

~~~~~~~~~~~~
+------------+             +------------+              +------------+
|            |             |            |              |            |
| Publisher  | ----(E)---> |   Broker   |              | Subscriber |
|            |             |            | <----(F)---- |            |
|            |             |            | -----(G)---> |            |
+------------+             +------------+              +------------+
~~~~~~~~~~~~~
{: #pubsub-3 title="Phase 3: Secure communication between Publisher and Subscriber"}
{: artwork-align="center"}

The (E) message corresponds to the publication of a topic on the Broker.
The publication (the resource representation) is protected with COSE ({{RFC8152}}).
The (F) message is the subscription of the Subscriber. The subscription MAY be unprotected in the case of CoAP, unless a profile of ACE {{I-D.ietf-ace-oauth-authz}} is used between Subscriber and Broker. The subscription is 
protected in the case of MQTT ({{#mqtt-pubsub}})
The (G) message is the response from the Broker, where the publication is protected with COSE.

The flow graph is presented below for CoAP. 
The message flow is similar for MQTT, where PUT corresponds to
a PUBLISH message, and GET corresponds to a SUBSCRIBE message. 
Whenever a Client publishes a new message, the Broker sends this message to all valid subscribers. 

~~~~~~~~~~~
  Publisher                Broker               Subscriber
      | --- PUT /topic ----> |                       |
      |  protected with COSE |                       |
      |                      | <--- GET /topic ----- |
      |                      |                       |
      |                      | ---- response ------> |
      |                      |  protected with COSE  |
~~~~~~~~~~~
{: #E-F-G-ex title="(E), (F), (G): Example of protected communication for CoAP"}
{: artwork-align="center"}

## Using COSE Objects To Protect The Resource Representation {#oscon}

The Publisher uses the symmetric COSE Key received from AS2 in exchange B ({{retr-cosekey}}) to protect the payload of the PUBLISH operation (Section 4.3 of {{I-D.ietf-core-coap-pubsub}} and {{MQTT-OASIS-Standard-v5}}). Specifically, the COSE Key is used to create a COSE\_Encrypt0 with algorithm specified by AS2. The Publisher uses the private key corresponding to the public key sent to the AS2 in exchange B ({{retr-cosekey}}) to countersign the COSE Object as specified in Section 4.5 of {{RFC8152}}. The payload is replaced by the COSE object before the publication is sent to the Broker.

The Subscriber uses the kid in the countersignature field in the COSE object to retrieve the right public key to verify the countersignature. It then uses the symmetric key received from AS2 to verify and decrypt the publication received in the payload from the Broker (in the case of CoAP the publication is received by the CoAP Notification and for MQTT, it is received as a PUBLISH message from the Broker to the subscribing client).

The COSE object is constructed in the following way:

* The protected Headers (as described in Section 3 of {{RFC8152}}) MAY contain the kid parameter, with value the kid of the symmetric COSE Key received in {{retr-cosekey}} and MUST contain the content encryption algorithm.
* The unprotected Headers MUST contain the Partial IV, with value a sequence number that is incremented for every message sent, and the counter signature that includes:
  - the algorithm (same value as in the asymmetric COSE Key received in (B)) in the protected header;
  - the kid (same value as the kid of the asymmetric COSE Key received in (B)) in the unprotected header;
  - the signature computed as specified in Section 4.5 of {{RFC8152}}.
* The ciphertext, computed over the plaintext that MUST contain the message payload.

The external_aad is an empty string.

An example is given in {{fig-cose-ex}}

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

The encryption and decryption operations are described in sections 5.3 and 5.4 of {{RFC8152}}.

# Security Considerations

In the profile described above, the Publisher and Subscriber use asymmetric crypto, which would make the message exchange quite heavy for small constrained devices. Moreover, all Subscribers must be able to access the public keys of all the Publishers to a specific topic to be able to verify the publications. Such a database could be set up and managed by the same entity having control of the topic, i.e. AS2.

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

### CoAP Profile Registration {#iana-mqtt-profile}

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

Profile: coap_pubsub_app

Description: COSE_Key object

References: {{RFC8152}}, \[\[This document\]\]

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

* REQ9: Specity the format of the identifiers of group members: TODO

* REQ10: Optionally, specify the format and content of 'group\_policies' entries: not defined

* REQ11: Specify the communication protocol the members of the group must use: CoAP pub/sub.

* REQ12: Specify the security protocol the group members must use to protect their communication. This must provide encryption, integrity and replay protection: Object Security of Content using COSE, see {{oscon}}.

* REQ13: Specify and register the application profile identifier : "coap_pubsub_app", see {{iana-profile}}.

* REQ14: Optionally, specify the encoding of public keys, of 'client\_cred', and of 'pub\_keys' if COSE_Keys are not used: NA.

* REQ15: Specify policies at the KDC to handle id that are not included in get_pub_keys: TODO

* REQ16: Specify the format and content of 'group_policies': TODO

* REQ17: Specify the format of newly-generated individual keying material for group members, or of the information to derive it, and corresponding CBOR label : not defined

* REQ18: Specify how the communication is secured between Client and KDC. Optionally, specify tranport profile of ACE {{I-D.ietf-ace-oauth-authz}} to use between Client and KDC: pre-set, as KDC is AS.

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
