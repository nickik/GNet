---
id: spider-datakit-reference
title: "SPIDER and Datakit"
aliases: ["Bell Labs SPIDER", "Bell Labs Datakit", "Datakit Reference"]
type: history
status: active
tags: ["gnet", "gnet/history", "gnet/reference", "networking", "bell-labs", "datakit"]
parent: "[[History MOC]]"
related: ["[[GNet Dominant Networking Evolution]]", "[[Decisions MOC]]"]
updated: 2026-09-06
---
# SPIDER and Datakit

Bell Labs pursued a substantial line of packet-network research in parallel with ARPANET and the later Internet work. The most relevant systems for GNet are **SPIDER** and its successor **Datakit**.

These projects are useful because they show that Bell Labs understood many important ideas very early: packet switching, virtual channels, fixed or small switching units, flow control, resource sharing, and integrated voice/data networking. They also illustrate why technically sophisticated networking can still lose if it is deployed as a comparatively closed vendor/carrier system while competing standards become open, multi-vendor ecosystems.

## SPIDER

SPIDER was developed at Bell Labs beginning around 1970 under Alexander G. Fraser and colleagues.

Its goals included connecting computers, terminals, storage, and shared services through a high-speed packet network. Work associated with SPIDER explored:

- packet switching;
- virtual channels;
- asynchronous time-division multiplexing;
- automatic flow control;
- shared file and printer resources;
- slotted-ring networking.

By the mid-1970s SPIDER connected a number of Bell Labs computers and shared resources and served as an important experimental predecessor to Datakit.

### Slotted-ring idea

A slotted ring continuously circulates fixed transmission slots around a ring. A station waits for an empty slot, fills it with data and addressing information, and the slot continues around the ring until the destination receives it and the slot is eventually released.

This differs from a token ring. A token ring circulates permission to transmit; a slotted ring can have many independent slots and therefore many transfers active simultaneously.

Relevant properties include:

- deterministic access behavior compared with collision-based LANs;
- multiple small transfers in flight simultaneously;
- natural support for fixed-size switching units;
- straightforward hardware implementation;
- suitability for mixed data and real-time traffic.

The idea is relevant to GNet because GNet also favors hardware-friendly packet handling and fixed packet size classes, although GNet is not itself a slotted-ring architecture.

## Datakit

Datakit emerged from the same Bell Labs research direction during the second half of the 1970s and became a real internal networking product around 1979.

Datakit was a **connection-oriented packet network based on virtual circuits**. Before normal traffic flowed, the network established a path. Intermediate switches retained state for that virtual circuit and subsequent packets could be switched using compact circuit identifiers rather than performing a complete destination lookup for every packet.

Conceptually:

```text
Host A
  |
  | establish virtual circuit
  v
Switch 1 ---- Switch 2 ---- Switch 3
   VC state     VC state      VC state
                               |
                             Host B
```

This approach provided several attractive properties:

- compact packet headers after circuit setup;
- fast hardware switching;
- predictable paths;
- explicit flow control;
- natural support for interactive traffic and voice/data integration;
- centralized network management.

Bell Labs deployed Datakit internally at substantial scale. It connected large numbers of Unix systems, terminals, and computing resources and was used in major Bell Labs development environments.

Datakit's design philosophy later resembled parts of X.25, Frame Relay, ATM, and other virtual-circuit networking systems more than IP's connectionless datagram model.

## GNet is not Datakit

GNet shares some goals with Datakit but should not be described as a virtual-circuit network in the Datakit sense.

In GNet, the **Flow/Tunnel ID identifies a conversation or flow**, but it does not require every intermediate router to establish and preserve an end-to-end circuit before packets can be forwarded.

The authoritative forwarding mechanism remains destination routing. Routers may cache flow information as an optimization, but packet delivery must not depend on that cache surviving.

Conceptually:

```text
New packet/flow
      |
      v
Destination routing lookup
      |
      +---- optionally cache Flow ID -> next hop
      |
      v
Forward packet
```

Subsequent packets may use the cached flow entry for faster forwarding. If topology changes or the cache entry disappears, the next packet can simply be routed normally again.

GNet therefore combines:

- independently routable packets;
- optional hardware flow-state acceleration;
- hop-by-hop credit/on-off flow control;
- globally routable addressing;
- hardware-friendly switching.

The Flow/Tunnel ID is closer to a transport conversation identifier or routing-cache key than to a mandatory installed Datakit circuit.

## Why Datakit did not become the dominant network standard

There was no single technical failure. Datakit worked and Bell Labs used it successfully. Its failure was primarily a **standards, ecosystem, and deployment-model failure**, reinforced by architectural differences.

### 1. Closed ecosystem versus open implementation

AT&T naturally approached networking as a telecommunications operator. Networking technology was closely tied to Bell equipment, Bell operating environments, and Bell-controlled deployment.

Ethernet and TCP/IP developed very differently.

Ethernet became a published multi-vendor standard through Xerox, DEC, Intel, and later IEEE standardization. TCP/IP specifications were openly published, and Berkeley Unix distributed widely available implementations with source code.

This meant that a university, workstation company, minicomputer vendor, router vendor, or government network could implement Ethernet/IP without becoming dependent on one manufacturer or carrier.

This openness was probably more important than many individual protocol differences.

### 2. Ethernet was easy to adopt incrementally

Early Ethernet was technically imperfect: shared-medium collisions, weak deterministic behavior under load, and no native global internetworking layer.

But it solved a narrow problem cheaply and openly: interoperable local packet transport.

IP could then sit above Ethernet and provide global internetworking across heterogeneous underlying networks.

The resulting modularity allowed different vendors to improve individual layers independently.

### 3. Internet architecture federated independent networks

TCP/IP was explicitly suited to connecting independently administered networks.

Datakit's virtual-circuit architecture was excellent inside a controlled network but depended more heavily on network-maintained connection state and a managed switching environment.

For a worldwide federation containing many independent operators, connectionless datagram routing proved easier to extend and interconnect.

This architectural difference mattered, but it should not be overstated: the major strategic failure was that AT&T did not create a sufficiently open ecosystem around its technology.

### 4. Unix helped spread the competitor

Bell Labs created Unix, but Berkeley Unix became one of the most important distribution mechanisms for TCP/IP.

Organizations could obtain Unix networking code, inspect it, port it, modify it, and connect machines from many manufacturers. Thus software originating from the Bell Labs ecosystem helped spread the open internetwork architecture that displaced AT&T's preferred networking approaches.

### 5. Bell optimized for carrier-style control

Bell's historical priorities included:

- centrally engineered reliability;
- controlled provisioning;
- long equipment lifetimes;
- integrated network management;
- predictable services.

Those priorities produced technically sophisticated networks but did not encourage the permissionless, rapidly evolving implementation ecosystem that formed around Ethernet and IP.

## Strategic lesson for GNet

GNet must be **fundamentally open**, even if DEC remains the principal designer and best hardware supplier.

The intended policy is:

1. Publish the complete GNet protocol suite and wire formats.
2. Make the core standards permanently royalty-free.
3. Publish reference implementations with source code and permissive redistribution rights.
4. Allow any company to manufacture compatible GNet equipment without DEC permission.
5. Sell inexpensive GNet adapters for non-DEC systems.
6. Sell the **R11-N** and related networking silicon to competitors and OEMs.
7. Publish conformance tests and interoperability suites.
8. Establish an independent standards process as outside adoption becomes significant.
9. Do not require proprietary DEC extensions for basic interoperability.

DEC should view third-party GNet implementations as strategic wins rather than lost peripheral sales.

If Data General, IBM-compatible vendors, PBX companies, industrial-control manufacturers, universities, and telecommunications companies build their own GNet interfaces, the value of the network increases for every participant and for DEC.

DEC's advantage should be:

> We designed GNet, manufacture excellent GNet silicon, and build the best integrated GNet systems.

It must **not** be:

> Only DEC is allowed to build GNet.

## Main historical takeaway

SPIDER and Datakit demonstrate that Bell Labs had sophisticated packet-network ideas well before networking standards settled around Ethernet and TCP/IP. Their limited long-term influence was not proof that packetized virtual channels, hardware flow control, or integrated voice/data networking were inherently bad ideas.

The larger lesson is that **technical quality does not compensate for a closed ecosystem**.

GNet should therefore take the strongest technical ideas from Bell's work while adopting the radically open deployment model that helped Ethernet and TCP/IP win.