---
id: address-configuration-packets
title: "Address Configuration Packets"
aliases: ["ADDRESS_OFFER","ADDRESS_CLAIM"]
type: packet
status: draft
layers: ["L2"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l2"]
parent: "[[Packet Formats MOC]]"
related: ["[[GCTL Protocol]]","[[Discovery and Bootstrap]]","[[Addressing and Routing]]","[[32-bit Flit Format]]"]
updated: 2026-09-02
---
# Address-configuration packets

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[GCTL Protocol]] · [[Discovery and Bootstrap]] · [[Addressing and Routing]] · [[32-bit Flit Format]]

Status: **DRAFT**

These link-local GCTL messages run after router discovery. Responses are sent directly using physical port/link identity.

The diagrams show actual 32-bit payload flits when each message begins at a DLP payload boundary. Every row contains a 4-bit VCID and 28 carried bits. The DLP segment header and integrity trailer are omitted. Protocol fields form one continuous bitstream and receive no internal flit alignment.

## ADDRESS_OFFER

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | GCTL Version  | Message Type  |     Flags [15:4]      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | F low |             Transaction ID [31:8]             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |  TxID [7:0]   |        Router Address [63:44]         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                Router Address [43:16]                 |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |     Router Address [15:0]     |Prefix Address [63:52] |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                Prefix Address [51:24]                 |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |             Prefix Address [23:0]             | PL hi |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | PL lo |Min Suffix Bits|         Reserved = 0          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                    Lifetime [31:4]                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | L low |                  Padding = 0                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

`F low`, `PL hi/lo`, and `L low` denote the indicated low or high portions of Flags, Prefix Length, and Lifetime. Unused host bits in Offered Prefix MUST be zero. Minimum Suffix Bits MUST be at least eight for a customer service. The final 24 carried bits are DLP padding.

## ADDRESS_CLAIM

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | GCTL Version  | Message Type  |     Flags [15:4]      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | F low |             Transaction ID [31:8]             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |  TxID [7:0]   |       Candidate Address [63:44]       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |               Candidate Address [43:16]               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |   Candidate Address [15:0]    | Client Nonce [63:52]  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                 Client Nonce [51:24]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |              Client Nonce [23:0]              |  Pad  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The final four carried bits are DLP padding.

## ADDRESS_ACK / ADDRESS_NAK

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | GCTL Version  | Message Type  |     Result [15:4]     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | R low |             Transaction ID [31:8]             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |  TxID [7:0]   |       Candidate Address [63:44]       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |               Candidate Address [43:16]               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |   Candidate Address [15:0]    |   Lifetime [31:20]    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |            Lifetime [19:0]            | Retry [31:24] |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |              Retry Delay [23:0]               |  Pad  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

`R low` denotes the remaining Result/Reason bits. Retry Delay is zero on ADDRESS_ACK. The final four carried bits are DLP padding.

The exact collision table, lease persistence, multi-router coordination, renumbering, and authentication binding remain OPEN.
