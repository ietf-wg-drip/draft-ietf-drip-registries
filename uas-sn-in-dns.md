---
coding: utf-8

title: UAS Serial Numbers in DNS
abbrev: uas-sn-dns
docname: draft-wiethuechter-drip-uas-sn-dns-00
category: std

ipr: trust200902
area: Internet
wg: drip Working Group
kw: Internet-Draft
cat: std

stand_alone: yes
pi: [toc, sortrefs, symrefs, comments]

author:
-   ins: A. Wiethuechter
    name: Adam Wiethuechter
    org: AX Enterprize, LLC
    street: 4947 Commercial Drive
    city: Yorkville
    region: NY
    code: 13495
    country: USA
    email: adam.wiethuechter@axenterprize.com

normative:
    RFC9153: # drip-requirements
    RFC9374: # drip-det
    RFC9434: # drip-arch

informative:
    drip-auth: I-D.ietf-drip-auth

    CTA2063A:
        title: ANSI/CTA 2063-A Small Unmanned Aerial Systems Numbers
        author:
        org: "Consumer Technology Association (CTA)"
        target: https://shop.cta.tech/products/small-unmanned-aerial-systems-serial-numbers
        date: 2019-09

--- abstract

TODO

--- middle

# Introduction

TODO

## Supported Scenarios

> Author Note: with the proposal to remove solving the issue of putting Serial Numbers into DNS from this document, this section most likely is removed or merged with another. The only point to merge is #6.

1. UA using manufacturer generated Serial Number for UAS ID. No additional information provided.
2. UA using manufacturer generated Serial Number for UAS ID. Manufacturer using a DIME. Manufacturer MUST provided pointer to additional information via DNS (even if null).
3. UA using manufacturer generated Serial Number which is mapped to a DET by manufacturer for UAS ID. UA using manufacturer generated DET for Authentication. Manufacturer using a DIME. DIME MUST place public DET information into DNS (i.e. HI). DIME MUST provide mapping of Serial Number to DET in DNS. Manufacturer MUST provide pointer to additional information via DNS (even if null).
4. UA using manufacturer generated DRIP enhanced Serial Number for UAS ID. UA using manufacturer generated DET for Authentication. Manufacturer using a DIME. DIME MUST place public information into DNS (i.e. HI) - either directly or as a mapping to a DET. DIME MUST provide pointer to additional information via DNS (even if null).
5. UA using manufacturer generated Serial Number for UAS ID. UA using user generated DET for Authentication. User uses DIME with capability to publicly map Serial Number to a DET (via a USS). DIME MUST place public DET information into DNS (i.e. HI). DIME MUST provide mapping of Serial Number to DET in DNS. DIME MUST provide pointer to additional information via DNS (even if null).

# Terminology {#terminology}

## Required Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

## Additional Definitions

This document makes use of the terms (PII, USS, etc.) defined in {{RFC9153}}. Other terms (DIME, Endorsement, etc.) are from {{RFC9434}}, while others (RAA, HDA, etc.) are from {{RFC9374}}.

# Serial Number Registration

There are four ways a Serial Number is supported by DRIP:

1. As a clear-text string with additional information ({{serial-1}})
2. As a clear-text string mapped to a DET "post" generation by the **manufacturer** (for use in authentication) and additional information ({{serial-2}})
3. As a clear-text string mapped to a DET "post" generation by the **user (via an HDA)** (for use in authentication) and additional information ({{serial-3}})
4. As an encoding of an HI and associated DET by the **manufacturer** (for use in authentication) with additional information ({{serial-4}})

> Note: additional information here refers to any subset of keys defined in {{ua-info-registry}}.

## Serial Method 1 {#serial-1}

This is where a UA is provisioned with a Serial Number by the manufacturer. The Serial Number is just text string, defined by {{CTA2063A}}. The manufacturer runs an Name Server delegated under the Serial Number apex and points to information using a DET RR (filling in only the Serial Number and URI fields).

~~~~
    +-------------------+
    | Unmanned Aircraft |
    +--o---o------------+
       |   ^
   (a) |   | (b)
       |   |
*******|***|*****************************
*      |   |    DIME: MAA               *
*      |   |                            *
*      v   |             +----------+   *
*   +--o---o--+          |          |   *
*   |   DPA   o--------->o          |   *
*   +----o----+   (d)    |          |   *
*        |               |          |   *
*        | (c)           | DIA/RDDS |   *
*        v               |          |   *
*   +----o--------+      |          |   *
*   | Registry/NS |      |          |   *
*   +-------------+      |          |   *
*                        +----------+   *
*                                       *
*****************************************

(a) Serial Number,
    UA Information
(b) Success Code
(c) DET RR
(d) UA Information
~~~~
{:fig #dime-sn1-example title="Example DIME:MAA with Serial Number Registration"}

## Serial Method 2 {#serial-2}

This is where a UAS is provisioned with a Serial Number and DET by the manufacturer enabling their devices to use {{drip-auth}} and provide additional information. A public mapping of the Serial Number to DET and all public artifacts MUST be provided by the manufacturer. The manufacturer MUST use an MAA for this task.

The device MAY allow the DET to be regenerated dynamically with the MAA.

~~~~
    +-------------------+
    | Unmanned Aircraft |
    +--o---o------------+
       |   ^
   (a) |   | (b)
       |   |
*******|***|*****************************
*      |   |    DIME: MAA               *
*      |   |                            *
*      v   |             +----------+   *
*   +--o---o--+          |          |   *
*   |   DPA   o--------->o          |   *
*   +----o----+   (d)    |          |   *
*        |               |          |   *
*        | (c)           | DIA/RDDS |   *
*        v               |          |   *
*   +----o--------+      |          |   *
*   | Registry/NS |      |          |   *
*   +-------------+      |          |   *
*                        +----------+   *
*                                       *
*****************************************

(a) Serial Number,
    UA Information,
    Self-Endorsement: UA
(b) Success Code,
    Broadcast Endorsement: MAA on UA
(c) DET RR, PTR RR
(d) UA Information
~~~~
{:fig #dime-sn2-example title="Example DIME:MAA with Serial Number + DET Registration"}

## Serial Method 3 {#serial-3}

This is where a UAS has a Serial Number (from the manufacturer) and the user (via a DIME) has a mechanism to generate and map a DET to the Serial Number after production. This can provide dynamic signing keys for DRIP Authentication Messages via {{drip-auth}} for UAS that MUST fly only using Serial Numbers. Registration SHOULD be allowed to any relevant DIME that supports it. A public mapping of the DET to the Serial Number SHOULD be provided.

~~~~
    +-------------------+
    | Unmanned Aircraft |
    +--o---o------------+
       |   ^
   (a) |   | (b)
       |   |
*******|***|*****************************
*      |   |      DIME                  *
*      |   |                            *
*      v   |             +----------+   *
*   +--o---o--+          |          |   *
*   |   DPA   o--------->o          |   *
*   +----o----+   (d)    |          |   *
*        |               |          |   *
*        | (c)           | DIA/RDDS |   *
*        v               |          |   *
*   +----o--------+      |          |   *
*   | Registry/NS |      |          |   *
*   +-------------+      |          |   *
*                        +----------+   *
*                                       *
*****************************************

(a) Serial Number,
    UA Information,
    Self-Endorsement: UA
(b) Success Code,
    Broadcast Endorsement: DIME on UA
(c) DET RR
(d) UA Information
~~~~
{:fig #dime-sn3-example title="Example DIME with Serial Number + DET Registration"}

## Serial Method 4 {#serial-4}

This is where a UAS manufacturer chooses to use the Serial Number scheme defined in {{RFC9374}} to create Serial Numbers, their associated DETs for {{drip-auth}} and provide additional information. This document RECOMMENDEDS that the manufacturer "locks" the device from changing its authentication method so identifiers in both the Basic ID Message and Authentication Message do not de-sync. The manufacturer MUST use an MAA for this task, with the mapping between their Manufacturer Code and the upper portion of the DET publicly available.

~~~~
    +-------------------+
    | Unmanned Aircraft |
    +--o---o------------+
       |   ^
   (a) |   | (b)
       |   |
*******|***|*****************************
*      |   |    DIME: MAA               *
*      |   |                            *
*      v   |             +----------+   *
*   +--o---o--+          |          |   *
*   |   DPA   o--------->o          |   *
*   +----o----+   (d)    |          |   *
*        |               |          |   *
*        | (c)           | DIA/RDDS |   *
*        v               |          |   *
*   +----o--------+      |          |   *
*   | Registry/NS |      |          |   *
*   +-------------+      |          |   *
*                        +----------+   *
*                                       *
*****************************************

(a) Serial Number,
    UA Information,
    Self-Endorsement: UA
(b) Success Code,
    Broadcast Endorsement: MAA on UA
(c) DET RR
(d) UA Information
~~~~
{:fig #dime-sn4-example title="Example DIME:MAA with DRIP Serial Number Registration"}

# Serial Numbers in DNS

> Author Note: There MUST be an entry point in DNS for the lookup of UAS Serial Numbers. This section is very much a shot in the dark.

This document specifies the creation and delegation to ICAO of the subdomain `uas.icao.arpa`. To enable lookup of Serial Numbers a subdomains of `sn.uas.icao.arpa` is maintained. All entries under `sn.uas.icao.arpa` are to follow the convention found in {{sn-fqdn}}. This is to enable a singular lookup point for Serial Numbers for UAS.

Note that other subdomains under `uas.icao.arpa` can be made to support other identifiers in UAS. The creation and use of other such other subdomains are out of scope for this document. The further use and creation of items under `icao.arpa` is the authority of ICAO (which has been delegated control).

DETs MUST not have a subdomain in `uas.icao.arpa` (such as `det.uas.icao.arpa`) as they fit within the predefined `ip6.arpa` as they are IPv6 addresses.

# IANA Considerations

TODO

# Security Considerations

TODO

--- back

# UAS Serial Number FQDN {#sn-fqdn}

> {id}.{length}.{manufacturer-code}.{apex}.

~~~~
Apex: .sn.uas.icao.arpa.
Serial: MFR0ADR1P1SC00L
Manufacturer Code: MFR0
Length: A
ID: DR1P1SC00L
FQDN: dr1p1sc00l.a.mfr0.sn.uas.icao.arpa.
~~~~

# DNS Examples

## Serial Method 1 {#dns-s1}

~~~~text
@ORIGIN mfr0.uas-sn.arpa
example1.8 IN URI ( https://example.com/sn/EXAMPLE1 )
~~~~

## Serial Method 2 {#dns-s2}

~~~~text
@ORIGIN mfr0.uas-sn.arpa
example2.8 IN DET ( 5 20010033e872f705f3ce91124b677d65 ... "MFR MFR0" MFR08EXAMPLE2 https://example.com/sn/EXAMPLE2 ... active )

@ORIGIN 3.2.f.7.0.f.a.1.3.0.0.1.0.0.2.ip6.arpa
6.5.d.7.7.6.b.4.2.1.1.9.e.c.3.f.5.0 IN PTR example2.8.mfr0.uas-sn.arpa
~~~~

## Serial Method 3 {#dns-s3}

~~~~text
@ORIGIN mfr0.uas-sn.arpa
example3.8 IN DET ( 5 20010033e872f70584b1fa2b70421112 ... "MFR MFR0" MFR08EXAMPLE3 https://example.com/sn/EXAMPLE3 ... active )

@ORIGIN 3.2.f.7.0.f.a.1.3.0.0.1.0.0.2.ip6.arpa
2.1.1.1.2.4.0.7.b.2.a.f.1.b.4.8.5.0 IN PTR example3.8.mfr0.uas-sn.arpa
~~~~

## Serial Method 4 {#dns-s4}

~~~~text
@ORIGIN mfr0.uas-sn.arpa
example4.8 IN DET ( 5 20010033e872f705ba8af5252a35030e ... "MFR MFR0" MFR08EXAMPLE4 https://example.com/sn/EXAMPLE4 ... active )

@ORIGIN 3.2.f.7.0.f.a.1.3.0.0.1.0.0.2.ip6.arpa
e.0.3.0.5.3.a.2.5.2.5.f.a.8.a.b.5.0 IN PTR example4.8.mfr0.uas-sn.arpa
~~~~