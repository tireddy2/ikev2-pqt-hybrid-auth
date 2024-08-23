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

docname: draft-hu-ipsecme-pqt-hybrid-auth-latest
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
 -
    fullname: Yasufumi Morioka
    organization: NTT DOCOMO, INC.
    email: yasufumi.morioka.dt@nttdocomo.com


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

 One IPsec area that would be impacted by Cryptographically Relevant Quantum Computer (CRQC) is IKEv2 authentication based on classic asymmetric cryptograph algorithms: e.g RSA, ECDSA; which are widely deployed authentication options of IKEv2. There are new Post-Quantum Cryptograph (PQC) algorithms for digital signature like NIST {{ML-DSA}}, however it takes time for new cryptograph algorithms to mature, so there is security risk to use only the new algorithm before it is field proven. This document describes a IKEv2 hybrid authentication scheme that could contain both classic and PQC algorithms, so that authentication is secure as long as one algorithm in the hybrid scheme is secure.


--- middle

# Introduction

A Cryptographically Relevant Quantum Computer (CRQC) could break classic asymmetric cryptograph algorithms: e.g RSA, ECDSA; which are widely deployed authentication options of IKEv2. New Post-Quantum Cryptograph (PQC) algorithms for digital signature are being standardized at the time of writing like NIST {{ML-DSA}}, however consider potential flaws in the new algorithm's specifications and implementations, it will take time for these new PQC algorithms to be field proven. So it is risky to only use PQC algorithms before they are mature. There is more detailed discussion on motivation of a hybrid approach for authentication in {{Section 1.3 of I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document describes a IKEv2 hybrid authentication scheme that contains both classic and PQC algorithms, so that authentication is secure as long as one algorithm in the hybrid scheme is secure.



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
        N(SIGNATURE_HASH_ALGORITHMS), N(USE_PPK) -->
                                <--  HDR, SAr1, KEr, Nr, [CERTREQ,] N(USE_PPK),
                                         N(SIGNATURE_HASH_ALGORITHMS),
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

Both peers includes SIGNATURE_HASH_ALGORITHMS as defined in {{Section 4 of RFC7427}} notification in IKE_SA_INIT exchange to indicate supported hash algorithms used with digital signature.

Responder includes SUPPORTED_AUTH_METHODS as defined in {{RFC9593}} notification include a list of supported authentication methods announcements includes a hybrid authentication announcements with following format:


                        1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Length (>3)  |  Auth Method  |   Cert Link 1 | Alg 1 Len     |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    ~                      AlgorithmIdentifier 1                    ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Cert Link 2   | Alg 2 Len     |                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               +
    |                                                               |
    ~                      AlgorithmIdentifier 2                    ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    ~                      ...                                      ~
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Cert Link N   | Alg N Len     |                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               +
    |                                                               |
    ~                      AlgorithmIdentifier N                     ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
{: #ds-announce title="Hybrid Authentication Announcement"}

The announcement include a list algorithms could be used for hybrid signature

* Auth Method: A new value to be allocated by IANA
* Cert Link N: Links corresponding signature algorithm N with a particular CA. as defined in {{Section 3.2.2 of RFC9593}}
* AlgorithmIdentifier N: The variable-length ASN.1 object that is encoded using Distinguished Encoding Rules (DER) {{X.690}} and identifies the signature algorithm;

Receiver of this announcement could choose one PQC algorithm and one traditional algorithm from the list to create the hybrid signature.

Initiator also includes SUPPORTED_AUTH_METHODS notification in IKE_AUTH request message to announce its support of hybrid authentication.



## AUTH Payload

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


The Signature Value is created via following procedure:

1. Choose a hash algorithm H from peer's SIGNATURE_HASH_ALGORITHMS, a PQC algorithm Sig1 and a traditional algorithm Sig2 from peer's hybrid authentication method announcement;
2. Follow the procedure defined in {{Section 4.2 of I-D.ietf-lamps-pq-composite-sigs}}, where the domain separator is the value in {{Section 7.1 of I-D.ietf-lamps-pq-composite-sigs}} correspond to the combination of Sig1, Sig2 and H.


The domain separator is also used as AlgorithmIdentifier in the Authentication Data.

The corresponding certificates of Sig1 and Sig2 MUST be put in the first and second CERT payload, in the order, of the message;

## RelatedCertificate
The signing certificate MAY contain RelatedCertificate extension, then the receiver SHOULD verify the extension according to {{Section 4.2 of I-D.ietf-lamps-cert-binding-for-multi-auth}}, failed verification SHOULD fail authentication.

## Certificate with Composite Keys
So far, this document assumes the signing certificate contains a key of single algorithm, however there might be certificate with composite key as define in {{I-D.ietf-lamps-pq-composite-sigs}}, in such case, AUTH payload is created with same procedure in {{auth-payload}}:

* Sig1 and Sig2 are specified by the signing certificate

Supported composite signature algorithms are announced via SUPPORTED_AUTH_METHODS notification with 3-Octet announcement as defined in {{Section 3.2.2 of RFC9593}}.




# Security Considerations

The security of general PQ/T hybrid authentication is discussed in {{I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document uses mechanisms defined in {{RFC7427}} and {{RFC9593}}, the security discussion in the corresponding RFCs also apply.


# IANA Considerations

This document requests a value in "IKEv2 Authentication Method" subregistry under IANA "Internet Key Exchange Version 2 (IKEv2) Parameters" registry


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
