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
Minimum local suffix width/policy
Lifetime
```

## ADDRESS_CLAIM semantics

A client supplies at least:

```text
Transaction ID
Candidate address or delegated-suffix choice
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
