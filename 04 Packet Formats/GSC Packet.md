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
updated: 2026-09-03
---
# Session-control messages

Status: **DRAFT logical envelope; message bodies OPEN**

GSC is carried above GDP. Its fields form a continuous logical bitstream; GSC does not define physical flit alignment.

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

These are logical words, not wire flits. The GDP/GSC bitstream is carried across the current baseline DLP flits (`2-bit VCID + 30 carried bits`) without requiring GSC-aligned flit boundaries.

Transaction ID supports repeatable requests and response matching. Dialog ID correlates an establishing/established session and is zero where no dialog exists.

DLP supplies hop-local integrity; GDP supplies no checksum. Any required end-to-end GSC/GTS integrity belongs above GDP.

## Required operations

REGISTER, LOOKUP, INVITE, OFFER, ALERT, ACCEPT, REJECT, CANCEL, RELEASE, UPDATE, TRANSFER, KEEPALIVE, RESERVE, and RESERVE_RESULT remain the required semantic family. Exact body encodings depend on the GTS identifier/security decisions.

Session signaling is not the media path: accepted media/data normally flows directly between endpoints, while external telephone numbering/signaling is handled by directory/session services and gateways rather than GDP routing.
