# GNet Name Service, DNS Alternatives, and RSA Licensing — 2025-04-08

> [!important] Historical design discussion
> This file archives an exploratory GNet discussion about a global name service and the possible use of public-key cryptography. It is **not a normative GNet specification** and does not change current protocol documents. Historical, cryptographic, patent, and licensing statements in the discussion should be independently verified before being used for engineering or legal decisions.

## Discussion summary

The discussion explored what a GNet-native alternative to DNS might look like, with the requirement that it scale to a global GNet rather than only to LANs.

### Naming-system direction explored

- DNS was used as the main comparison point. Weaknesses discussed included its hierarchical administrative model, security being added after the original design, privacy leakage, cache/TTL behavior, amplification/DDoS exposure, and awkwardness for highly dynamic or mobile environments.
- Modern alternatives and experiments mentioned for inspiration included GNUnet GNS, Tor `.onion` naming, IPFS/IPNS, Namecoin, ENS, mDNS/Bonjour, LISP-style locator/identifier separation, Secure Scuttlebutt, and related decentralized naming ideas.
- A first proposed architecture used federated zones, signed records, caching, and a DHT. This was then challenged on historical grounds because modern DHTs such as Chord, Pastry, Tapestry, and Kademlia are products of the early 2000s, not the early-1980s GNet design period.
- For an historically plausible 1980s system, the discussion moved toward **federated authoritative zones plus directory servers and caches**, with optional static hash partitioning or catalog replication rather than a modern self-organizing DHT.

### Who could publish zones

The exploratory model allowed zones to be operated by many kinds of organizations rather than only governments or a single central registry. Likely publishers included:

- companies;
- universities and research institutions;
- governments and public agencies;
- network/backbone operators;
- communities and associations;
- potentially individuals.

A zone would contain names and service/address records controlled by that publisher. The intention was to separate global naming from the physical or administrative topology of the network.

### How zones could be discovered

Several complementary mechanisms were discussed:

1. **Zone Directory Servers (ZDS):** distributed catalog servers mapping zone names to authoritative GNet servers, keys, and validity information.
2. **Mirrored zone catalogs:** signed or replicated catalogs that network operators, universities, companies, or service providers could mirror.
3. **Bootstrap hints:** clients or local resolvers ship with a small set of known directory/catalog servers, analogous to bootstrap configuration rather than a complete global host table.
4. **Caching:** local and regional resolvers retain successful zone and record lookups, reducing global query traffic.
5. **Zone advertisements/registration:** a possible mechanism for zone operators to submit or announce their zone to directory services. This was exploratory and not specified.

The resulting resolution path would be roughly:

`human name -> determine zone -> find authoritative zone service through a directory/cache -> query authoritative service -> verify result -> cache it`

This provides hierarchy for **lookup scaling and delegation** without necessarily making a single administrative root the sole source of trust.

### Public-key cryptography

The discussion then considered using public-key signatures for zones and records so that cached or mirrored data could be verified independently.

Algorithms and historical availability discussed included:

- Diffie–Hellman key exchange (1976);
- RSA (published 1977);
- ElGamal (1985);
- elliptic-curve cryptography as a later/less practical 1980s possibility.

RSA was viewed as especially attractive for an early GNet name system because signed zone information would make replication and caching much safer: a directory or cache would not need to be fully trusted to modify the data if the client could verify the zone's signature.

### RSA patent/licensing issue

A substantial part of the discussion concerned the U.S. RSA patent and whether an open GNet protocol could rely on RSA.

The main strategic idea explored was an **umbrella or protocol-wide license**: the GNet developer or sponsor would negotiate rights broad enough that *any compliant implementation of official GNet protocols* could use RSA, including independent commercial hardware and software vendors, rather than forcing every GNet implementer to negotiate separately.

The discussion considered two possible negotiation periods:

- **Before RSA Data Security existed/commercial licensing was established:** approach MIT directly while the RSA patent application was pending and negotiate broad rights for GNet implementations.
- **Later:** negotiate with the party controlling commercial RSA licensing for a standards-wide license.

The desired license concept would be:

- implementation-neutral;
- usable by third parties, not just the original GNet developer;
- applicable to software and hardware implementations;
- broad enough to cover official GNet naming, authentication, routing/security, and session protocols;
- valid for the relevant life of the U.S. patent;
- preferably royalty-free to downstream GNet implementers, with the original sponsor paying any negotiated consideration.

No decision was made that such a license actually existed or that MIT/RSA would certainly have accepted these terms. The discussion established it as a **strategic possibility worth historical/legal research**.

### Important historical constraint

A key design lesson from the discussion is that GNet should not import later Internet mechanisms merely because they are elegant. In particular, a modern DHT is anachronistic for an early-1980s launch. The GNet naming architecture should be constructed from mechanisms plausible at that time: delegation, replicated directories, caching, authoritative servers, compact queries, and—if licensing permits—public-key signatures.

## Design ideas to revisit later

The discussion deliberately stopped before turning these ideas into a current specification. Items left for future work include:

- exact global namespace syntax and whether names are strictly hierarchical, zone-qualified, or support several naming contexts;
- who is permitted to claim a human-readable top-level zone name and how collisions/disputes are handled;
- whether GNet has one global zone catalog, several competing/federated catalogs, or a trust-anchor model;
- how zone catalogs are replicated and incrementally updated with 1980s hardware and bandwidth limits;
- whether local resolvers can return signed cached answers without contacting the authoritative server;
- key sizes, signature formats, hash functions, rollover, revocation, and recovery for compromised zone keys;
- whether cryptographic naming is mandatory from the first release or added as an extension;
- historical verification of MIT/RSA licensing ownership, patent dates, licensing practices, and whether a standards-wide downstream implementation license would have been commercially realistic.

## Archived discussion notes

The original discussion began by asking what an alternative to DNS for GNet should look like and what existing systems could provide inspiration. A decentralized/federated naming system was proposed, initially including self-authenticating names, signed records, DHT-based lookup, and GNet-native transport.

The DHT proposal was then rejected as historically inappropriate for an early-1980s GNet. The conversation identified early-2000s systems such as Chord, Kademlia, Pastry, and Tapestry as the point when modern distributed hash tables became established. Plausible earlier substitutes were static or replicated directories, hierarchical delegation, broadcast only within local domains, and caching.

The discussion then asked who would actually publish a zone. The proposed answer was that organizations such as companies, governments, universities, backbone operators, communities, and possibly individuals could operate authoritative zones. Discovery would rely on Zone Directory Servers, signed/mirrored zone catalogs, bootstrap hints, and caching. A client looking up a name such as a company/service pair would first find the zone authority and then resolve the record from that authority or a verified cache.

Public-key signatures were introduced as a way to reduce the amount of trust placed in directory mirrors and caches. RSA was considered because it was published in the late 1970s and could in principle be implemented on contemporary systems, albeit with significant computational cost compared with later machines. Other public-key work such as Diffie–Hellman and later ElGamal was also noted.

The conversation then shifted to the U.S. RSA patent. It examined the distinction between source-code availability and patent rights: even independently written or openly distributed code can implement a patented method, so an open implementation does not itself remove patent risk in a jurisdiction where the patent applies.

Finally, the discussion considered an alternate-history GNet strategy in which the GNet sponsor negotiates **very early** with MIT for broad permission to use RSA in the protocol suite. The strongest desired version would not merely license one DEC/GNet product; it would explicitly cover third-party implementations of official GNet protocols so outside companies could build interoperable GNet routers, hosts, name servers, and software without obtaining their own RSA license for those protocol uses.

This remains an exploratory historical/legal strategy, not a committed GNet architecture or licensing fact.
