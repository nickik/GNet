---
id: gnet-overview
title: "GNet Protocol Suite Overview"
aliases: ["Overview","GNet overview","Protocol overview"]
type: overview
status: active
layers: ["L1","L2","L3","L4","L5","L6","L7"]
tags: ["gnet","gnet/overview","gnet/architecture"]
related: ["[[Specification Status]]","[[GDP Protocol]]","[[GTS Protocol]]","[[Canonical Service Selector]]","[[GCTL Protocol]]","[[Direct Link Protocol]]"]
updated: 2026-09-14
---
# GNet Protocol Suite Overview

GNet is a routed networking architecture built around small physical **flits**, hop-local flow control, compact network-layer datagrams, and end-to-end transport sessions.

This page is the best starting point for somebody who already understands Ethernet/IP-style networking and wants to understand how the GNet pieces fit together. It is an explanatory overview, not the final authority for individual wire fields; the linked protocol documents remain normative.

The central idea is simple:

```text
Application / service
        |
        v
       GTS           reliable tunnels and streams
        |
        v
       GDP           routed datagrams and addresses
        |
        v
       DLP           hop-local carried flits / VC state
        |
        v
   GNet physical link
```

Control is split by scope rather than forced into one protocol:

```text
GLCP   directly attached physical/link infrastructure control
GCTL   control messages carried on the normal GNet data path
```

GNet is deliberately **not** an Ethernet clone. There is no global Layer-2 MAC-address space and no requirement for Ethernet-style learning bridges. Native GNet traffic is addressed by GDP even on local networks.

---

## 1. The protocol stack

The OSI mapping is approximate. GNet follows the same separation of concerns, but some control functions naturally cross textbook layer boundaries.

| OSI layer | GNet component | Main responsibility |
|---|---|---|
| L7 Application | GTerm, file, boot, RPC, voice, other services | application semantics |
| L6 Presentation / naming | directory, identity, service naming | names, service discovery, representation, identity |
| L5 Session | GTS tunnel control + CSS | service selection, tunnel establishment, reset/close, stream creation |
| L4 Transport | GTS | sequencing, acknowledgement, retransmission, receive flow control, end-to-end integrity |
| L3 Network | GDP + GCTL | addressing, routing, hop limit, packet sizing, errors, diagnostics, discovery/control |
| L2 Data link | DLP + hop-local VC state | carried flits, bounded local transfer state, link backpressure relationship |
| L1 Physical | GNet PHY + infrastructure-local GLCP functions | signalling, rate/mode negotiation, local medium/path permission |

A useful comparison for TCP/IP readers is:

| Familiar concept | Rough GNet counterpart | Important difference |
|---|---|---|
| Ethernet physical/link | GNet PHY + DLP/GLCP | GNet has no Ethernet MAC-address layer |
| IP | GDP | GDP has Local and Global address forms, fixed payload Size Classes, and no fragmentation |
| ICMP plus parts of bootstrap/management | GCTL | GCTL is broader and also carries link-local credit control on the normal data path |
| TCP | GTS | GTS uses tunnels containing multiple streams rather than repeating TCP-style ports in every data packet |
| UDP | no frozen direct equivalent yet | initial GTS is reliable/ordered |
| TCP/UDP ports | Canonical Service Selector + Tunnel ID + Stream ID | CSS selects a service only during CONNECT; ordinary data identifies the established tunnel/stream |
| DNS/service discovery | GNet directory/service model | directory names resolve to GDP address + CSS; exact directory protocol remains under design |

---

## 2. Physical flits: the basic unit on a native GNet link

The baseline native GNet data unit is a **34-bit flit**:

```text
+--------+----------------------------------+
| VCID   | Carried bits                     |
| 2 bits | 32 bits                          |
+--------+----------------------------------+
```

The 32 carried bits are deliberately convenient for computers built around 16- and 32-bit words. The 2-bit **VCID** is hop-local metadata used to distinguish active transfers sharing one physical link.

Baseline VC2 therefore provides four wire VCID values. VCID `0` is reserved for future deadlock-resolution/escape use; ordinary traffic normally uses VCIDs `1-3`.

A VCID is **not**:

- an address;
- a port;
- a tunnel ID;
- a globally meaningful circuit number.

It exists only on one link and direction and may be replaced at every forwarding node.

There is no Ethernet-style start-of-frame bit in every packet. Once a local VC context is active, carried 32-bit words flow on that VC.

See [[34-bit Flit Format]] and [[Virtual Channels and VCIDs]].

---

## 3. DLP: the deliberately small link data path

The **Direct Link Protocol (DLP)** is intentionally thin.

Its job is to define the hop-local data path carrying 32-bit words inside physical flits. It does not contain network addresses, transport sessions, service identifiers, or application semantics.

Native DLP traffic normally carries GDP. GDP's Size Class tells forwarding endpoints how much GDP payload belongs to the packet, so DLP does not duplicate GDP with a second packet-size system.

Current DLP does **not** add a periodic payload CRC or an Ethernet-style frame FCS. This is deliberate:

```text
DLP   moves carried flits on the hop
GDP   protects the routing/header information
GTS   protects end-to-end transport content
```

This keeps the forwarding data path suitable for cut-through/wormhole operation rather than forcing every switch to buffer a complete packet merely to validate a link frame.

See [[Direct Link Protocol]].

---

## 4. Link control and backpressure

GNet separates two different questions:

1. **Can the next forwarding endpoint receive another flit?**
2. **May this sender use the physical medium/path right now?**

They are not the same thing.

### Receiver credit

One GNet credit means:

> guaranteed receive capacity for exactly one physical flit at the next forwarding endpoint on the current link.

Credits are strictly **hop-local**. A multi-hop route therefore has an independent credit relationship at every forwarding boundary.

Credit exchange uses GCTL messages on the normal data path:

```text
CREDIT_REQUEST
CREDIT
```

A router or switch advertises its own local receive capacity; it does not mirror some distant destination's credit balance.

This creates explicit backpressure:

```text
downstream receive space disappears
                |
                v
          credit disappears
                |
                v
       upstream sender stops
```

### Physical/path permission

Permission to drive a shared medium or use a switched path is a separate link/infrastructure concern. Native media may use dedicated physical control signalling for bootstrap, capability/rate negotiation, reset, or medium/path permission.

That directly attached infrastructure-control mechanism is called **GLCP**. Its exact runtime responsibilities depend on the link profile and remain narrower than GCTL.

The important architectural boundary is:

```text
GCTL credit = remote/adjacent forwarding endpoint receive capacity
GLCP        = directly attached physical/infrastructure operation
```

See [[GNet Link Control Protocol]] and [[GCTL Protocol]].

---

## 5. Wormhole / cut-through forwarding

GNet is designed so an intermediate forwarding node does not need to receive an entire large GDP packet before useful forwarding work begins.

Conceptually:

```text
Host A            Router             Router             Host B

header  --------> header  ---------> header  --------->
data    --------> data    ---------> data    --------->
data    --------> data    ---------> data    --------->
...                 ...                 ...
```

The header establishes where the packet is going; following flits can then be pipelined across the route while the tail of the packet is still arriving upstream.

This is attractive because it reduces:

- per-hop serialization latency;
- fast packet-buffer memory;
- memory bandwidth inside routers/switches.

A blocked path instead propagates backpressure through the hop-local credit relationships.

This does **not** eliminate congestion. Persistent overload can still block resources across several hops, so virtual channels, scheduling, deadlock avoidance, routing policy, and end-to-end congestion behavior remain important.

See [[Wormhole Routing Benefits]].

---

## 6. GDP: the routed network layer

The **GNet Datagram Protocol (GDP)** is GNet's Layer-3 protocol. Its role is intentionally close to the classic datagram-network role: carry a packet from a source endpoint to a destination endpoint through a routed network.

GDP provides:

- Source and Destination addressing;
- Local and Global address representations;
- a payload protocol Type;
- a fixed Size Class;
- Hop Limit;
- a small header CRC;
- conventional destination-based routing.

GDP does **not** provide:

- reliable delivery;
- acknowledgements;
- retransmission;
- sequence numbers;
- sessions;
- transport windows;
- fragmentation;
- payload integrity;
- encryption.

Those belong above or below GDP as appropriate.

### GDP Type

The initial 4-bit Type registry is intentionally small:

```text
0x0       Reserved
0x1       GCTL
0x2       GTS
0x3-0xF   Reserved
```

So the two fundamental payloads of the initial network are control traffic and transport traffic.

---

## 7. GDP Global and Local addressing

GNet has one canonical global address space but two packet representations.

### Global form

Global GDP carries complete endpoint addresses:

```text
Destination    64 bits
Source         64 bits
```

The fixed Global header is exactly five 32-bit carried flits:

```text
Flit 1
31..30  Version        2
29..26  Type           4
25..22  Size Class     4
21      Address Form   1
20..16  Reserved       5
15..8   CRC-8          8
7..0    Hop Limit      8

Flit 2  Destination[63:32]
Flit 3  Destination[31:0]
Flit 4  Source[63:32]
Flit 5  Source[31:0]
```

Destination appears before Source so a forwarding implementation can begin destination lookup as early as possible.

### Local form

Within a known local GDP context, the same endpoints may use compact 16-bit local identifiers:

```text
Flit 1
31..30  Version        2
29..26  Type           4
25..22  Size Class     4
21      Address Form   1
20..16  Reserved       5
15..8   CRC-8          8
7..4    Reserved       4
3..0    Hop Limit      4

Flit 2
31..16  Destination    16
15..0   Source         16
```

The complete Local header is therefore only **64 bits / two carried flits**.

A Local ID is not a second global identity. It is a compact representation valid inside one known local GDP context. At a boundary, a router can construct the equivalent Global form using the canonical 64-bit endpoint identities. Mixed Local/Global source/destination packets are not used.

This is particularly useful where many small packets stay inside a local fabric, because the address overhead is much smaller without introducing a separate MAC-address layer.

See [[GDP Datagram]] and [[Addressing and Routing]].

---

## 8. GDP header integrity

GDP uses **CRC-8-GNET** to protect the immutable header information that routers and endpoints must interpret correctly.

The CRC covers:

```text
Version
Type
Size Class
Address Form
Destination
Source
```

It does not cover:

```text
CRC field itself
Hop Limit
Reserved bits
GDP payload
```

Hop Limit is excluded because every router changes it. A router can therefore decrement Hop Limit without regenerating the header CRC.

The CRC algorithm is:

```text
polynomial  0x07
init        0x00
refin       false
refout      false
xorout      0x00
```

A failed GDP header CRC invalidates the packet.

The important design distinction is:

> GDP verifies that its routing/header information is meaningful; it does not promise that the carried payload is error-free.

For GTS traffic, end-to-end payload integrity is provided by GTS CRC-32.

---

## 9. GDP Size Classes

GDP does not carry an arbitrary payload-length field. Instead, a 4-bit Size Class selects one of 16 exact payload budgets:

| ID | Name | Payload bytes |
|---:|---|---:|
| 0 | `empty` | 0 |
| 1 | `tiny3B` | 3 |
| 2 | `ctrl32B` | 32 |
| 3 | `ctrl64B` | 64 |
| 4 | `msg128B` | 128 |
| 5 | `msg192B` | 192 |
| 6 | `msg256B` | 256 |
| 7 | `msg384B` | 384 |
| 8 | `medium512B` | 512 |
| 9 | `medium768B` | 768 |
| 10 | `bulk1K` | 1024 |
| 11 | `bulk1280B` | 1280 |
| 12 | `legacyet` | 1500 |
| 13 | `xmtu2K` | 2048 |
| 14 | `jumbo4K` | 4096 |
| 15 | `jumbo8K` | 8192 |

Different physical/link profiles may support only a subset of these classes.

GDP does **not fragment** a packet in transit. If the selected next hop cannot carry the packet's Size Class, the packet cannot simply be split into smaller GDP fragments; the appropriate network error is reported instead.

The size-class approach gives forwarding hardware a bounded packet size very early in the header while avoiding a larger arbitrary-length field.

---

## 10. Routing

Global GDP routing uses hierarchical 64-bit addresses and longest-prefix matching.

The initial routed profile is intentionally conservative:

- directly connected routes;
- administratively configured static prefix routes;
- optional static default route;
- longest-prefix match.

Dynamic route exchange, metrics, automatic convergence, inter-domain routing, and multipath policy are separate work rather than hidden inside the base GDP packet format.

The initial address-configuration profile supports routed on-link prefix lengths such as `/16`, `/32`, `/48`, and `/56`; the global address itself remains a full 64-bit value.

### Hop Limit

Global GDP carries an 8-bit Hop Limit. Local GDP uses a 4-bit Hop Limit because a local context is expected to be much shallower.

Every router decrements the applicable Hop Limit. Expiry discards the packet and may generate the appropriate GCTL error.

See [[Addressing and Routing]].

---

## 11. GCTL: network control, diagnostics, and link-local credit messages

**GCTL** is GDP Type `0x1` and is carried on the same normal data path as other GDP traffic.

It combines several control functions that must work across normal GNet forwarding:

- scoped discovery;
- address offer/claim/bootstrap;
- `CREDIT_REQUEST` / `CREDIT` for adjacent forwarding endpoints;
- ECHO reachability tests;
- destination-unreachable and malformed-packet errors;
- Hop Limit errors;
- path probes;
- mandatory status queries;
- read-only management/status enumeration;
- bounded asynchronous event reports.

This makes GCTL broader than ICMP. It is the common network-control vocabulary for GNet rather than merely an error-report packet family.

Automatic error reporting remains best effort and rate limited. GCTL errors do not make GDP reliable.

See [[GCTL Protocol]] and [[GCTL Message Registry]].

---

## 12. GTS: reliable tunnels and streams

**GTS** is GDP Type `0x2` and provides the initial end-to-end transport/session service.

The core object is a **tunnel** containing one or more independently sequenced **streams**:

```text
GTS tunnel
    |
    +-- Stream 0
    +-- Stream 2
    +-- Stream 4
    ...
```

The initial profile provides reliable, ordered streams.

Important identifiers are:

```text
Tunnel ID       32 bits, receiver-local
Reset ID        32 bits, receiver-local reset capability
Stream ID        8 bits, tunnel-scoped
Packet Sequence 32 bits, stream-scoped
```

A CONNECT exchange selects one CSS, creates a service-bound tunnel, and creates Stream 0. Additional streams use STREAM_OPEN/STREAM_ACK and remain inside that same selected service.

### DATA

An ordinary GTS DATA packet contains:

```text
Version/Type          1 B
Tunnel ID             4 B
Stream ID             1 B
Sequence              4 B
Data                   N
CRC-32                4 B
```

The sequence number counts DATA/DATA_END packets, not bytes.

### Reliability

ACK packets are separate. They contain:

- cumulative ACK Base;
- a 32-bit selective receive bitmap;
- receive credit for additional DATA packets.

A visible hole can therefore be retransmitted selectively rather than retransmitting every later packet.

A retransmission timer covers losses that cannot be inferred from later packets.

### End-to-end integrity

Ordinary GTS uses CRC-32-GNET over its transport content plus a canonical GDP pseudo-header. The pseudo-header binds the transport packet to the effective endpoint identities and GDP Size Class.

When Local GDP representation is used, the 16-bit Local IDs are expanded to the canonical 64-bit endpoint identities for the GTS CRC. A Local/Global representation change in the network therefore does not invalidate the end-to-end transport CRC.

GTS reliability is an endpoint function. Ordinary routers do not maintain GTS tunnel or stream state.

See [[GTS Protocol]] and [[GTS Transport Packets]].

---

## 13. Canonical Service Selector: services instead of per-packet ports

GTS deliberately separates **service selection** from ordinary transport demultiplexing.

A service is selected once, in CONNECT, using the **Canonical Service Selector (CSS)**. If CONNECT succeeds, the tunnel is bound to that service for its lifetime. Ordinary DATA packets use Tunnel ID and Stream ID and do not repeat a source/destination port pair or CSS in every packet.

CSS is one **128-bit canonical service namespace** with three wire representations:

| Representation | Wire size | Example | Meaning |
|---|---:|---|---|
| Registered-8 | 8 bits | `-FILE`, `-GRPC` | registered compressed form |
| Short-32 | 32 bits | `#CAPI`, `#MYEP` | exactly four ASCII characters |
| Full-128 | 128 bits | `0123456789ABCDEF0123456789ABCDEF` | complete canonical value |

They are not separate namespaces.

Short-32 occupies the most-significant 32 bits of CSS128 and the low 96 bits are zero:

```text
#CAPI
    32-bit value = 0x43415049
    CSS128       = 43415049000000000000000000000000
```

Registered-8 expands through the global registry to a Short-32 mnemonic and then to the same CSS128 form:

```text
-FILE
    Registered-8 = 0x01
    Short-32     = 0x46494C45   "FILE"
    CSS128       = 46494C45000000000000000000000000
```

The shortest available representation is canonical. Because `FILE` is registered, `#FILE` is not a second service identity and is not the canonical wire encoding.

An endpoint plus service may be written as:

```text
<GDP-address>:-FILE
<GDP-address>:#CAPI
<GDP-address>:0123456789ABCDEF0123456789ABCDEF
```

This produces four distinct identifiers with different jobs:

```text
GDP address      where is the endpoint?
CSS              what service is requested?
Tunnel ID        which established transport association?
Stream ID        which stream inside that tunnel?
```

A directory service can map a human-facing service name to one or more `GDP address + CSS` pairs. The exact global directory/naming protocol is still open.

STREAM_OPEN does not contain CSS. It creates another stream inside the already service-bound tunnel. Selecting a different service requires another CONNECT and another tunnel.

See [[Canonical Service Selector]], [[CSS Registered Service Registry]], and [[GNet Service Model]].

---

## 14. A packet's journey through GNet

Consider an application sending reliable data to a remote service.

### At the sender

1. The application obtains a destination GDP address and CSS, directly or through a directory.
2. If necessary, GTS sends CONNECT carrying the CSS; successful CONNECT establishes receiver-local Tunnel IDs and Stream 0 bound to that service.
3. GTS creates a DATA packet containing Tunnel ID, Stream ID, Sequence, application data, and CRC-32.
4. GDP wraps that GTS packet, sets Type=`GTS`, chooses an appropriate Size Class, supplies Source/Destination, and calculates the GDP header CRC-8.
5. The link data path carries the GDP bits as 32-bit words inside VC-tagged physical flits.

### At a router

1. The router receives the GDP fixed header.
2. It validates GDP header CRC-8 before treating the packet as valid.
3. It performs destination lookup.
4. It decrements Hop Limit.
5. It establishes the outgoing hop-local forwarding/VC state and observes downstream credit/path availability.
6. It forwards the carried packet onward; it does not inspect or repair GTS application content.

Because the destination is early in the GDP header and forwarding is flit-oriented, the architecture is intended to support forwarding before the complete payload has arrived.

### At the destination

1. GDP validates/delivers the packet according to Type.
2. Type=`GTS` sends the payload to GTS.
3. GTS validates its CRC-32.
4. Tunnel ID and Stream ID select the already service-bound transport state.
5. Sequence/ACK logic provides ordered reliable delivery to the selected service.

The important layering is therefore:

```text
link state is hop-local
routing state is packet/network-local
service selection happens at tunnel setup
transport state is end-to-end
application state stays above transport
```

---

## 15. What GNet intentionally does not do

Several absences are deliberate rather than missing features.

### No Ethernet-style global Layer-2 address

DLP does not carry MAC source/destination addresses. Native endpoints are addressed with GDP.

### No general Layer-2 bridging architecture

The network is designed around routed GDP domains rather than one indefinitely extended learning-bridge broadcast domain.

### No GDP fragmentation

GDP packets are selected from explicit Size Classes. A router does not fragment an oversized packet in transit.

### No GDP payload checksum

GDP CRC-8 protects routing/header interpretation only. End-to-end payload integrity belongs to protocols such as GTS.

### No transport state in normal routers

Tunnel IDs, stream sequences, ACKs, retransmission timers, and application services remain endpoint state.

### No mandatory TCP-style ports in every packet

CSS selects the service during CONNECT. Established traffic uses tunnel and stream identifiers, so service identity is not repeated in every DATA packet.

---

## 16. Current specification boundaries

The core architecture is defined, but GNet is still an active protocol design. Important areas that are not yet fully closed include:

- trustworthy resynchronization after a GDP header CRC failure when Size Class itself may be corrupt;
- the final minimum runtime control set for some switched/native link profiles;
- dynamic routing and routing-policy protocols beyond the initial static profile;
- complete end-to-end congestion-control behavior above the hop-local credit system;
- final directory/naming wire formats and replication;
- authentication, encryption, and authenticated management;
- some timing constants and conformance vectors in GTS.

For current normative status, see [[Specification Status]] and [[Open Questions]].

---

## 17. The shortest useful mental model

If only five ideas are remembered, they should be these:

1. **GDP is the routed datagram layer.** It is the GNet equivalent of the network layer, with 64-bit Global addresses and compact 16-bit Local representation.
2. **The wire moves 32-bit carried words as small VC-tagged flits.** This enables cut-through/wormhole-style forwarding and small intermediate buffers.
3. **Flow control is hop-local.** Credits describe actual receive capacity at the next forwarding endpoint; they are not end-to-end transport windows.
4. **GTS owns end-to-end reliability and CSS selects the service.** CONNECT binds one service to a tunnel; the tunnel then contains independently sequenced streams.
5. **The layers protect different things.** GDP CRC-8 protects routing/header information; GTS protects transport content; applications can define additional semantics as required.

From there, the detailed specifications are:

- [[Direct Link Protocol]]
- [[GNet Link Control Protocol]]
- [[GDP Protocol]] / [[GDP Datagram]]
- [[GCTL Protocol]]
- [[GTS Protocol]] / [[GTS Transport Packets]]
- [[Canonical Service Selector]] / [[CSS Registered Service Registry]]
- [[Addressing and Routing]]
- [[GNet Service Model]]
