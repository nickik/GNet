---
id: glcp-control-flits
title: "GLCP Control Flits"
aliases: ["GLCP HELLO", "GLCP CAPABILITIES"]
type: packet
status: accepted
layers: ["L1", "L2"]
tags: ["gnet", "gnet/packet", "gnet/glcp", "gnet/status/accepted"]
parent: "[[Packet Formats MOC]]"
related: ["[[GNet Link Control Protocol]]", "[[34-bit Flit Format]]", "[[Minimum GNet-3 NIC]]"]
updated: 2026-09-06
---
# GLCP control flits

Status: **ACCEPTED GNet 0.1 bootstrap encoding**

GLCP uses its dedicated CONTROL-UP and CONTROL-DOWN pairs. A GLCP control flit is exactly 34 logical bits, sent most-significant bit first. It is not a DLP data flit: it has no VCID and does not consume DLP credit or grant state.

The electrical line code and serialization remain PHY work. The 34-bit logical control-flit boundary is normative for GNet 0.1.

## HELLO

`HELLO` establishes a current per-port link generation. The client MUST send `HELLO(initial)` as its first control flit after detecting link presence.

```text
33      30 29      26 25  24 23          18 17          12 11       0
+----------+----------+------+------+--------------+--------------+----------+
| opcode=1 | version=1| kind | SGEN |     PGEN     |   reserved   |
+----------+----------+------+------+--------------+--------------+----------+
     4 bits     4 bits  2 bits    6 bits       6 bits       12 bits
```

| Field | Bits | Meaning |
|---|---:|---|
| `opcode` | 4 | `0x1` for `HELLO`. |
| `version` | 4 | `0x1` for GNet 0.1 GLCP. |
| `kind` | 2 | `0=initial`, `1=acknowledgement`; `2–3` reserved. |
| `sgen` | 6 | Sender's link generation, incremented when that endpoint begins a new negotiation. |
| `pgen` | 6 | Peer's generation being acknowledged; zero only in `initial`. |
| `reserved` | 12 | Transmit zero; ignore on receipt. |

The required exchange is:

```text
client -> infrastructure  HELLO(initial, client-generation, 0)
infrastructure -> client  HELLO(ack, infrastructure-generation, client-generation)
client -> infrastructure  HELLO(ack, client-generation, infrastructure-generation)
```

On each later GLCP control flit, the receiver checks that the sender generation equals the peer generation established by this exchange on that physical port. A mismatched generation is stale and MUST be discarded. Generation filters stale port state; it is neither a global address nor authentication.

## CAPABILITIES

After the HELLO exchange, the client sends one `CAPABILITIES(offer)` control flit. The infrastructure chooses one supported profile and rate, and the client confirms that exact selection. No data flit, `REQUEST`, `CREDIT`, or `GRANT` is valid before confirmation.

```text
33      30 29      26 25          20 19  18 17      14 13    11 10       0
+----------+----------+--------------+------+------+----------+---------+----------+
| opcode=2 | version=1|     SGEN     | kind | profile  | rate  | reserved |
+----------+----------+--------------+------+------+----------+---------+----------+
     4 bits     4 bits       6 bits   2 bits   4 bits   3 bits    11 bits
```

| Field | Bits | Meaning |
|---|---:|---|
| `opcode` | 4 | `0x2` for `CAPABILITIES`. |
| `version` | 4 | `0x1` for GNet 0.1 GLCP. |
| `sgen` | 6 | Sender generation established by `HELLO`. |
| `kind` | 2 | `0=offer`, `1=selection`, `2=confirmation`, `3=reject/reserved`. |
| `profile` | 4 | Profile set in an offer; exactly one selected profile in selection/confirmation. |
| `rate` | 3 | Rate set in an offer; exactly one selected rate in selection/confirmation. |
| `reserved` | 11 | Transmit zero; ignore on receipt. |

Profile bits are `bit 0 = VC2` and `bit 1 = VC4`; bits `2–3` are reserved. VC2 is the sole 0.1 implementation profile. VC4 is a future profile and MUST NOT be offered by the current CGNet implementation.

Rate bits are `bit 0 = 0.75 Mbit/s`, `bit 1 = 1.5 Mbit/s`, and `bit 2 = 3 Mbit/s`. Minimum GNet-3 endpoints offer all three rates. Infrastructure selection MUST be a one-bit subset of both the client offer and its local port capability; client confirmation MUST repeat that selection exactly.

