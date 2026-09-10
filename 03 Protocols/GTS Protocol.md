---
id: gts-protocol
title: "GTS Protocol"
aliases: ["GTS","GNet Transport and Session Protocol"]
type: protocol
status: open
layers: ["L4","L5"]
tags: ["gnet","gnet/protocol","gnet/status/open","gnet/layer/l4","gnet/layer/l5"]
parent: "[[Protocols MOC]]"
related: ["[[GTS Transport Packets]]","[[Transport and Flows]]","[[ADR-0005 Tunnels and Streams]]","[[ADR-0009 No GDP Integrity Field]]"]
updated: 2026-09-10
---
# GNet Transport and Session Protocol (GTS)

> [!info] Knowledge graph
> **Up:** [[Protocols MOC]] · **Related:** [[GTS Transport Packets]] · [[Transport and Flows]] · [[ADR-0005 Tunnels and Streams]] · [[ADR-0009 No GDP Integrity Field]]

Status: **FROZEN initial packet-routed reliability/lifecycle semantics; OPEN exact control encodings**

GTS is the endpoint transport/session protocol above GDP. The initial traditional packet-routed profile is deliberately narrow: reliable ordered streams over ordinarily routed GDP packets. Unreliable, multicast/group, flow-ID routed, and congestion-control extensions are deferred.

## Frozen identifier and sequencing model

- Tunnel ID: 32 bits, receiver-local.
- Stream ID: 8 bits, scoped to one tunnel.
- Packet Sequence: 32 bits, scoped to one stream and counting DATA/DATA_END packets rather than bytes.
- DATA and ACK are separate packet types; ACK state is never piggybacked on DATA in the initial profile.

A CONNECT exchange establishes the receiver-local Tunnel ID for each direction. Ordinary packets carry only the peer-assigned destination Tunnel ID.

## Frozen reliability model

GTS uses selective-repeat reliability with a cumulative ACK base and 32-bit receive bitmap.

Normal ACK policy is:

- ACK every two correctly received DATA/DATA_END packets;
- if only one packet is pending acknowledgement, ACK on a short delayed-ACK timer;
- ACK immediately when a gap is detected;
- ACK immediately when a reported gap is filled and ACK Base advances;
- ACK immediately when Receive Credit reaches zero or must change to restart a stalled sender.

When a bitmap shows a missing outstanding packet followed by one or more positively received later packets, that missing packet is an explicit hole and is retransmitted immediately. Duplicate-ACK counting is not required.

A retransmission timer remains mandatory for losses that cannot be exposed by later packets. The initial profile uses an adaptive RTO based on smoothed measured RTT, with one retransmission timer per stream. Exact coefficients/minimum/maximum timing constants remain to be frozen.

Receive Credit is measured in packets, not bytes.

## Tunnel and stream lifecycle

CONNECT simultaneously establishes the tunnel and opens Stream 0 as the initial service stream. Additional streams use STREAM_OPEN/STREAM_ACK.

Stream IDs use parity ownership to avoid allocation collisions:

- CONNECT initiator allocates even Stream IDs;
- CONNECT responder allocates odd Stream IDs;
- Stream 0 belongs to the initiator convention and is created by CONNECT.

Graceful stream close is directional. STREAM_CLOSE means the sender will transmit no further DATA/DATA_END in that direction; the opposite direction may remain open. A stream is fully closed only after both directions are closed and required acknowledgement/confirmation state completes.

RESET is the fatal tunnel-wide termination mechanism. It destroys the tunnel and all streams immediately. Exact RESET encoding and the final Reset-ID/capability decision remain open.

## DATA_END

Ordinary DATA retains the fixed no-length format. A partial final transport unit uses the separate DATA_END packet type, which carries an explicit Valid Length for the meaningful application bytes in that final packet. DATA_END consumes a normal sequence number and is acknowledged exactly like DATA.

The exact Valid Length field width and packing remain open.

## End-to-end integrity

GDP carries no checksum/CRC, so GTS provides mandatory end-to-end integrity.

- future compact forms using the 0-byte/3-byte tiny GDP classes use CRC-8-GNET;
- every ordinary larger-class GTS packet uses CRC-32-GNET.

CRC parameters are frozen:

CRC-8-GNET: polynomial `0x07`, init `0x00`, no reflection, xorout `0x00`, check `123456789 -> 0xF4`.

CRC-32-GNET: polynomial `0x04C11DB7`, init `0xFFFFFFFF`, no reflection, xorout `0xFFFFFFFF`, most-significant byte first, check `123456789 -> 0xFC891918`.

The CRC covers canonical GDP context plus the complete GTS header/payload. The canonical GDP contribution is GDP Version, GDP Type=`0x2`, GDP Size Class, effective 64-bit Source Address, and effective 64-bit Destination Address. Local GDP addresses are expanded to their canonical endpoint identities first. Hop Limit, Local/Global representation, reserved bits, and other mutable forwarding state are excluded.

Failed CRC means the packet is discarded and treated as not received.

## Service selection

GTS does not use TCP-style fixed ports. Service selection is setup-only through the accepted variable-width Service Selector model; ordinary DATA/ACK packets carry only Tunnel ID and Stream ID.

## Explicitly deferred

- flow-ID/virtual-circuit routed optimization;
- multicast/group transport;
- unreliable stream profiles;
- end-to-end congestion-control algorithm;
- cryptographic authentication/encryption;
- dynamic routing behavior.

## Remaining initial-profile work

The major behavioral decisions are now frozen. Remaining work is mostly wire closure:

- numeric GTS packet-type registry;
- CONNECT / CONNECT_ACK exact layout;
- STREAM_OPEN / STREAM_ACK exact layout;
- DATA_END Valid Length width/placement;
- STREAM_CLOSE / close-confirmation exact layout;
- RESET layout and Reset-ID decision;
- numerical delayed-ACK/RTO constants;
- stale-packet/Stream-ID reuse rules;
- padding rules and golden vectors.
