---
title: "The \"testing\" flag for Service Binding (SVCB) Records"
abbrev: "The SVCB \"testing\" flag"
category: std

docname: draft-manuben-svcb-testing-flag-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: Operations
# workgroup: DNSOP
venue:
  github: "bemasc/svcb-testing-flag"

author:
 -
    fullname: Benjamin M. Schwartz
    organization: Meta Platforms, Inc.
    email: ietf@bemasc.net
 -
    fullname: Manu Bretelle
    organization: Meta Platforms, Inc.
    email: chantr4@gmail.com

normative:

informative:


--- abstract

This draft defines a flag to mark a service endpoint as being potentially unreliable.  This flag may be useful when introducing new features that could have a negative impact on availability.

--- middle

# Introduction

The Service Binding (SVCB) DNS record type {{!RFC9460}}, and SVCB-compatible types like HTTPS, convey a collection of endpoints that can provide a service, along with metadata about each of those endpoints.  This metadata can indicate protocol features that are available and supported on those endpoints.

In most cases, advertising new features is unlikely to render the service unavailable.  Clients that are unaware of these features will ignore them, and clients that are aware will fall back to other SVCB records or other connection modes if the feature doesn't work.  However, for security-enhancing features, this fallback behavior would create a loss of security against an active attacker, so it is generally not allowed.  Instead, if the feature does not work as expected, the client will "fail closed".  This behavior can make it challenging to deploy security-enhancing features, as the initial public deployment can create an outage if the service is misconfigured.

This document defines a new SVCB SvcParam to help service operators offer new security features.  By marking these features as still being tested, the operator advises the client to interpret problems as an accidental failure by the operator, not a malicious action by an active attacker.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Specification

The "testing" flag is a SvcParam that always has an empty value in presentation and wire format.  When present, this flag indicates that this ServiceMode record is subject to outages, and clients SHOULD NOT interpret connection failures as evidence of an active attack.

Service owners SHOULD ensure that this flag is mandatory, either explicitly (by adding `mandatory=testing` to the SvcParams) or implicitly if this parameter is "automatically mandatory" for the protocol mapping.  Future protocol mappings SHOULD make this SvcParam "automatically mandatory".

## Interaction with SvcPriority

Clients SHOULD NOT alter the priority of SVCB records based on the presence of the "testing" flag.  Deprioritizing SVCB records with this flag would result in little or no user traffic making use of the testing record, which would defeat the goal of validating that new features function correctly for real users.

## Example: Encrypted DNS Protocol

Consider the case of a plaintext DNS server operator at "dns.example.com" who would like to announce support for DNS over TLS {{?RFC7858}}.  Per {{?RFC9461}}, this operator could publish a record like:

~~~
_dns.dns.example.com. SVCB 1 . alpn=dot
~~~

Clients following {{?RFC9461}} would retrieve this record, observe that DNS over TLS is available, and attempt to use it on TCP port 853.  If the TLS session cannot be established for any reason, a compliant client will not fall back to plaintext DNS on UDP port 53, because the failure could indicate an active attack ({{?RFC9461, Section 8.2}}).  If the operator of "dns.example.com" does not have operational confidence in their DNS over TLS service, this failure mode could raise concerns about the potential consequences of offering this new service.

To reduce the risk associated with this new service, the operator could instead use the new "testing" flag as follows:

~~~
_dns.dns.example.com. SVCB 1 . alpn=dot testing mandatory=testing
~~~

Clients that do not implement this specification will ignore the record because it specifies an unrecognized mandatory SvcParam.  They will continue to use plaintext DNS.  Clients that respect the "testing" flag will attempt to use DNS over TLS, but they will fall back to plaintext DNS if DNS over TLS is non-functional.

# Security Considerations

Use of the "testing" flag explicitly disables SVCB's defense against active attackers.  This is a loss in security.  However, the intent of this flag is to facilitate the deployment of security-enhancing protocols.  Downgrade-resistant security is achieved only when the testing period is complete and the "testing" flag is removed.

# IANA Considerations

IANA is requested to add this entry to the SVCB SvcParams Registry:

| Number | Name    | Meaning                    | Change Controller | Reference       |
| ------ | ------- | -------------------------- | ----------------- | --------------- |
| TBD    | testing | Endpoint may be unreliable | IETF              | (This document) |

--- back

# Acknowledgments
{:numbered="false"}

This proposal is inspired by deployment considerations related to {{?I-D.dnsop-deleg}}.
