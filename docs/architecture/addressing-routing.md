# Addressing and routing

Status: **FROZEN principles; OPEN bit allocation and routing protocol**

## Address model

A GDP address is an unsigned 64-bit global address. Prefixes aggregate administratively and geographically. The working hierarchy is:

`Region / Metro / District / Neighbourhood-or-Facility / Customer / Device`

This is a semantic hierarchy, not yet a frozen bit partition. Variable prefix lengths allow organizations, campuses, households, and mobile providers to receive appropriately sized blocks. Every retail customer MUST receive a prefix leaving at least eight device bits.

Zero is reserved for an unconfigured source. All-ones values are not defined as global broadcast addresses.

## Local configuration

1. A device discovers a router without requiring a GDP address.
2. The router advertises a customer/link prefix and minimum suffix width.
3. The device chooses a random suffix, normally at least eight bits.
4. The device sends ADDRESS_CLAIM directly to the router.
5. The router accepts the address or rejects a collision and supplies configuration lifetime information.

Physical port identity is useful input to policy, but is not a globally visible MAC address.

## Routing

Forwarding uses longest/deepest prefix match. A route identifies an egress link, next router, cost, allowed service classes, and validity. Horizontal peering is permitted at every hierarchy level; a parent/top-level router is fallback, not a mandatory transit point.

The proposed inter-domain protocol, provisionally **G-RIB/GNet-PGP**, advertises reachable prefixes, policy/cost, and a path or equivalent loop-prevention value. Its wire protocol and convergence rules remain OPEN.

## Mobility

Identity and current routing location must be separable enough to support roaming. Whether this uses stable endpoint addresses, home registrars, temporary care-of addresses, or directory/session indirection remains OPEN. NAT is not an acceptable mobility mechanism.
