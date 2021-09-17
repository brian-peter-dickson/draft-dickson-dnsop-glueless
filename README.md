# Introduction

DNS Security extensions (DNSSEC) are additions to the DNS protocol which provide data integrity and authenticity protections, but do not provide privacy.


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 [@RFC2119] [@!RFC8174]
when, and only when, they appear in all capitals, as shown here.

# Background

Use of DNSSEC requires upgrades to software for authorative servers, resolvers, and optionally clients, in order to benefit from these protections. It also requires that DNS operators actually sign their zones.

When a given zone is unsigned, those protections to the zone contents are not available.

FIXME

For example, querying for the NS records for "example.com", at the name servers for the "com" TLD, where the published com zone has "example.com NS ns1.example.net", is not protected against MITM attacks, even if the domain "example.net" (the domain serving records for "ns1.example.net") is DNSSEC signed.

# Recommendations

The following practice is RECOMMENDED for unsigned zones:

* Do not use in-bailiwick name server names for unsigned zones.
* Use out-of-zone names for the name servers for unsigned zones

Example:

    Do NOT do the following (delegations requiring glue):
    unsigned-zone.example NS ns1.unsigned-zone.example
    unsigned-zone.example NS ns2.unsigned-zone.example
    // glue
    ns1.unsigned-zone.example A (IP address)
    ns1.unsigned-zone.example AAAA (IP address)
    ns2.unsigned-zone.example A (IP address)
    ns2.unsigned-zone.example AAAA (IP address)

    Instead, do the following (glueless delegations):
    unsigned-zone.example NS ns1.nameserver-signed-zone.example
    unsigned-zone.example NS ns2.nameserver-signed-zone.example
    //
    // Delegation to signed zone containing name server names
    nameserver-signed-zone.example NS ns1.nameserver-signed-zone.example
    nameserver-signed-zone.example NS ns2.nameserver-signed-zone.example
    nameserver-signed-zone.example DS (DS record data)
    // glue records for this delegation
    ns1.nameserver-signed-zone.example A (IP address)
    ns1.nameserver-signed-zone.example A (IP address)
    ns2.nameserver-signed-zone.example AAAA (IP address)
    ns2.nameserver-signed-zone.example AAAA (IP address)

The following practice is RECOMMENDED (for signed name server name zones, i.e. large operators' zones):

* For name server name zones (zones containing data for name servers), use dedicated name server names for the zone itself
* Consider use of another zone for the dedicated name server names, to make the name server name zone itself fully glueless
* For this additional zone, also consider using a different name server _name_ for its delegation's exclusive use 
* Decoupling the respective NS names, ensures changes and updates to the zone that uses glue, don't affect any other zones
* Depending on parent zone policy (e.g. TLD database policy), renaming or renumbering name servers may affect delegations using them (NS entries)
* A single zone with non-reused NS names guarantees side effects of this sort are not possible
* Additional lookups are required on the initial reference to any NS in the main glueless zone
* Subsequent (new) queries for the IP addresses of glueless name servers only require single queries

Example:

    // Entries in the example TLD
    //
    // Same unsigned zone uses the same name servers
    // However, the name server is in its own glueless zone
    unsigned-zone.example NS ns1.nameserver-signed-zone.example
    unsigned-zone.example NS ns2.nameserver-signed-zone.example
    //
    nameserver-signed-zone.example NS ns1.separate-signed-zone.example
    nameserver-signed-zone.example NS ns2.separate-signed-zone.example
    nameserver-signed-zone.example DS (DS record data)
    //
    separate-signed-zone.example NS special-ns1.separate-signed-zone.example
    separate-signed-zone.example NS special-ns2.separate-signed-zone.example
    separate-signed-zone.example DS (DS record data)
    // glue for special-ns1 and -2
    // special-ns1 and -2 are used only for/by separate-signed-zone
    special-ns1.separate-signed-zone.example A (IP address)
    special-ns1.separate-signed-zone.example AAAA (IP address)
    special-ns2.separate-signed-zone.example A (IP address)
    special-ns2.separate-signed-zone.example AAAA (IP address)

    Zone file for nameserver-signed-zone:
    nameserver-signed-zone.example SOA (soa record data)
    // glueless NS are used
    nameserver-signed-zone.example NS ns1.separate-signed-zone.example
    nameserver-signed-zone.example NS ns2.separate-signed-zone.example
    // actual glueless address records for "real" name server names
    ns1.nameserver-signed-zone.example A (IP address)
    ns1.nameserver-signed-zone.example AAAA (IP address)
    ns2.nameserver-signed-zone.example A (IP address)
    ns2.nameserver-signed-zone.example AAAA (IP address)
    // etc etc etc

    Zone file for separate-signed-zone:
    separate-signed-zone.example SOA (soa record data)
    // This is the only non-glueless NS in use, matches glue record in parent
    separate-signed-zone.example NS special-ns1.separate-signed-zone.example
    separate-signed-zone.example NS special-ns2.separate-signed-zone.example
    special-ns1.separate-signed-zone.example A (IP address)
    special-ns1.separate-signed-zone.example AAAA (IP address)
    special-ns2.separate-signed-zone.example A (IP address)
    special-ns2.separate-signed-zone.example AAAA (IP address)
    // actual glueless address records for "real" name server name
    // "real" name server name is only used by nameserver-signed-zone
    ns1.separate-signed-zone.example A (IP address)
    ns1.separate-signed-zone.example AAAA (IP address)
    ns2.separate-signed-zone.example A (IP address)
    ns2.separate-signed-zone.example AAAA (IP address)



# Security Considerations

This guidance is not a substitute for use of DNSSEC for DNS domains.

This guidance is useful in preventing off-path attackers from poisoning DNS cache entries necessary for delegations.

However, an on-path attacker is still able to manipulate DNS responses sent over UDP or unencrypted TCP.

Use of an encrypted transport is one potential method of preventing MITM attacks (i.e. DNS over TLS from resolver to authoritative server, aka ADoT), but this is still less secure than use of DNSSEC.

# IANA Considerations

This document has no IANA actions.
