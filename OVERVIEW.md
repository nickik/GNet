# GNet architecture overview

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

A 32-bit flit is the common presentation unit used throughout the packet specifications. Each row in an RFC-style packet diagram is exactly one flit. DLP frames carry an exact octet length, so the final payload flit may contain zero padding. See [packets/flit-format.md](packets/flit-format.md).

## Layer 1 — physical media

Layer 1 moves 32-bit flits across a physical connection.

- **GNET-L** is the approximately 3 Mb/s local request/grant star over four twisted pairs.
- **GNET-A** is the centrally scheduled shared residential access system for roughly 256 premises, targeting 10 Mb/s.
- **GNET-P** is the synchronous point-to-point trunk family at 10, 25, and 50 Mb/s over coax and later fiber.

Layer 1 owns signaling, clocking, line coding, request/grant or polling, and physical fault detection. It does not own global addresses, sessions, or names.

## Layer 2 — Direct Link Protocol

**DLP/GNET-LINK** packages a payload into 32-bit flits, identifies the carried protocol, supplies the exact payload length, and detects link corruption with CRC-8. It has no source or destination network address and no session state. Local delivery comes from the physical port or scheduled channel.

DLP is protocol-neutral and can carry GDP, link-local GCTL, DECnet, IP, XNS, or gateway traffic.

## Layer 3 — Global Data Protocol

**GDP** is the common routed datagram. Its fixed header is five flits:

1. Version, Type, Hop Limit, and QoS;
2. source-address high word;
3. source-address low word;
4. destination-address high word;
5. destination-address low word.

GDP uses hierarchical 64-bit global addresses. It deliberately has no length, checksum, fragmentation, options, tunnel/flow identifier, sequence number, acknowledgment, or encryption metadata.

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
