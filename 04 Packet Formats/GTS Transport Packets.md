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

When a CONNECT or STREAM_OPEN selects a logical service, it carries the following logical fields:

```text
+----------------+----------------------------------+
| Size Class 4b  | Service ID 8/16/32/64/128 bits |
+----------------+----------------------------------+
```

The Service ID width is determined by Size Class:

| Size class | Service ID width |
|---:|---:|
| 0 | 8 bits |
| 1 | 16 bits |
| 2 | 32 bits |
| 3 | 64 bits |
| 4 | 128 bits |
| 5-15 | reserved |

The exact octet/bit placement is not yet frozen. The four-bit Size Class SHOULD be packed with another four-bit setup field where practical so that the class-0 case requires only 12 logical selector bits rather than forcing a standalone service-width byte.

The selector is setup-only. Once a service has been accepted and bound to tunnel/stream state, subsequent DATA packets use the Tunnel ID and Stream ID rather than repeating the Service Selector.

Large selector widths permit sparse allocation. This can make exhaustive service scanning impractical, but it is not cryptographic protection: predictable values remain guessable and a passive observer can learn selectors from unencrypted setup traffic.

Earlier CONNECT/CONNECT_ACK diagrams were exploratory and assumed obsolete 4-bit-VCID/28-carried-bit DLP packing. They are not current wire encodings and are intentionally not reproduced as normative diagrams here.

The next transport decision must define CONNECT, CONNECT_ACK, STREAM_OPEN, STREAM_ACCEPT, DATA, ACK, RESET, and CLOSE together with exact state machines, retransmission/congestion behavior, end-to-end integrity, and golden vectors. Physical carriage then uses the current baseline VC2 DLP format without transport-specific flit alignment.
