---
id: gts-transport-packets
title: "GTS Transport Packets"
aliases: ["GTS packets"]
type: packet
status: open
layers: ["L4","L5"]
tags: ["gnet","gnet/packet","gnet/status/open","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Packet Formats MOC]]"
related: ["[[GTS Protocol]]","[[ADR-0005 Tunnels and Streams]]","[[GDP Datagram]]"]
updated: 2026-09-09
---
# GTS transport packets

Status: **OPEN exact wire packing; service-selector semantics accepted**

The tunnel/stream model requires encodings for:

- `CONNECT` — propose Tunnel ID/reset authority/service/profile and initial transport flow control;
- `CONNECT_ACK` — accept and return negotiated state/handle;
- `STREAM_OPEN` / `STREAM_ACCEPT` — establish stream IDs, optionally select a service, and negotiate delivery profiles;
- `DATA` / `ACK` — carry stream data plus only profile-required sequence/ack/window state;
- `STREAM_CLOSE` — close one stream;
- `TUNNEL_CLOSE`, `RESET`, `REBIND` — destructive operations that prove reset authority.

Normal DATA MUST NOT contain Reset ID or Service Selector. Link `CREDIT`/`GRANT` are not GTS fields; they are GLCP/DLP hop-local state.

## Service Selector field

When a CONNECT or STREAM_OPEN selects a logical service, it carries a 2-bit Size Class followed by a class-specific selector field:

| Size class | Selector field | Representation |
|---:|---:|---|
| `00` | 8 bits | numeric registered service code |
| `01` | 32 bits | 4 ASCII characters |
| `10` | 128 bits | 16 ASCII characters |
| `11` | reserved | future expansion |

Logical layout:

```text
+--------------+------------------------------------+
| Size Class 2 | Selector: 8, 32, or 128 bits      |
+--------------+------------------------------------+
```

ASCII selector fields are fixed-width byte strings. A shorter name is terminated with a zero byte and the remaining bytes are zero padded. The exact permitted ASCII subset and case rules remain OPEN.

Examples:

```text
00 05
    -> registered service 5

01 "FILE"
    -> four-character service name

10 "oooooofilesy\0\0\0\0"
    -> 16-byte selector field containing a private name
```

The exact placement of the two Size Class bits in CONNECT or STREAM_OPEN is not yet frozen. They SHOULD share an existing setup byte with unrelated small flags where practical. The class-0 form therefore has only 10 logical selector bits.

The selector is setup-only. Once a service has been accepted and bound to tunnel/stream state, subsequent DATA packets use the Tunnel ID and Stream ID rather than repeating the Service Selector.

The 128-bit textual form allows a very sparse service namespace. Exhaustive scanning can therefore be impractical, but this is not cryptographic protection. Predictable names can still be dictionary-scanned, and a passive observer can learn selectors from unencrypted setup traffic.

Earlier CONNECT/CONNECT_ACK diagrams were exploratory and assumed obsolete 4-bit-VCID/28-carried-bit DLP packing. They are not current wire encodings and are intentionally not reproduced as normative diagrams here.

The next transport decision must define CONNECT, CONNECT_ACK, STREAM_OPEN, STREAM_ACCEPT, DATA, ACK, RESET, and CLOSE together with exact state machines, retransmission/congestion behavior, end-to-end integrity, and golden vectors. Physical carriage then uses the current baseline VC2 DLP format without transport-specific flit alignment.
