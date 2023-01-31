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
  IANA.cose_algorithms:
  IANA.cose_key-type:
  IANA.cose_header-parameters:
  RFC2119:
  RFC6690:
  RFC6749:
  RFC7252:
  RFC7925:
  RFC8174:
  RFC8392:
  RFC8949:
  RFC9052:
  RFC9053:
  RFC9200:
  RFC9237:
  I-D.draft-ietf-rats-uccs-01:
  I-D.ietf-cose-cbor-encoded-cert:
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

This document defines an application profile for enabling secure group
communication for a constrained pub-sub scenario, where Publishers and Subscribers communicate through a broker, using the ACE framework. This profile relies on transport layer or application layer security profiles of ACE to achieve communication security, server authentication and proof-of-possession for a key owned by the Client and bound to an OAuth 2.0 Access Token. The document describes how to request and provision keying
material for group communication, and protect the content of the pub-sub client message exchange, focusing mainly on the pub-sub scenarios using the Constrained Application Protocol (CoAP) {{I-D.ietf-core-coap-pubsub}}.
--- middle

# Introduction

In the publish-subscribe (pub-sub) scenario, devices with limited reachability communicate via a broker, which enables store-and-forward messaging between these devices. This document defines a way to authorize pub-sub clients using the ACE framework {{RFC9200}} to obtain the keys for protecting the content of their pub-sub messages when communicating through the broker.

This document specifies how to request, distribute and renew keying material and configuration parameters to protect message exchanges for pub-sub communication, using {{I-D.ietf-ace-key-groupcomm}}, which expands from the ACE framework ({{RFC9200}}).  Message exchanges among the participants as well as message formats and processing follow the specifications for provisioning and renewing keying material in group communication scenarios in {{I-D.ietf-ace-key-groupcomm}}.

The pub-sub communication using the Constrained Application Protocol (CoAP) {{RFC7252}} is specified in {{I-D.ietf-core-coap-pubsub}}.This document gives detailed specifications for CoAP pub-sub, and describes how it can be adapted for MQTT {{MQTT-OASIS-Standard-v5}}; similar adaptations can extend to other transport protocols as well.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 {{RFC2119}} {{RFC8174}}  when, and only when, they appear in all
capitals, as shown here.

Readers are expected to be familiar with:

* The terms and concepts described in {{RFC9200}}, and Authorization Information Format (AIF) {{RFC9237}} to express authorization information. In particular, analogously to {{RFC9200}}, terminology for entities in the architecture such as Client (C), Resource Server (RS), and Authorization Server (AS) is defined in OAuth 2.0 {{RFC6749}}.
* The terms and concept related to the message formats and processing, specified in {{I-D.ietf-ace-key-groupcomm}}, for provisioning and renewing keying material in group communication scenarios. 
* The terms and concepts of pub-sub group communication, as described in {{I-D.ietf-core-coap-pubsub}}.
* The terms and concepts described in CBOR {{RFC8949}} and COSE {{RFC9052}}{{RFC9053}}.

# Application Profile Overview {#overview}

The architecture of the scenario is shown in {{archi}}. A Client can act both as a publisher and a subscriber, publishing to some topics, and subscribing to others. However, for the simplicity of presentation, this profile describes Publisher and Subscriber clients separately. The Broker acts as the ACE RS, and also corresponds to the Dispatcher in {{I-D.ietf-ace-key-groupcomm}}).

Both Publishers and Subscribers use the same pub-sub communication protocol and the same transport profile of ACE in their interaction with the broker. The pub-sub communication protocol considered in this document is CoAP, as described in {{I-D.ietf-core-coap-pubsub}}, but the specification can apply to other pub-sub protocols such as MQTT {{MQTT-OASIS-Standard-v5}}, or other transport.  All clients MUST use CoAP when communicating to the KDC.

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

All communications between the involved entities MUST be secured. This profile expects the establishment of a secure connection between a Client and Broker, using an ACE transport profile such as DTLS {{I-D.ietf-ace-dtls-authorize}} or OSCORE {{I-D.ietf-ace-oscore-profile}} (A and C). Once the client establishes a secure association with KDC with the help of AS, it can request to join the security groups of its pub-sub topics (A and B), and  can communicate securely with the other group members, using the keying material provided by the KDC.

(C) corresponds to the exchange between the Client and  the Broker, where the Client sends its access token to the Broker and establishes a secure connection with the Broker.
Depending on the Information received in (A), the connection set-up may involve, for example, a DTLS handshake, or other protocols. Depending on the application, the set up phase may be skipped: for example, if OSCORE is used directly. 

It must be noted that Clients maintain two different security associations. On the one hand, the Publisher and the Subscriber clients have a security association with the Broker,which, as the ACE RS, verifies that the Clients are authorized (Security Association 1). On the other hand, the Publisher has a security association with the Subscriber, to protect the publication content (Security Association 2) while sending it through the broker. The Security Association 1 is set up using AS and a transport profile of {{RFC9200}}, the Security Association 2 is set up using AS, KDC and {{I-D.ietf-ace-key-groupcomm}}.

Given that the publication content is protected, the Broker MAY accept unauthorised Subscribers. In this case, the Subscriber client MAY skip setting up Security Association 1 with the Broker and connect to it as an anonymous client to subscribe to topics of interest at the Broker.

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

This document describes how to use {{I-D.ietf-ace-key-groupcomm}} and {{RFC9200}} to perform authentication, authorization and key distribution actions as overviewed in Section 2 of {{I-D.ietf-ace-key-groupcomm}}, when the considered group is Publishers and Subsribers belonging to the same security group.

To this end, this profile describes how:

1. A Client gets the authorization to join a security group, and providing it with the group keying material to communicate with other group members.
2. A Client retrieves group keying material to publish protected publications to the Broker or read protected publications.
3. A Client retrieves authentication credentials of other group members, and provides and updates own authentication credentials.
4. A Client leaves a group.
5. A ? evicts a Client from the group. (ToDo: Who - the KDC?)
6. The KDC renews and redistributes the group keying (rekeying) material due to membership change in the group.

Appendix {{groupcomm_requirements}} lists the specifications on this application
profile of ACE, based on the requirements defined in Appendix A of {{I-D.ietf-ace-key-groupcomm}}.

# Client Interface the KDC {#kdc-interface}

The Clients uses the following KDC resources:

| KDC resource | Description | Operations  |
| ------------ | ----------- | ------------|
| /ace-group  | Required. Contains a set of group names, each corresponding to one of the specified group identifiers | FETCH (All Clients) |
| /ace-group/GROUPNAME | Required. Contains symmetric group keying material associated with GROUPNAME | GET, POST (All) |
| /ace-group/GROUPNAME/creds | Required. Contains the authentication credentials of all the Publisher members of the group with name GROUPNAME | GET, FETCH (All) |
| /ace-group/GROUPNAME/num | Required. Contains the current version number for the symmetric group keying material of the group with name GROUPNAME | GET (All) |
| /ace-group/GROUPNAME/nodes/NODENAME | Required. Contains the group keying material and the individual keying material for that group member NODENAME in GROUPNAME. | GET, DELETE (All). PUT not supported. |
| /ace-group/GROUPNAME/nodes/NODENAME/cred | Required. Authentication credential for NODENAME in the group GROUPNAME |  POST (Publisher) |
| /ace-group/GROUPNAME/kdc-cred | MUST be hosted if a group re-keying mechanism is used. Contains the authentication credential of the KDC for the group with name GROUPNAME. | GET (All) |
| /ace-group/GROUPNAME/policies | Optional. Contains the group policies of the group with
name GROUPNAME. | GET (All) |

Note that the use of these resources follows what is defined in {{I-D.ietf-ace-key-groupcomm}} applies, and only additions or modifications to that specification are defined in this document.

The Resource Type (rt=) Link Target Attribute value "core.ps.gm" is registered in {{core_rt}} (REQ10), and can be used to describe group-membership resources and its sub-resources at Broker, e.g., by using a link-format document {{RFC6690}}}.
Applications can use this common resource type to discover links to group-membership resources for joining pub-sub groups.
(ToDo: Check this discovery is feasible in core pub-sub)

# Joining a pub-sub security group (A-B) {#authorisation}

Figure {{message-flow}} provides a high level overview of the message flow for a node joining a group. This message flow is expanded in the subsequent sections.

~~~~~~~~~~~
   Client                                       Broker   AS      KDC
      | [--Resource Request (CoAP/MQTT or other)-->] |    |       |
      |                                              |    |       |
      | [<----AS Information (CoAP/MQTT or other)--] |    |       |
      |                                                   |       |
      | ---Authorisation Request (CoAP/HTTP or other)---->|       |
      |                                                   |       |
      | <---Authorisation Response (CoAP/HTTP or other) --|       |
      |                                                           |
      |----------------------Token Post (CoAP)------------------->|
      |                                                           |
      |------------------- Joining Request (CoAP) --------------->|
      |                                                           |
      |------------------ Joining Response (CoAP) --------------->|

~~~~~~~~~~~
{: #message-flow title="Authorisation Flow"}
{: artwork-align="center"}

 Since {{RFC9200}} recommends the use of CoAP and CBOR, this document describes the exchanges assuming CoAP and CBOR are used. However, using HTTP instead of CoAP is possible, using the corresponding parameters and methods. Analogously, JSON {{RFC8259}} can be used instead of CBOR, using the conversion method specified in Sections 6.1 and 6.2 of {{RFC8949}}. In case JSON is used, the Content Format or Media Type of the message has to be changed accordingly. Exact definition of these exchanges are considered out of scope for this document.

## AS Discovery (Optional) {#AS-discovery}

Complementary to what is defined in {{RFC9200}} (Section 5.1) for AS discovery, the Broker MAY send the address of the AS to the Client in the 'AS' parameter in the AS Information as a response to an Unauthorized Resource Request (Section 5.2).  An example using CBOR diagnostic notation and CoAP is given below:

~~~~~~~~~~~
    4.01 Unauthorized
    Content-Format: application/ace-groupcomm+cbor
    {"AS": "coaps://as.example.com/token"}
~~~~~~~~~~~
{: #AS-info-ex title="AS Information example"}
{: artwork-align="center"}

## Authorisation Request/Response for the KDC and the Broker {#auth-request}

After retrieving the AS address, the Client sends two Authorisation Requests to the AS for two audiences: the Broker and the KDC, respectively. AS handles authorisation requests for topics a Client is allowed to Publish or Subscribe to the Broker, corresponding to an application group.  To protect the message content, the client needs to request to be part of the security group(s) for those application groups.

Communications between the Client and the AS MUST be secured, according to what is defined by the used transport profile of ACE. Both Authorisation Requests include the following fields(Section 3.1 of {{I-D.ietf-ace-key-groupcomm}}):

* 'scope', specifying the name of the groups, that the Client requests to access. This parameter is a CBOR byte string that encodes a CBOR array, whose format MUST follow the data model AIF-PUBSUB-GROUPCOMM defined below.
* 'audience', an identifier, corresponding to either the KDC or the Broker.

Other additional parameters can be included if necessary, as defined in {{RFC9200}}.

For the Broker, the scope represents pub-sub topics i.e., the application group, and for the KDC, the scope represents the security group. If there is a one-to-one mapping between the application group and the security group, the client uses the same scope for both requests. If there is not a one-to-one mapping, the correct policies regarding both sets of scopes MUST be available to the AS.

The client MUST ask for the correct scopes in its Authorization Requests. How the client discovers the (application group, security group) association is out of scope of this document.
(ToDo: Can pub-sub discovery handle this?) 

### Format of Scope 

The 'scope' parameter used for the KDC follows the AIF format. Based on the generic AIF model

~~~~~~~~~~~
      AIF-Generic<Toid, Tperm> = [* [Toid, Tperm]]
~~~~~~~~~~~

the value of the CBOR byte string used as scope encodes the CBOR array [* [Toid, Tperm]], where each [Toid, Tperm] element corresponds to one scope entry.

This document defines the new AIF specific data model
AIF-PUBSUB-GROUPCOMM, that this profile MUST use to format and encode scope entries. In particular, the object identifier ("Toid") is a CBOR text string, specifying the topic name for the scope entry. The permission set ("Tperm") is a CBOR unsigned integer with value, specifying the role(s) that the Client wishes to take in the group. The set of numbers representing the role is converted into a single number by taking two to the power of each method number and computing the inclusive OR of the binary representations of all the power values.

~~~~~~~~~~~
  AIF-PUBSUB-GROUPCOMM = AIF-Generic<pubsub-topic pubsub-permissions>
   pubsub-topic = tstr

   pubsub-permissions = uint . bits pubsub-roles

   pubsub-roles = &(
    Pub: 0,
    Sub: 1
   )

   scope_entry = [pubsub-topic, pubsub-permissions]
~~~~~~~~~~~
{: #scope-aif title="Pub-Sub scope using the AIF format"}
{: artwork-align="center"}

## Authorisation response

The AS responds with an Authorization Response to each request, containing claims, as defined in Section 5.8.2 of {{RFC9200}} and Section 3.2 of {{I-D.ietf-ace-key-groupcomm}} with the following additions:

*  The AS MUST include the 'scope' parameter, when the value included in the Access Token differs from the one specified by the joining node in the Authorization Request.  In such a case, the second element of each scope entry MUST be present, and specifies the set
of roles that the joining node is actually authorized to take in for that scope entry, encoded as specified in {{auth-request}}.

The 'profile' claim is set to "coap_pubsub_app" as defined in {{iana-coap-profile}}.

On receiving the Authorisation Response, the Client needs to manage which token to use for which audience, i.e., the KDC or the Broker.

## Token Transfer to KDC {#token-post}
After receiving a token from the AS, the Client transfers the token to the KDC using one of the methods defined Section 3.3 {{I-D.ietf-ace-key-groupcomm}}). This includes sending a POST request to the authz-info endpoint,  and if using the DTLS transport profile of ACE, the Client MAY provide the access token to the KDC during the secure session establishment.

When using the authz-info endpoint, a Subscriber Client MAY ask in addition for the format of the public keys in the group, used for source authentication, as well as any other group parameters. In this case, the message MUST have Content-Format set to "application/ace+cbor" defined in Section 8.16 of {{RFC9200}}.
The message payload MUST be formatted as a CBOR map, which MUST include the access token and MAY include the 'sign_info' parameter, specifying the CBOR simple value "null" (0xf6) to request information about the signature algorithm, signature algorithm parameters, signature key parameters and about the exact format of authentication credentials used in the groups that the Client has been authorized to join. Alternatively, the joining node MAY retrieve this information by other means as described in {{I-D.ietf-ace-key-groupcomm}}.

The KDC verifies the token to check of the Client is authorized to access the topic with the requested role. After successful verification, the Client is authorized to receive the group keying material from the KDC and join the group. The KDC replies to the Client with a 2.01 (Created) response, using Content-Format "application/ace+cbor". The payload of the 2.01 response is a CBOR map.

For Publisher Clients, i.e., the Clients whose scope of the access token includes the "Pub" role, the KDC MUST include the parameter 'kdcchallenge' in the CBOR map, specifying a dedicated challenge N_S generated by the KDC. For the N_S value, it is RECOMMENDED to use a 8-byte long random nonce. Later when joining the group, the Publisher Client MUST send its own public key to the KDC and use the 'kdcchallenge' as part of proving possession of the corresponding private key (see {{I-D.ietf-ace-key-groupcomm}}).

If 'sign_info' is included in the Token Transfer Request, the KDC SHOULD include the 'sign_info' parameter in the Token Transfer Response.

* 'sign_alg' MUST take value from the "Value" column of one of the Recommended algorithms in the "COSE Algorithms" registry {{IANA.cose_algorithms}}.
* 'sign_parameters' is a CBOR array.  Its format and value are
the same of the COSE capabilities array for the algorithm
indicated in 'sign_alg' under the "Capabilities" column of the "COSE Algorithms" registry {{IANA.cose_algorithms}}.
* 'sign_key_parameters' is a CBOR array.  Its format and value
are the same of the COSE capabilities array for the COSE key
type of the keys used with the algorithm indicated in
'sign_alg', as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{IANA.cose_key-type}}.
* 'cred_fmt' takes value from the "Label" column of the "COSE
  Header Parameters" registry {{IANA.cose_header-parameters}}. 
  Acceptable values denote a format of authentication credential that MUST explicitly provide the public key as well as the comprehensive set of information related to the public key algorithm, including, e.g., the used elliptic curve (when applicable). Current acceptable formats of authentication credentials are CBOR Web Tokens (CWTs) and CWT Claims Sets (CCSs) {{RFC8392}}, X.509 certificates {{RFC7925}} and C509 certificates {{I-D.ietf-cose-cbor-encoded-cert}}. Future formats would be acceptable to use as long as they comply with the criteria defined above.

ToDo: Need to specify N_S generation if we are allowing DTLS profile token transfer.

## Join Request {#join-request}

In the next step, a node MUST have established a secure communication association
established before attempting to join a group.  Possible ways to provide a secure communication association are described in the DTLS transport profile {{I-D.ietf-ace-dtls-authorize}} and OSCORE transport profile {{I-D.ietf-ace-oscore-profile}} of ACE.

After establishing a secure communication, the Client sends a Join Request to the KDC as described in Section 4.3 of {{I-D.ietf-ace-key-groupcomm}}. More specifically, the Client sends a POST request to the /ace-group/GROUPNAME endpoint, with Content-Format "application/ace-groupcomm+cbor". The payload MUST contain the following information formatted as a CBOR map, which MUST be encoded as defined in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}:

* 'scope' parameter set to the specific group that the Client is attempting to join, i.e., the group name, and the roles it wishes to have in the group. This value corresponds to one scope entry, as defined in {{auth-request}}.
* 'get_creds' parameter MAY be present if the Client needs to retrieve the public keys of the other members. The Subscriber Clients MUST have access to the public keys of all the Publisher Clients. This may be achieved by requesting the public keys of all the Publisher Clients. In this case, parameter MUST be set to "null". 
* 'client\_cred' parameter MUST be included if the Client is a Publisher and contains the Client's public key formatted according to the encoding of the public keys used in the group. For a Subscriber-only Client,  the Joining Request MUST NOT contain the 'client\_cred' parameter.
* 'cnonce', includes a dedicated nonce N_C generated by the Client, if 'client\_cred' is present.  It is RECOMMENDED to use a 8-byte long random nonce.
* 'client\_cred\_verify', if 'client\_cred' is present. This parameter contains the proof-of-possession evidence. Client signs the scope, concatenated with N\_S and concatenated with N\_C using the private key corresponding to the public key in the 'client_cred' paramater. N\_S is the challenge received from the KDC in the 'kdcchallenge' parameter of the 2.01 (Created) response to the Token Transfer Request (see {{token-post}}).
* OPTIONALLY, the 'creds\_repo', if the format of the Client's authentication credential in the 'client_cred' parameter is a certificate.
* 'control_uri', using the default url-path is /ace-group/GROUPNAME/node

An example of the Join Request for a CoAP Publisher using CoAP and CBOR is specified in {{fig-req-pub-kdc}}, where SIG is a signature computed using the private key associated to the public key and the algorithm in 'client\_cred'.

~~~~~~~~~~~~
{
  "scope" : [["Broker1/Temp", 1]],
  "client_cred" : ToDo:Fix,
  "cnonce" : h'd36b581d1eef9c7c,
  "client_cred_verify" : SIG
    }
}
~~~~~~~~~~~~
{: #fig-req-pub-kdc title="Joining Request payload for a Publisher"}
{: artwork-align="center"}

An example of the payload of a Join Request for a Subscriber using CoAP and CBOR is specified in {{fig-req-sub-kdc}}.

~~~~~~~~~~~~
{
  "scope" : [["Broker1/Temp", 2]],
  "get_creds" : "null"
}
~~~~~~~~~~~~
{: #fig-req-sub-kdc title="Joining Request payload for a Subscriber"}
{: artwork-align="center"}

## Join Response
On receiving the Join Request, the KDC processes the request as defined in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}, and may return a success or error response.

If 'client_cred' field is present, the KDC verifies signature in the the 'client_cred_verify'. As PoP input, the KDC uses the value of the 'scope' parameter from the Join Request as a CBOR byte string, concatenated with N_S encoded as a CBOR byte string, concatenated with N_C encoded as a CBOR byte string.
As public key of the joining node, the KDC uses either
the one included in the authentication credential retrieved from
the 'client_cred' parameter of the Join Request or the already stored authentication credential from previous interactions with the joining node. The KDC verifies the PoP evidence, which is a signature, by using the public key of the joining node, as well as the signature algorithm used in the group and possible corresponding parameters.

In the case of success,the Client is added to the list
of current members, if not already a member. The Client is assigned a NODENAME and assigned a sub-resource /ace-group/GROUPNAME/nodes/NODENAME. NODENAME is associated to the access token and secure session of the Client. For Publishers, their public key is also associated with tuple containing NODENAME, GROUPNAME and access token. The public key of the Publisher is stored.

Then, the KDC responds with a Join Response with response code 2.01 (Created) if the Client has been added to the list of group members, and 2.04 (Changed) otherwise (e.g., if the Client is re-joining).  The Content-Format  is "application/ace-groupcomm+cbor". The payload (formatted as a CBOR map) MUST contain the following fields from the Join Response  and encode them as defined in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}:

- 'gkty', the key type for the 'key' parameter. 
- 'key', the keying material for group communication. This is a "COSE\_Key" object (defined in {{RFC9052}} {{RFC9053}}, containing:
    * 'kty' with value 4 (symmetric)
    * 'alg' with value defined by the AS (Content Encryption Algorithm)
    * 'Base IV' with value defined by the AS
    * 'k' with value the symmetric key value
    * OPTIONALLY, 'kid' with an identifier for the key value 
ToDo: Check the format of the 'key' value (REQ17)
ToDo: Additionally, documents specifying the key format MUST register it in the "ACE Groupcomm Key Types" registry including its name, type and application profile to be used with.
- 'num' containing the version number for the 'key'. The initial version number is set to 1.
- 'exp', the value of the expiration time of the 'key'
- 'creds', MUST be present, if the 'get\_creds' parameter was present. Otherwise, it MUST NOT be present. If the joining node has asked for the authentication credentials of all the group members, i.e., 'get_creds' had value the CBOR simple value "null" (0xf6) in the Join Request, then the Group Manager provides the authentication credentials of all the Publisher Clients in the group.
- 'peer\_roles' MUST be present if 'creds' is also present. Otherwise, it MUST NOT be present.
ToDo: Requested a change for this, and see how the Groupcomm draft is updated
- 'peer\_identifiers' MUST be present if 'creds' is also present. Otherwise, it MUST NOT be present.
- 'kdc\_cred', MUST be present if group re-keying is used, and encoded as a CBOR byte string, with value the original binary representation of the KDC's authentication credential.
- 'kdc\_nonce', MUST be present, if 'kdc\_cred' is present and encoded as a CBOR byte string, and including a dedicated nonce N_KDC generated by the KDC.
- 'kdc_cred_verify' MUST be present, if 'kdc\_cred' is present and encoded as a CBOR byte string. The PoP evidence is computed over the nonce N_KDC, which is specified in the 'kdc_nonce' parameter and taken as PoP input. KDC MUST compute the signature
by using the signature algorithm used in the group, as
well as its own private key associated with the authentication
credential specified in the 'kdc_cred' parameter.

An example of the Join Response for a CoAP Publisher using CoAP and CBOR (corresponding to Join Request in {{fig-req-pub-kdc}}) is shown in.  {{fig-resp-pub-kdc}}.

~~~~~~~~~~~~
{
  "gkty" : "COSE_Key",
  "key" : {1: 4, 2: h'1234', 3: 12, 5: h'1f389d14d17dc7',
          -1: h'02e2cc3a9b92855220f255fff1c615bc'},
  "num" : 1
}
~~~~~~~~~~~~
{: #fig-resp-pub-kdc title="Join Response payload for a Publisher"}
{: artwork-align="center"}


An example of the payload of a Join Response for a Subscriber using CoAP and CBOR (corresponding to the request in {{fig-req-sub-kdc}}) is shown in {{fig-resp-sub-kdc}}.

~~~~~~~~~~~~
{
  "gkty" : "COSE_Key"
  "key" : {1: 4, 2: h'1234', 3: 12, 5: h'1f389d14d17dc7',
          -1: h'02e2cc3a9b92855220f255fff1c615bc'},
  "creds" : [{ToDo:Fix Example}]
  "peer_roles": [1],
  "peer_identifiers": ["1"]
}
~~~~~~~~~~~~
{: #fig-resp-sub-kdc title="Joining Response payload for a Subscriber"}
{: artwork-align="center"}


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


(D) corresponds to the publication of a topic on the Broker, using a CoAP PUT.
The publication (the resource representation) is protected with COSE  ({{RFC9052}}{{RFC9053}}) by the Publisher.
The (E) message is the subscription of the Subscriber, and uses a CoAP
GET with the Observe option set to 0 (zero) {{I-D.ietf-core-coap-pubsub}}. The subscription MAY be unprotected.
The (F) message is the response from the Broker, where the publication is protected with COSE by the Publisher.

~~~~~~~~~~~
  Publisher                Broker               Subscriber
      | --- PUT /topic ----> |                       |
      |  protected with COSE |                       |
      |                      | <--- GET /topic ----- |
      |                      |      Observe:0        |
      |                      | ---- response ------> |
      |                      |  protected with COSE  |
~~~~~~~~~~~
{: #flow title="Example of protected communication for CoAP"}
{: artwork-align="center"}


## Using COSE Objects To Protect The Resource Representation {#oscon}

The Publisher uses the symmetric COSE Key received from the KDC to protect the payload of the PUBLISH operation (Section 4.3 of {{I-D.ietf-core-coap-pubsub}}). Specifically, the COSE Key is used to create a COSE\_Encrypt0 object with algorithm specified by the KDC. The Publisher uses the private key corresponding to the public key sent to the KDC in exchange B  to countersign the COSE Object as specified in {{RFC9052}} {{RFC9053}}. The payload is replaced by the COSE object before the publication is sent to the Broker.
ToDo: Check RFC9338 for counter signatures

The Subscriber uses the 'kid' in the 'countersignature' field in the COSE object to retrieve the right public key to verify the countersignature. It then uses the symmetric key received from KDC to verify and decrypt the publication received in the payload from the Broker (in the case of CoAP the publication is received by the CoAP Notification).

The COSE object is constructed in the following way:

* The protected Headers (as described in {{RFC9052}} {{RFC9053}}) MUST contain the kid parameter if it was provided in the Joining Response, with value the kid of the symmetric COSE Key received and MUST contain the content encryption algorithm.
* The unprotected Headers MUST contain the Partial IV, with value a sequence number that is incremented for every message sent, and the counter signature that includes:
  - the algorithm (same value as in the asymmetric COSE Key received in (B)) in the protected header;
  - the kid (same value as the kid of the asymmetric COSE Key received in (B)) in the unprotected header;
  - the signature computed as specified in {{RFC9052}} {{RFC9053}}.
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

The encryption and decryption operations are described in  {{RFC9052}} {{RFC9053}}.

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
KDC MUST trigger a group rekeying as described in Section 6 
of {{I-D.ietf-ace-key-groupcomm}} due to a change in the group membership or the current group keying material approaching its expiration time. KDC MAY trigger regularly scheduled update of the group keying material.

Default rekeying scheme is Point-to-point (Section 6.1 of {{I-D.ietf-ace-key-groupcomm}}), where KDC individually targets ach node to rekey, using the pairwise secure communication association with that node.

If the group rekeying is performed due to one or multiple
Publisher Clients that have joined the group, then a rekeying
message MUST also include the authentication credentials that those Clients use in the group, together with the roles and node identifier that the corresponding Client has in the group.  This information is specified by means of the parameters 'creds',
'peer_roles' and 'peer_identifiers', like done in the Join Response message.

ToDo: Any additional rekeying mechanisms?
The pub-sub model would make sense. In this case, the KDC acts
as publisher (KDC authentication credential??) and publishes each rekeying message to a specific "rekeying topic", which is associated with the group and is hosted at a broker server.  Following their group joining, the group members subscribe to the rekeying topic at the broker, thus receiving the group rekeying messages as they are published by the KDC.

# Considerations for Supporting MQTT PubSub Profile {#mqtt-pubsub}

The steps MQTT clients go through would be similar to the CoAP clients, and the payload of the MQTT PUBLISH messages will be protected using COSE.

In MQTT, topics are organised as a tree, and in the {{I-D.ietf-ace-mqtt-tls-profile}}, 'scope' captures permissions for not a single topic but a topic filter. Therefore, topic names (i.e., group names) may include wildcards spanning several levels of the topic tree. Hence, it is important to distinguish application groups and security groups defined in {{I-D.ietf-core-groupcomm-bis}}.

Also differently for MQTT, the Client sends a token request to AS for the requested topics (application groups) using AIF-MQTT data model for representing the requested scopes is described in Section 3 of the {{I-D.ietf-ace-mqtt-tls-profile}}. In the authorisation response, the 'profile' claim is set to "mqtt_pubsub_app" as defined in {{iana-mqtt-profile}}.
Both Publisher and Subscriber Clients authorise to the Broker with their respective tokens (described in {{I-D.ietf-ace-mqtt-tls-profile}}).

A Publisher Client sends PUBLISH messages for a given topic and protects the payload with the corresponding key for the associated security group. RS validates the PUBLISH message by verifying its topic in the stored token. 

A Subscriber Client may send SUBSCRIBE messages with one or multiple topic filters. A topic filter may correspond to multiple topics. RS validates the SUBSCRIBE message by checking the stored token for the Client.

Authorisation Server (AS) Discovery is defined in Section 2.2.6.1 of {{I-D.ietf-ace-mqtt-tls-profile}} for MQTT v5 clients (and not supported for MQTT v3 clients).

# Security Considerations

All the security considerations in {{I-D.ietf-ace-key-groupcomm}} apply.

In the profile described above, the Publisher and Subscriber use asymmetric crypto, which would make the message exchange quite heavy for small constrained devices. Moreover, all Subscribers must be able to access the public keys of all the Publishers to a specific topic to be able to verify the publications. 

 Even though Access Tokens have expiration times, an Access Token may need to be revoked before its expiration time (see {{I-D.draft-ietf-ace-revoked-token-notification-02}} for a list of possible circumstances). Clients can be excluded from future publications through re-keying for a certain topic. This could be set up to happen on a regular basis, for certain applications. How this could be done is out of scope for this work. 
 The method described in {{I-D.draft-ietf-ace-revoked-token-notification-02}} MAY be used to allow an Authorization Server to notify the KDC about revoked Access Tokens.

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

References: {{RFC9052}} {{RFC9053}}, \[\[This document\]\]

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

## CoRE Resource Type {#core_rt}

IANA is asked to register the following entry in the "Resource Type (rt=) Link Target Attribute Values" registry within the "Constrained Restful Environments (CoRE) Parameters" registry group.

   *  Value: "core.ps.gm"

   *  Description: Group-membership resource for Pub-Sub communication.

   *  Reference: [RFC-XXXX]

Clients can use this resource type to discover a group membership resource at a Broker. 

--- back

# Requirements on Application Profiles {#groupcomm_requirements}

This section lists the specifications on this profile based on the requirements defined in Appendix A of {{I-D.ietf-ace-key-groupcomm}}.

* REQ1: Specify the format and encoding of 'scope'. : TODO.

* REQ2: If the AIF format of 'scope' is used, register its specific instance of "Toid" and "Tperm" as Media Type parameters and a corresponding Content-Format, as per the guidelines in {{RFC9237}}.

* REQ3: If used, specify the acceptable values for 'sign_alg': TODO

* REQ4: If used, specify the acceptable values for 'sign_parameters': TODO

* REQ5: If used, specify the acceptable values for 'sign_key_parameters' : TODO

* REQ6: Specify the acceptable formats for authentication
credentials and, if used, the acceptable values for 'cred_fmt': TODO

* REQ7: If the value of the GROUPNAME URI path and the group name in the access token scope (gname) are not required to
coincide, specify the mechanism to map the GROUPNAME value in the
URI to the group name: TODO

* REQ8: Define whether the KDC has an authentication credential and if this has to be provided through the 'kdc_cred' parameter : TODO

* REQ9: Specify if any part of the KDC interface as defined in {{I-D.ietf-ace-key-groupcomm}} is not supported by the KDC: TODO

* REQ10: Register a Resource Type for the root url-path, which is
used to discover the correct url to access at the KDC : the Resource Type
(rt=) Link Target Attribute value "core.ps.gm" is registered in
Section {{core_rt}}.

* REQ11: Define what specific actions (e.g., CoAP methods) are
allowed on each resource provided by the KDC interface, depending
on whether the Client is a current group member; the roles that a
Client is authorized to take as per the obtained access token;  and the roles that the Client has as current group member.

* REQ12: Categorize possible newly defined operations for Clients
into primary operations expected to be minimally supported and
secondary operations, and provide accompanying considerations: None added.

* REQ13: Specify the encoding of group identifier: ToDo.

* REQ14: Specify the approaches used to compute and verify the PoP evidence to include in 'client_cred_verify', and which of those approaches is used in which case: ToDo

* REQ15: Specify how the nonce N_S is generated, if the token is not provided to the KDC through the Token Transfer Request to the
authz-info endpoint (e.g., if it is used directly to validate TLS
instead): TODO

* REQ16: Define the initial value of the 'num' parameter: 1

* REQ17: Specify the format of the 'key' parameter: ToDo

* REQ18: Specify the acceptable values of the 'gkty' parameter: ToDo

* REQ19: Specify and register the application profile identifier: coap_pubsub_app

*  REQ20: If used, specify the format and content of 'group_policies' and its entries.  Specify the policies default values: N/A

* REQ21: Specify the approaches used to compute and verify the PoP evidence to include in 'kdc_cred_verify', and which of those approaches is used in which case

* REQ22: Specify the communication protocol the members of the group must use.

*  REQ23: Specify the security protocol the group members must use to protect their communication. This must provide encryption, integrity and replay protection.

* REQ24: Specify how the communication is secured between Client and KDC.  Optionally, specify transport profile of ACE [RFC9200] to use between Client and KDC.

* REQ25: Specify the format of the identifiers of group members.

*  REQ26: Specify policies at the KDC to handle ids that are not
included in 'get_creds'.

* REQ27: Specify the format of newly-generated individual keying
material for group members, or of the information to derive it,
and corresponding CBOR label.

* REQ28: Specify which CBOR tag is used for identifying the
semantics of binary scopes, or register a new CBOR tag if a
suitable one does not exist already.

* REQ29: Categorize newly defined parameters according to the same criteria of Section 8 of {{I-D.ietf-ace-key-groupcomm}}.

* REQ30: Define whether Clients must, should or may support the
conditional parameters defined in Section 8 of {{I-D.ietf-ace-key-groupcomm}}, and under which
circumstances.

* OPT1: Optionally, if the textual format of 'scope' is used,
specify CBOR values to use for abbreviating the role identifiers
in the group: N/A

* OPT2: Optionally, specify the additional parameters used in the
  exchange of Token Transfer Request and Response : N/A

* OPT3: Optionally, specify the negotiation of parameter values for signature algorithm and signature keys, if 'sign_info' is not used: N/A

* OPT4: Optionally, specify possible or required payload formats for specific error cases.: not defined

*  OPT5: Optionally, specify additional identifiers of error types, as values of the 'error' field in an error response from the KDC: not defined.

*  OPT6: Optionally, specify the encoding of 'creds_repo' if the
default is not used.: N/A

*  OPT7: Optionally, specify the functionalities implemented at the 'control_uri' resource hosted at the Client, including message exchange encoding and other details

* OPT8: Optionally, specify the behavior of the handler in case of failure to retrieve an authentication credential for the specific node:

* OPT9: Optionally, define a default group rekeying scheme, to refer to in case the 'rekeying_scheme' parameter is not included in the Join Response:

*  OPT10: Optionally, specify the functionalities implemented at the 'control_group_uri' resource hosted at the Client, including
message exchange encoding and other details

* OPT11: Optionally, specify policies that instruct Clients to
retain messages and for how long, if they are unsuccessfully
decrypted

* OPT12: Optionally, specify for the KDC to perform group rekeying (together or instead of renewing individual keying material) when receiving a Key Renewal Request

* OPT13: Optionally, specify how the identifier of a group member's authentication credential is included in requests sent to other group members

*  OPT14: Optionally, specify additional information to include in rekeying messages for the "Point-to-Point" group rekeying scheme:

*  OPT15: Optionally, specify if Clients must or should support any of the parameters defined as optional in {{I-D.ietf-ace-key-groupcomm}}:

# Acknowledgments
{: numbered="no"}

The author wishes to thank Ari Keränen, John Mattsson, Ludwig Seitz, Göran Selander, Jim Schaad and Marco Tiloca for the useful discussion and reviews that helped shape this document.

--- fluff

<!-- Local Words: -->
<!-- Local Variables: -->
<!-- coding: utf-8 -->
<!-- ispell-local-dictionary: "american" -->
<!-- End: -->
