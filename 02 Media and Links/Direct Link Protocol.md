---
id: direct-link-protocol
title: "Direct Link Protocol"
aliases: ["DLP","GNET-LINK"]
type: protocol
status: mixed
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[32-bit Flit Format]]","[[Virtual Channels and VCIDs]]","[[DLP Segment Size Classes]]","[[GDP Protocol]]","[[ADR-0004 Direct Point-to-Point Link]]"]
updated: 2026-09-02
---
# Direct Link Protocol (DLP / GNET-LINK)

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[32-bit Flit Format]] · [[Virtual Channels and VCIDs]] · [[DLP Segment Size Classes]] · [[GDP Protocol]]

Status: **FROZEN role, 4-bit VCID, and size classes; DRAFT integrity encoding**

DLP is the common L2 envelope used by GNET-L, GNET-A, and GNET-P. It multiplexes bounded segments with a hop-local VCID, identifies the carried protocol, delimits the carried bitstream, and detects link corruption. The medium supplies physical transmitter, receiver, port, or premise-channel identity.

## Fundamental flit rule

Every transmitted GNet flit is exactly 32 bits. Bits 0–3 are the VCID and bits 4–31 carry 28 bits of header, payload, padding, or trailer data.

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                  Carried bits [27:0]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The VCID is part of the 32-bit flit. It is not sideband and does not create a 36-bit transfer. Consequently, a flit carries exactly 28 bits of protocol data.

## Draft segment encoding

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |   Protocol    |LC |SC |    Payload Length (octets)    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                 Payload bits 0 .. 27                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                Payload bits 28 .. 55                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                          ...                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | Last payload bits; trailing padding MUST be zero      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |     CRC-8     |             Reserved = 0              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The header uses 8 carried bits for **Protocol**, 2 for **Link Class** (LC), 2 for **Segment Class** (SC), and 16 for **Payload Length**. Payload Length is the exact number of meaningful octets in the carried protocol bitstream and excludes padding and the trailer.

Segment Class bounds the segment's meaningful payload length:

- `00` — **Small:** at most 64 octets.
- `01` — **Normal:** at most 256 octets.
- `10` — **Large:** at most 1,024 octets.
- `11` — reserved.

The size class bounds buffer reservation and VCID holding time; Payload Length gives the exact byte count, including a final segment shorter than its class limit. See [[DLP Segment Size Classes]].

For a payload of N octets, the number of payload flits is:

```text
K = ceil((8 × N) / 28) = ceil((2 × N) / 7)
```

The complete segment uses `K + 2` flits. The last payload flit has between zero and 27 trailing zero-padding bits. Protocol octets are packed most-significant bit first without inserting alignment between flits.

## VCID operation

- The first flit using an inactive VCID is its segment header.
- Every following payload and trailer flit repeats the same VCID.
- Flits belonging to different active VCIDs may be interleaved on a common link.
- Payload Length tells the receiver how many payload flits belong to the segment.
- Reception of the counted trailer completes the segment and releases the VCID for reuse.
- A VCID has meaning only on one link, in one direction, and—on a shared medium—within the physical transmitter or premise-channel context.
- A router terminates the incoming VCID and chooses a new VCID for the outgoing link.

The detailed allocation handshake, reserved VCID values, timeout/recovery behavior, relation to a persistent Link Flow ID, and Link Class meanings remain OPEN. See [[Virtual Channels and VCIDs]].

## Integrity boundary

The current trailer draft carries CRC-8 in the first eight carried bits and zero in the remaining 20. The CRC belongs to DLP, not GDP. Its polynomial, initialization, reflection, exact coverage of repeated VCID bits, and whether a full 28-bit or multi-flit check is preferable remain OPEN.

Preamble, synchronization, idle symbols, and request/grant signaling are physical-medium concerns and are not represented as DLP payload bits.

DLP has no universal MAC source or destination fields. Point-to-point wiring, a hub port, a GNET-A premise channel, or a GNET-P trunk supplies local delivery identity. Link-local fanout is an explicit control operation, never learned switching.
