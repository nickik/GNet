---
id: voice-over-gnet
title: "Voice over GNet"
aliases: ["VoG","GNet Voice","packet voice"]
type: protocol-profile
status: draft
layers: ["L4","L5","L7"]
tags: ["gnet","gnet/protocol","gnet/voice","gnet/status/draft"]
parent: "[[Protocols MOC]]"
related: ["[[GSC Protocol]]","[[GTS Protocol]]","[[GDP Protocol]]","[[GNet Coupler]]","[[GNet Switch]]"]
updated: 2026-09-03
---
# Voice over GNet

Status: **ACCEPTED architecture; DRAFT detailed media profile**

Voice over GNet makes interactive voice an ordinary native GNet service rather than requiring a parallel circuit-switched/TDM network.

## Architectural split

```text
Directory / identity
        |
GSC / GNet Session
registration, lookup, invite, alert,
offer, accept, transfer, release
        |
        +------------------------------+
                                       |
                              accepted realtime flow
                                       |
Endpoint A ======================== Endpoint B
          packet voice media
```

- **GSC** owns session/call signaling and media negotiation.
- **GTS/GNet flow facilities** own end-to-end stream/reservation behavior where used.
- **GDP** routes media packets.
- **GC/GS/GCT and other links** schedule admitted realtime traffic using their normal GNet mechanisms.
- Voice does not introduce a second address space or switching fabric.

## Session establishment

A voice session normally follows:

```text
REGISTER / LOOKUP
INVITE
OFFER      codec + packet interval + resource needs
ALERT
ACCEPT     selected media parameters + endpoint/flow information
<direct media>
UPDATE / TRANSFER as needed
RELEASE
```

Exact encodings remain under GSC/GTS specifications.

## Realtime service

Voice traffic requires bounded delay and low jitter. Endpoints request appropriate realtime service during session establishment; the network admits/reserves only what it can support. Local media links use existing GNet REALTIME scheduling rather than a voice-specific MAC.

REALTIME priority does not mean unlimited priority or permission for bulk traffic. Admission control and bounded packetization are required.

## Media

The media profile must support negotiated codec/encoding identifiers, sample/packet interval, sequence/timing information sufficient for playout, loss handling and optional security. Exact first-generation codecs and packet-header encoding remain OPEN.

The architecture permits programmable endpoints to perform codec, filtering, gain, tone and echo-related processing without exposing implementation details to GNet.

## Gateways

Legacy analog telephone lines, PBXs, PCM trunks and public telephone networks terminate at gateways:

```text
legacy telephone network
        |
      gateway
 signaling/media conversion
        |
 GSC + Voice over GNet
        |
      GNet
```

Gateways convert external signaling/numbering/media into GNet session and media semantics. They do not make legacy circuit identifiers part of GDP addressing.

## Product neutrality

This specification defines network behavior only. It does **not** define particular telephones, computers, PBXs, audio cards, switch chassis or vendor products. Product strategy belongs outside the canonical GNet repository.
