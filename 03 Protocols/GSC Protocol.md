---
id: gsc-protocol
title: "GSC Protocol"
aliases: ["GSC","GNet Session Control","GNet Session","GNet-Session"]
type: protocol
status: draft
layers: ["L7"]
tags: ["gnet","gnet/protocol","gnet/status/draft","gnet/layer/l7"]
parent: "[[Protocols MOC]]"
related: ["[[GSC Packet]]","[[GSC Message Registry]]","[[GTerm Protocol]]","[[Voice over GNet]]"]
updated: 2026-09-03
---
# GNet Session Control (GSC)

> [!info] Knowledge graph
> **Up:** [[Protocols MOC]] · **Related:** [[GSC Packet]] · [[GSC Message Registry]] · [[GTerm Protocol]] · [[Voice over GNet]]

Status: **DRAFT protocol; ACCEPTED signaling/media separation**

**GSC**, commonly described as **GNet Session** or **GNet-Session**, is the SIP-like control protocol for interactive sessions including terminal, voice, video and collaborative services.

Session/control services participate in setup, naming, policy, admission and accounting. Once accepted, media or application data should normally travel directly between endpoints through GNet rather than being hairpinned through the signaling server.

## Message set

| Message | Purpose |
|---|---|
| REGISTER | associate a person/service name with current devices/endpoints |
| LOOKUP | resolve a service name, person, or telephone number |
| INVITE | request a new session |
| OFFER | describe media, codecs, encryption, capacity and other resources |
| ALERT | report ringing or user notification |
| ACCEPT | accept and return media addresses/flow parameters |
| REJECT | refuse as busy, unavailable, unauthorized or unsupported |
| CANCEL | withdraw an unanswered invitation |
| RELEASE | end a session and release resources |
| UPDATE | change media, codec, keys or reservations |
| TRANSFER | redirect or transfer a participant |
| KEEPALIVE | maintain registration or session soft state |

GSC negotiation may cover encoding, packet interval, GDP addresses, flow identifiers, bandwidth/delay needs, optional media, security and charging policy.

## Voice

The normative voice-session architecture is described in [[Voice over GNet]]. GSC owns call/session signaling; it does not become the voice media stream itself.

For an internal GNet-to-GNet call, signaling may traverse directory/session services while accepted voice packets flow directly. Admission control confirms suitable realtime resources before ACCEPT.

External telephony terminates at a gateway. The gateway maps GSC to external signaling, converts packet voice to/from legacy analog/PCM forms where required, and maps external telephone numbers through directory/address-mapping services. External numbering/signaling is not part of GDP routing.

On RELEASE, reservations, gateway trunks and session security state are released and accounting is finalized outside the forwarding path.
