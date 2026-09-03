# GNet Layers 2-4 and DEC Compatibility Strategy Chat

This archive note summarizes the project conversation that refined GNet Layer 2 through Layer 4 and then explored the strategic consequences of a successful GNet ecosystem for DEC in the 1980s, including workstation, networking, telephony, IBM midrange, and IBM-compatible mainframe strategy.

## GNet protocol stack

The discussion converged on a deliberately simple layered architecture:

- Layer 1 provides the physical links.
- Layer 2 provides framing, arbitration, and local integrity, but does not need conventional MAC addressing on the SmartHub LAN.
- Layer 3 is the routed GNet packet layer, with full global addressing or a compact local-only form.
- Layer 4 is tunnel-first and mobility-aware. A tunnel binds a service once, then carries multiple streams with different delivery properties.

A core design principle is that GNet should remain simple enough for late-1970s hardware while avoiding the complexity and inefficiencies of contemporary Ethernet, ARCNET, Token Ring, and early DECnet approaches.

---

# Layer 2: SmartHub LAN

## Physical topology

The preferred early office-access topology is a physical star using ordinary telephone-style twisted-pair cabling.

A minimum installation can use two pairs:

- one pair for control/arbitration
- one pair for data

A four-pair installation can provide additional data capacity, separate upstream/downstream paths, or future expansion.

The SmartHub sits in the wiring closet or on each office floor. The hub should be extremely cheap and require almost no configuration.

## Logical bus over physical star

Although physically star-wired, the data side behaves logically like a broadcast bus:

1. a client that wants to transmit asserts a request on the control pair
2. the SmartHub grants permission according to its scheduler
3. only the granted client transmits
4. the hub distributes the transmission to attached clients
5. each NIC examines the Layer 3 destination and accepts or ignores the packet

The hub therefore owns the token/arbitration state. A token never circulates among clients.

This removes collisions entirely while also avoiding the logical-ring complexity of ARCNET or Token Ring.

## No Layer 2 addresses on SmartHub LAN

A major decision is that the SmartHub LAN does not need source or destination addresses at Layer 2.

The source is already known implicitly because the hub knows which physical port it granted. The destination is identified by Layer 3.

Layer 2 therefore becomes principally:

- frame synchronization/boundary
- payload type/class
- length or size indication
- CRC/integrity
- arbitration via the separate control pair

Conceptually:

```text
[SYNC][TYPE/CLASS][LENGTH][PAYLOAD][CRC]
```

No MAC-address equivalent is required.

## Local and global traffic types

A small Layer 2 type field distinguishes at least:

- Global GNet traffic
- Local GNet traffic
- maintenance/control/test traffic

Global traffic carries the full routed GNet Layer 3 form with 48-bit addresses.

Local traffic carries a compact local-only Layer 3 form using 8-bit local addresses. This is intended for communication entirely inside one LAN/fabric and avoids paying the full global-address overhead for local office traffic.

The Layer 2 framing remains identical regardless of which Layer 3 form is carried.

## Two-level priority

The SmartHub can implement priority much more cheaply than a distributed-token LAN because arbitration is centralized.

The intended model has two request priorities on the control pair:

- high-priority request: permitted only for a small packet class, intended for latency-sensitive traffic such as telephone/real-time control
- normal-priority request: permits larger packets for normal data transfers

The hub schedules requests centrally, preferably round-robin within a priority class, with limits preventing permanent starvation of normal traffic.

This is a major reason the architecture is attractive for integrated voice and data.

## Passive versus active hub

The original attraction of a passive repeater was minimum cost. The refined direction is a minimally active SmartHub rather than a sophisticated switch.

The hub only needs enough electronics to:

- detect requests
- issue grants
- gate/replicate the data path
- optionally reshape/regenerate signals
- isolate a faulty port

It should not contain routing, service logic, or complicated site configuration.

## DDCMP relationship

DEC's DDCMP is useful as a design reference for framing, count-oriented transparency, CRC discipline, and robust point-to-point communication, but it is too heavy to copy directly for the SmartHub access LAN.

The SmartHub LAN should not inherit DDCMP's link-layer ACK/NAK, message numbering, multipoint addressing, or other reliability machinery. End-to-end reliability belongs primarily at Layer 4.

A DDCMP-inspired point-to-point profile is more appropriate for router-to-router links.

---

# Layer 3: GNet routed packet layer

## Global packet

The global GNet packet is the IP-equivalent routed datagram.

The design direction uses:

- fixed, cheap-to-parse header
- 48-bit source address
- 48-bit destination address
- version
- hop limit
- packet size class
- minimal routing/service metadata as required

Layer 3 does not contain session identity, ports, reliability state, transport sequence numbers, or mandatory encryption/authentication.

No Layer 3 checksum is required if the link layer provides local CRC and higher layers provide end-to-end integrity where required.

The earlier idea of a `NextProto` field was rejected as unnecessary if every valid Layer 3 payload is always the GNet tunnel protocol.

## Addressing

The preferred global address model is hierarchical and 48 bits wide, conceptually divided into fields such as:

- Top
- Organization
- Division
- Node

The exact textual notation remains subject to final specification, but the project has favored human-readable decimal forms rather than hex-centric notation.

## Local packet

A compact local-only Layer 3 form is permitted inside a SmartHub LAN.

The essential concept is:

- 8-bit local source
- 8-bit local destination
- compact size/class information
- no global routing fields
- no hop limit because the packet cannot leave the local fabric

The Layer 2 type tells the receiver whether the following Layer 3 header is Global GNet or Local GNet.

The system can automatically use Local GNet when it knows the peer is on the same local fabric, falling back to the global form when routing is required.

---

# Layer 4: tunnel-first transport

## Tunnel as the mandatory Layer 4 abstraction

Instead of allowing arbitrary Layer 4 protocols directly inside Layer 3, every valid GNet Layer 3 payload enters the GNet Tunnel Protocol.

A tunnel is created first. Streams then exist inside the tunnel.

This provides one stable mobility-aware connection abstraction while allowing individual streams to choose different delivery semantics.

## Tunnel ID

Tunnel establishment uses two 32-bit contributions:

1. the initiator sends a 32-bit tunnel value in CONNECT
2. the responder returns that value plus its own 32-bit contribution
3. the established Tunnel ID is therefore 64 bits

The Tunnel ID is public and appears on normal stream packets. It identifies the tunnel independently of the current Layer 3 addresses, allowing the connection to survive address changes.

## Reset ID

The earlier `Handle ID` concept was renamed `Reset ID` after clarifying its purpose.

The Reset ID is a capability-like value established early in tunnel setup. It is not transmitted on ordinary data packets.

It is required for privileged tunnel operations such as:

- REBIND
- tunnel CLOSE
- tunnel RESET

This provides lightweight protection against off-path injection: an observer that did not see the initial exchange does not know the Reset ID.

This is not strong cryptographic protection against an on-path observer; stronger encrypted/authenticated stream versions may later be negotiated.

## Service binding

A tunnel binds a Service ID during CONNECT.

Because the service is already part of the established tunnel state, the Service ID does not need to be repeated in every data packet.

This is preferred to TCP-style ports on every packet.

Human-readable service names can resolve to compact numeric Service IDs on the wire.

## Stream IDs

Each tunnel supports 256 Stream IDs using an 8-bit field.

The intended reserved/default model is:

- Stream 0: tunnel control / reserved control semantics
- Stream 1: default data stream
- remaining Stream IDs: additional streams

A client should normally be able to open a new stream simply by sending the first packet on an unused valid Stream ID rather than requiring a separate stream-open round trip.

To prevent simultaneous-open collisions, one possible convention is odd IDs for client-created streams and even IDs for server-created streams, though this remains a design choice rather than a fully frozen rule.

## Stream profiles

Mobility belongs to the tunnel, so every stream remains mobile regardless of its delivery profile.

Streams can have different profiles, including:

- reliable ordered byte stream
- unreliable datagram
- sequenced datagram
- future reliable message-oriented profile
- future encrypted profiles
- future compression profiles

The first packet of a new stream can set an Open flag and carry a fixed-format extension describing the stream profile. Only the opening packet pays for those setup fields; subsequent packets use the compact established-stream header.

Future encrypted versions can therefore add key/agreement material during stream open without permanently enlarging every ordinary packet.

---

# Reliable stream profile

The default reliable profile is intentionally close to the proven TCP reliability model rather than inventing a radically different recovery mechanism.

It requires:

- Tunnel ID
- Stream ID
- sequence number
- acknowledgement number
- receive credit/window
- flags
- payload

The provisional fixed reliable header discussed is 20 bytes.

Conceptually:

```text
Flags       8 bits
Stream ID   8 bits
Credit      compact/scaled field
Profile/reserved
Tunnel ID   64 bits
Sequence    32 bits
ACK         32 bits
```

Service ID and Reset ID are not repeated on normal packets.

## Reliability

Credit does not replace acknowledgements.

Credit answers:

> how much may the peer send?

ACK answers:

> what data definitely arrived?

Reliable streams therefore require sequence tracking, ACKs, retransmission timers, and selective retransmission or later SACK-like extensions.

Block hashes may be useful as additional end-to-end integrity verification, but should not replace ACK-based loss recovery.

## Credit encoding

The discussion considered compressing receive credit into 8 bits rather than carrying an exact 16-bit byte window.

A preferable compact design is to negotiate a Credit Unit and transmit an 8-bit count of units. Example units could be 64, 128, 256, or 1024 bytes.

This keeps packet arithmetic simple while allowing a large effective receive window.

Two distinct credit concepts must remain separate:

- hop/link-level wormhole or flit credit: Layer 2/link/fabric flow control
- end-to-end stream receive credit: Layer 4 transport flow control

## Stream opening

A packet on an unused stream with the Open flag can establish the stream and carry profile options.

The default implicit profile is a reliable ordered byte stream.

The receiver may acknowledge the first payload immediately, making stream establishment a one-packet operation rather than requiring a separate stream-opening handshake.

---

# Datagram profiles

## Unreliable datagram

The minimal UDP-like profile needs only:

- flags/type
- Stream ID
- Tunnel ID
- payload

This produces a provisional 10-byte Layer 4 header.

There is no ACK, sequence number, retransmission, or stream receive credit.

## Sequenced datagram

The earlier term `unreliable ordered datagram` was replaced by `sequenced datagram` because the receiver does not wait for missing packets.

A sequenced datagram adds a small message sequence number. The receiver can determine which update is newer and discard stale late arrivals.

Use cases include:

- voice/media
- telemetry
- cursor/state updates
- remote graphics state
- any 'latest state wins' traffic

A provisional layout is 12 bytes at Layer 4:

- flags/type
- Stream ID
- 16-bit message sequence
- 64-bit Tunnel ID

---

# Header-overhead comparison

Using the provisional figures discussed:

- GNet Layer 2 framing: approximately 4 bytes
- Global GNet Layer 3: approximately 16 bytes
- reliable stream Layer 4: 20 bytes

A normal globally routed reliable GNet packet therefore has approximately 40 bytes of L2-L4 overhead.

A first packet that includes a small stream-open extension is roughly 44 bytes.

For comparison, minimal Ethernet II + IPv4 + TCP is approximately:

- Ethernet header + FCS: 18 bytes
- IPv4: 20 bytes
- TCP: 20 bytes
- total: 58 bytes

The local GNet form reduces overhead further because the full 16-byte global Layer 3 header is replaced with a very small local header.

---

# GNet office-network product strategy

The discussion extended the protocol design into a late-1970s DEC product strategy.

By approximately 1977-1978, DEC could plausibly have:

- a 3 Mb/s GNet SmartHub office LAN
- PDP-11 and VAX network cards
- VT-series terminal attachment over GNet rather than direct serial wiring
- file and print service appliances
- a small-office telephone/key-system/PBX appliance analogous in market role to the later AT&T MERLIN family

The important product architecture is to separate the cheap hub from all services.

## FloorHub

Each floor or office zone gets a nearly configuration-free hub. The hub is infrastructure, not a router/server.

## Separate service boxes

Optional boxes attach to the hub and independently provide:

- router/uplink
- file/print server
- telephone PBX/key system
- terminal server/concentrator
- WAN/public-network gateway

This allows customers to install the wiring/fabric once and add services independently.

## Open attachment standard

DEC should publish the GNet wire/attachment specification and operate a conformance program.

Any vendor may build a GNet NIC or appliance if it passes the required interoperability tests.

Higher-level DEC file/print and other services may remain proprietary, allowing DEC to monetize the ecosystem while keeping the LAN attachment standard open.

This strategy aims to combine roles historically occupied by ARCNET, Novell, Cisco, and small-office telephony vendors.

---

# Alternate DEC strategic scale

The discussion then considered an alternate DEC that also:

- designs VAX as a RISC-like architecture from the beginning
- maintains leading price/performance through the 1980s
- captures much more of the workstation market
- develops ECL high-end/vector implementations
- uses GNet to dominate office LAN/networking
- aggressively attacks IBM commercial midrange systems

A non-PC version of this DEC was estimated in the rough range of $22B-$32B annual revenue by the end of the 1980s, versus real DEC's approximately $12.7B FY1989 revenue.

A successful PC strategy on top of those wins could move DEC materially closer to IBM's scale, though IBM would still possess enormous mainframe, software, maintenance, and service revenues.

---

# IBM commercial-system compatibility strategy

DEC historically already possessed important commercial-computing assets:

- Commercial Products / Business Products organization
- DEC Datasystem products
- DIBOL
- RPG II support on PDP-11 and later VAX
- commercial OEM/VAR channels

The strategic proposal is to expand this dramatically into explicit IBM replacement products rather than merely advertising that VAX can run COBOL or RPG.

## DECSystem/34

A supported System/34 replacement emphasizing:

- RPG compatibility
- file compatibility
- job/spool/operator compatibility
- terminal/printer compatibility
- migration tooling

## DECSystem/36

The primary commercial attack product, with full practical software and operations compatibility as the objective.

Important components include:

- RPG II
- 5250 terminal/printer behavior
- SSP-compatible runtime behavior
- file/data conversion
- packaged-application certification
- nationwide DEC migration/support services

## DECSystem/38

A deeper compatibility environment requiring much more than a compiler.

DEC would need to reproduce enough of:

- System/38 Machine Interface behavior
- CPF/CL runtime semantics
- object model
- integrated database behavior
- operator/admin model

The System/38 MI makes an alternate implementation plausible, but database/runtime compatibility is the difficult part.

The overall target is full customer-visible software compatibility even though the physical processor underneath is PDP-11/VAX/RISC-VAX technology.

---

# IBM 360/370-compatible mainframe strategy

The project also distinguishes the System/34/36/38 compatibility program from IBM System/360/370 plug-compatible mainframes.

The latter had a real independent ecosystem including:

- Amdahl
- Itel
- National Advanced Systems (NAS)
- Magnuson
- IPL Systems
- Cambex
- StorageTek CPU efforts
- Trilogy
- Fujitsu/Hitachi at larger scale

The proposed DEC end-state is not to preserve all of these hardware lines. It is to acquire compatibility expertise, customer bases, field-service organizations, I/O knowledge, and test suites, then virtualize IBM operating environments on the common DEC architecture.

## Recommended acquisitions / targets

### 1. Itel Advanced Systems, around 1979

The preferred foundational acquisition.

Value:

- active IBM-compatible customer base
- compatibility engineering
- IBM software/operational expertise
- sales/service entry into IBM accounts
- existing Japanese supplier relationships

Historically Itel's computer activities ended up becoming National Advanced Systems under National Semiconductor. In the alternate strategy DEC should attempt to buy this business first.

### 2. Magnuson assets, around 1983

Acquire during distress/bankruptcy for:

- IBM 4300-compatible engineering
- diagnostics
- low/midrange mainframe implementation knowledge
- test infrastructure

### 3. IPL Systems

Useful for additional IBM 4300-compatible implementation, independent compatibility tests, customers, and engineering redundancy.

### 4. Cambex selected assets

Especially valuable for:

- IBM-compatible memory
- channels
- storage/controller behavior
- subsystem attachment

### 5. StorageTek partnership/assets

Especially valuable for:

- disk/tape compatibility
- IBM channels
- datacenter migration
- backup/archive integration

### 6. NAS if Itel was missed

NAS becomes the substitute acquisition if DEC fails to acquire Itel's business at the earlier opportunity.

### 7. Trilogy engineers/assets

Useful mainly for advanced packaging/high-performance implementation talent after the company becomes distressed.

### Amdahl

Amdahl is strategically different because it is the strongest high-end IBM-compatible competitor and has important Fujitsu ties.

The preferred strategy is partnership, cross-licensing, or an equity relationship unless an unusually early acquisition opportunity can be created.

---

# Virtualization end-state

The IBM-compatible acquisition program should move through three phases.

## Phase 1: native plug-compatible hardware

DEC continues selling acquired compatible systems while learning the ecosystem and supporting customers.

## Phase 2: assisted compatibility on RISC-VAX/ECL-VAX

DEC builds a virtual-machine environment with hardware/software assists for common IBM operations such as:

- condition codes
- decimal arithmetic
- storage protection/keys
- privileged transitions
- address translation
- channel operations

Rare instructions can trap into firmware/hypervisor implementation rather than requiring a literal clone CPU.

## Phase 3: supported IBM virtual machines on DEC architecture

The end goal is to run important IBM operating environments and applications as supported guests on the common DEC platform.

Compatibility must include more than the instruction set:

- CPU architectural state
- memory/protection semantics
- channel I/O
- disk/tape behavior
- 3270 and batch operations
- MVS/VM/VSE-class operating environments
- diagnostics and recovery behavior

Virtualization is used as a customer-capture and migration strategy:

1. preserve existing IBM applications unchanged
2. integrate them with GNet, DEC storage, print, and applications
3. migrate workloads gradually to native DEC software where beneficial

The strategic principle is:

> DEC should acquire IBM-compatible companies for their compatibility knowledge, customer relationships, field support, I/O expertise, and test suites—not to maintain a museum of incompatible clone hardware forever.

The long-term objective is for important IBM operating environments to run as supported virtual machines on the common DEC architecture while customers progressively adopt native DEC networking, storage, and application platforms.
