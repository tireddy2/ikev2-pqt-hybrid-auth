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
title: "Post-Quantum Traditional (PQ/T) Hybrid Authentication in the Internet Key Exchange Version 2 (IKEv2)"
abbrev: "IKEv2 PQTH Auth"
category: std

docname: draft-hu-ipsecme-pqt-hybrid-auth-00
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: sec
workgroup: ipsecme
keyword:
 - Post-Quantum
 - Hybrid Authentication
 - IKEv2
venue:
  group: WG
  type: Working Group
  mail: ipsec@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/ipsec/
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Hu, Jun
    organization: Nokia
    email: jun.hu@nokia.com
    country: United States of America
 -
    fullname: Yasufumi Morioka
    organization: NTT DOCOMO, INC.
    email: yasufumi.morioka.dt@nttdocomo.com
    country: Japan


normative:
  I-D.ietf-lamps-pq-composite-sigs:
  I-D.ietf-pquip-hybrid-signature-spectrums:
  I-D.ietf-lamps-cert-binding-for-multi-auth:
  RFC4739:
  RFC7296:
  RFC7427:
  RFC9593:
  X.690:
    title: "Information Technology - ASN.1 encoding rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)"
    seriesinfo:
      ISO/IEC: 8825-1:2021 (E)
      ITU-T: Recommendation X.690
    date: Feb.2021

informative:

  ML-DSA:
    title: Module-Lattice-Based Digital Signature Standard
    date: Aug.2023
    seriesinfo:
      NIST: FIPS-204
      State: Initial Public Draft
    target: https://csrc.nist.gov/pubs/fips/204/ipd


  QRPKI:
    title: Transitioning to a Quantum-Resistant Public Key Infrastructure
    author:
      -
       name: Nina Bindel
      -
       name: Udyani Herath
      -
       name: Mattew McKague
      -
       name: Douglas Stebila
    date: 2017
    target: https://eprint.iacr.org/2017/460




  RFC8784:
  RFC9370:


--- abstract

 One IPsec area that would be impacted by Cryptographically Relevant Quantum Computer (CRQC) is IKEv2 authentication based on traditional asymmetric cryptograph algorithms: e.g RSA, ECDSA; which are widely deployed authentication options of IKEv2. There are new Post-Quantum Cryptograph (PQC) algorithms for digital signature like NIST {{ML-DSA}}, however it takes time for new cryptograph algorithms to mature, so there is security risk to use only the new algorithm before it is field proven. This document describes a IKEv2 hybrid authentication scheme that could contain both traditional and PQC algorithms, so that authentication is secure as long as one algorithm in the hybrid scheme is secure.


--- middle

# Introduction

A Cryptographically Relevant Quantum Computer (CRQC) could break traditional asymmetric cryptograph algorithms: e.g RSA, ECDSA; which are widely deployed authentication options of IKEv2. New Post-Quantum Cryptograph (PQC) algorithms for digital signature are being standardized at the time of writing like NIST {{ML-DSA}}, however consider potential flaws in the new algorithm's specifications and implementations, it will take time for these new PQC algorithms to be field proven. So it is risky to only use PQC algorithms before they are mature. There is more detailed discussion on motivation of a hybrid approach for authentication in {{Section 1.3 of I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document describes a IKEv2 hybrid authentication scheme that contains both traditional and PQC algorithms, so that authentication is secure as long as one algorithm in the hybrid scheme is secure.

Following two types of setup are covered: 

1. Type-1: A single certificate that has composite key as defined in {{I-D.ietf-lamps-pq-composite-sigs}}
2. Type-2: Two certificates, one with traditional algorithm key and one with PQC algorithm key


# Conventions and Definitions

{::boilerplate bcp14-tagged}

Cryptographically Relevant Quantum Computer (CRQC): A quantum computer that is capable of breaking real world cryptographic systems.

Post-Quantum Cryptograph (PQC) algorithms: Asymmetric cryptograph algorithms are thought to be secure against CRQC.

Traditional Cryptograph algorithms: Existing asymmetric cryptograph algorithms could be broken by CRQC, like RSA, ECDSA ..etc.

# IKEv2 Key Exchange
There is no changes introduced in this document to the IKEv2 key exchange process, although it MUST be also resilient to CRQC when using along with the PQ/T hybrid authentication, for example key exchange using the PPK as defined in {{RFC8784}}, or hybrid key exchanges that include PQC algorithm via multiple key exchange process as defined in {{RFC9370}}.





# Exchanges

The hybrid authentication exchanges is illustrated in an example depicted in {{hybrid-auth-figure}}, the key exchange uses PPK, however it could be other key exchanges that involves PQC algorithm since how key exchange is done is transparent to authentication.

~~~~~~~~~~~
   Initiator                         Responder
   -------------------------------------------------------------------
   HDR, SAi1, KEi, Ni,
              N(USE_PPK) -->
                                <--  HDR, SAr1, KEr, Nr, [CERTREQ,] N(USE_PPK),
                                         N(SUPPORTED_AUTH_METHODS)

   HDR, SK {IDi, CERT+, [CERTREQ,]
            [IDr,] AUTH, SAi2,
            TSi, TSr, N(PPK_IDENTITY, PPK_ID),
            N(SUPPORTED_AUTH_METHODS)} -->
                                <--  HDR, SK {IDr, CERT+, [CERTREQ,]
                                         AUTH, [N(PPK_IDENTITY)]}
~~~~~~~~~~~
{: #hybrid-auth-figure title="Hybrid Authentication Exchanges with RFC8784 Key Exchange"}

## Announcement

Announcement of support hybrid authentication is through SUPPORTED_AUTH_METHODS notification as defined in {{RFC9593}}, which includes a list of acceptable authentication methods announcements, this document defines a hybrid authentication announcements with following format:


                         1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Length (>3)  |  Auth Method  |   Cert Link 1 | Alg 1 flag    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Alg 1 Len     |                                               |
    +-+-+-+-+-+-+-+-+                                               |
    ~                      AlgorithmIdentifier 1                    ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Cert Link 2   | Alg 2 flag    |  Alg 2 Len    |               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
    |                                                               |
    ~                      AlgorithmIdentifier 2                    ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    ~                      ...                                      ~
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Cert Link 3   | Alg 3 flag    |  Alg 3 Len    |               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
    |                                                               |
    ~                      AlgorithmIdentifier N                    ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
{: #ds-announce title="Hybrid Authentication Announcement"}

The announcement include a list of N algorithms could be used for hybrid signature

* Auth Method: A new value to be allocated by IANA
* Cert Link N: Links corresponding signature algorithm N with a particular CA. as defined in {{Section 3.2.2 of RFC9593}}
* Alg N Flag: 
  * C: set to 1 if the algorithm could be used in type-1 setup
  * S: set to 1 if the algorithm could be used in type-2 setup
  * C and S MUST NOT be zero at the same time
  * RESERVED: set to 0

~~~~~~~~~~~             
     0 1 2 3 4 5 6 7 
    +-+-+-+-+-+-+-+-+
    |C|S| RESERVED  |
    +-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #announce-flag title="Algorithm Flag"}

* AlgorithmIdentifier N: The variable-length ASN.1 object that is encoded using Distinguished Encoding Rules (DER) {{X.690}} and identifies the  algorithm of a composite signature as defined in {{Section 7 of I-D.ietf-lamps-pq-composite-sigs}}.


### Sending Announcement

As defined in {{RFC9593}}, responder include SUPPORTED_AUTH_METHODS in IKE_SA_INIT response (and potentially also in IKE_INTERMEDIATE response), while initiator include the notification in IKE_AUTH request. 

Sender include a hybrid authentication announcement in SUPPORTED_AUTH_METHODS, which contains 0 or N AlgorithmIdentifiers sender accepts, each AlgorithmIdentifier identifies a combination of algorithms:

* a traditional PKI algorithm with corresponding hash algorithm (e.g. id-RSASA-PSS with id-sha256)
* a PQC algorithm (e.g. id-ML-DSA-44)
  * in case of Hash ML-DSA, there is also a pre-hash algorithm (e.g. id-sha256)

C and S bits in flag field are set according to whether sender accept the algorithm in setup-1/setup-2. 

Announcement without any algorithm signals that there is no particular restrictions on algorithm. 

### Receiving Announcement

If hybrid authentication announcement is received, and receiver choose to authenticate itself using hybrid authentication, then based on its local policy and certificates, one AlgorithmIdentifier in the hybrid authentication announcement and a PKI setup (type-1 or type-2) are chosen to create its AUTH and CERT payload(s).


## AUTH & CERT payload

The IKEv2 AUTH payload has following format as defined in {{Section 3.8 of RFC7296}}:


                            1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | Next Payload  |C|  RESERVED   |         Payload Length        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | Auth Method   |                RESERVED                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      ~                      Authentication Data                      ~
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
{: #rfc7296-auth title="AUTH payload"}

For hybrid authentication, the AUTH Method has value defined in {{announcement}}

The Authentication Data field follows format defined in {{Section 3 of RFC7427}}:



                           1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | ASN.1 Length  | AlgorithmIdentifier ASN.1 object              |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      ~        AlgorithmIdentifier ASN.1 object continuing            ~
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      ~                         Signature Value                       ~
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
{: #ha-auth-data title="Authentication Data in hybrid AUTH payload"}


### Type-1

Based on selected AlgorithmIdentifier A and setup type T, the Signature Value is created via following procedure, 

1. There is no change on data to be signed, e.g. InitiatorSignedOctets/ResponderSignedOctets as defined in {{Section 2.15 of RFC7296}}
2. Follow Sign operation identified by A, e.g. {{Section 4.2.1 of I-D.ietf-lamps-pq-composite-sigs}} or {{Section 4.3.1 of I-D.ietf-lamps-pq-composite-sigs}}; the ctx input is the string of "IKEv2". 

The signing composite certificate MUST be the first CERT payload. 

### Type-2

The procedure is same as Type-1, use private key of traditional and PQC certificate accordingly; e.g. in Sign procedure define in {{Section 4.2.1 of I-D.ietf-lamps-pq-composite-sigs}}, the `mldsaSK` is the private key of ML-DSA certificate, while `tradSK` is the private key of traditional certificate. 

The signing PQC certificate MUST be the first CERT payload, while traditional certificate MUST be the second CERT payload.

#### RelatedCertificate
The signing certificate MAY contain RelatedCertificate extension, then the receiver SHOULD verify the extension according to {{Section 4.2 of I-D.ietf-lamps-cert-binding-for-multi-auth}}, failed verification SHOULD fail authentication.


# Security Considerations

The security of general PQ/T hybrid authentication is discussed in {{I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document uses mechanisms defined in {{I-D.ietf-lamps-pq-composite-sigs}}, {{RFC7427}} and {{RFC9593}}, the security discussion in the corresponding RFCs also apply.


# IANA Considerations

This document requests a value in "IKEv2 Authentication Method" subregistry under IANA "Internet Key Exchange Version 2 (IKEv2) Parameters" registry


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
