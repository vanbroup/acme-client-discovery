---
title: "Auto-discovery mechanism for ACME authorized clients"
abbrev: "ACME Client Discovery"
category: std

docname: draft-vanbrouwershaven-acme-client-discovery-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Automated Certificate Management Environment"
keyword:
 - ACME
 - Auto-discovery
venue:
  group: "Automated Certificate Management Environment"
  type: "Working Group"
  mail: "acme@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/acme/"
  github: "vanbroup/acme-client-discovery"
  latest: "https://vanbroup.github.io/acme-client-discovery/draft-vanbrouwershaven-acme-client-discovery.html"

author:
  - ins: P. van Brouwershaven
    name: Paul van Brouwershaven
    org: Entrust Limited
    abbrev: Entrust
    street: 2500 Solandt Road – Suite 100
    city: 'Ottawa, Ontario'
    country: Canada
    code: K2K 3G5
    email: paul.vanbrouwershaven@entrust.com
  - ins: M. Ounsworth
    name: Mike Ounsworth
    org: Entrust Limited
    abbrev: Entrust
    street: 2500 Solandt Road – Suite 100
    city: 'Ottawa, Ontario'
    country: Canada
    code: K2K 3G5
    email: mike.ounsworth@entrust.com
  - ins: C. Bonnell
    name: Corey Bonnell
    organization: 'DigiCert, Inc'
    abbrev: DigiCert
    city: Pittsburgh
    region: PA
    country: United States of America
    email: corey.bonnell@digicert.com
  - ins: I.Barreira
    name: Iñigo Barreira
    org: Sectigo (Europe) SL
    abbrev: Sectigo
    street: 'Rambla Catalunya 86, 3 1. 08008 Barcelona.'
    city: Barcelona
    country: Spain
    code: 08008
    email: inigo.barreira@sectigo.com

normative:
  RFC8555:

informative:
  RFC5785:
  I-D.vanbrouwershaven-acme-auto-discovery:

--- abstract

A significant challenge in the widespread adoption of the Automated Certificate Management Environment (ACME) [RFC8555] is the trust establishment between ACME servers and clients. While ACME clients can automatically discover the URL of the ACME server through ACME Auto Discovery {{I-D.vanbrouwershaven-acme-auto-discovery}}, they face difficulty in identifying authorized clients. This draft proposes a solution to this problem by allowing Certification Authority (CA) customers to specify which ACME keys are authorized to request certificates on their behalf by simply providing the domain name of the service provider.

Specifically, this document registers the URI "/.well-known/acme-keys" at which all compliant service providers can publish their ACME client public keys. This mechanism allows the ACME server to identify the specific service provider, enhancing the trust relationship. Furthermore, it provides flexibility to service providers as they can use multiple keys and rotate them as often as they like, thereby improving security and control over their ACME client configurations while giving CA customers the ability to specifically authorize which service providers can request certificates on their behalf.


--- middle

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Introduction {#sec-intro}

The Automated Certificate Management Environment (ACME) [RFC8555] has been instrumental in streamlining the process of certificate issuance and validation. However, a significant challenge that hinders its widespread adoption is the establishment of trust between ACME servers and clients. While ACME clients can automatically discover the URL of the ACME server through ACME Auto Discovery {{I-D.vanbrouwershaven-acme-auto-discovery}}, identifying authorized clients remains a complex task.

This document proposes a solution to this problem by introducing a mechanism that allows Certification Authority (CA) customers to specify which ACME keys are authorized to request certificates on their behalf. This is achieved by simply providing the domain name of the service provider.

Specifically, this document registers the URI "/.well-known/acme-keys" where all compliant service providers can publish their ACME client public keys. This mechanism not only enhances the trust relationship by allowing the ACME server to identify the specific service provider but also provides flexibility to service providers. They can use multiple keys and rotate them as often as they like, thereby improving security and control over their ACME client configurations.

Moreover, this mechanism empowers CA customers by giving them the ability to specifically authorize which service providers can request certificates on their behalf.

# Protocol Overview

1. A user creates an account at `server.example`.
2. The user specifies at `server.example` that `client.example` is authorized to request certificats on their behalf for the domain `customer.example`.
   1. The ACME server `server.example` downloads the known public keys from `https://client.example/.well-known/acme-keys` and will regularly check for changes.
3. The ACME client `client.example` registers its key at `server.example`, which will only succeed if any of the customers have authorized `client.example`.
4. The ACME client `client.example` makes an ACME request to the ACME server from `server.example` for domain `customer.example`.
5. Based on the domain `customer.example` the ACME server `server.example` authenticates the ACME client against the known public keys of the service providers that the customer has authorized.
6. The ACME client continues normal operation according to [RFC8555].

~~~ aasvg
+----------------+     +----------------+     +----------------+
|                |     |                |     |                |
|                |     |  ACME Server   |     |  ACME Client / |
|    User        |     |     / CA       |     |  Service Prov. |
|                |     |                |     |                |
+----------------+     +----------------+     +----------------+
       |                      |                      |
       | 1. Create account    |                      |
       +--------------------->|                      |
       |                      |                      |
       | 2. Authorize client  |                      |
       +--------------------->|                      |
       |                      | 3. Register key      |
       |                      |<---------------------|
       |                      |                      |
       |                      | 4. Verify key with   |
       |                      | info from well-known |
       |                      +--------------------->|
       |                      |                      |
       |                      | 5. Request cert      |
       |                      |<---------------------+
       |                      |                      |
       |                      | 6. Normal operation  |
       |                      |<-------------------->|
       |                      |                      |

~~~


# Implementation Considerations

# IANA Considerations {#sec-iana}

##  Well-Known URI for the ACME Directory

The following value has been registered in the "Well-Known URIs" registry (using the template from [RFC5785]):

~~~
URI suffix: acme-keys
Change controller: IETF
Specification document(s): RFC XXXX, Section Y.Z
Related information: N/A
~~~

> **RFC Editor's Note:** Please replace XXXX above with the RFC number assigned to this document

# Security Considerations

## Attacker Control Over Well-Known Directory

This document introduces a mechanism where ACME client keys are published in a well-known directory of a service provider. This introduces a potential risk if an attacker gains control over this well-known directory. In such a scenario, the attacker could add their own ACME client keys, posing as the service provider. This could potentially allow the attacker to request certificates on behalf of the service provider.

However, it's important to note that even if an attacker manages to publish their own keys in the well-known directory, they would still need to prove control over the domain name to obtain a certificate, as per the ACME protocol [RFC8555]. This provides an additional layer of security and significantly reduces the risk of unauthorized certificate issuance.

Service providers should ensure the security of their well-known directories to prevent unauthorized access.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
