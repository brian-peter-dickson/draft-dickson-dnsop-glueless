# Introduction

FIXME

There are new privacy goals and DNS server capability discovery goals, which cannot be met without the ability to validate the name of the name servers for a given domain at the delegation point.

Specifically, a query for NS records over an unprotected transport path returns results which do not have protection from tampering by an active on-path attacker, or against successful cache poisoning attackes.

This is true regardless of the DNSSEC status of the domain containing the authoritative information for the name servers for the queried domain.

For example, querying for the NS records for "example.com", at the name servers for the "com" TLD, where the published com zone has "example.com NS ns1.example.net", is not protected against MITM attacks, even if the domain "example.net" (the domain serving records for "ns1.example.net") is DNSSEC signed.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 [@RFC2119] [@!RFC8174]
when, and only when, they appear in all capitals, as shown here.

# Background

FIXME


# Security Considerations

This guidance is not a substitute for use of DNSSEC for DNS domains.

This guidance is useful in preventing off-path attackers from poisoning DNS cache entries necessary for delegations.

However, an on-path attacker is still able to manipulate DNS responses sent over UDP or unencrypted TCP.

Use of an encrypted transport is one potential method of preventing MITM attacks (i.e. DNS over TLS from resolver to authoritative server, aka ADoT), but this is still less secure than use of DNSSEC.

# IANA Considerations

This document has no IANA actions.
