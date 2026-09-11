---
id: adr-0018-gdp-header-crc-local16
title: "ADR-0018 GDP Header CRC and Local 16-bit Form"
aliases: ["Decision 0018","GDP header CRC","GDP local 16-bit form"]
type: decision
status: accepted
layers: ["L3"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l3"]
parent: "[[Decisions MOC]]"
related: ["[[GDP Protocol]]","[[GDP Datagram]]","[[ADR-0015 Restore Minimal GDP Header]]"]
updated: 2026-09-11
---
# Decision 0018: GDP header CRC and Local 16-bit form

Status: **ACCEPTED 2026-09-11**

This decision supersedes the no-GDP-integrity portion of [[ADR-0015 Restore Minimal GDP Header]]. GDP remains a minimal routed datagram protocol, but its fixed header now includes a small CRC protecting immutable header information that routers and endpoints must interpret safely.

## Address forms

- Global Source and Destination remain 64 bits each.
- Local Source and Destination change from 8 bits each to 16 bits each.
- Destination is serialized before Source in both forms.
- Mixed Local/Global source and destination forms remain unsupported.

The Local fixed header is exactly two 32-bit carried flits. The Global fixed header remains exactly five 32-bit carried flits.

## Common fields

GDP Version is 2 bits. Type remains 4 bits. Size Class remains 4 bits. Address Form remains 1 bit.

The initial Type registry remains:

```text
0x0       RESERVED
0x1       GCTL
0x2       GTS
0x3-0xF   RESERVED
```

## First flit

Global Flit 1 is:

```text
Bits 31..30   Version        2
Bits 29..26   Type           4
Bits 25..22   Size Class     4
Bit  21       Address Form   1
Bits 20..16   Reserved       5
Bits 15..8    CRC-8          8
Bits 7..0     Hop Limit      8
```

Local Flit 1 is:

```text
Bits 31..30   Version        2
Bits 29..26   Type           4
Bits 25..22   Size Class     4
Bit  21       Address Form   1
Bits 20..16   Reserved       5
Bits 15..8    CRC-8          8
Bits 7..4     Reserved       4
Bits 3..0     Hop Limit      4
```

The CRC therefore occupies bits 15..8, the third byte of the first carried 32-bit word, in both forms.

Local Flit 2 is:

```text
Bits 31..16   Destination ID   16
Bits 15..0    Source ID        16
```

Global Flits 2-3 carry Destination Address most-significant word first. Global Flits 4-5 carry Source Address most-significant word first.

## CRC-8

GDP reuses the existing `CRC-8-GNET` primitive rather than defining a second CRC algorithm:

```text
width       8
polynomial  0x07   (x^8 + x^2 + x + 1)
init        0x00
refin       false
refout      false
xorout      0x00
check       "123456789" -> 0xF4
```

CRC processing is most-significant bit first.

For GDP the CRC input is the canonical concatenation, with no padding, of:

```text
Version
Type
Size Class
Address Form
Destination
Source
```

Each field is processed most-significant bit first. Destination and Source use their transmitted width: 64 bits each in Global form and 16 bits each in Local form.

The CRC does not cover:

- the CRC field itself;
- Hop Limit;
- currently Reserved bits.

Hop Limit is deliberately excluded because routers modify it. A router may therefore decrement Hop Limit without recomputing the GDP CRC. Reserved bits are excluded in this revision; a later specification assigning semantics to a reserved bit may separately define whether that field becomes CRC protected.

## Integrity boundary

GDP CRC-8 is a header-integrity check only. It is not a payload checksum and does not provide end-to-end payload integrity. Payload corruption may be forwarded when the GDP header remains valid. Higher-layer protocols provide whatever payload integrity they require; GTS retains its own end-to-end CRC protection.

A GDP header CRC failure invalidates the entire GDP packet and the packet is not passed upward or routed as valid.

## Resynchronization after a bad header

The current DLP definition does not yet provide an independent packet boundary that remains trustworthy when GDP Size Class itself is corrupted. Therefore exact resynchronization behavior after an invalid GDP header is **not frozen by this ADR**. Implementations must not reinterpret following carried bits as a fresh GDP header merely because the preceding header failed CRC. The link/profile recovery rule that establishes the next trustworthy GDP boundary remains an explicit specification issue.

## DLP scope

This decision does not introduce periodic DLP CRC windows, CHECK/count blocks, negotiated CRC cadence, DLP transfer identifiers, CRC poisoning, or CRC-window ABORT propagation. Those explored mechanisms are not part of this decision.
