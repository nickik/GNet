---
id: gsc-packet
title: "GSC Packet"
aliases: ["Session control packet"]
type: packet
status: draft
layers: ["L7"]
tags: ["gnet","gnet/packet","gnet/status/draft","gnet/layer/l7"]
parent: "[[Packet Formats MOC]]"
related: ["[[GSC Protocol]]","[[GSC Message Registry]]","[[GDP Datagram]]","[[32-bit Flit Format]]"]
updated: 2026-09-02
---
# Session-control messages

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[GSC Protocol]] · [[GSC Message Registry]] · [[GDP Datagram]] · [[32-bit Flit Format]]

Status: **DRAFT common header; message bodies OPEN**

## Logical GSC envelope

GSC defines the following continuous bitstream after the GDP header. These rows are logical 32-bit words, not transmitted flits.

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  GSC Version  | Message Type  |             Flags             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Transaction ID                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         Dialog ID                             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Message-specific body                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                              ...                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Actual GDP/GSC flit boundary

GSC is not aligned to a new DLP flit. The first GSC octet follows the 20-octet GDP header in the same carried bitstream. The transition and common GSC header are transmitted as follows; earlier GDP fields are shown in [[GDP Datagram]].

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |      Destination Address [19:0]       |  GSC Version  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | Message Type  |             Flags             | T hi  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                 Transaction ID [27:0]                 |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                   Dialog ID [31:4]                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  | D low |              Message body [0:23]              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                 Message body [24:51]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                          ...                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

`T hi` contains Transaction ID bits 31–28 and `D low` contains Dialog ID bits 3–0. Transaction ID makes requests repeatable and matches immediate responses. Dialog ID correlates messages belonging to an established or establishing session; it is zero when no dialog exists. DLP supplies the exact total octet length and its own hop-local integrity trailer. GSC/GTS must eventually define any required end-to-end integrity; GDP supplies none.

## Required operations

- **REGISTER:** endpoint to registrar; publish current device/service reachability.
- **LOOKUP:** endpoint to directory; resolve a name or number.
- **INVITE:** caller to service/peer; propose a session and requested capabilities/resources.
- **OFFER:** either endpoint to peer; describe media, codecs, security, and resources.
- **ALERT:** peer to caller; indicate that the target is being notified.
- **ACCEPT:** peer to caller; accept and return session parameters.
- **REJECT:** peer to caller; refuse with a reason.
- **CANCEL:** caller to peer; abandon an unanswered invitation.
- **RELEASE:** either endpoint; close an established session.
- **UPDATE:** either endpoint; renegotiate media, keys, or reservations.
- **TRANSFER:** endpoint/service to peer; redirect or transfer a participant.
- **KEEPALIVE:** either direction; maintain registration or session soft state.
- **RESERVE:** endpoint to network service; request bounded-delay/bandwidth resources.
- **RESERVE_RESULT:** network service to endpoint; accept, modify, or reject a reservation.

All requests need an idempotent transaction or invitation identifier above GDP. Session operations bind to endpoint addresses and, when present, authenticated identities. Encoding waits on the GTS identifier and security decisions.

Session signaling is not the media path. Once accepted, media/data packets normally flow directly between endpoints. Internal calls bypass a PSTN gateway; E.164 and SS7 are handled by directory/session services and external gateways rather than GDP routing.
