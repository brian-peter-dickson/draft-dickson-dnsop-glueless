%%%
title = "Operating a Glueless DNS Authority Server"
abbrev = "Glueless DNS"
docName = "draft-dickson-dnsop-glueless"
category = "info"

[seriesInfo]
name = "Internet-Draft"
value = "draft-dickson-dnsop-glueless-01"
stream = "IETF"
status = "informational"


ipr = "trust200902"
area = "Operations"
workgroup = "DNSOP Working Group"
keyword = ["Internet-Draft"]

[pi]
toc = "yes"
sortrefs = "yes"
symrefs = "yes"
stand_alone = "yes"

[[author]]
initials = "B."
surname = "Dickson"
fullname = "Brian Dickson"
organization = "GoDaddy"
  [author.address]
  email = "brian.peter.dickson@gmail.com"

%%%

.# Abstract

This Internet Draft proposes a method for protecting authority servers against MITM and poisoning attacks, using a domain naming strategy to not require glue A/AAAA records and use of DNSSEC.

This technique assumes the use of validating resolvers.

MITM and poisoning attacks should only be effective/possible against unsigned domains.

However, until all domains are signed, this guidance is relevant, in that it can limit the attack surface of unsigned domains.

This guidance should be combined with [@?I-D.dickson-dnsop-ds-hack]

{mainmatter}
{{README.md}}
{backmatter}

# Acknowledgments

Thanks to everyone who helped create the tools that let everyone use Markdown to create 
Internet Drafts, and the RFC Editor for xml2rfc.

Thanks to Dan York for his Tutorial on using Markdown (specifically mmark) for writing IETF drafts.

Thanks to YOUR NAME HERE for contributions, reviews, etc.
