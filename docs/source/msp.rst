Membership Service Providers (MSP)
==================================

The document serves to provide details on the setup and best practices for MSPs.

Membership Service Provider (MSP) is a component that aims to offer an
abstraction of a membership operation architecture.

会员服务提供商（MSP）是一个组件，旨在提供一个抽象成员操作体系结构。

In particular, MSP abstracts away all cryptographic mechanisms and protocols
behind issuing and validating certificates, and user authentication. An
MSP may define their own notion of identity, and the rules by which those
identities are governed (identity validation) and authenticated (signature
generation and verification).
特别是，MSP将发放和验证证书，以及用户认证的所有密码机制和协议抽象出来。一个MSP可以定义自有的身份概念，以及身份的管理（身份验证）和认证（签名生成和验证）规则。

A Hyperledger Fabric blockchain network can be governed by one or more MSPs.
This provides modularity of membership operations, and interoperability
across different membership standards and architectures.
Hyperledger Fabric区块链网络可以由一个或多个MSP管理。这提供了模块化的成员操作，并联通各种不同的成员标准和体系结构。

In the rest of this document we elaborate on the setup of the MSP
implementation supported by Hyperledger Fabric, and discuss best practices
concerning its use.
在本文的其余部分中，我们将详细介绍Hyperledger Fabric提供的MSP机制如何设置，并讨论它的最佳实践用法。

MSP Configuration
-----------------

To setup an instance of the MSP, its configuration needs to be specified
locally at each peer and orderer (to enable peer, and orderer signing),
and on the channels to enable peer, orderer, client identity validation, and
respective signature verification (authentication) by and for all channel
members.
要设置MSP的实例，需要在每个peer和order的本地指定其配置（以启用peer和order的签名），并在各channel上启用peer，order，client的身份验证以及相应的签名验证（认证）为所有渠道成员提供服务。

Firstly, for each MSP a name needs to be specified in order to reference that MSP
in the network (e.g. ``msp1``, ``org2``, and ``org3.divA``). This is the name under
which membership rules of an MSP representing a consortium, organization or
organization division is to be referenced in a channel. This is also referred
to as the *MSP Identifier* or *MSP ID*. MSP Identifiers are required to be unique per MSP
instance. For example, shall two MSP instances with the same identifier be
detected at the system channel genesis, orderer setup will fail.
首先，对于每个MSP，需要指定一个名称以代表网络中的MSP（例如``msp1``，`org2``和``org3.divA``）。这个名称对应了渠道中的MSP成员规则，代表了联盟，组织或组织部门。这也被称为* MSP标识符*或* MSP ID *。 MSP标识符必须是每个MSP实例唯一的。例如在系统channel初始化时，如果检测到具有相同标识符的两个MSP实例，order setup将会失败。

In the case of default implementation of MSP, a set of parameters need to be
specified to allow for identity (certificate) validation and signature
verification. These parameters are deduced by
`RFC5280 <http://www.ietf.org/rfc/rfc5280.txt>`_, and include:
在MSP的默认实现中，需要指定一组参数以进行身份（证书）验证和签名验证。

- A list of self-signed (X.509) certificates to constitute the *root of
  trust*
  一组self-signed（X.509）证书构成的*根证书*
  
- A list of X.509 certificates to represent intermediate CAs this provider
  considers for certificate validation; these certificates ought to be
  certified by exactly one of the certificates in the root of trust;
  intermediate CAs are optional parameters
  一组提供商用于证书验证的中间CA的X.509证书;这些证书应该由其中一个可信的根证书签发;中间CA是可选参数
  
- A list of X.509 certificates with a verifiable certificate path to
  exactly one of the certificates of the root of trust to represent the
  administrators of this MSP; owners of these certificates are authorized
  to request changes to this MSP configuration (e.g. root CAs, intermediate CAs)
  一组代表该MSP的管理员的X.509证书，可验证的证书路径来自其中一个可信的根证书;这些证书的所有者有权请求更改此MSP配置（例如，根CA，中间CA）
  
- A list of Organizational Units that valid members of this MSP should
  include in their X.509 certificate; this is an optional configuration
  parameter, used when, e.g., multiple organisations leverage the same
  root of trust, and intermediate CAs, and have reserved an OU field for
  their members
  MSP的有效成员应在其X.509证书中包含的组织单位列表;这是一个可选的配置参数，用于多个组织利用相同的根证书和中间CA，以及为其成员保留OU字段的情况
  
- A list of certificate revocation lists (CRLs) each corresponding to
  exactly one of the listed (intermediate or root) MSP Certificate
  Authorities; this is an optional parameter
  证书撤销列表（CRLs），对应每个（中间或根）MSP证书颁发机构;这是一个可选参数
  
- A list of self-signed (X.509) certificates to constitute the *TLS root of
  trust* for TLS certificate.
  一组作为* TLS根证书*的self-signed（X.509）证书，用于生成TLS证书。
  
- A list of X.509 certificates to represent intermediate TLS CAs this provider
  considers; these certificates ought to be
  certified by exactly one of the certificates in the TLS root of trust;
  intermediate CAs are optional parameters.
  一组提供商用于证书验证的中间TLS CA的X.509证书;这些证书应该由其中一个可信的TLS根证书签发;中间CA是可选参数

*Valid*  identities for this MSP instance are required to satisfy the following conditions:
MSP实例的*有效*身份必须满足以下条件：

- They are in the form of X.509 certificates with a verifiable certificate path to
  exactly one of the root of trust certificates;
  它们的形式是X.509证书，其证书路径恰好是信任证书的根证书之一;
  
- They are not included in any CRL;
  它们不包含在任何CRL中

- And they *list* one or more of the Organizational Units of the MSP configuration
  in the ``OU`` field of their X.509 certificate structure.
  并且他们在其X.509证书结构的“OU”字段中列出一个或多个MSP配置的组织单位。

For more information on the validity of identities in the current MSP implementation,
we refer the reader to :doc:`msp-identity-validity-rules`.

In addition to verification related parameters, for the MSP to enable
the node on which it is instantiated to sign or authenticate, one needs to
specify:

- The signing key used for signing by the node (currently only ECDSA keys are
  supported), and
- The node's X.509 certificate, that is a valid identity under the
  verification parameters of this MSP.

It is important to note that MSP identities never expire; they can only be revoked
by adding them to the appropriate CRLs. Additionally, there is currently no
support for enforcing revocation of TLS certificates.

How to generate MSP certificates and their signing keys?
--------------------------------------------------------

To generate X.509 certificates to feed its MSP configuration, the application
can use `Openssl <https://www.openssl.org/>`_. We emphasise that in Hyperledger
Fabric there is no support for certificates including RSA keys.

Alternatively one can use ``cryptogen`` tool, whose operation is explained in
:doc:`getting_started`.

`Hyperledger Fabric CA <http://hyperledger-fabric-ca.readthedocs.io/en/latest/>`_
can also be used to generate the keys and certificates needed to configure an MSP.

MSP setup on the peer & orderer side
------------------------------------

To set up a local MSP (for either a peer or an orderer), the administrator
should create a folder (e.g. ``$MY_PATH/mspconfig``) that contains six subfolders
and a file:

1. a folder ``admincerts`` to include PEM files each corresponding to an
   administrator certificate
2. a folder ``cacerts`` to include PEM files each corresponding to a root
   CA's certificate
3. (optional) a folder ``intermediatecerts`` to include PEM files each
   corresponding to an intermediate CA's certificate
4. (optional) a file ``config.yaml`` to include information on the
   considered OUs; the latter are defined as pairs of
   ``<Certificate, OrganizationalUnitIdentifier>`` entries of a yaml array
   called ``OrganizationalUnitIdentifiers``, where ``Certificate`` represents
   the relative path to the certificate of the certificate authority (root or
   intermediate) that should be considered for certifying members of this
   organizational unit (e.g. ./cacerts/cacert.pem), and
   ``OrganizationalUnitIdentifier`` represents the actual string as
   expected to appear in X.509 certificate OU-field (e.g. "COP")
5. (optional) a folder ``crls`` to include the considered CRLs
6. a folder ``keystore`` to include a PEM file with the node's signing key;
   we emphasise that currently RSA keys are not supported
7. a folder ``signcerts`` to include a PEM file with the node's X.509
   certificate
8. (optional) a folder ``tlscacerts`` to include PEM files each corresponding to a TLS root
   CA's certificate
9. (optional) a folder ``tlsintermediatecerts`` to include PEM files each
   corresponding to an intermediate TLS CA's certificate

In the configuration file of the node (core.yaml file for the peer, and
orderer.yaml for the orderer), one needs to specify the path to the
mspconfig folder, and the MSP Identifier of the node's MSP. The path to the
mspconfig folder is expected to be relative to FABRIC_CFG_PATH and is provided
as the value of parameter ``mspConfigPath`` for the peer, and ``LocalMSPDir``
for the orderer. The identifier of the node's MSP is provided as a value of
parameter ``localMspId`` for the peer and ``LocalMSPID`` for the orderer.
These variables can be overridden via the environment using the CORE prefix for
peer (e.g. CORE_PEER_LOCALMSPID) and the ORDERER prefix for the orderer (e.g.
ORDERER_GENERAL_LOCALMSPID). Notice that for the orderer setup, one needs to
generate, and provide to the orderer the genesis block of the system channel.
The MSP configuration needs of this block are detailed in the next section.

*Reconfiguration* of a "local" MSP is only possible manually, and requires that
the peer or orderer process is restarted. In subsequent releases we aim to
offer online/dynamic reconfiguration (i.e. without requiring to stop the node
by using a node managed system chaincode).

Channel MSP setup
-----------------

At the genesis of the system, verification parameters of all the MSPs that
appear in the network need to be specified, and included in the system
channel's genesis block. Recall that MSP verification parameters consist of
the MSP identifier, the root of trust certificates, intermediate CA and admin
certificates, as well as OU specifications and CRLs.
The system genesis block is provided to the orderers at their setup phase,
and allows them to authenticate channel creation requests. Orderers would
reject the system genesis block, if the latter includes two MSPs with the same
identifier, and consequently the bootstrapping of the network would fail.

For application channels, the verification components of only the MSPs that
govern a channel need to reside in the channel's genesis block. We emphasise
that it is **the responsibility of the application** to ensure that correct
MSP configuration information is included in the genesis blocks (or the
most recent configuration block) of a channel prior to instructing one or
more of their peers to join the channel.

When bootstrapping a channel with the help of the configtxgen tool, one can
configure the channel MSPs by including the verification parameters of MSP
in the mspconfig folder, and setting that path in the relevant section in
``configtx.yaml``.

*Reconfiguration* of an MSP on the channel, including announcements of the
certificate revocation lists associated to the CAs of that MSP is achieved
through the creation of a ``config_update`` object by the owner of one of the
administrator certificates of the MSP. The client application managed by the
admin would then announce this update to the channels in which this MSP appears.

Best Practices
--------------

In this section we elaborate on best practices for MSP
configuration in commonly met scenarios.

**1) Mapping between organizations/corporations and MSPs**

We recommend that there is a one-to-one mapping between organizations and MSPs.
If a different mapping type of mapping is chosen, the following needs to be to
considered:

- **One organization employing various MSPs.** This corresponds to the
  case of an organization including a variety of divisions each represented
  by its MSP, either for management independence reasons, or for privacy reasons.
  In this case a peer can only be owned by a single MSP, and will not recognize
  peers with identities from other MSPs as peers of the same organization. The
  implication of this is that peers may share through gossip organization-scoped
  data with a set of peers that are members of the same subdivision, and NOT with
  the full set of providers constituting the actual organization.
- **Multiple organizations using a single MSP.** This corresponds to a
  case of a consortium of organisations that are governed by similar
  membership architecture. One needs to know here that peers would propagate
  organization-scoped messages to the peers that have an identity under the
  same MSP regardless of whether they belong to the same actual organization.
  This is a limitation of the granularity of MSP definition, and/or of the peer’s
  configuration.

**2) One organization has different divisions (say organizational units), to**
**which it wants to grant access to different channels.**

Two ways to handle this:

- **Define one MSP to accommodate membership for all organization’s members**.
  Configuration of that MSP would consist of a list of root CAs,
  intermediate CAs and admin certificates; and membership identities would
  include the organizational unit (``OU``) a member belongs to. Policies can then
  be defined to capture members of a specific ``OU``, and these policies may
  constitute the read/write policies of a channel or endorsement policies of
  a chaincode. A limitation of this approach is that gossip peers would
  consider peers with membership identities under their local MSP as
  members of the same organization, and would consequently gossip
  with them organisation-scoped data (e.g. their status).
- **Defining one MSP to represent each division**.  This would involve specifying for each
  division, a set of certificates for root CAs, intermediate CAs, and admin
  Certs, such that there is no overlapping certification path across MSPs.
  This would mean that, for example, a different intermediate CA per subdivision
  is employed. Here the disadvantage is the management of more than one
  MSPs instead of one, but this circumvents the issue present in the previous
  approach.  One could also define one MSP for each division by leveraging an OU
  extension of the MSP configuration.

**3) Separating clients from peers of the same organization.**

In many cases it is required that the “type” of an identity is retrievable
from the identity itself (e.g. it may be needed that endorsements are
guaranteed to have derived by peers, and not clients or nodes acting solely
as orderers).

There is limited support for such requirements.

One way to allow for this separation is to to create a separate intermediate
CA for each node type - one for clients and one for peers/orderers; and
configure two different MSPs - one for clients and one for peers/orderers.
Channels this organization should be accessing would need to include
both MSPs, while endorsement policies will leverage only the MSP that
refers to the peers. This would ultimately result in the organization
being mapped to two MSP instances, and would have certain consequences
on the way peers and clients interact.

Gossip would not be drastically impacted as all peers of the same organization
would still belong to one MSP. Peers can restrict the execution of certain
system chaincodes to local MSP based policies. For
example, peers would only execute “joinChannel” request if the request is
signed by the admin of their local MSP who can only be a client (end-user
should be sitting at the origin of that request). We can go around this
inconsistency if we accept that the only clients to be members of a
peer/orderer MSP would be the administrators of that MSP.

Another point to be considered with this approach is that peers
authorize event registration requests based on membership of request
originator within their local MSP. Clearly, since the originator of the
request is a client, the request originator is always doomed to belong
to a different MSP than the requested peer and the peer would reject the
request.

**4) Admin and CA certificates.**

It is important to set MSP admin certificates to be different than any of the
certificates considered by the MSP for ``root of trust``, or intermediate CAs.
This is a common (security) practice to separate the duties of management of
membership components from the issuing of new certificates, and/or validation of existing ones.

**5) Blacklisting an intermediate CA.**

As mentioned in previous sections, reconfiguration of an MSP is achieved by
reconfiguration mechanisms (manual reconfiguration for the local MSP instances,
and via properly constructed ``config_update`` messages for MSP instances of a channel).
Clearly, there are two ways to ensure an intermediate CA considered in an MSP is no longer
considered for that MSP's identity validation:

1. Reconfigure the MSP to no longer include the certificate of that
   intermediate CA in the list of trusted intermediate CA certs. For the
   locally configured MSP, this would mean that the certificate of this CA is
   removed from the ``intermediatecerts`` folder.
2. Reconfigure the MSP to include a CRL produced by the root of trust
   which denounces the mentioned intermediate CA's certificate.

In the current MSP implementation we only support method (1) as it is simpler
and does not require blacklisting the no longer considered intermediate CA.

**6) CAs and TLS CAs**

MSP identities' root CAs and MSP TLS certificates' root CAs (and relative intermediate CAs)
need to be declared in different folders. This is to avoid confusion between
different classes of certificates. It is not forbidden to reuse the same
CAs for both MSP identities and TLS certificates but best practices suggest
to avoid this in production.

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
