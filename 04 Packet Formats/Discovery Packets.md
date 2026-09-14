---
id: discovery-packets
title: "Discovery Packets"
aliases: ["SOLICIT and ADVERTISE"]
type: packet
status: frozen
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/frozen","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GCTL Protocol]]","[[Discovery and Bootstrap]]","[[Service Type Registry]]","[[ADR-0019 GCTL Credit and Bootstrap Wire Profile]]"]
updated: 2026-09-14
---
# Service discovery packets

Status: **FROZEN for GNet 0.1**

Discovery is generic: Router, Directory, Terminal Server, and later services use SOLICIT/ADVERTISE with different Service Type values.

These are GCTL messages carried through GDP on the normal data path. They are distinct from GLCP and any dedicated physical control pair.

All multi-byte values use network byte order. The first 8 bytes are the normal GCMP common header.

## SOLICIT

Common header:

```text
Version          1 byte
Message Type     1 byte = 0x01 SOLICIT
Code             1 byte
Flags            1 byte
Transaction ID   4 bytes
```

Body:

```text
Service Type      2 bytes
Scope             1 byte
Reserved          1 byte = 0
```

Initial Scope registry:

```text
0   directly attached / link scope
```

Other Scope values are reserved.

An unconfigured endpoint first establishes its link, then may send `SOLICIT(Router)` using its FE80/16 link-local source address.

Before the provider address is known, the destination is the reserved link-scoped bootstrap address:

```text
FE80:0000:0000:0000
```

This value MUST NOT be routed.

## ADVERTISE

Common header:

```text
Version          1 byte
Message Type     1 byte = 0x02 ADVERTISE
Code             1 byte
Flags            1 byte
Transaction ID   4 bytes
```

Body:

```text
Service Type      2 bytes
Preference        1 byte
Reserved          1 byte = 0
Provider Address  8 bytes
Lifetime          4 bytes
Capabilities      4 bytes
```

`ADVERTISE` echoes the request Transaction ID and Service Type.

`Service Type = 0x0001` identifies Router.

`Lifetime = 0` means the advertisement MUST NOT be cached beyond the immediate exchange.

The Provider Address is the GDP address to use for subsequent directed bootstrap/control exchange.

## Padding

The body occupies the beginning of the selected GDP/GCTL payload after the common header. Remaining bytes in the selected GDP Size Class are zero padding and MUST be transmitted as zero. Receivers MUST reject nonzero padding for these frozen typed messages.

## Router bootstrap sequence

For initial routed-address configuration:

```text
SOLICIT(Router)
    -> ADVERTISE(Router)
    -> ADDRESS_OFFER
    -> ADDRESS_CLAIM
    -> ADDRESS_ACK or ADDRESS_NAK
```

`ADVERTISE` and `ADDRESS_OFFER` may be emitted back-to-back for the same Transaction ID.
