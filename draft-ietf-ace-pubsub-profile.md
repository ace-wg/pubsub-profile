---
v: 3

title: Publish-Subscribe Profile for Authentication and Authorization for Constrained Environments (ACE)
abbrev: ACE Pub-sub Profile
docname: draft-ietf-ace-pubsub-profile-latest

coding: utf-8

ipr: trust200902
area: Security
workgroup: ACE Working Group
keyword: Internet-Draft
cat: std
submissiontype: IETF

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
-
    ins: M. Tiloca
    name: Marco Tiloca
    org: RISE AB
    email: marco.tiloca@ri.se

normative:
  IANA.cose_algorithms:
  IANA.cose_key-type:
  IANA.cose_header-parameters:
  RFC2119:
  RFC5705:
  RFC6347:
  RFC6690:
  RFC6749:
  RFC7252:
  RFC7925:
  RFC8174:
  RFC8392:
  RFC8446:
  RFC8447:
  RFC8613:
  RFC8949:
  RFC9052:
  RFC9053:
  RFC9147:
  RFC9200:
  RFC9237:
  RFC9277:
  RFC9338:
  I-D.ietf-cose-cbor-encoded-cert:
  I-D.ietf-core-coap-pubsub:
  I-D.ietf-ace-key-groupcomm:

informative:
  RFC8259:
  RFC9202:
  RFC9203:
  I-D.ietf-ace-edhoc-oscore-profile:
  I-D.ietf-ace-mqtt-tls-profile:
  I-D.draft-ietf-ace-revoked-token-notification:
  I-D.ietf-ace-key-groupcomm-oscore:
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

This document defines an application profile of the Authentication and Authorization for Constrained Environments (ACE) framework, to enable secure group communication in the Publish-Subscribe (pub/sub) architecture for the Constrained Application Protocol (CoAP) \[draft-ietf-core-coap-pubsub\], where Publishers and Subscribers communicate through a Broker. This profile relies on protocol-specific transport profiles of ACE to achieve communication security, server authentication, and proof-of-possession for a key owned by the Client and bound to an OAuth 2.0 Access Token. This document specifies the provisioning and enforcement of authorization information for Clients to act as Publishers and/or Subscribers, as well as the provisioning of keying material and security parameters that Clients use for protecting their communications end-to-end through the Broker.

Note to RFC Editor: Please replace "\[draft-ietf-core-coap-pubsub\]" with the RFC number of that document and delete this paragraph.
--- middle

# Introduction

In a publish-subscribe (pub/sub) scenario, devices with limited reachability communicate via a Broker, which enables store-and-forward messaging between these devices. This effectively enables a form of group communication, where all the Publishers and Subscribers participating in the same pub/sub topic are considered members of the same group associated with that topic.

With a focus on the pub/sub architecture defined in {{I-D.ietf-core-coap-pubsub}} for the Constrained Application Protocol (CoAP) {{RFC7252}}, this document defines an application profile of the Authentication and Authorization for Constrained Environments (ACE) framework {{RFC9200}}, which enables pub/sub communication where a group of Publishers and Subscribers securely communicate through a Broker using CoAP.

Building on the message formats and processing defined in {{I-D.ietf-ace-key-groupcomm}}, this document specifies the provisioning and enforcement of authorization information for Clients to act as Publishers and/or Subscribers at the Broker, as well as the provisioning of keying material and security parameters that Clients use for protecting end-to-end their communications via the Broker.

In order to protect the pub/sub operations at the Broker as well as the provisioning of keying material and security parameters, this profile relies on protocol-specific transport profiles of ACE (e.g., {{RFC9202}}, {{RFC9203}}, or {{I-D.ietf-ace-edhoc-oscore-profile}}) to achieve communication security, server authentication, and proof-of-possession for a key owned by the Client and bound to an OAuth 2.0 Access Token.

Furthermore, the content of published messages that are circulated by the Broker is protected end-to-end between the corresponding Publisher and the intended Subscribers. To this end, this profile relies on COSE {{RFC9052}}{{RFC9053}} and on keying material provided to the Publishers and Subscribers participating in the same pub/sub topic. In particular, source authentication of published content is achieved by means of the corresponding Publisher signing such content with its own private key.

While this profile focuses on the pub/sub architecture for CoAP, this document also describes how it can be applicable to MQTT {{MQTT-OASIS-Standard-v5}}. Similar adaptations can also extend to further transport protocols and pub/sub architectures.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 {{RFC2119}} {{RFC8174}}  when, and only when, they appear in all capitals, as shown here.

Readers are expected to be familiar with:

* The terms and concepts described in {{RFC9200}}, and Authorization Information Format (AIF) {{RFC9237}} to express authorization information. In particular, analogously to {{RFC9200}}, terminology for entities in the architecture such as Client (C), Resource Server (RS), and Authorization Server (AS) is defined in OAuth 2.0 {{RFC6749}}.
* The terms and concept related to the message formats and processing, specified in {{I-D.ietf-ace-key-groupcomm}}, for provisioning and renewing keying material in group communication scenarios.
* The terms and concepts of pub/sub group communication, as described in {{I-D.ietf-core-coap-pubsub}}.
* The terms and concepts described in CBOR {{RFC8949}} and COSE {{RFC9052}}{{RFC9053}}.

A principal interested to participate in group communication as well as already participating as a group member is interchangeably denoted as "Client", "pub/sub client" or "node".

* Group: a set of nodes that share common keying material and security parameters to protect their communications with one another. That is, the term refers to a "security group". This is not to be confused with an "application group", which has relevance at the application level and whose members are in this case the Clients acting as Publishers and/or Subscribers for a topic.

# Application Profile Overview {#overview}

This document describes how to use {{RFC9200}} and {{I-D.ietf-ace-key-groupcomm}} to perform authentication, authorization, and key distribution operations as overviewed in {{Section 2 of I-D.ietf-ace-key-groupcomm}}, where the considered group is the security group composed of the pub/sub Clients that exchange end-to-end protected content through the Broker.

Pub/sub Clients communicate within their application groups, each of which is mapped to a topic. Depending on the application, a topic may consist of one or more sub-topics, which may have their own sub-topics and so on, thus forming a hierarchy. A security group SHOULD be associated with a single application group. However, the same application group MAY be associated with multiple security groups. Further details and considerations on the mapping between the two types of groups are out of the scope of this document.

This profile considers the architecture shown in {{archi}}. A Client can act as a Publisher, or a Subscriber, or both, e.g., by publishing to some topics and subscribing to others. However, for the simplicity of presentation, this profile describes Publisher and Subscriber Clients separately.

Both Publishers and Subscribers act as ACE Clients. The Broker acts as an ACE RS, and corresponds to the Dispatcher in {{I-D.ietf-ace-key-groupcomm}}. The Key Distribution Center (KDC) also acts as an ACE RS, and builds on what is defined for the KDC in {{I-D.ietf-ace-key-groupcomm}}. From a high-level point of view, the Clients interact with the KDC in order to join security groups and obtain the group keying material to protect end-to-end and verify the content published in the corresponding topics.

~~~~~~~~~~~~
             +---------------+   +--------------+
             | Authorization |   |     Key      |
             |    Server     |   | Distribution |
             |      (AS)     |   |    Center    |
             |               |   |    (KDC)     |
             +---------------+   +--------------+
                      ^                  ^
                      |                  |
     +---------(A)----+                  |
     |                                   |
     |   +--------------------(B)--------+
     v   v
+------------+             +------------+
|            | <-- (O) --> |            |
|  pub/sub   |             |   Broker   |
|  Client    | <-- (C)---> |            |
|            |             |            |
+------------+             +------------+
~~~~~~~~~~~~~
{: #archi title="Architecture for Pub/sub with Authorization Server and Key Distribution Center"}
{: artwork-align="center"}

Both Publishers and Subscribers MUST use the same pub/sub communication protocol for their interaction with the Broker. When using the profile defined in this document, this protocol MUST be CoAP, which is used as described in {{I-D.ietf-core-coap-pubsub}}. What is specified in this document can apply to other pub/sub protocols such as MQTT {{MQTT-OASIS-Standard-v5}}, or to further transport protocols.

All Publishers and Subscribers MUST use CoAP when communicating with the KDC.

Furthermore, both Publishers and Subscribers MUST use the same transport profile of ACE (e.g., {{RFC9202}} for DTLS; or {{RFC9203}} or {{I-D.ietf-ace-edhoc-oscore-profile}} for OSCORE) in their interaction with the Broker. In order to reduce the number of libraries that Clients have to support, it is RECOMMENDED that the same transport profile of ACE is used also for the interaction between the Clients and the KDC.

All communications between the involved entities MUST be secured.

The Client and the Broker MUST have a secure association, which they establish with the help of the AS and using a transport profile of ACE. This is shown by the interactions A and C in {{archi}}. During this process, the Client obtains an Access Token from the AS and uploads it to the Broker, thus providing an evidence of the topics that it is authorized to participate in, and with which permissions.

The Client and the KDC MUST have a secure association, which they also establish with the help of the AS and using a transport profile of ACE. This is shown by the interactions A and B in {{archi}}. During this process, the Client obtains an Access Token from the AS and uploads it to the KDC, thus providing an evidence of the security groups that it can join, as corresponding to the topics of interest at the Broker. Based on the permissions specified in the Access Token, the Client can request the KDC to join a security group, after which the Client obtains from the KDC the keying material to use for communicating with the other group members. This builds on the process for joining security groups with ACE defined in {{I-D.ietf-ace-key-groupcomm}} and further specified in this document.

In addition, this profile allows an anonymous Client to perform some of the discovery operations defined in {{Section 2.3 of I-D.ietf-core-coap-pubsub}} through the Broker, as shown by the interaction O in {{archi}}. That is, an anonymous Client can discover:

* the Broker itself, by relying on the resource type "core.ps" (see {{Section 2.3.1 of I-D.ietf-core-coap-pubsub}}); and

* topics of interest at the Broker (i.e., the corresponding topic resources hosted at the Broker), by relying on the resource type "core.ps.conf" (see {{Section 2.3.2 of I-D.ietf-core-coap-pubsub}}).

However, an anonymous Client is not allowed to access topic resources at the Broker and obtain from those any additional information or metadata about the corresponding topic (e.g., the topic status, the URI of the topic-data resource where to publish or subscribe for that topic, or the URI to the KDC).

As highlighted in {{associations}}, each Client maintains two different security associations pertaining to the pub/sub group communication. On the one hand, the Client has a pairwise security association with the Broker, which, as the ACE RS, verifies that the Client is authorized to perform data operations (i.e., publish, subscribe, read, delete) on a certain set of topics (Security Association 1). As discussed above, this security association is set up with the help of the AS and using a transport profile of ACE, when the Client obtains the Access Token to upload to the Broker.

On the other hand, separately for each topic, all the Publisher and Subscribers for that topic have a common, group security association, through which the published content sent through the Broker is protected end-to-end (Security Association 2). As discussed above, this security association is set up and maintained as the different Clients request the KDC to join the security group, upon which they obtain from the KDC the corresponding group keying material to use for protecting end-to-end and verifying the content of their pub/sub group communication.

~~~~~~~~~~~~
+------------+             +------------+              +------------+
|            |             |            |              |            |
| Publisher  |             |   Broker   |              | Subscriber |
|            |             |            |              |            |
|            |             |            |              |            |
+------------+             +------------+              +------------+
      :   :                     :   :                      :   :
      :   '----- Security ------'   '------ Security ------'   :
      :        Association 1              Association 1        :
      :                                                        :
      '---------------------- Security ------------------------'
                            Association 2
~~~~~~~~~~~~~
{: #associations title="Security Associations between Publisher, Broker, and Subscriber."}
{: artwork-align="center"}

In summary, this profile specifies the following functionalities.

1. A Client obtains the authorization to participate in a pub/sub topic at the Broker with certain permissions. This pertains operations defined in {{I-D.ietf-core-coap-pubsub}} for taking part in pub/sub communication with CoAP.

2. A Client obtains the authorization to join a security group with certain permissions. This allows the Client to obtain from the KDC the group keying material for communicating with other group members, i.e., to protect end-to-end and verify the content published at the Broker on topics associated with the security group.

3. A Client obtains from the KDC the authentication credentials of other group members, and provides the KDC with its own (updated) authentication credential.

4. A Client leaves the group or is removed from the group by the KDC.

5. The KDC renews and redistributes the group keying material (rekeying), e.g., due to a membership change in the group.

{{groupcomm_requirements}} lists the specifications on this application profile of ACE, based on the requirements defined in {{Section A of I-D.ietf-ace-key-groupcomm}}.


# Getting Authorisation to Join a Pub/sub security group (A) {#authorisation}

{{message-flow}} provides a high level overview of the message flow for a Client getting authorisation to join a group. Square brackets denote optional steps. The message flow is expanded in the subsequent sections.

~~~~~~~~~~~
Client                                          Broker   AS   KDC
   |                                                 |    |     |
   |[<-------- Discovery of Topic Resource -------->]|    |     |
   |                                                 |    |     |
   |[--------------- Resource Request ------------->]|    |     |
   |[<--------------- AS Information ---------------]|    |     |
   |                                                 |    |     |
   |                                                 |    |     |
   |----- Authorisation Request (Audience: Broker) ------>|     |
   |<---- Authorisation Response (Audience: Broker) ------|     |
   |                                                 |    |     |
   |                                                 |    |     |
   |------ Upload of authorisation information ----->|    |     |
   |<----- Establishment of secure association ----->|    |     |
   |                                                 |    |     |
   |                                                 |    |     |
   |[<-- Discovery of KDC and name of sec. group -->]|    |     |
   |                                                 |    |     |
   |                                                 |    |     |
   |------- Authorisation Request (Audience: KDC) ------->|     |
   |<------ Authorisation Response (Audience: KDC) -------|     |
   |                                                 |    |     |
   |                                                 |    |     |
   |--------- Upload of authorisation information ------------->|
   |<-------- Establishment of secure association ------------->|
   |                                                 |    |     |
   |                                                 |    |     |
   |----- Request to join the security group for the topic ---->|
   |<-------- Keying material for the security group -----------|
   |                                                 |    |     |
   |                                                 |    |     |
   |---------------- Resource Request -------------->|    |     |
   |                                                 |    |     |
~~~~~~~~~~~
{: #message-flow title="Authorisation Flow"}
{: artwork-align="center"}

Since {{RFC9200}} recommends the use of CoAP and CBOR, this document describes the exchanges assuming CoAP and CBOR are used.

However, using HTTP instead of CoAP is possible, by leveraging the corresponding parameters and methods. Analogously, JSON {{RFC8259}} can be used instead of CBOR, using the conversion method specified in {{Sections 6.1 and 6.2 of RFC8949}}. In case JSON is used, the Content-Format of the message has to be changed accordingly. Exact definitions of these exchanges are out of scope for this document.

## Topic Discovery at the Broker (Optional) {#topic-discovery}

The discovery of a topic at the Broker can be performed by discovering the corresponding topic resources hosted at the Broker. For example, the Client can send a lookup request to /.well-known/core at the Broker, specifying as lookup criterion the resource type "core.ps.conf" (see {{Section 2.3.2 of I-D.ietf-core-coap-pubsub}}).

Although the links to the topic resources are also specified in the representation of the collection resource at the Broker (see {{Section 2.4 of I-D.ietf-core-coap-pubsub}}), the Client is not supposed to access such a resource, as intended for administrative operations that are out of the scope of this document.

## AS Discovery at the Broker (Optional) {#AS-discovery}

Complementary to what is defined in {{Section 5.1 of RFC9200}} for AS discovery, the Broker MAY send the address of the AS to the Client in the 'AS' parameter of the AS Request Creation Hints, as a response to an Unauthorized Resource Request (see {{Section 5.2 of RFC9200}}). An example using CBOR diagnostic notation and CoAP is given below:

~~~~~~~~~~~
    4.01 Unauthorized
    Content-Format: application/ace+cbor
    Payload:
    {
     "AS": "coaps://as.example.com/token"
    }
~~~~~~~~~~~
{: #AS-info-ex title="AS Request Creation Hints Example"}
{: artwork-align="left"}

## KDC Discovery at the Broker (Optional) {#kdc-discovery}

Once a Client has obtained an Access Token from the AS and accordingly established a secure association with the Broker, the Client has the permission to access the topic resources at the Broker that pertain to the topics on which the Client is authorized to operate.

In particular the Client is authorized to retrieve the representation of a topic resource, from which the Client can retrieve information related to the topic in question, as specified in {{Section 2.5 of I-D.ietf-core-coap-pubsub}}.

This profile extends the set of CoAP Pub/sub Parameters that is possible to specify within the representation of a topic resource, as originally defined in {{Section 3 of I-D.ietf-core-coap-pubsub}}. In particular, this profile defines the following two parameters that the Broker can specify in a response from a topic resource (see {{Section 2.5 of I-D.ietf-core-coap-pubsub}}). Note that, when these parameters are transported in their respective fields of the message payload, the Content-Format application/core-pubsub+cbor MUST be used.

* 'kdc_uri', with value the URI of the group membership resource at KDC, where Clients can send a request to join the security group associated with the topic in question. The URI is encoded as a CBOR text string. Clients will have to obtain an Access Token from the AS to upload to the KDC, before starting the process to join the security group at the KDC.

* 'sec_gp', specifying the name of the security group associated with the topic in question, as a stable and invariant identifier. The name of the security group is encoded as a CBOR text string.

Furthermore, the Resource Type (rt=) Link Target Attribute value "core.ps.gm" is registered in {{core_rt}} (REQ10), and can be used to describe group-membership resources and its sub-resources at KDC, e.g., by using a link-format document {{RFC6690}}. As an alternative to the discovery approach defined above and provided by the Broker, applications can use this common resource type to discover links to group-membership resources at the KDC for joining security groups associated with pub/sub topics.

## Authorisation Request/Response for the KDC and the Broker {#auth-request}

A Client sends two Authorisation Requests to the AS, targeting two different audiences, i.e, the Broker and the KDC.

As to the former, the AS handles Authorisation Requests for topics that the Client is allowed to perform topic data operations at the Broker, as corresponding to an application group.

As to the latter, the AS handles Authorization Requests for security groups that the Client is allowed to join, in order to obtain the group keying material for protecting end-to-end and verifying the content of exchanged messages on the associated pub/sub topics.

This section builds on Section 3 of {{I-D.ietf-ace-key-groupcomm}} and defines only additions or modifications to that specification.

Both Authorisation Requests include the following fields (see {{Section 3.1 of I-D.ietf-ace-key-groupcomm}}):

* 'scope': Optional. If present, it specifies the following information, depending on the specifically targeted audience.

   If the audience is the Broker, the scope specifies the name of the topics that the Client wishes to access, together with the corresponding permissions. If the audience is the KDC, the scope specifies the name of the security groups that the Client wishes to join, together with the corresponding permissions.

   This parameter is encoded as a CBOR byte string, whose value is the binary encoding of a CBOR array. The format MUST follow the data model AIF-PUBSUB-GROUPCOMM defined in {{scope}}.

* 'audience': Required identifier corresponding to either the Broker or the KDC.

Other additional parameters can be included if necessary, as defined in {{RFC9200}}.

When using this profile, it is expected that a one-to-one mapping is enforced between the application group and the security group (see {{overview}}). If this is not the case, the correct access policies corresponding both sets of scopes have to be available to the AS.

### Format of Scope {#scope}

Building on {{Section 3.1 of I-D.ietf-ace-key-groupcomm}}, this section defines the exact format and encoding of scope used in this profile.

To this end, this profile uses the Authorization Information Format (AIF) {{RFC9237}} (REQ1). With reference to the generic AIF model

~~~~~~~~~~~
      AIF-Generic<Toid, Tperm> = [* [Toid, Tperm]]
~~~~~~~~~~~

the value of the CBOR byte string used as scope encodes the CBOR array \[* \[Toid, Tperm\]\], where each \[Toid, Tperm\] element corresponds to one scope entry.

Furthermore, this document defines the new AIF specific data model AIF-PUBSUB-GROUPCOMM that this profile MUST use to format and encode scope entries.

In particular, the following holds for each scope entry.

* The object identifier ("Toid") is specialized as a CBOR item specifying the name of the application groups (topics) or the name of the corresponding security groups pertaining to the scope entry.

* The permission set ("Tperm") is specialized as a CBOR unsigned integer with value R, specifying the permissions that the Client wishes or has been authorized to have in the groups indicated by "Toid".

More specifically, the following applies when, as defined in this document, a scope entry specifies a Client's set of permissions for accessing: i) topic-related resources at the Broker; or ii) resources at the KDC associated with the corresponding security groups.

* The object identifier ("Toid") is a CBOR text string, specifying the name of one application group (topic) or of the corresponding security group to which the scope entry pertains.

* The permission set ("Tperm") is a CBOR unsigned integer, whose value R specifies the operations that the Client wishes to or has been authorized to perform on the resources at the Broker associated with the application group (topic) indicated by "Toid", or on the resources at the KDC associated with the security group indicated by "Toid" (REQ1). The value R is computed as follows.

   * Each operation (i.e., permission detail) in the permission set is converted into the corresponding numeric identifier X taken from the following set.

      - Admin (0): This operation is reserved for scope entries that express permissions for Administrators of pub/sub groups.

      - AppGroup (1): This operation is signaled as wished/authorized when "Toid" specifies the name of an application group (topic)+, while it is signaled as not wished/authorized when Toid specifies the name of a security group.

      - Publish (2): This operation concerns the publication of data to the topic in question, performed by means of a PUT request sent by a Publisher Client to the corresponding topic-data resource at the Broker.

      - Read (3): This operation concerns both: i) the subscription at the topic-data resource for the topic in question at the Broker, performed by means of a GET request with the CoAP Observe Option set to 0 and sent by a Subscriber Client; and ii) the simple reading of the latest data published to the topic in question, performed by means of a simple GET request sent to the same topic-data resource.

      - Delete (4): This operation concerns the deletion of the topic-data resource for the topic in question at the Broker, performed by means of a DELETE request sent to that resource.

   * The set of N numeric identifiers is converted into the single value R, by taking two to the power of each numeric identifier X_1, X_2, ..., X_N, and then computing the inclusive OR of the binary representations of all the power values.

   Since this application profile considers user-related operations, the "Admin" operation is signaled as not wished/authorized. That is, the scope entries MUST have the least significant bit of "Tperm" set to 0.

If the "Toid" of a scope entry in an access token specifies the name of an application group (i.e., the "AppGroup" operation is signaled as authorized), the Client has also the permission to retrieve the configuration of the application group (topic) whose name is indicated by "Toid", by sending a GET or FETCH request to the corresponding topic resource at the Broker.

The specific interactions between the Client and the Broker are defined in {{I-D.ietf-core-coap-pubsub}}.

~~~~~~~~~~~
  AIF-PUBSUB-GROUPCOMM = AIF-Generic<pubsub-topic, pubsub-perm>
   pubsub-topic = tstr ; pub/sub topic name
                       ; (the associated security group)

   pubsub-perm = uint .bits pubsub-perm-details

   pubsub-perm-details = &(
    Admin: 0,
    AppGroup: 1
    Publish: 2,
    Read: 3,
    Delete: 4
   )

   scope_entry = [pubsub-topic, pubsub-perm]
~~~~~~~~~~~
{: #scope-aif title="Pub/sub scope using the AIF format"}
{: artwork-align="center"}

## Authorisation response {#as-response}

The AS responds with an Authorization Response to each request, containing claims, as defined in Section 5.8.2 of {{RFC9200}} and Section 3.2 of {{I-D.ietf-ace-key-groupcomm}} with the following additions:

* The AS MUST include the 'expires_in' parameter.  Other means for the AS to specify the lifetime of Access Tokens are out of the scope of this document.
* The AS MUST include the 'scope' parameter, when the value included in the Access Token differs from the one specified by the Client in the Authorization Request.  In such a case, the second element of each scope entry MUST be present, and specifies the set of interactions that the Client is authorized for that scope entry, encoded as specified in {{auth-request}}.

Furthermore, the AS MAY use the extended format of scope defined in Section 7 of {{I-D.ietf-ace-key-groupcomm}} for the 'scope' claim of the Access Token.  In such a case, the AS MUST use the CBOR tag with tag number TAG_NUMBER, associated with the CoAP Content-Format CF_ID for the media type application/aif+cbor registered in {{content_format}} of this document (REQ28).

Note to RFC Editor: In the previous paragraph, please replace "TAG_NUMBER" with the CBOR tag number computed as TN(ct) in Section 4.3 of {{RFC9277}}, where ct is the ID assigned to the CoAP Content-Format registered in {{content_format}} of this document.  Then, please replace "CF_ID" with the ID assigned to that CoAP Content-Format.  Finally, please delete this paragraph.

This indicates that the binary encoded scope follows the scope semantics defined for this application profile in {{scope}} of this document.

## Token Transfer to KDC {#token-post}

The Client transfers its access token to the KDC using one of the methods defined Section 3.3 {{I-D.ietf-ace-key-groupcomm}}. This typically includes sending a POST request to the authz-info endpoint. However, if using the DTLS transport profile of ACE {{RFC9202}} and the Client uses a symmetric proof-of-possession key in the DTLS handshake, the Client MAY provide the access token to the KDC in the "psk_identity" field of the DTLS ClientKeyExchange message when using DTLS 1.2 {{RFC6347}}, or in the "identity" field of a PskIdentity within the PreSharedKeyExtension of the ClientHello message when using DTLS 1.3 {{RFC9147}}. In addition to that, the following applies.

In the token transfer response to the Publishers, i.e., the Clients whose scope of the access token includes the "Publish" permission for at least one scope entry, the KDC MUST include the parameter 'kdcchallenge' in the CBOR map. 'kdcchallange' is a challenge N_S generated by the KDC, and is RECOMMENDED to be a 8-byte long random nonce. Later when joining the group, the Publisher can use the 'kdcchallenge' as part of proving possession of its private key (see {{I-D.ietf-ace-key-groupcomm}}). If a Publisher provides the access token to the KDC through an authz-info endpoint, the Client MUST support the parameter 'kdcchallenge'.

If 'sign_info' is included in the Token Transfer Request, the KDC SHOULD include the 'sign_info' parameter in the Token Transfer Response. Note that the joining node may have obtained such information by alternative means e.g., the 'sign_info'  may have been pre-configured (OPT3).

The following applies for each element 'sign_info_entry'.

* 'sign_alg' MUST take its value from the "Value" column of one of the recommended algorithms in the "COSE Algorithms" registry {{IANA.cose_algorithms}} (REQ3).
* 'sign_parameters' is a CBOR array.  Its format and value are the same of the COSE capabilities array for the algorithm indicated in 'sign_alg' under the "Capabilities" column of the "COSE Algorithms" registry {{IANA.cose_algorithms}} (REQ4).
* 'sign_key_parameters' is a CBOR array.  Its format and value are the same of the COSE capabilities array for the COSE key type of the keys used with the algorithm indicated in 'sign_alg', as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{IANA.cose_key-type}} (REQ5).
* 'cred_fmt' takes value from the "Label" column of the "COSE Header Parameters" registry {{IANA.cose_header-parameters}} (REQ6). Acceptable values denote a format of authentication credential that MUST explicitly provide the public key as well as the comprehensive set of information related to the public key algorithm, including, e.g., the used elliptic curve (when applicable). Acceptable formats of authentication credentials include CBOR Web Tokens (CWTs) and CWT Claims Sets (CCSs) {{RFC8392}}, X.509 certificates {{RFC7925}} and C509 certificates {{I-D.ietf-cose-cbor-encoded-cert}}. Future formats would be acceptable to use as long as they comply with the criteria defined above.

# Client Group Communication Interface at the KDC {#kdc-interface}

The pub/sub Clients uses the following KDC resources to enable group communication:

| KDC resource | Description | Operations  |
| ------------ | ----------- | ------------|
| /ace-group  | Required. Contains a set of group names, each corresponding to one of the specified group identifiers. | FETCH (All Clients) |
| /ace-group/GROUPNAME | Required. Contains symmetric group keying material associated with GROUPNAME. | GET, POST (All Clients) |
| /ace-group/GROUPNAME/creds | Required. Contains the authentication credentials of all the Publishers of the group with name GROUPNAME. | GET, FETCH (All Clients) |
| /ace-group/GROUPNAME/num | Required. Contains the current version number for the symmetric group keying material of the group with name GROUPNAME. | GET (All Clients) |
| /ace-group/GROUPNAME/nodes/NODENAME | Required. Contains the group keying material for that group member NODENAME in GROUPNAME. | GET, DELETE (All Clients). PUT (Publishers). |
| /ace-group/GROUPNAME/nodes/NODENAME/cred | Required. Authentication credential for NODENAME in the group GROUPNAME. |  POST (Publishers) |
| /ace-group/GROUPNAME/kdc-cred | MUST be hosted if a group re-keying mechanism is used. Contains the authentication credential of the KDC for the group with name GROUPNAME. | GET (All Clients) |
| /ace-group/GROUPNAME/policies | Optional. Contains the group policies of the group with name GROUPNAME. | GET (All Clients) |

Note that the use of these resources follows what is defined in {{I-D.ietf-ace-key-groupcomm}}, and only additions or modifications to that specification are defined in this document.

## Joining a Security Group {#join}

This section describes the interactions between a node and the KDC to join a security group. Source authentication of a message sent within the group is ensured by means of a digital signature embedded in the message. Subscribers must be able to retrieve Publishers' authentication credentials from a trusted repository, to verify source authenticity of received messages. Hence, on joining a group, a Publisher is expected to provide its own authentication credential to the KDC.

On a successful join, the Clients receive the symmetric COSE Key from the KDC to protect the payload of a published topic data.

The message exchange between the joining node and the KDC follows what is defined in Section 4.3.1.1 of {{I-D.ietf-ace-key-groupcomm}} and only additions or modifications to that specification are defined in this document.

~~~~~~~~~~~
   Client                               KDC
      |----- Join Request (CoAP) ------>|
      |                                 |
      |<-----Join Response (CoAP) ------|

~~~~~~~~~~~
{: #join-flow title="Join Flow"}
{: artwork-align="center"}

### Join Request {#join-request}

After establishing a secure communication, the Client sends a Join Request to the KDC as described in Section 4.3 of {{I-D.ietf-ace-key-groupcomm}}. More specifically, the Client sends a POST request to the /ace-group/GROUPNAME endpoint, with Content-Format "application/ace-groupcomm+cbor". The payload MUST contain the following information formatted as a CBOR map, which MUST be encoded as defined in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}:

* 'scope': Required. MUST be set to the specific group that the Client is attempting to join, i.e., the group name, and the permissions it wishes to have in the group. This value corresponds to one scope entry, as defined in {{scope}}.
* 'get_creds': Optional, present if the Subcriber wants to retrieve the public keys of all the Publishers upon joining. Otherwise, this parameter MUST NOT be present. If the parameter is present, the parameter MUST encode the CBOR simple value "null" (0xf6). Note that no 'role_filter' is necessary as KDC returns the authentication credentials of Publishers by default.
* 'client\_cred': The use of this parameter is detailed in {{client_cred}}.
* 'cnonce': Required. It is a dedicated nonce N\_C generated by the Client. It is RECOMMENDED to use an 8-byte long random nonce. Join Requests MUST include a new 'cnonce' at each join attempt.
* 'client\_cred\_verify': Optional, it MUST be present if 'client\_cred' is present. The use of this parameter is detailed in {{pop}}.

As a Publisher has its own authentication credential to use in a group, it MUST support the client_cred', 'cnonce', and 'client_cred_verify' parameters.

#### Client Credentials-'client_cred' {#client_cred}

One of the following cases can occur when a new node attempts to join a group.

* The joining node isn't a Publisher, i.e., it is not going to send messages to the group.  In this case, the joining node is not required to provide its own authentication credential to the KDC. In case the joining node still provides an authentication credential in the 'client_cred' parameter of the Join Request (see {{join-request}}), the KDC silently ignores that parameter, as well as the related parameter 'client_cred_verify'.
* The joining node is a Publisher, and
    -  the KDC already acquired the authentication credential of the joining node either during a past group joining process, or during establishing a secure communication association, and the joining node and the KDC use a symmetric proof-of-possession key. If the authentication credential and the proof-of-possession key are compatible with the signature or ECDH algorithm, and possible associated parameters, then the key can be used for the authentication credential in the group. In this case, the joining node MAY choose not to provide again its own authentication credential to the KDC, in order to limit the size of the Join Request.
    - the KDC hasn't acquired an authentication credential. Then, the joining node MUST provide a compatible authentication credential in the 'client_cred' parameter of the Join Request (see {{join-request}}).

Finally, the joining node MUST provide its own authentication credential again if it has provided the KDC with multiple authentication credentials during past joining processes intended for different groups.  If the joining node provides its own authentication credential, the KDC performs consistency checks as per {{join-request}} and, in case of success, considers it as the authentication credential associated with the joining node in the group.

#### Proof-of-Possession {#pop}

The 'client\_cred\_verify' parameter contains the proof-of-possession evidence, and is computed as defined below (REQ14).

The Publisher signs the scope, concatenated with N\_S and concatenated with N\_C using the private key corresponding to the public key in the 'client_cred' parameter.

The N\_S may be either:

* The challenge received from the KDC in the 'kdcchallenge' parameter of the 2.01 (Created) response to the Token Transfer Request (see {{token-post}}).
* If the Publisher used a symmetric proof-of-possession key in the DTLS handshake {{RFC9202}} with the KDC,  then it is an exporter value computed as defined in Section 7.5 of {{RFC8446}}.  Specifically, N_S is exported from the DTLS session between the joining node and the KDC, using an empty 'context_value', 32 bytes as 'key_length', and the exporter label "EXPORTER-ACE-Sign-Challenge-coap-group-pubsub-app" defined in {{tls_exporter}} of this document.
* If the Join Request is a retry in response to an error response from the KDC, which included a new 'kdcchallenge' parameter, N_S MUST be this new challenge parameter.

### Join Response {#join-response}

On receiving the Join Request, the KDC processes the request as defined in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}, and returns a success or error response.

If the 'client\_cred' field is present, the KDC verifies the signature in the 'client\_cred\_verify'. As PoP input, the KDC uses the value of the 'scope' parameter from the Join Request as a CBOR byte string, concatenated with N_S encoded as a CBOR byte string, concatenated with N_C encoded as a CBOR byte string. As public key of the joining node, the KDC uses either the one included in the authentication credential retrieved from the 'client\_cred' parameter of the Join Request or the already stored authentication credential from previous interactions with the joining node. The KDC verifies the PoP evidence, which is a signature, by using the public key of the joining node, as well as the signature algorithm used in the group and possible corresponding parameters.

For a Publisher Client, the KDC assigns an available Sender ID that has not been used <!-- since the latest time when the current Group Identifier (Gid) value was assigned to --> in the group. The KDC MUST NOT assign a Sender ID to the joining node if the node isn't a Publisher. The Sender ID MUST be unique within the group. Similar to {{RFC8613}}, the Sender ID can be short: the maximum length of Sender ID in bytes equals the length of the AEAD nonce minus 6; for AES-CCM-16-64-128 the maximum length of Sender ID is 7 bytes.

In the case of any join request error, the KDC and the Client attempting to join follow the procedure defined in {{join-error}}.

In the case of success, the Client is added to the list of current members, if not already a member. The Client is assigned a NODENAME and a sub-resource /ace-group/GROUPNAME/nodes/NODENAME. NODENAME is associated to the access token and secure session of the Client. Publishers' client credentials are also associated with the tuple containing NODENAME, GROUPNAME, <!-- Group Identifier (Gid), --> a newly assigned sender ID and the access token. Note that, as long as the secure association between the client and the KDC persists, then the same client re-joining the group is recognized by the KDC by virtue of their secure association. As a consequence, the re-joining Publisher keeps the same NODENAME and related subresource /ace-group/GROUPNAME/nodes/NODENAME, while receiving a new Sender ID according to the same criteria above.

The KDC responds with a Join Response with response code 2.01 (Created) if the Client has been added to the list of group members, and 2.04 (Changed) otherwise (e.g., if the Client is re-joining).  The Content-Format  is "application/ace-groupcomm+cbor". The payload (formatted as a CBOR map) MUST contain the following fields from the Join Response and encode them as defined in Section 4.3.1 of {{I-D.ietf-ace-key-groupcomm}}:

- 'gkty': the key type "Group_PubSub_Keying_Material" for the 'key' parameter defined in {{iana-ace-groupcomm-key}} of this document.
- 'key': The keying material for group communication includes the following parameters:
    * 'group_SenderId', specifying the Client's Sender ID encoded as a CBOR byte string. This field MUST be included if the Client is a Publisher, and MUST NOT be included otherwise.
    * 'cred_fmt', specifying the format of authentication credentials used in the group. This parameter takes its value from the "Label" column of the "COSE Header Parameters" registry {{IANA.cose_header-parameters}}. At the time of writing this specification, acceptable formats of authentication credentials are CBOR Web Tokens (CWTs) and CWT Claims Sets (CCSs) {{RFC8392}}, X.509 certificates {{RFC7925}} and C509 certificates {{I-D.ietf-cose-cbor-encoded-cert}}. Further formats may be available in the future, and would be acceptable to use as long as they comply with the criteria defined above (REQ6).
    * 'sign_alg', specifying the Signature Algorithm used to sign messages in the group. This parameter takes values from the "Value" column of the "COSE Algorithms" registry {{IANA.cose_algorithms}}.
    * 'sign_params', specifying the parameters of the Signature Algorithm. This parameter is a CBOR array, which includes the following two elements:
      - 'sign_alg_capab'is a CBOR array, with the same format and value of the COSE capabilities array for the Signature Algorithm indicated in 'sign_alg', as specified for that algorithm in the "Capabilities" column of the "COSE Algorithms" registry {{IANA.cose_algorithms}}.
      - 'sign_key_type_capab' is a CBOR array, with the same format and value of the COSE capabilities array for the COSE key type of the keys used with the Signature Algorithm indicated in 'sign_alg', as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{IANA.cose_key-type}}.
    * 'group_key', specifying a COSE\_Key object as defined in {{RFC9052}} and conveying the group key to use in the security group. The COSE\_Key object MUST contain the following parameters:
       - 'kty', with value 4 (Symmetric).
       - 'alg', with value the identifier of the AEAD algorithm used in the security group.
       - 'Base IV', with value the Base Initialization Vector (Base IV) to use in the security group with this group key.
       - 'k', with value the symmetric encryption key to use as group key (REQ17).
       - 'kid', with value the identifier of the COSE\_Key object, hence of the group key. This value is used as Group Identifier (Gid) of the security group, as long as the present key is used as group key in the security group.
- 'num', specifying the version number of the keying material specified in the 'key' field (with initial value set to 0 on creation of the group).
- 'exp', which MUST be present.
- 'ace-groupcomm-profile', which MUST be present and has value "coap_group_pubsub_app" (PROFILE_TBD), which is defined in {{iana-profile}} of this document.
- 'creds', which MUST be present if the 'get\_creds' parameter was present. Otherwise, it MUST NOT be present. The KDC provides the authentication credentials of all the Publishers in the security group.
- 'peer\_roles', which MAY be omitted even if 'creds' is present, since each authentication credential conveyed in the 'creds' parameter: i) is associated with a Client authorized to be Publisher in the group; and ii) plays no role if such Client was also authorized to be Subscriber in the group. If 'creds' is not present, 'peer\_roles' MUST NOT be present.
- 'peer\_identifiers', which MUST be present if 'creds' is also present. Otherwise, it MUST NOT be present. The identifiers are the Publisher Sender IDs whose authentication credential is specified in the 'creds' parameter (REQ 25).
- 'kdc\_cred', which MUST be present if group re-keying is used, and encoded as a CBOR byte string, with value the original binary representation of the KDC's authentication credential (REQ8).
- 'kdc\_nonce', which MUST be present, if 'kdc\_cred' is present and encoded as a CBOR byte string, and including a dedicated nonce N\_KDC generated by the KDC. For N\_KDC, it is RECOMMENDED to use a 8-byte long random nonce.
- 'kdc_cred_verify', which MUST be present, if 'kdc\_cred' is present and encoded as a CBOR byte string. The PoP evidence is computed over the nonce N\_KDC, which is specified in the 'kdc\_nonce' parameter and taken as PoP input. KDC MUST compute the signature by using the signature algorithm used in the group, as well as its own private key associated with the authentication credential specified in the 'kdc\_cred' parameter (REQ21).
- 'group_rekeying', which MAY be omitted if the KDC uses the "Point-to-Point" group rekeying scheme registered in {{Section 11.12 of I-D.ietf-ace-key-groupcomm}} as the default rekeying scheme in the group (OPT9). In any other case, the 'group_rekeying' parameter MUST be included.

A Publisher Client MUST support 'group\_SenderId' parameter (REQ29).

If the application requires backward security, the KDC MUST generate updated security parameters and group keying material, and provide it to the current group members, upon the new node's joining (see {{rekeying}}).  In such a case, the joining node is not able to access secure communication in the pub/sub group prior its joining.

 Upon receiving the Join Response, the joining node retrieves the KDC's authentication credential from the 'kdc_cred' parameter. The joining node MUST verify the proof-of-possession (PoP) evidence, which is a signature, specified in the 'kdc_cred_verify' parameter of the Join Response (REQ21).

### Join Error Handling {#join-error}

The KDC MUST reply with a 4.00 (Bad Request) error response to the Join Request in the following cases:

* The 'client_cred' parameter is present in the Join Request and its value is not an eligible authentication credential (e.g., it is not of the format accepted in the group) (OPT8).
* The 'client_cred' parameter is present but does not include the 'cnonce' and 'client_cred_verify' parameters.
* The 'client_cred' parameter is not present while the joining node is not going to join the group exclusively as a Subscriber, and any of the following conditions holds:

    -  The KDC does not store an eligible authentication credential (e.g., of the format accepted in the group) for the joining node.
    -  The KDC stores multiple eligible authentication credentials (e.g., of the format accepted in the group) for the joining node.
* The 'scope' parameter is not present in the Join Request, or it is present and specifies any set of permissions not included in the list defined in {{scope}}.

 A 4.00 (Bad Request) error response from the KDC to the joining node MAY have content format application/ace-groupcomm+cbor and contain a CBOR map as payload. The CBOR map MAY include the 'kdcchallenge' parameter.  If present, this parameter is a CBOR byte string, which encodes a newly generated 'kdcchallenge' value that the Client can use when preparing a new Join Request.  In such a case the KDC MUST store the newly generated value as the 'kdcchallenge' value associated with the joining node, possibly replacing the currently stored value.

 On receiving the Join Response, if 'kdc_cred' is present but the Client cannot verify the PoP evidence, the Client MUST stop processing the Join Response and MAY send a new Join Request to the KDC.

 The Group Manager MUST return a 5.03 (Service Unavailable) response to a Publisher's join request in case there are currently no Sender IDs available.

## Other Group Operations through the KDC

### Querying for Group Information {#query}

* '/ace-group': All Clients send FETCH requests to retrieve a set of group names associated with their group identifiers. ToDo:Each element of the CBOR array 'gid' is a CBOR byte string (REQ13), which encodes the Gid of the group for which the group name and the URI to the group-membership resource are provided.
* '/ace-group/GROUPNAME':  All Clients can use GET requests to retrieve the symmetric group keying material of the group with the name GROUPNAME. The value of the GROUPNAME URI path and the group name in the access token scope ('gname') MUST coincide.
* '/ace-group/GROUPNAME/creds': KDC acts as a repository of authentication credentials for Publishers. The Subscribers of the group use GET/FETCH requests to retrieve the authentication credentials of all or a subset of the group members of the group with name GROUPNAME. The KDC silently ignores the Sender IDs included in the 'get_creds' parameter of the request that are not associated with any current group member (REQ26).

   The response from the KDC MAY omit the parameter 'peer\_roles', since each authentication credential conveyed in the 'creds' parameter: i) is associated with a Client authorized to be Publisher in the group; and ii) plays no role if such Client was also authorized to be Subscriber in the group. If 'creds' is not present, 'peer\_roles' MUST NOT be present.

* '/ace-group/GROUPNAME/num': All group member Clients use GET requests to retrieve the current version number for the symmetric group keying material of the group with name GROUPNAME.
* '/ace-group/GROUPNAME/kdc-cred': All group member Clients use GET requests to retrieve the current authentication credential of the KDC.

### Updating Authentication Credentials

A Publisher can contact the KDC to upload a new authentication credential to use in the group, and replace the currently stored one. To this end, it sends a CoAP POST request to the /ace-group/GROUPNAME/nodes/NODENAME/cred. The KDC replaces the stored authentication credential of this Client (identified by NODENAME) with the one specified in the request at the KDC, for the group identified by GROUPNAME.

### Removal from a Group

A Client can actively request to leave the group.  In this case, the Client sends a CoAP DELETE request to the endpoint /ace-group/GROUPNAME/nodes/NODENAME at the KDC, where GROUPNAME is the group name and NODENAME is its node name. KDC can also remove a group member due to any of the reasons described in Section 5 of {{I-D.ietf-ace-key-groupcomm}}.

### Rekeying a Group {#rekeying}

The KDC MUST trigger a group rekeying as described in Section 6 of {{I-D.ietf-ace-key-groupcomm}} due to a change in the group membership or the current group keying material approaching its expiration time. KDC MAY trigger regularly scheduled update of the group keying material.

Upon generating the new group keying material and before starting its distribution, the KDC MUST increment the version number of the group keying material. The KDC MUST preserve the current value of the Sender ID of each member in that group.
<!-- The KDC MUST also generate a new Group Identifier (Gid) for the group as introduced in {{I-D.ietf-ace-key-groupcomm}}. -->

Default rekeying scheme is Point-to-point (Section 6.1 of {{I-D.ietf-ace-key-groupcomm}}), where KDC individually targets each node to rekey, using the pairwise secure communication association with that node.

If the group rekeying is performed due to one or multiple Publishers joining the group, then a rekeying message includes sender IDs, and authentication credentials that the Publishers use in the group. This information is specified by means of the parameters 'creds' and 'peer_identifiers', like done in the Join Response message (i.e., 'peer_roles' MAY be omitted).

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

(D) corresponds to the publication of a topic on the Broker, using a CoAP PUT. The publication (the resource representation) is protected with COSE ({{RFC9052}}{{RFC9053}}) by the Publisher. The (E) message is the subscription of the Subscriber, and uses a CoAP GET with the Observe option set to 0 (zero) {{I-D.ietf-core-coap-pubsub}}. The (F) message is the response from the Broker, where the publication is protected with COSE by the Publisher.
(ToDo: Add Delete to the flow?)

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

The Publisher uses the symmetric COSE Key received from the KDC to protect the payload of the Publish operation (Section 4.3 of {{I-D.ietf-core-coap-pubsub}}). Specifically, the Publisher creates a COSE\_Encrypt0 object {{RFC9052}}{{RFC9053}} by means of the COSE Key currently used as group key. The encryption algorithm and Base IV to use are specified by the 'alg' and 'Base IV' parameters of the COSE Key, together with its key identifier in the 'kid' parameter.

Also, the Publisher uses its private key corresponding to the public key sent to the KDC for countersigning the COSE\_Encrypt0 object as specified in {{RFC9338}}. The countersignature is specified in the 'Countersignature version 2' parameter, within the 'unprotected' field of the COSE\_Encrypt0 object.

Finally, the Publisher sends the COSE\_Encrypt0 object conveying the countersignature to the Broker, as payload of the PUT request sent to the topic-data of the topic targeted by the Publish operation.

Upon receiving a response from the topic-data resource at the Broker, the Subscriber uses the 'kid' parameter from the 'Countersignature version 2' parameter within the 'unprotected' field of the COSE\_Encrypt0 object, in order to retrieve the Publisher's public key from the Broker or from its local storage. Then, the Subscriber uses that public key to verify the countersignature.

In case of successful verification, the Subscriber uses the 'kid' parameter from the 'unprotected' field of the COSE\_Encrypt0 object, in order to retrieve the COSE Key used as current group key from its local storage. Then, the Subscriber uses that group key to verify and decrypt the COSE\_Encrypt0 object. In case of successful verification, the Subscriber delivers the received topic data to the application.

The COSE\_Encrypt0 object is constructed as follows.

The 'protected' field MUST include:

* The 'alg' parameter, with value the identifier of the AEAD algorithm specified in the 'alg' parameter of the COSE Key used as current group key.

The 'unprotected' field MUST include:

* The 'kid' parameter, with the same value specified in the 'kid' parameter of the COSE Key used as current group key. This value represent the current Group ID (Gid) of the security group associated with the application group (topic).

* The 'Partial IV' parameter, with value set to the current Sender Sequence Number of the Publisher. All leading bytes of value zero SHALL be removed when encoding the Partial IV, except in the case of Partial IV value 0, which is encoded to the byte string 0x00.

   The Publisher MUST increment its Sender Sequence Number value by 1, after having completed an encryption operation by means of the current group key.

* The 'Countersignature version 2' parameter, specifying the countersignature of the COSE\_Encrypt0 object. In particular:

   - The 'protected' field specifies the serialized parameters from the 'protected' field of the COSE\_Encrypt0 object, i.e., the 'alg' parameter with value the identifier of the Signature Algorithm used in the security group.

   - The 'unprotected' field includes the 'kid' parameter, with value the Publisher's Sender ID that the Publisher obtained from the KDC when joining the security group, as value of the 'group_SenderId' parameter of the Join Response (see {{join-response}}).

   - The 'signature' field, with value the countersignature.

   The countersignature is computed as defined in {{RFC9338}}, by using the private key of the Publisher as signing key, and by means of the Signature Algorithm used in the group. The fields of the Countersign_structure are populated as follows:

   - 'context' takes "CounterSignature".
   - 'body_protected' takes the serialized parameters from the 'protected' field of the COSE\_Encrypt0 object, i.e., the 'alg' parameter.
   - 'sign_protected' takes the serialized parameters from the 'protected' field of the 'Countersignature version 2' parameter, i.e., the 'alg' parameter.
   - 'external_aad is not supplied.
   - 'payload' is the ciphertext of the COSE\_Encrypt0 object.

* The 'ciphertext' parameter, with value the ciphertext computed over the topic data to publish.

   The ciphertext is computed as defined in {{RFC9052}}{{RFC9053}}, by using the current group key as encryption key, the AEAD Nonce computed as defined in {{ssec-aead-nonce}}, the topic data to publish as plaintext, and the Enc_structure populated as follows:

   - 'context' takes "Encrypt".
   - 'protected' takes the serialization of the protected parameter 'alg' from the 'protected' field of the COSE\_Encrypt0 object.
   - 'external_aad is not supplied.

## AEAD Nonce # {#ssec-aead-nonce}

The encryption and decryption operations. The AEAD nonce is generated following the construction in Section 5.2 of {{RFC8613}} using the sender ID, Partial IV, and Base IV from the symmetric COSE Key received.

# Applicability to MQTT PubSub Profile {#mqtt-pubsub}

The steps MQTT clients go through would be similar to the CoAP clients, and the payload of the MQTT PUBLISH messages will be protected using COSE. The MQTT clients needs to use CoAP to communicate to the KDC, to join security groups, and be part of the pair-wise rekeying initiated by the KDC.

Authorisation Server (AS) Discovery is defined in Section 2.2.6.1 of {{I-D.ietf-ace-mqtt-tls-profile}} for MQTT v5 clients (and not supported for MQTT v3 clients). $SYS/ has been widely adopted as a prefix to topics that contain server-specific information or control APIs, and may be used for topic and KDC discovery.

When the Client sends an authorisation request to the AS using the AIF-PUBSUB-GROUPCOMM data model, in the authorisation response, the 'profile' claim is set to "mqtt_pubsub_app" as defined in {{iana-profile}}.

Both Publishers and Subscribers MUST authorise to the Broker with their respective tokens (described in {{I-D.ietf-ace-mqtt-tls-profile}}). A Publisher Client sends PUBLISH messages for a given topic and protects the payload with the corresponding key for the associated security group. A Subscriber Client may send SUBSCRIBE messages with one or multiple topic filters. A topic filter may correspond to multiple topics. The Broker forwards all PUBLISH messages to all authorised Subscribers, including the retained messages.

# Security Considerations

All the security considerations in {{I-D.ietf-ace-key-groupcomm}} apply.

In the profile described above, the Publishers and Subscribers use asymmetric crypto, which would make the message exchange quite heavy for small constrained devices. Moreover, all Subscribers must be able to access the public keys of all the Publishers to a specific topic to verify the publications.

 Even though access tokens have expiration times, an access token may need to be revoked before its expiration time (see {{I-D.draft-ietf-ace-revoked-token-notification}} for a list of possible circumstances). Clients can be excluded from future publications through re-keying for a certain topic. This could be set up to happen on a regular basis, for certain applications. How this could be done is out of scope for this work. The method described in {{I-D.draft-ietf-ace-revoked-token-notification}} MAY be used to allow an Authorization Server to notify the KDC about revoked access tokens.

The Broker is only trusted with verifying that the Publisher is authorized to publish, but is not trusted with the publications itself, which it cannot read nor modify.

With respect to the reusage of nonces for Proof-of-Possession input, the same considerations apply as in the
{{I-D.ietf-ace-key-groupcomm-oscore}}.

TODO: expand on security and privacy considerations

# IANA Considerations

Note to RFC Editor: Please replace "{{&SELF}}" with the RFC number of this document and delete this paragraph.

This document has the following actions for IANA.

## ACE Groupcomm Key Registry {#iana-ace-groupcomm-key}

IANA is asked to register the following entry in the "ACE Groupcomm Key Types" registry defined in Section 11.7 of {{I-D.ietf-ace-key-groupcomm}}.

* Name: Group_PubSub_Keying_Material

* Key Type Value: GROUPCOMM_KEY_TBD

* Profile: coap_group_pubsub_app, defined in {{iana-profile}} of this document.

* Description: Encoded as described in the {{join-response}} of this document.

* References: {{RFC9052}}, {{RFC9053}}, {{&SELF}}

## ACE Groupcomm Profile Registry {#iana-profile}

IANA is asked to register the following entries in the "ACE Groupcomm Profiles" registry defined in Section 11.8 of {{I-D.ietf-ace-key-groupcomm}}.

* Name: coap_group_pubsub_app

* Description: Profile for delegating client authentication and authorization for publishers and subscribers in a CoAP pub/sub setting scenario in a constrained environment.

* CBOR Value: TBD

* Reference: {{&SELF}}

&nbsp;

* Name: mqtt_pubsub_app

* Description: Profile for delegating client authentication and authorization for publishers and subscribers in a MQTT pub/sub setting scenario in a constrained environment.

* CBOR Value: TBD

* Reference: {{&SELF}}

## CoRE Resource Type {#core_rt}

IANA is asked to register the following entry in the "Resource Type (rt=) Link Target Attribute Values" registry within the "Constrained Restful Environments (CoRE) Parameters" registry group.

*  Value: "core.ps.gm"

*  Description: Group-membership resource for pub/sub communication.

*  Reference: {{&SELF}}

Clients can use this resource type to discover a group membership resource at the KDC.

## AIF Media-Type Sub-Parameters {#aif}

For the media-types application/aif+cbor and application/aif+json defined in Section 5.1 of {{RFC9237}}, IANA is requested to register the following entries for the two media-type parameters Toid and Tperm, in the respective sub-registry defined in Section 5.2 of {{RFC9237}} within the "MIME Media Type Sub-Parameter" registry group.

* Parameter: Toid

* Name: pubsub-topic

* Description/Specification: Pub/sub topic name, corresponding to the security group

* Reference: {{&SELF}}

&nbsp;

* Parameter: Tperm

* Name: pubsub-perm

* Description/Specification: Permissions corresponding to the topic data interactions in the pub/sub group

* Reference: {{&SELF}}

## CoAP Content-Formats {#content_format}

IANA is asked to register the following entries to the "CoAP Content-Formats" registry within the "Constrained RESTful Environments (CoRE) Parameters" registry group.

* Content Type: application/aif+cbor;Toid="pubsub-topic",Tperm="pubsub-perm"

* Content Coding: -

* ID: 294 (suggested)

* Reference: {{&SELF}}

&nbsp;

* Content Type: application/aif+json;Toid="pubsub-topic",Tperm="pubsub-perm"

* Content Coding: -

* ID: 295 (suggested)

* Reference: {{&SELF}}

## TLS Exporter Labels {#tls_exporter}

IANA is asked to register the following entry to the "TLS Exporter Labels" registry defined in Section 6 of {{RFC5705}} and updated in Section 12 of {{RFC8447}}.

* Value: EXPORTER-ACE-Sign-Challenge-coap-group-pubsub-app

* DTLS-OK: Y

* Recommended: N

* Reference: {{Section 4.1.1.2 of &SELF}}

--- back

# Requirements on Application Profiles {#groupcomm_requirements}

This section lists the specifications on this profile based on the requirements defined in Appendix A of {{I-D.ietf-ace-key-groupcomm}}.

* REQ1: Specify the format and encoding of 'scope'. : See {{scope}}.

* REQ2: If the AIF format of 'scope' is used, register its specific instance of "Toid" and "Tperm" as Media Type parameters and a corresponding Content-Format, as per the guidelines in {{RFC9237}}.:See {{aif}}.

* REQ3: If used, specify the acceptable values for 'sign_alg':  values from the "Value" column of the "COSE Algorithms" registry {{IANA.cose_algorithms}}.

* REQ4: If used, specify the acceptable values for 'sign_parameters': format and values from the COSE algorithm
capabilities as specified in the "COSE Algorithms" registry {{IANA.cose_algorithms}}.

* REQ5: If used, specify the acceptable values for 'sign_key_parameters' : Its format and value are the same of the COSE capabilities array for the COSE key type of the keys used with the algorithm indicated in 'sign_alg', as specified for that key type in the "Capabilities" column of the "COSE Key Types" registry {{IANA.cose_key-type}}.

* REQ6: Specify the acceptable formats for authentication credentials and, if used, the acceptable values for 'cred_fmt':  Acceptable formats explicitly provide the public key as well as the comprehensive set of information related to the public key algorithm. Takes value from the "Label" column of the "COSE Header Parameters" registry {{IANA.cose_header-parameters}}.

* REQ7: If the value of the GROUPNAME URI path and the group name in the access token scope (gname) are not required to coincide, specify the mechanism to map the GROUPNAME value in the URI to the group name: not applicable; a perfect matching is required.

* REQ8: Define whether the KDC has an authentication credential and if this has to be provided through the 'kdc_cred' parameter : Optional, see {{join-response}} of this document.

* REQ9: Specify if any part of the KDC interface as defined in {{I-D.ietf-ace-key-groupcomm}} is not supported by the KDC: Some left optional, see {{kdc-interface}} of this document.

* REQ10: Register a Resource Type for the root url-path, which is used to discover the correct url to access at the KDC : the Resource Type (rt=) Link Target Attribute value "core.ps.gm" is registered in Section {{core_rt}}.
ToDo: This possibly will not stay as the final method, KDC discovery done differently through topic discovery?

* REQ11: Define what specific actions (e.g., CoAP methods) are allowed on each resource provided by the KDC interface, depending on whether the Client is a current group member; the roles that a Client is authorized to take as per the obtained access token;  and the roles that the Client has as current group member.: See {{kdc-interface}} of this document.

* REQ12: Categorize possible newly defined operations for Clients into primary operations expected to be minimally supported and secondary operations, and provide accompanying considerations: None added.

* REQ13: Specify the encoding of group identifier: CBOR byte string, see {{query}}.

* REQ14: Specify the approaches used to compute and verify the PoP evidence to include in 'client_cred_verify', and which of those approaches is used in which case: See {{pop}} in this document.

* REQ15: Specify how the nonce N_S is generated, if the token is not provided to the KDC through the Token Transfer Request to the authz-info endpoint (e.g., if it is used directly to validate TLS
instead): See {{pop}} in this document.

* REQ16: Define the initial value of the 'num' parameter: The initial value MUST be set to 0.

* REQ17: Specify the format of the 'key' parameter: See {{join-response}}.

* REQ18: Specify the acceptable values of the 'gkty' parameter: Group_PubSub_Keying_Material, see {{iana-ace-groupcomm-key}}.

* REQ19: Specify and register the application profile identifier: coap_group_pubsub_app, see {{iana-profile}}.

*  REQ20: If used, specify the format and content of 'group_policies' and its entries.  Specify the policies default values: ToDo.

* REQ21: Specify the approaches used to compute and verify the PoP evidence to include in 'kdc_cred_verify', and which of those approaches is used in which case: see {{join-response}}.

* REQ22: Specify the communication protocol the members of the group must use.: CoAP {{RFC7252}}, and for
pub/sub communication {{I-D.ietf-core-coap-pubsub}}

*  REQ23: Specify the security protocol the group members must use to protect their communication. This must provide encryption, integrity and replay protection.: Symmetric COSE Key is used to create a COSE\_Encrypt0 object with an AEAD algorithm specified by the KDC.

* REQ24: Specify how the communication is secured between Client and KDC.  Optionally, specify transport profile of ACE {{RFC9200}} to use between Client and KDC.: ACE transport profile such as DTLS {{RFC9202}} or OSCORE {{RFC9203}}.

* REQ25: Specify the format of the identifiers of group members.: the Sender ID defined in {{join-response}}.

* REQ26: Specify policies at the KDC to handle ids that are not included in 'get_creds'.: See {{query}}.

* REQ27: Specify the format of newly-generated individual keying material for group members, or of the information to derive it, and corresponding CBOR label.: Not applicable.

* REQ28: Specify which CBOR tag is used for identifying the semantics of binary scopes, or register a new CBOR tag if a suitable one does not exist already.: See {{as-response}} and {{content_format}} of this document.

* REQ29: Categorize newly defined parameters according to the same criteria of Section 8 of {{I-D.ietf-ace-key-groupcomm}}.:  A Publisher Client MUST support 'group\_SenderId' in 'key'; see {{join-response}}

* REQ30: Define whether Clients must, should or may support the conditional parameters defined in Section 8 of {{I-D.ietf-ace-key-groupcomm}}, and under which circumstances.: A Publisher Client MUST support the client_cred', 'cnonce', and 'client_cred_verify' parameters; see {{join-request}}. A Publisher Client that provides the token to the KDC, through the authz-info endpoint, MUST support the parameter 'kdcchallenge'; see {{token-post}}.

* OPT1: Optionally, if the textual format of 'scope' is used, specify CBOR values to use for abbreviating the role identifiers in the group: No.

* OPT2: Optionally, specify the additional parameters used in the exchange of Token Transfer Request and Response : No.

* OPT3: Optionally, specify the negotiation of parameter values for signature algorithm and signature keys, if 'sign_info' is not used: See {{token-post}}.

* OPT4: Optionally, specify possible or required payload formats for specific error cases.: See {{join-error}}.

*  OPT5: Optionally, specify additional identifiers of error types, as values of the 'error' field in an error response from the KDC: No.

*  OPT6: Optionally, specify the encoding of 'creds_repo' if the default is not used: No.

*  OPT7: Optionally, specify the functionalities implemented at the 'control_uri' resource hosted at the Client, including message exchange encoding and other details.: No.

* OPT8: Optionally, specify the behavior of the handler in case of failure to retrieve an authentication credential for the specific node: The KDC MUST reply with a 4.00 (Bad Request) error response to the Join Request; see {{join-error}}.

* OPT9: Optionally, define a default group rekeying scheme, to refer to in case the 'rekeying_scheme' parameter is not included in the Join Response: the "Point-to-Point" rekeying scheme registered in Section 11.12 of
{{I-D.ietf-ace-key-groupcomm}}.

*  OPT10: Optionally, specify the functionalities implemented at the 'control_group_uri' resource hosted at the Client, including message exchange encoding and other details. : No.

* OPT11: Optionally, specify policies that instruct Clients to retain messages and for how long, if they are unsuccessfully decrypted.: No.

* OPT12: Optionally, specify for the KDC to perform group rekeying (together or instead of renewing individual keying material) when receiving a Key Renewal Request: ToDo.

* OPT13: Optionally, specify how the identifier of a group member's authentication credential is included in requests sent to other group members: No.

*  OPT14: Optionally, specify additional information to include in rekeying messages for the "Point-to-Point" group rekeying scheme: ToDo.

*  OPT15: Optionally, specify if Clients must or should support any of the parameters defined as optional in {{I-D.ietf-ace-key-groupcomm}}: No.

# Document Updates # {#sec-document-updates}

RFC EDITOR: PLEASE REMOVE THIS SECTION.

## Version -07 to -08 ## {#sec-07-08}

* Revised presentation of the scope format.

* The 'cnonce' parameter must be present in the Join Request.

* The 'kid' of the group key is used as Group Identifier.

* Relaxed inclusion of the 'peer_roles' parameter.

* More detailed description of the encryption and signing operations.

## Version -06 to -07 ## {#sec-06-07}

* Revised abstract and introduction.

* Clarified use of "application groups".

* Revised use of protocols and transport profiles with Broker and KDC.

* Revised overview of the profile and its security associations.

* Revised presentation of authorization flow.

* Subscribers cannot be anonymous anymore.

* Revised scope definition.

* Revised Join Response.

* Revised COSE countersignature, COSE encrypt objects.

* Further clarifications, fixes and editorial improvements.

# Acknowledgments
{: numbered="no"}

The authors wish to thank {{{Ari Keränen}}}, {{{John Preuß Mattsson}}}, {{{Jim Schaad}}}, {{{Ludwig Seitz}}}, and {{{Göran Selander}}} for the useful discussion and reviews that helped shape this document.

The work on this document has been partly supported by the H2020 project SIFIS-Home (Grant agreement 952652).

--- fluff

<!-- Local Words: -->
<!-- Local Variables: -->
<!-- coding: utf-8 -->
<!-- ispell-local-dictionary: "american" -->
<!-- End: -->
