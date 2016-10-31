---
title: "ACME Identifiers and Challenges for Telephone Numbers"
abbrev: ACME for TNs
docname: draft-peterson-acme-telephone-latest
category: std
ipr: trust200902

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: J. Peterson
    name: Jon Peterson
    org: Neustar
    email: jon.peterson@neustar.biz

--- abstract

This document specifies identifiers and challenges required to enable the
Automated Certificate Management Environment (ACME) to issue certificate for
telephone numbers.

--- middle

# Introduction

ACME is a mechanism for automating certificate management on the Internet
{{!I-D.ietf-acme-acme}}. It enables administrative entities to prove effective
control over resources like domain names, and automates the process of
generating and issuing certificates.

The STIR problem statement {{?RFC7340}} identifies the need for Internet
credentials that can attest authority for telephone numbers in order to detect
impersonation, which is currently an enabler for common attacks associated with
illegal robocalling, voicemail hacking, and swatting. These credentials are used
to sign PASSporTs {{?I-D.ietf-stir-passport}}, which may be carried in using
protocols such as SIP {{?I-D.ietf-stir-rfc4474bis}} or delivered outside of the
signaling channel of call setup {{?I-D.rescorla-stir-fallback}}. Currently, the
only defined credentials for this purpose are the certificates specified in
{{!I-D.ietf-stir-certificates}}.

{{I-D.ietf-stir-certificates}} describes certificate extensions suitable for
associating telephone numbers with certificates. To help enable certificate
authorities to issue certificates with these extensions, this specification
defines extensions to ACME suitable to enable certificate authorities to
validate effective control of numbering resources and to issue corresponding
certificates.

However, note that the aim of the ACME challenges described in this document are
not to prove the assignment and delegation of resources in the telephone
network: it is instead to establish whether Internet-enabled entites have
effective control over the devices associated with those resources. These
credentials are not mutual exclusive with credentials delegated from national
authorities, and in fact for the purposes of a call set-up protocol like SIP,
there may be multiple attestations (for example, multiple SIP Identity header
fields) signed by different parties.

# Terminology

In this document, the key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" are to be interpreted as described in {{!RFC2119}}.

# Telephone Number Identifier Type {#ident}

In order to issue certificates for telephone numbers with ACME, a new ACME
identifier type for telephone numbers is required for use in ACME authorization
objects.  The baseline ACME specification only defines one type of identifier,
for a fully-qualified domain name ("dns"). This document thus defines a new ACME
identifier type for telephone numbers ("tn"). This represents a telephone
number, specifically a number of the type that is specified in the TN
Authorization List certificate extension of {{I-D.ietf-stir-certificates}}.

~~~~~
{
  "status": "valid",
  "expires": "2015-03-01T14:09:00Z",

  "identifier": {
    "type": "tn",
    "value": "2125551212"
  },

  "challenges": [
    {
      "type": "sms-01",
      "status": "valid",
      "validated": "2014-12-01T12:05:00Z",
      "keyAuthorization": "SXQe-2XODaDxNR...vb29HhjjLPSggwiE"
    }
  ]
}
~~~~~

# Challenges for Telephone Numbers {#challenges}

Proving that a device on the Internet has effective control over a telephone
number is not as easy as proving control over an Internet resources like a DNS
zone or a resource on the web. Issuing certificates for telephone numbers is
perhaps most closely analogous to certificates for email addresses: end user
control over an email address boils down to the capabilities to read and send
email associated with that address. While a user typically has control over an
email address for a long period of time, control over email addresses can change
when users leave companies or other institutions, and addresses may subsequently
end up in the control of another party. Moreover, while it is relatively easy to
spoof the sender of any email address, as it unfortunately is with telephone
numbers, it is harder to intercept traffic to a target email address or
telephone number.

The likely challenges for proving effective control over a telephone number
therefore rely largely on routing some kind of secret to the telephone number in
question and requesting that the receiving device play that secret back to the
ACME server. The Short Message Service (SMS) provides a key building block for
challenges because of its ability to route a secret addressed to a telephone
number to a user-controlled device. However, because of the diverse capabilities
of Internet-connected devices that control telephone numbers, an SMS could be
used in different ways for different challenges. Some devices will be able to
interrogate their operating system to learn their own telephone number, for
example, while others cannot. Some devices will be able to receive a text
message and suppress it from being rendered to the user, while others cannot.

Because the assignment of numbering resources can change over time,
demonstrations of effective control must be regularly refreshed -- though again,
because of the diverse capabilities of the devices involved, different schemes
for refreshing the challenge, ones that require less direct user supervision,
may be available to some devices and not others.

## Telephone Number Routability Validation {#route}

With basic telephone number routability validation, the client in an ACME
transaction proves its control over a telephone by proving that it can receive
traffic sent to that telephone number over the PSTN. The ACME server challenges
the client to dereference a URI containing a token that is sent to the client
over SMS. Typically that token will be embedded in a URL that the end user will
dereference in order to be guided to a web resource that will enable account
creation with the CA.


type (required, string):
: The string "sms-00"

token (required, string):
: A random value that uniquely identifies the challenge. This value MUST have at
least 128 bits of entropy, in order to prevent an attacker from guessing it. It
MUST NOT contain any characters outside the URL-safe Base64 alphabet and MUST
NOT contain any padding characters ("=").

~~~~~
{
  "type": "sms-00",
  "token": "evaGxfADs6pSRb2LAv9IZf17Dt3juxGJ-PCt92wr-oA"
}
~~~~~

A client responds to this challenge by dereferencing the URI chosen by the ACME
server. For this challenge, this may lead to an out-of-band requirement for the
client to create a user account through human interaction.

### Advanced Routability Validation {#suppress}

Future versions of this specification will explore ways to increase the
automation of the challenge process when the client device has an application
capable of creating ACME accounts and requesting certificates to be issued.

### Telephone Number Range Validation {#range}

Future versions of this specification will explore ways to validate bulk
allocations of telephone numbers such as those used by IP PBXs.

# Acknowledgments

We would like to thank you for your contributions to this problem statement and
framework.

# IANA Considerations

Future versions of this specification will include registrations for the ACME
Identifier type and ACME Challenge type registries here.

# Security Considerations

TBD
