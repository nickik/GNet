---
id: discovery-packets
title: "Discovery Packets"
aliases: ["SOLICIT and ADVERTISE"]
type: packet
status: draft
layers: ["L2","L3"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l2","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GCTL Protocol]]","[[Discovery and Bootstrap]]","[[Service Type Registry]]","[[32-bit Flit Format]]"]
updated: 2026-09-02
---
# Service discovery packets

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[GCTL Protocol]] · [[Discovery and Bootstrap]] · [[Service Type Registry]] · [[32-bit Flit Format]]

Status: **DRAFT**

Discovery is generic: Router, Directory, Terminal Server, and later services use the same SOLICIT/ADVERTISE messages with different Service Type values.

The following diagrams show actual 32-bit flits when the GCTL message begins directly at a DLP payload boundary. Every row contains the four-bit VCID and 28 carried bits. The DLP segment header and integrity trailer are not repeated in each diagram. When GCTL is carried inside GDP, its logical bitstream follows the GDP header and is sliced continuously rather than realigned.

## SOLICIT

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | GCTL Version  | Message Type  |     Flags [15:4]      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | F low |             Transaction ID [31:8]             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |  TxID [7:0]   |         Service Type          | S hi  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | S lo  | Reserved = 0  |          Padding = 0          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

`F low` denotes Flags bits 3–0; `S hi/lo` divide the eight-bit Scope. The final 16 carried bits are DLP padding and are not part of GCTL.

GCTL Version is draft value 1. Message Type is SOLICIT. Flags are zero until allocated. Transaction ID is a random request value. Scope values are LINK=0, ROUTER_DOMAIN=1, DISTRICT=2, and METRO=3.

An unconfigured endpoint may send only `SOLICIT(Router, LINK)` using DLP link-local GCTL. A configured endpoint sends other solicitations as GDP/GCTL. Intermediaries must not expand the requested scope.

## ADVERTISE

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | GCTL Version  | Message Type  |     Flags [15:4]      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | F low |             Transaction ID [31:8]             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |  TxID [7:0]   |         Service Type          | P hi  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |   Preference [11:0]   |   Provider Address [63:48]    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |               Provider Address [47:20]                |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |        Provider Address [19:0]        | Life [31:24]  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                Lifetime [23:0]                | C hi  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                  Capabilities [27:0]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Message Type is ADVERTISE. Transaction ID and Service Type echo SOLICIT. Larger Preference values are preferred unless policy overrides them. Provider Address may be zero only for a router advertisement returned by physical-port identity before GDP configuration. Lifetime zero means do not cache. Capabilities is service-specific.

`P hi` contains Preference bits 15–12 and `C hi` contains Capabilities bits 31–28.

The logical SOLICIT message is 12 octets and occupies four VCID-tagged payload flits with 16 padding bits. ADVERTISE is 28 octets and occupies exactly eight payload flits. These layouts remain DRAFT.
