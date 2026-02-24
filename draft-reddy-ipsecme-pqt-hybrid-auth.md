---
title: "Hybrid Post-Quantum and Traditional Authentication for IKEv2"
abbrev: "IKEv2 PQ/T Hybrid Auth"
category: std

docname: draft-reddy-ipsecme-pqt-hybrid-auth-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "IP Security Maintenance and Extensions"
keyword:
 - Post-Quantum
 - Hybrid Authentication
 - IKEv2
venue:
  group: "IP Security Maintenance and Extensions"
  type: "Working Group"
  mail: "ipsec@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ipsec/"
  github: "tireddy2/ipsecme-pqt-hybrid"
  latest: "https://tireddy2.github.io/ipsecme-pqt-hybrid/draft-reddy-ipsecme-pqt-hybrid-auth.html"

author:
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "k.tirumaleswar_reddy@nokia.com"
-
    ins: S. Fluhrer
    name: Scott Fluhrer
    org: Cisco Systems
    email: sfluhrer@cisco.com

normative:
  I-D.ietf-lamps-pq-composite-sigs:
  RFC7296:
  RFC7427:
  RFC9593:
  RFC4739:
  RFC9242:
  RFC7383:

informative:

  ML-DSA:
    title: Module-Lattice-Based Digital Signature Standard
    date: Aug.2023
    seriesinfo:
      NIST: FIPS-204
      State: Initial Public Draft
    target: https://csrc.nist.gov/pubs/fips/204/ipd
  RFC9794:


--- abstract

A Cryptographically Relevant Quantum Computer (CRQC) can break traditional public-key algorithms (e.g., RSA, ECDSA), which are typically used for authentication in IKEv2. Combining the post-quantum ML-DSA signature algorithm with a traditional signature algorithm provides protection against potential weaknesses or implementation flaws in ML-DSA. This draft defines hybrid PKI authentication methods for IKEv2 that ensure that authentication remains secure as long as at least one of the component signature algorithms remains unbroken.

--- middle

# Introduction

The advent of quantum computing poses a significant threat to current cryptographic systems. Traditional cryptographic algorithms such as RSA, Diffie-Hellman, DSA, and their elliptic curve variants are vulnerable to quantum attacks. During the transition to post-quantum cryptography (PQC), uncertainty remains regarding the long-term security of both traditional and PQC algorithms. Hybrid approaches allow deployments to mitigate risk during this transition period.

Unlike previous migrations between cryptographic algorithms, the decision of when to migrate and which algorithms to adopt is far from straightforward. Even after the migration period, it may be advantageous for an entity's cryptographic identity to incorporate multiple public-key algorithms to enhance security.

Cautious implementers may opt to combine cryptographic algorithms such that a successful forgery requires breaking each of the component signature algorithms used in the hybrid construction.

This document defines hybrid authentication mechanisms for IKEv2 that combine traditional and post-quantum (PQC) signature algorithms. The security objective of these mechanisms is that authentication remains secure as long as at least one of the component signature algorithms in the hybrid construction remains secure against forgery.

The mechanisms specified in this document provide a general framework for combining PQC and traditional signature algorithms in IKEv2. Although this document primarily describes combinations involving ML-DSA {{ML-DSA}} variants and traditional algorithms, the framework is not limited to ML-DSA and can accommodate other PQC and traditional signature algorithm combinations.

The following two deployment models are specified:

1. Composite Certificate:
   A single certificate containing a composite public key and composite signature, as defined in {{I-D.ietf-lamps-pq-composite-sigs}}. In this model, a single certificate chain and a single AUTH payload are used to provide hybrid authentication assurance.

2. Dual Certificates:
   One certificate containing a traditional public key and one certificate containing a PQC public key. This model exemplifies a PQ/T hybrid protocol with non-composite authentication, as defined in {{Section 4 of RFC9794}}. In this approach, two independent single-algorithm certificate chains are used in parallel, each validated according to standard PKIX procedures. Both certificates are conveyed during the IKE_AUTH exchange, and each corresponding signature is computed over the IKEv2 signed octets defined in Section 2.15 of {{RFC7296}}. Consequently, both authentications are cryptographically bound to the same IKE SA without requiring modifications to the X.509 certificate format.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

Cryptographically Relevant Quantum Computer (CRQC): A quantum computer that is capable of breaking real-world cryptographic systems.

Post-Quantum Cryptographic (PQC) algorithms: Asymmetric cryptographic algorithms designed to resist attacks by cryptographically relevant quantum computers (CRQC).

Traditional Cryptographic algorithms: Existing asymmetric Cryptographic  algorithms could be broken by CRQC, like RSA, ECDSA, etc.

# IKEv2 Key Exchange

When hybrid authentication is used to achieve post-quantum security goals, the key exchange mechanism should provide comparable post-quantum resilience; otherwise, the overall security of the IKE SA will still depend on the traditional key exchange.

# Exchanges

The hybrid authentication exchanges are illustrated in {{hybrid-auth-figure}}, using an ML-KEM key exchange carried in an IKE_SA_INTERMEDIATE exchange as defined in {{RFC9242}}. The key exchange mechanism is independent of the authentication mechanism defined in this document.

~~~
Initiator                         Responder
-------------------------------------------------------------------
HDR, SAi1, KEi, Ni -->
                  <--  HDR, SAr1, KEr, Nr, [CERTREQ,]
                                      N(SUPPORTED_AUTH_METHODS)

HDR, SK {INTERMEDIATE KEi, Ni} -->
                  <--  HDR, SK {INTERMEDIATE KEr, Nr}

HDR, SK {IDi, CERT+, [CERTREQ,]
        [IDr,] AUTH, SAi2,
        TSi, TSr,
        N(SUPPORTED_AUTH_METHODS)} -->
                            <--  HDR, SK {IDr, CERT+, [CERTREQ,]
                                      AUTH}
-------------------------------------------------------------------
~~~
{: #hybrid-auth-figure title="Hybrid Authentication Exchanges with ML-KEM via IKE_SA_INTERMEDIATE"}


# Composite Certificate

This draft extends and complements {{!PQC-AUTH=I-D.ipsecme-ikev2-pqc-auth}} which defines how to use Post-Quantum Cryptographic (PQC) signature algorithms (such as ML-DSA and SLH-DSA) in IKEv2 authentication. Both drafts share the same overarching goal:

Enable IKEv2 to authenticate peers using PQC signature algorithms, ensuring security against quantum-capable adversaries.

Whereas {{PQC-AUTH}} specifies PQC-only authentication, this draft specifies how to deploy PQC and traditional algorithms together to provide hybrid assurance during the migration phase.

Both drafts:

* Do not require any changes to IKEv2 base protocol messages.
* Rely on the standard IKEv2 AUTH payload format {{RFC7296}}.
* Use SUPPORTED_AUTH_METHODS ({{RFC9593}}). In this case, the IKev2 peers use the SUPPORTED_AUTH_METHODS notification to advertise supported composite signature algorithms.

IKEv2 can use arbitrary signature algorithms as described in {{RFC7427}}, where the "Digital Signature" authentication method replaces older signature authentication methods. Both standalone PQC signature algorithms and composite signature algorithms can be incorporated using the "Signature Algorithm" field in the AUTH payload, as defined in {{!RFC7427}}.

For composite signatures, a single AlgorithmIdentifier describes a composite public key and a composite signature that combines multiple constituent algorithms (e.g., a traditional and a PQC algorithm) in accordance with {{I-D.ietf-lamps-pq-composite-sigs}}. This allows a single certificate and AUTH payload to provide hybrid assurance without requiring multiple exchanges.

AlgorithmIdentifier ASN.1 objects are used to uniquely identify composite schemes, including the full parameter set for each constituent algorithm. This ensures unambiguous selection and verification of composite signature during authentication.

## Composite Certificate Processing

Authentication using composite certificates follows the generic digital signature authentication method defined in {{RFC7427}} and the AUTH computation defined in Section 2.15 of {{RFC7296}}. If one or more IKE_SA_INTERMEDIATE exchanges occurred, the signed octets are constructed as specified in {{RFC9242}}.

The end-entity certificate MUST contain a composite public key as defined in {{I-D.ietf-lamps-pq-composite-sigs}}. The composite signature algorithm used in the AUTH payload MUST correspond to the composite public key algorithm in the certificate. A mismatch between the AUTH signature algorithm and the certificate public key algorithm MUST cause the IKE_SA negotiation to fail.

Signature generation and verification are performed using the composite signature scheme as defined in {{I-D.ietf-lamps-pq-composite-sigs}}. Any internal hashing or message preprocessing is performed as specified by that document.

# Dual Certificate Hybrid Authentication

This section describes how this draft leverages the mechanisms defined in {{RFC4739}} to enable PQ/T hybrid authentication in IKEv2.

When using dual certificates, each peer performs multiple rounds of authentication as specified in {{RFC4739}}:

* During capability negotiation, each peer indicates support for multiple authentications by including the MULTIPLE_AUTH_SUPPORTED notification in the initial key exchange.
* During the first IKE_AUTH exchange, the ANOTHER_AUTH_FOLLOWS notification is included to indicate that a subsequent authentication round will follow.

The authentication process is as follows:

1. First IKE_AUTH exchange
   - Uses the traditional certificate and signature.
   - Includes the ANOTHER_AUTH_FOLLOWS notification to signal that another authentication will occur.

2. Second IKE_AUTH exchange
   - Uses the PQC certificate and signature.
   - This completes the dual certificate authentication process.

Both authentication exchanges compute the AUTH payload as defined in Section 2.15 of {{RFC7296}}, as specified by {{RFC4739}}. Each signature binds the peer’s identity to the IKE SA transcript. If one or more IKE_SA_INTERMEDIATE exchanges occurred, the signed octets are constructed as specified in {{RFC9242}}. Since both authentication rounds occur within the same IKE SA and both must succeed, the resulting IKE SA is authenticated only if both signatures are valid.

Each party MUST validate both authentication rounds. If either round fails, the IKE SA negotiation MUST fail.

## Example Flow

- IKE_SA_INIT: ECDH exchange, MULTIPLE_AUTH_SUPPORTED
- IKE_SA_INTERMEDIATE: ML-KEM exchange
- First IKE_AUTH: traditional CERT, traditional AUTH, ANOTHER_AUTH_FOLLOWS
- Second IKE_AUTH: PQC CERT, PQC AUTH

## Certificate Validation

When dual certificate authentication is used, each certificate chain is validated according to the applicable PKIX procedures as defined in {{RFC7296}}.

Both authentication rounds defined in {{RFC4739}} MUST succeed. Failure to validate either certificate chain or failure to verify either corresponding AUTH payload signature MUST cause the IKE_SA negotiation to fail.

Deployments should ensure that the selected traditional and PQC algorithms provide comparable security strength to avoid unintended weakening of the overall authentication assurance.

# IKEv2 Fragmentation

Post-quantum signature algorithms and certificate chains may significantly increase the size of IKE_AUTH messages. Implementations supporting the mechanisms defined in this document MUST support IKEv2 Fragmentation as defined in {{RFC7383}}.

# Negotiation

Note: currently, this section talks mostly about the problems that we'll need to addressed.
Eventually, it'll be updated to talk about the actual mechanism, including the bits-on-the-wire format.

To support brown field upgrades, we will need for an IKE device to be able to support negotiating with devices with only conventional (e.g. RSA) certificates, and with devices with hybrid or dual certificates (e.g. RSA and ML-DSA).
In addition, for the network to be entire postquantum safe, devices will need to be able to mandate that hybrid or dual certificates be used.
Furthermore, it would be cleaner if the device sent its policy upfront to the peer, rather than letting it try to negotiate and failing.

For hybrid certificates, this is straight-forward.  A hybrid certificate can be listed in the SUPPORTED_AUTH_METHODS list as yet another algorithm.
A device can decide to list support for both that and RSA, or it could decide to list only support for hybrid certificates.
The existing mechanism in IKE will cleanly decide which is appropriate.
The only remaining issue is one of preference (if we have multiple algorithm OIDs that are acceptable to the peer, which one should we use).
This is not a security issue; instead, it is a performance issue (preferring hybrid would allow us to find performance problems earlier).

For dual certificates, there is currently no mechanism addressing it.
Any such mechanism needs to address the current issues (backwards compatibility to conventional certificates vs possible mandatory support for dual).
In addition, it would be ideal if it is somewhat generalizable, with the hope that it might be useful to address future requirements.
Furthermore, it should not be so complex to make it difficult to understand, to describe or to implement.
In addition, I do not believe for it to be appropriate for this document to take a strong stand about the category of any specific algorithm (whether or not it is 'postquantum')
'
Here is one plausible way to do that:

1. For each algorithm listed in the supported auth method announcement, we somehow tag it with a general type.  For example, RSA might get the type 0 (which an implementation might use to describe a conventional algorithm), while ML-DSA-87 might be tagged as type 1 (which might be used to describe a PQ algorithm).

2. In addition, there is a separate list of allowable combinations.  For example, a device in the process of being upgraded might list either "0" (conventional only is acceptable) or "0+1" (conventional plus postquantum is allowed).  A device after the network has been upgraded might list "0+1" (dual certificates, including a conventional certificate and an PQ certificate) as the only allowable option.  An even later transition (deprecating conventional certificates) might list "0+1" or "1" as allowable combinations.

Of course, were we to use this approach, the method of tagging the algorithms, plus the method of listing the allowable combintations, would need to be specified.

# Security Considerations

The hybrid mechanisms defined in this document aim to provide authentication security as long as at least one component signature algorithm remains secure against forgery.

The security of general PQ/T hybrid authentication is discussed in {{RFC9794}}. This document relies on mechanisms defined in {{I-D.ietf-lamps-pq-composite-sigs}}, {{RFC7427}}, {{RFC9593}}, and {{RFC4739}}, and the security considerations of those specifications need to be taken into account.

Traditional signature algorithms such as ECDSA, Ed25519, and Ed448 provide existential unforgeability under chosen-message attack (EUF-CMA), which is sufficient for IKEv2 authentication. When used as the traditional component in a composite construction with ML-DSA, these algorithms contribute to defense-in-depth during the transition to post-quantum cryptography, maintaining IKEv2 authentication security as long as at least one component algorithm remains secure.

However, composite signature schemes do not in general preserve strong unforgeability (SUF-CMA) once the traditional component algorithm is broken, for example due to the availability of CRQCs. In such cases, a forged traditional signature component can be combined with a valid post-quantum component to produce a composite signature that verifies successfully, violating SUF. This loss of SUF is inherent to the composite construction and does not impact IKEv2, which relies only on the composite signature verification result.

## Downgrade Considerations

The AUTH computation in IKEv2 signs the IKE_SA_INIT exchange as specified in Section 2.15 of {{RFC7296}}. Therefore, negotiation signals exchanged during IKE_SA_INIT (such as `SUPPORTED_AUTH_METHODS` {{RFC9593}} or `MULTIPLE_AUTH_SUPPORTED` {{RFC4739}}) are cryptographically bound to the authentication exchange and cannot be modified by an active attacker without causing authentication failure.

If hybrid authentication is required by local policy, implementations MUST enforce that the negotiated authentication method satisfies that policy.

Specifically:

* If composite authentication is required, receipt of a non-composite certificate or non-composite signature algorithm during IKE_AUTH MUST cause the IKE_SA negotiation to fail.

* If dual certificate authentication is required:
  - Both authentication rounds defined in {{RFC4739}} MUST complete successfully.
  - Failure of either authentication round MUST cause the IKE_SA negotiation to fail.

# IANA Considerations

None.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
