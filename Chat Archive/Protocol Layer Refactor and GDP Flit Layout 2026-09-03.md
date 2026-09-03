# Protocol layer refactor and GDP flit layout — 2026-09-03

> [!important] Historical design checkpoint
> This note archives the decisions reached in the 2026-09-03 GNet design discussion. Canonical packet-format and protocol documents take precedence if later work supersedes this checkpoint.

## Layer split

The protocol suite is explicitly separated into three layers:

- **Layer 2 — Direct Link Protocol (DLP):** point-to-point framing and link-local flit transport. It has no network addresses or sessions. An 8-bit CRC is sufficient for link error detection; exact CRC encoding/polynomial remains a separate implementation detail.
- **Layer 3 — GDP (Global Data Protocol):** routed datagrams. GDP contains routing/addressing/QoS/flow metadata only. It does not contain transport sessions or reliability state.
- **Layer 4/5 — GNet:** connection/tunnel/stream protocol analogous in role to TCP, with reliability and flow-control semantics to be developed above GDP.

## 32-bit flit

Every transmitted flit is exactly 32 bits:

```text
[ VCID:2 | SOF:1 | carried payload:29 ]
```

The first carried bit of the **first flit only** is a one-bit **Frame Type** discriminator. Therefore the first flit has 28 bits remaining after Frame Type, while continuation flits retain all 29 carried bits.

Current Frame Type registry:

- `0` — GDP data frame
- `1` — Hello frame

Everything other than Hello is carried as GDP data and is defined above GDP rather than adding more DLP frame types.

## Hello frame

Hello is link-local, single-flit, contains no addresses, no interval, no routing information, and is never forwarded.

After `VCID | SOF | Frame Type=Hello`, the remaining 28 carried bits are:

```text
[ Type:4 | Version:4 | Reserved:12 | Checksum:8 ]
```

Reserved bits transmit as zero. The 8-bit Hello checksum is the final field in the flit.

## GDP header packing

GDP uses a six-flit fixed header. GDP addresses are **58 bits** so each address occupies exactly two 29-bit continuation flits.

```text
Flit 1:
[ VCID:2 | SOF:1 | FrameType=GDP:1 | Type:4 | Version:4 | SizeClass:4 | HopLimit:8 | QoS:8 ]

Flit 2:
[ VCID:2 | SOF:0 | HeaderChecksum:8 | FlowControlID:16 | Reserved:5 ]

Flit 3:
[ VCID:2 | SOF:0 | SourceAddress[57:29]:29 ]

Flit 4:
[ VCID:2 | SOF:0 | SourceAddress[28:0]:29 ]

Flit 5:
[ VCID:2 | SOF:0 | DestinationAddress[57:29]:29 ]

Flit 6:
[ VCID:2 | SOF:0 | DestinationAddress[28:0]:29 ]
```

GDP fields are therefore:

- Frame Type: 1 bit in first flit, DLP discriminator
- Type: 4 bits
- Version: 4 bits
- Size Class: 4 bits
- Hop Limit: 8 bits
- QoS: 8 bits
- Header Checksum: 8 bits
- Flow Control ID: 16 bits
- Source Address: 58 bits
- Destination Address: 58 bits

The GDP checksum protects the GDP header only. End-to-end payload integrity/reliability belongs to GNet or another protocol carried by GDP.

## GDP payload size classes

The four-bit Size Class selects one of sixteen fixed GDP payload budgets:

| ID | Name | Bytes |
|---:|---|---:|
| 0 | empty | 0 |
| 1 | tiny3B | 3 |
| 2 | ctrl32B | 32 |
| 3 | ctrl64B | 64 |
| 4 | msg128B | 128 |
| 5 | msg256B | 256 |
| 6 | medium512B | 512 |
| 7 | bulk1K | 1024 |
| 8 | legacyet | 1500 |
| 9 | xmtu2K | 2048 |
| 10 | jumbo8K | 8192 |
| 11 | ultra16K | 16384 |
| 12 | mega32K | 32768 |
| 13 | giga64K | 65536 |
| 14 | jumbogram256K | 262144 |
| 15 | jumbogram1M | 1048576 |

Class 1 exists specifically to provide the smallest GDP payload class, approximately one additional carried-data flit. Larger classes support progressively more efficient bulk transfer up through ultra-jumbo/jumbogram operation.

## GNet transport direction

GNet is no longer embedded in GDP. It will establish a tunnel/connection above GDP. The current design direction separates a long-lived tunnel identity from individual connections/ports and uses explicit connection-establishment packets, receive-window information, and higher-layer integrity. Detailed retransmission and congestion-control behavior remains to be finalized.
