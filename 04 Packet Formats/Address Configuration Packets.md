---
id: address-configuration-packets
title: "Address Configuration Packets"
aliases: ["ADDRESS_OFFER","ADDRESS_CLAIM"]
type: packet
status: draft
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GCTL Protocol]]","[[Discovery and Bootstrap]]","[[Addressing and Routing]]"]
updated: 2026-09-03
---
# Address-configuration packets

Status: **DRAFT — semantics retained; old direct-DLP flit packing superseded**

Address configuration occurs after GLCP link establishment and router discovery. GCTL carries the network-level offer/claim/result exchange using the provisional/link-local GDP bootstrap rules.

## ADDRESS_OFFER semantics

A router supplies at least:

```text
Transaction ID
Router identity/address
Delegated/offered prefix
Prefix length
Candidate client address within that prefix
Lifetime
```

For the accepted GNet 0.1 on-link prefix profile, Prefix Length is one of `/16`, `/32`, `/48`, or `/56`. The prefix MUST be normalized (all suffix bits zero), and the candidate address MUST match it. The remaining bits are router-managed endpoint space; there is no endpoint subnet delegation or separate local-suffix policy in 0.1.

## ADDRESS_CLAIM semantics

A client supplies at least:

```text
Transaction ID
Candidate address supplied in ADDRESS_OFFER
Client nonce/claim identifier
```

## ADDRESS_ACK / ADDRESS_NAK semantics

The router returns:

```text
Transaction ID
candidate/confirmed address
result/reason
lifetime
retry delay when applicable
```

Physical port identity may bind the transaction to the directly attached client but is not a global MAC address.

The earlier 4-bit-VCID/28-carried-bit direct-DLP diagrams are obsolete. Exact GCTL widths, collision policy, lease persistence, multi-router coordination, renumbering, and authentication binding remain OPEN.
