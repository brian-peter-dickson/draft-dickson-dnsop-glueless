# Introduction

DNS Security Extensions (DNSSEC) are additions to the DNS protocol which provide data integrity and authenticity protections, but do not provide privacy.


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 [@RFC2119] [@!RFC8174]
when, and only when, they appear in all capitals, as shown here.

# Background

Use of DNSSEC requires upgrades to software for authorative servers, resolvers, and optionally clients, in order to benefit from these protections. It also requires that DNS operators actually sign their zones and secure the corresponding delegations at the parent.

When a given zone is unsigned or not securely delegated, those protections to the zone contents are not available.

Any such insecure zone is trivially able to be altered by an on-path attacker.

An off-path attacker is limited to use of cache poisoning attacks.

However, some class of cache poisoning attacks target unsigned delegation data. These records consist of the necessary NS records, and when necessary, "glue" records for IP addresses corresponding to these NS records.

The impact to successful cache poisoning of delegation records is that the attacker may substitute their own name servers for the legitimate name server. In other words, the attacker is able to promote itself to being effectively on-path, and trivially modify unsigned domain results.

# Proposed Solutions

This work does not propose any protocol changes. It provides guidance on strategies and techniques for name server naming.

There are two kinds of delegation records that require protection against off-path attackers, for unsigned domains.

For protecting NS records used in delegations, there is a new proposal for use of a new DS record. See [@I-D.dickson-dnsop-ds-hack] for details.

The present draft addresses the "glue" records, by recommending methods to make them mostly unnecessary. If there is no delegation glue data, an attacker cannot poison that data. The resolver cache would contain only authoritative address records associated with NS names. Authoritative data cannot be pre-empted by such poisoning attacks, since those are only able to replace less trusted glue records. 

Additional recommendations are made to reduce the chances for errors caused by DNS operators when changing delegation records, by avoiding re-use of name server names which require glue address records.

# Terminology:
The following terms are used to disambiguate zones and server names:

* Registered domain - end-user (registrant) domain
    * In the parent zone, the registered domain is the left-hand side of the NS record
* Registered domain name server - the name of the name server serving the registered domain
    * In the parent zone, the registered domain name server is the right-hand side of the NS record

# Recommendations

The following practice is RECOMMENDED for unsigned zones:

* Do not use in-bailiwick registered domain name servers for unsigned zones.
* Instead, use out-of-zone names for the registered domain name servers of unsigned zones.

Example:

    Do NOT do the following (delegations requiring glue):
    unsigned-zone.example NS ns1.unsigned-zone.example
    unsigned-zone.example NS ns2.unsigned-zone.example
    // glue
    // "strictly necessary glue"
    // always required for successful resolution
    ns1.unsigned-zone.example A (IP address)
    ns1.unsigned-zone.example AAAA (IP address)
    ns2.unsigned-zone.example A (IP address)
    ns2.unsigned-zone.example AAAA (IP address)

    Instead, do the following (glueless delegations):
    // This is the minimum "glueless" set-up
    // NS target name is not a "registered" host
    // NS target is not used for glue for any domains
    unsigned-zone.example NS ns1.nameserver-signed-zone.example
    unsigned-zone.example NS ns2.nameserver-signed-zone.example
    //
    // Delegation to signed zone containing name server names
    // (This zone serves the address records of name servers
    //  such as the glueless example above)
    nameserver-signed-zone.example NS ns1.nameserver-signed-zone.example
    nameserver-signed-zone.example NS ns2.nameserver-signed-zone.example
    nameserver-signed-zone.example DS (DS record data)
    // However, this zone needs to be resolvable, and needs glue
    // glue records for this delegation
    ns1.nameserver-signed-zone.example A (IP address)
    ns1.nameserver-signed-zone.example A (IP address)
    ns2.nameserver-signed-zone.example AAAA (IP address)
    ns2.nameserver-signed-zone.example AAAA (IP address)

The following practice is RECOMMENDED:

* For any name server domain (domain containing addresses and related data for name servers used by registered domains), use distinct dedicated name servers for the domain itself
    * I.e. avoid sharing name servers between the name server domain and any registered domains
* Consider making the name server domain itself fully glueless, with an out-of-zone name server (using a tertiary domain)
* For this tertiary domain, also consider using separating the in-bailiwick name servers, from the names used for serving the name server domain
* Decoupling the respective NS names ensures that changes and updates to the domain that uses glue don't affect any other domains
* Depending on parent zone policy (e.g. TLD database policy), renaming or renumbering name servers may affect delegations using them (NS entries)
* A single domain with non-reused NS names guarantees side effects of this sort are not possible
* Additional lookups are required on the initial reference to get the addresses of name servers for the main glueless domain
* Subsequent (new) queries for the IP addresses of glueless name servers only require single queries

Example:

    Entries in the example TLD
    //
    // Same unsigned zone uses the same name servers
    // However, the name server is in its own glueless zone
    unsigned-zone.example NS ns1.nameserver-signed-zone.example
    unsigned-zone.example NS ns2.nameserver-signed-zone.example
    //
    nameserver-signed-zone.example NS ns1.separate-zone.example
    nameserver-signed-zone.example NS ns2.separate-zone.example
    nameserver-signed-zone.example DS (DS record data)
    //
    separate-zone.example NS special-ns1.separate-zone.example
    separate-zone.example NS special-ns2.separate-zone.example
    separate-zone.example DS (DS record data)
    // glue for special-ns1 and -2
    // special-ns1 and -2 are used only for/by separate-zone
    special-ns1.separate-zone.example A (IP address)
    special-ns1.separate-zone.example AAAA (IP address)
    special-ns2.separate-zone.example A (IP address)
    special-ns2.separate-zone.example AAAA (IP address)

    Zone file for nameserver-signed-zone:
    nameserver-signed-zone.example SOA (soa record data)
    // glueless NS are used
    nameserver-signed-zone.example NS ns1.separate-zone.example
    nameserver-signed-zone.example NS ns2.separate-zone.example
    // actual glueless address records for "real" name server names
    ns1.nameserver-signed-zone.example A (IP address)
    ns1.nameserver-signed-zone.example AAAA (IP address)
    ns2.nameserver-signed-zone.example A (IP address)
    ns2.nameserver-signed-zone.example AAAA (IP address)
    // etc etc etc

    Zone file for separate-zone:
    separate-zone.example SOA (soa record data)
    // This is the only non-glueless NS in use
    // NB: matches glue in parent
    separate-zone.example NS special-ns1.separate-zone.example
    separate-zone.example NS special-ns2.separate-zone.example
    special-ns1.separate-zone.example A (IP address)
    special-ns1.separate-zone.example AAAA (IP address)
    special-ns2.separate-zone.example A (IP address)
    special-ns2.separate-zone.example AAAA (IP address)
    // actual address records for "real" name server name
    // (only used by nameserver-signed-zone)
    ns1.separate-zone.example A (IP address)
    ns1.separate-zone.example AAAA (IP address)
    ns2.separate-zone.example A (IP address)
    ns2.separate-zone.example AAAA (IP address)



# Security Considerations

This guidance is not a substitute for use of DNSSEC for DNS domains.

This guidance is useful in preventing off-path attackers from poisoning DNS cache entries necessary for delegations.

However, an on-path attacker is still able to manipulate DNS responses sent over UDP or unencrypted TCP.

Use of an encrypted transport is one potential method of preventing MITM attacks (i.e. DNS over TLS from resolver to authoritative server, aka ADoT), but this still does not provide the same security properties as DNSSEC (in particular that the zone contents are authorized by the DNSSEC key holder).

# IANA Considerations

This document has no IANA actions.
