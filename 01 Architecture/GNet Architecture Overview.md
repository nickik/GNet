---
id: gnet-architecture-overview
title: "GNet Architecture Overview"
aliases: ["Architecture overview"]
type: architecture
status: frozen
layers: ["L1","L2","L3","L4","L5","L6","L7"]
tags: ["gnet","gnet/architecture","gnet/status/frozen","gnet/layer/l1","gnet/layer/l2","gnet/layer/l3","gnet/layer/l4","gnet/layer/l5","gnet/layer/l6","gnet/layer/l7"]
parent: "[[Architecture MOC]]"
related: ["[[GNet Layer Model]]","[[32-bit Flit Format]]","[[Virtual Channels and VCIDs]]"]
updated: 2026-09-02
---
# GNet architecture overview

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[GNet Layer Model]] · [[32-bit Flit Format]] · [[Virtual Channels and VCIDs]]


Status: **FROZEN layer boundaries; individual protocols retain their own status**

GNet is one protocol architecture carried across several physical systems. A terminal on GNET-L, a household reached through GNET-A, and a router connected by GNET-P all use the same GDP network packet and the same higher-layer protocols.

## Encapsulation

```text
Application message
    inside a GSC, GTerm, boot, directory, file, voice, or RPC protocol
        inside a GTS stream/tunnel or application datagram
            inside a GDP packet
                inside a DLP frame
                    transmitted as 32-bit flits by GNET-L, GNET-A, or GNET-P
```

A transmitted flit is exactly 32 bits: a 4-bit hop-local VCID followed by 28 carried bits. Protocol bitstreams are packed continuously across those 28-bit regions, so octets and logical 32-bit fields may cross flit boundaries. DLP segments carry an exact octet length, so the final carried region may contain zero padding. See [[32-bit Flit Format]] and [[Virtual Channels and VCIDs]].

## Layer 1 — physical media

Layer 1 moves 32-bit flits across a physical connection.

- **GNET-L** is the approximately 3 Mb/s local request/grant star over four twisted pairs.
- **GNET-A** is the centrally scheduled shared residential access system for roughly 256 premises, targeting 10 Mb/s.
- **GNET-P** is the synchronous point-to-point trunk family at 10, 25, and 50 Mb/s over coax and later fiber.

Layer 1 owns signaling, clocking, line coding, request/grant or polling, and physical fault detection. It does not own global addresses, sessions, or names.

## Layer 2 — Direct Link Protocol

**DLP/GNET-LINK** packages a payload into bounded 32-bit flit segments, identifies the carried protocol, supplies the exact payload length, and detects link corruption. Every flit begins with a 4-bit VCID, allowing several active segments to be interleaved on one link. DLP size classes bound the meaningful payload at 64, 256, or 1,024 octets. The VCID is local to a link and direction and is replaced at each router. DLP has no source or destination network address and no end-to-end session state; local delivery comes from the physical port or scheduled premise channel.

DLP is protocol-neutral and can carry GDP, link-local GCTL, DECnet, IP, XNS, or gateway traffic.

## Layer 3 — Global Data Protocol

**GDP** is the common routed datagram. Its fixed header is 20 logical octets:

1. Version, Type, Hop Limit, and QoS;
2. a 64-bit source address;
3. a 64-bit destination address.

When carried by DLP, the 160 GDP-header bits occupy portions of six 28-bit carried regions; the sixth can also carry the first eight payload bits. GDP uses hierarchical 64-bit global addresses. It deliberately has no length, checksum, CRC, integrity flag/trailer, fragmentation, options, tunnel/flow identifier, sequence number, acknowledgment, or encryption metadata.

## Layer 4 — GNet Transport

**GTS transport** adds endpoint-owned delivery behavior. It creates tunnels, multiplexes streams, sequences and acknowledges reliable data, controls receive windows, retransmits, and fragments/reassembles when needed.

A tunnel has a Tunnel ID and separate Reset ID. The Reset ID authorizes close, reset, or rebind and is not repeated in ordinary data. Each stream may independently negotiate reliable/unreliable, ordered/sequenced, message/byte, encryption, and compression behavior.

## Layer 5 — session and security

**GTS tunnel control and GNET-S** establish security associations, authorize destructive tunnel operations, request reserved flows, and support mobility/rebinding. Routers may maintain reservation/scheduling state, but ordinary GDP forwarding remains independent of transport sessions.

## Layer 6 — directory and identity

Directory services map logical service or person names to one or more reachable providers. Results may carry address candidates, capabilities, authentication method, load/preference, location, access group, and lifetime.

The identity service separates device address, device credential, and user identity. A removable DigitalKey may carry protected identity, telephone/loading-number, billing, and online credentials.

## Layer 7 — application protocols

- **GSC** establishes interactive terminal, voice, video, or collaborative sessions while leaving accepted media on the direct endpoint path.
- **GTerm** provides routable, multiplexed virtual terminals and a trusted local SESSION selector.
- **Boot** locates and transfers network boot images.
- **Directory**, file, print, block storage, voice, and RPC protocols supply their own application semantics.
- Universal Digital Bytecode/DVM and the Universal Server are consumers of GNet RPC and capability services, not new network layers.

## Implementation boundary

PLIO is a system-I/O/backplane contract. QDX is a queued device ABI. QDX-GNET may perform DMA, framing, queues, CRC, header parsing, and compatible forwarding acceleration, but it does not redefine routing, transport, discovery, identity, or application protocols.
