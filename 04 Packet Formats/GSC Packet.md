---
id: gsc-packet
title: "GSC Packet"
aliases: ["Session control packet"]
type: packet
status: draft
layers: ["L7"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l7"]
parent: "[[Packet Formats MOC]]"
related: ["[[GSC Protocol]]","[[GSC Message Registry]]","[[GDP Datagram]]"]
updated: 2026-09-06
---
# Session-control messages

Status: **DRAFT logical envelope; message bodies OPEN**

GSC is carried above GDP. Its fields are defined as logical 32-bit words; current GDP payload carriage uses 32-bit DLP data flits.

```text
    Word 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  GSC Version  | Message Type  |             Flags             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    Word 2
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Transaction ID                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    Word 3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         Dialog ID                             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The current 20-octet GDP header occupies exactly five 32-bit flits, so a GSC payload begins on a flit boundary. Each complete 32-bit GSC word can therefore occupy one data flit directly when carried in this layout. Baseline VC metadata is associated separately by DLP/PHY and is not part of the GSC word.

Transaction ID supports repeatable requests and response matching. Dialog ID correlates an establishing/established session and is zero where no dialog exists.

DLP supplies hop-local integrity; GDP supplies no checksum. Any required end-to-end GSC/GTS integrity belongs above GDP.

## Required operations

REGISTER, LOOKUP, INVITE, OFFER, ALERT, ACCEPT, REJECT, CANCEL, RELEASE, UPDATE, TRANSFER, KEEPALIVE, RESERVE, and RESERVE_RESULT remain the required semantic family. Exact body encodings depend on the GTS identifier/security decisions.

Session signaling is not the media path: accepted media/data normally flows directly between endpoints, while external telephone numbering/signaling is handled by directory/session services and gateways rather than GDP routing.
