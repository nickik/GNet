---
id: gts-transport-packets
title: "GTS Transport Packets"
aliases: ["GTS packets"]
type: packet
status: open
layers: ["L4","L5"]
tags: ["gnet","gnet/packet","gnet/status/open","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Packet Formats MOC]]"
related: ["[[GTS Protocol]]","[[ADR-0005 Tunnels and Streams]]"]
updated: 2026-09-02
---
# GTS transport packets

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[GTS Protocol]] · [[ADR-0005 Tunnels and Streams]]


Status: **OPEN; historical field requirements preserved**

## Current semantic packet family

The tunnel/stream model requires encodings for:

- **CONNECT** proposes Tunnel ID, reset authority, service selector, security/profile, and initial flow-control values.
- **CONNECT_ACK** accepts and returns a local Tunnel Handle and negotiated values.
- **STREAM_OPEN** requests a Stream ID and delivery/security/compression profile.
- **STREAM_ACCEPT** accepts or modifies the stream profile.
- **DATA** carries Stream ID, data/fragment position, and only the state required by that profile.
- **ACK** acknowledges reliable/sequenced delivery and advertises flow-control state.
- **STREAM_CLOSE** ends one stream without destroying the tunnel.
- **TUNNEL_CLOSE, RESET, and REBIND** are destructive controls that MUST prove the Reset ID.

Normal DATA MUST NOT contain the Reset ID. Service names, ports, or service codes appear during CONNECT/STREAM_OPEN and are not repeated on DATA.

## Logical packing of the historical CONNECT proposal

The latest recorded field sequence totals 164 meaningful bits. The following historical diagram packs it into six logical 32-bit words with 28 final padding bits. These are not transmitted flits: actual DLP flits reserve four bits for VCID and carry only 28 bits of the continuous GDP/GTS bitstream.

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Type  | Rsvd  |  Recv Window  |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Receive Window (12)  |          Window Scale [23:4]          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | WS lo |                   Tunnel ID [63:36]                   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Tunnel ID [35:4]                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |TID lo |          Source Port          |    Checksum [15:4]    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Sum lo |                      Padding = 0                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

This diagram exposes rather than repairs the problems: two different Receive Window fields and several fields split across flit boundaries. It is **not** an accepted wire encoding.

When appended after the 160-bit GDP header, the meaningful GDP/CONNECT bitstream is 324 bits and requires 12 DLP payload flits, each containing a 4-bit VCID plus 28 carried bits. Its final carried region has 12 padding bits.

## Logical packing of the historical CONNECT_ACK proposal

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Type  |     Reserved (12)     |           Checksum            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Receive Window (12)  |          Window Scale [23:4]          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | WS lo |                   Tunnel ID [63:36]                   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Tunnel ID [35:4]                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |TID lo |                 Tunnel Handle [63:36]                 |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                     Tunnel Handle [35:4]                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | TH lo |                      Padding = 0                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

CONNECT_ACK contains 196 meaningful bits, occupies seven logical words, and needs 28 logical-word padding bits. When appended after GDP, the 356 meaningful bits require 13 actual DLP payload flits and eight final carried-region padding bits. It is **not** an accepted wire encoding.

Earlier requirements also called for 32-bit tunnel sequence and acknowledgment numbers. The next transport decision must define CONNECT, CONNECT_ACK, DATA, ACK, RESET, and CLOSE together, with exact state machines and test vectors, rather than repairing these layouts in isolation.
