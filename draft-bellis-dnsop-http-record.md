---
title: A DNS Resource Record for HTTP
docname: draft-bellis-dnsop-http-record

ipr: trust200902
area: Internet
wg: DNSOP Working Group
kw: Internet-Draft
cat: std

coding: utf-8
pi:
  - toc
  - symrefs
  - sortrefs

author:
  -
    ins: R. Bellis
    name: Ray Bellis
    org: Internet Systems Consortium, Inc.
    abbrev: ISC
    street: 950 Charter Street
    city: Redwood City
    code: CA 94063
    country: USA
    phone: +1 650 423 1200
    email: ray@isc.org

--- abstract

This document specifies an "HTTP" resource record type for the DNS to
facilitate the lookup of the server hostname of HTTP(s) URIs.  It is
intended to replace the use of CNAME records for this purpose, and in
the process provides a solution for the inability of the DNS to allow a
CNAME to be placed at the apex of a domain name.

--- middle

# Introduction

It is very common for HTTP(s) URIs to contain a domain name that is not
the same as the hostname of the actual server that hosts the content.

This is typically achieved via a CNAME record where the owner name of
that record (the "Alias") is the domain name from the URI and the
Canonical name field in its RDATA corresponds with the target hostname
(although it should be noted that this strictly a violation of the
original design semantics of the CNAME record).

It is also impossible to store a CNAME at the apex of a domain name,
which causes signficant difficulties if you wish to redirect your domain
name without a "www" prefix to a content delivery network (CDN).  The
only portable solution at the moment is to determine the IP address
records of the content host and insert them directly at the apex of the
zone, but this is brittle, and prevents the correct operation of typical
CDN features.

While there have been previous attempts to promote the use of the SRV
record instead of CNAME records, there have been concerns raised about
the performance impact of the additional DNS lookup an SRV record
would typically require.

To achieve equivalent end-user performance as existing CNAME-based
solutions, this document permits (but does not mandate) recursive
resolvers to pre-emptively look up the target of an HTTP Record and
return the corresponding records to the client.

Also, the presence of the Port field in an SRV record is incompatible
with the "Same Origin" security policy enforced by web browsers and in
practise the load-balancing / fallback capabilities of the SRV record
are not widely used either, and non-DNS based solutions for this are
already widely deployed for HTTP traffic.

This document therefore specifies a minimal "HTTP" resource record type
for the DNS to facilitate the redirection from the domain name portion
of an HTTP(s) URI to the server hostname and thence to A or AAAA
records.  It is specifically intended to replace the use of CNAME
records for this purpose, and in the process provides a solution for the
inability of the DNS to allow a CNAME to be placed at the apex of a
domain name.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP 14
{{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

# Description

The owner name of an HTTP RR is the domain name portion of an HTTP(s) URI.

The use of underscore label prefixes (e.g. _http._tcp) was considered,
but rejected since it prohibits the use of wildcard records which us a
valuable technique for offering per-customer domain prefixes without
requiring that every prefix be individually provisioned.

## Wire Format {#wire}

The RDATA of an HTTP RR is a domain name in uncompressed wire format.

## Presentation Format

The RDATA of an HTTP RR is presented as a domain name in standard
master file format.

## Server Operation

Recursive resolvers MAY on receiving a request for an HTTP record look
up the A and AAAA records for the target (either from cache, or via new
iterative queries) and include the results in the Additional Section of
the response.

If the recursive resolver is performing DNSSEC resolution but is unable
to validate the A or AAAA responses it MUST NOT include them in the
response unless the client has specified the +CD (checking disabled)
flag.

Where EDNS Client Subnet {{?RFC7871}} is configured on the resolver those
A and AAAA lookups MUST be performed as if the client had made those
queries directly to the resolver.

## Client Operation

HTTP clients supporting this specification MUST issue parallel DNS
requests for the A, AAAA and HTTP records for the domain portion of an
http: or https: URI.

If an HTTP record is returned, the client MUST either use the A and AAAA
records contained in the Additional Section of the response, or issue
further parallel requests for the A and AAAA records corresponding to
the domain name in the RDATA of the HTTP record and then use those
IP addresses to access the URI.

If the original A and AAAA lookups return IP addresses these MUST only
be used if no HTTP record is returned.

<< the above needs more text around timing, happy eyeballs, etc. >>

# Security Considerations {#security}

TBD

# Implementation status {#impstatus}

<< RFC Editor Note: Please remove this entire section prior to
publication as an RFC. >>

# Privacy Considerations

TBD (if any)

# IANA Considerations

<< a copy of the RFC 6895 IANA RR TYPE application template will appear
here >>

# Acknowledgements

--- back
