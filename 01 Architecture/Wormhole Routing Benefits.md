---
id: wormhole-routing-benefits
title: "Wormhole Routing Benefits"
aliases: ["Wormhole routing","Wormhole versus ATM","Wormhole versus Ethernet/IP"]
type: architecture
status: active
layers: ["L2","L3"]
tags: ["gnet","gnet/architecture","gnet/routing","gnet/flow-control","gnet/performance"]
parent: "[[Architecture MOC]]"
related: ["[[GNet Architecture Overview]]","[[Direct Link Protocol]]","[[Virtual Channels and VCIDs]]","[[GNet Link Control Protocol]]","[[Transport and Flows]]"]
updated: 2026-09-07
---
# Wormhole routing benefits

GNet uses wormhole-style forwarding at the link/switching layer: a forwarding node can begin transmitting a package onward before the complete package has arrived. Native GNet combines this with hop-local virtual channels, receiver credits, and separate scheduling grants.

This note explains why that design is attractive, especially under heavy offered load, and where its weaknesses are compared with ATM and conventional Ethernet/IP packet switching.

> [!important]
> Wormhole forwarding is not a substitute for end-to-end congestion control. GNet's hop-local credits prevent a sender from overrunning downstream receive capacity, but sustained overload still requires admission, scheduling, routing policy, and end-to-end GTS/application behavior.

## Historical origin

Modern wormhole routing is generally traced to William J. Dally and Charles L. Seitz at the California Institute of Technology in 1986. Dally's Caltech dissertation and the Caltech **Torus Routing Chip (TRC)** described forwarding flits onward as soon as they arrive rather than buffering an entire packet at every intermediate node. The TRC also introduced the virtual-channel method for constructing deadlock-free routing functions.

The immediate Caltech context was message-passing multicomputers. The earlier **Caltech Cosmic Cube** used store-and-forward routing; the TRC represented the major transition to cut-through/wormhole techniques. Caltech's later **Mosaic C** multicomputer incorporated a two-dimensional hardware router using closely related ideas.

A later actual LAN built around this technology was **ATOMIC**, developed at USC/Information Sciences Institute in the early 1990s using Caltech Mosaic technology. ATOMIC combined Mosaic's native wormhole routing with additional store-and-forward/path-concatenation mechanisms to provide a general-purpose high-speed LAN and TCP/IP compatibility.

### Important historical references

- W. J. Dally, *A VLSI Architecture for Concurrent Data Structures*, Caltech Ph.D. thesis, 1986.
- W. J. Dally and C. L. Seitz, *The Torus Routing Chip*, 1986.
- W. J. Dally and C. L. Seitz, *Deadlock-Free Message Routing in Multiprocessor Interconnection Networks*, IEEE Transactions on Computers, 1987.
- W. J. Dally, *Virtual-Channel Flow Control*, IEEE Transactions on Parallel and Distributed Systems, 1992.
- L. M. Ni and P. K. McKinley, *A Survey of Wormhole Routing Techniques in Direct Networks*, IEEE Computer, 1993.

## Core latency advantage

With store-and-forward packet switching, an intermediate switch normally receives the whole packet before forwarding it. If a packet of length `L` traverses `D` links, serialization contributes at every hop.

In a simplified model:

```text
store-and-forward latency  ~ D * L / bandwidth
wormhole latency           ~ L / bandwidth + D * router_delay
```

The exact equation depends on link and router implementation, but the architectural point is important: **wormhole forwarding pipelines the packet across the path**.

The header can already be several switches downstream while the tail is still entering the network.

For GNet this is particularly useful because GDP packages may be substantially larger than a single physical flit. A switch does not need to wait for an entire GDP package before beginning the next-hop transfer.

## Small and bounded switch buffering

A conventional packet switch must be prepared to buffer complete packets when an output is busy. With variable-sized packets, high-speed links quickly make this memory expensive:

```text
required packet-buffer bandwidth ~= ingress bandwidth + egress bandwidth
```

and buffering multiple full packets per port can dominate switch memory and memory bandwidth.

Classic wormhole routing instead needs only enough storage for a small number of flits per active virtual channel. A blocked package remains distributed across the path and backpressure stops upstream transmission.

For GNet this is one of the central architectural advantages:

- no requirement for a full-package buffer at every hop;
- less high-speed packet memory;
- less memory bandwidth inside the switch;
- a simpler forwarding datapath;
- implementation cost scales more closely with ports and active VCs than with worst-case packet size.

This mattered strongly in the 1980s, when fast SRAM was expensive, but it remains attractive whenever link speed grows faster than economical buffering.

## Backpressure under traffic pressure

GNet receiver credits represent actual downstream flit capacity. A sender cannot legally transmit more flits than downstream capacity guarantees.

This provides a useful property during overload:

```text
congestion
   -> downstream credits disappear
   -> sender stops
   -> pressure propagates upstream
```

The network therefore does not need to accept an arbitrary amount of traffic and later discover that a queue has overflowed.

In a conventional best-effort Ethernet/IP network, congestion is usually absorbed by queues until those queues fill, after which packets are dropped. End systems then infer congestion from loss, delay, or explicit congestion signals. Large buffers can postpone loss but create long queueing delay.

GNet's design moves part of that mechanism into explicit hop-local flow control. Under transient bursts, this can keep the network lossless without requiring deep queues.

### What this does *not* solve

Backpressure turns buffer overflow into blocking. It does not create bandwidth.

Under persistent overload, the blocked dependency can extend over many switches. A congested destination or link can therefore consume VC state and block unrelated traffic unless the network has sufficient virtual channels, scheduling isolation, and suitable routing.

This is the classic wormhole congestion problem.

## Virtual channels are critical

The original wormhole weakness is that one blocked packet can hold the path resources occupied by its body and prevent another packet from using otherwise idle physical bandwidth.

Dally's virtual-channel work addressed this by separating **buffer/resource ownership** from **physical-link ownership**. Several logical channels share one physical link; a blocked packet can occupy one VC while another packet uses another VC over the same wire.

Dally's 1992 simulations reported large throughput improvements from virtual channels for the studied interconnection networks, including about a 3.5x improvement in one fixed-buffer comparison and throughput approaching network capacity in the modeled cases.

GNet follows the same broad principle but keeps the baseline deliberately small:

```text
34-bit physical flit
    = 2-bit hop-local VCID
    + 32 carried bits
```

VCID `0` is reserved for future deadlock-resolution/escape use. Baseline ordinary transfers use VCIDs `1-3`.

This is not the large per-flow VC space of ATM. GNet VCs are cheap, short-lived, hop-local flow-control resources.

## Behavior near saturation

Wormhole routing has two very different regimes.

### Below saturation

It is extremely attractive:

- very small per-hop latency;
- low buffer requirements;
- good physical-link utilization;
- no full-packet store-and-forward delay;
- simple hardware forwarding;
- natural credit/backpressure control.

### At or above saturation

Naive wormhole networks can degrade sharply:

- a blocked packet occupies resources across multiple hops;
- congestion can propagate backward over a large part of the topology;
- head-of-line blocking can waste unrelated output capacity;
- cyclic resource dependencies can deadlock unless routing/VC rules prevent them;
- unfair schedulers can starve some traffic;
- deterministic routing can concentrate traffic on hot links.

John Ngai's 1989 Caltech work reported that highly developed oblivious wormhole routing on the then-current multicomputer networks reached stable sustained throughput around 45-50% of bisection-bandwidth limits under the random-traffic model he studied, motivating adaptive routing to diffuse local congestion.

The lesson for GNet is not that wormhole performs badly under load. It is that **flow control alone is insufficient**. GNet needs the complete system:

1. VC separation so one blocked transfer does not monopolize a physical link;
2. deadlock-free/escape routing rules;
3. fair scheduling and bounded priority service;
4. admission/rate control where appropriate;
5. adaptive or multipath routing where topology permits it;
6. end-to-end congestion behavior above GDP.

With those mechanisms, wormhole forwarding retains its latency and buffering advantages without relying on unlimited queues.

# Comparison with ATM

ATM and GNet share more ideas than their packet formats initially suggest. Both aim to make switches fast by moving complexity into compact hardware forwarding operations rather than performing general-purpose packet processing at every hop.

However, their resource models are substantially different.

| Property | GNet wormhole | ATM |
|---|---|---|
| Transfer unit | Variable GDP package carried as flits | Fixed 53-byte cell |
| Intermediate forwarding | Flit-level cut-through/wormhole | Cell switching |
| Flow identity in fabric | Small hop-local VCID | VPI/VCI virtual circuit identifiers |
| Buffer requirement | A few flits per VC can be sufficient | Switch queues cells |
| Connection model | GDP remains connectionless/routed | Connection-oriented virtual circuits |
| Backpressure | Native credit-based hop control | Depends on ATM mechanism/profile; not fundamental to basic ATM service |
| Large-message overhead | No mandatory conversion into tiny independent cells | Segmentation into 48-byte payload cells and reassembly |
| QoS model | Scheduler/QoS policy plus higher-layer mechanisms | QoS and traffic contracts central to architecture |
| Failure/congestion style | Blocking/backpressure unless isolated | Cells normally queued; sophisticated admission/traffic management possible |

## Where GNet has an advantage over ATM

### 1. No 48-byte segmentation tax

ATM's 53-byte cell contains a 5-byte header and only 48 payload bytes. Large packets must be segmented into many cells and reassembled at the far side.

GNet can preserve a large logical package while physically forwarding it a flit at a time. The switch gets the hardware advantages of small transfer units without making the network-layer packet itself a sequence of independently addressed 53-byte cells.

### 2. Less per-unit header processing

Once a GNet worm has established its next-hop forwarding state, body flits follow that state. The network does not need to repeat a large virtual-circuit header for every small portion of the original package.

### 3. Connectionless routing remains natural

ATM's virtual circuits make resource reservation and QoS natural, but setup and per-connection state are fundamental architectural concepts.

GDP remains a routed datagram. GNet's hop-local VCID is not an end-to-end circuit identifier and is released after the local transfer.

### 4. Potentially less buffering

A credit-controlled wormhole switch can be built around very small per-VC buffers rather than deep cell queues.

That reduces SRAM requirements and memory traffic, which is particularly important for a historically plausible DEC implementation.

## Where ATM has an advantage

ATM's architecture makes traffic contracts, admission control, explicit virtual circuits, and service guarantees central concepts. A sufficiently engineered ATM network can isolate heavy users and maintain fairness under severe congestion better than a simplistic lossless wormhole network.

A 2001 simulation study comparing TCP over wormhole and ATM networks found that comparable TCP throughput required more hardware/software complexity in the ATM case, but that ATM handled severe congestion and fairness better in the configurations studied.

That is an important warning for GNet: **small buffers and backpressure are not automatically superior during sustained overload**. The GNet scheduler, VC allocation policy, routing system, and higher-layer congestion control must prevent a few blocked worms from dominating network resources.

# Comparison with Ethernet/IP-style packet switching

"Ethernet/IP" is not one switching algorithm. Ethernet switches can themselves perform cut-through forwarding, and IP routers can use many internal fabrics. The comparison here is specifically with the conventional datagram design in which complete packets are accepted into packet queues and congestion is handled principally through queueing and packet loss.

| Property | GNet wormhole | Conventional Ethernet/IP packet fabric |
|---|---|---|
| Forwarding granularity | Flits | Packets/frames |
| Typical forwarding | Pipeline before whole package arrives | Store-and-forward is common; cut-through also exists |
| Buffering | Small per-VC flit buffers | Packet queues, often large |
| Local overflow behavior | Credits stop upstream transmission | Queue fills, then packet is dropped |
| Congestion visibility | Immediate hop-local backpressure | Queue delay/loss/ECN observed by endpoints |
| State in core | Small temporary hop-local VC state | Usually no per-flow IP forwarding state, but substantial queue state |
| End-to-end congestion | Still required | TCP/transport congestion control is fundamental |
| Long packet at every hop | Serialization is pipelined | Store-and-forward repeats receive-before-forward delay |

## Main GNet advantages

### Lower latency across multiple switches

The package does not pay complete receive-then-transmit serialization at every hop.

### Much smaller fast-memory requirement

A switch does not need enough fast memory to absorb many maximum-sized GDP packages on every port.

### No packet loss merely because an intermediate queue happened to fill

Credit exhaustion stops transmission before downstream capacity is overrun.

This is particularly useful for a network carrying mixtures of computer data, terminal traffic, RPC, storage, and real-time voice where retransmission caused by local queue overflow is undesirable.

### Congestion is visible immediately to upstream hardware

The pressure appears as a lack of credits/grants rather than only after milliseconds of queue growth and eventual packet loss.

### Router cost can remain low as line rates rise

Memory capacity and especially memory bandwidth are major costs in packet switches. Wormhole forwarding permits switch performance to track the switching fabric and link logic rather than requiring proportionally deeper and faster packet memories.

# Traffic-pressure example

Consider several ingress links sending large packages toward one saturated egress.

## Deep-buffer packet network

```text
input -> packet queue -> packet queue -> packet queue -> bottleneck
                         queues grow
                         latency grows
                         buffers eventually overflow
                         packet loss/retransmission
```

The network temporarily hides overload in memory.

## GNet wormhole network

```text
input -> VC -> VC -> VC -> bottleneck
                         no credit
                  <- backpressure <-
```

The overloaded path stops consuming new downstream storage almost immediately.

This makes GNet attractive when the objective is **bounded buffering and controlled overload rather than absorbing arbitrary bursts into queues**.

But if the blocked worm holds the only usable VC on upstream links, congestion spreads. Therefore the real GNet design target is:

```text
bounded buffers
+ multiple VCs
+ fair arbitration
+ deadlock escape
+ traffic classes
+ adaptive routing where useful
+ end-to-end congestion control
```

not wormhole routing alone.

# Architectural benefits for GNet

The strongest reasons to retain wormhole-style forwarding are therefore:

1. **Low multi-hop latency** — routing and transmission are pipelined across the network.
2. **Tiny intermediate buffers** — switches need flit storage rather than full-package storage.
3. **Low memory bandwidth** — packet payloads do not repeatedly enter and leave large intermediate packet memories.
4. **Natural hardware implementation** — route setup, VC state, credits, and crossbar forwarding map cleanly onto dedicated logic.
5. **Fast hop-local congestion response** — credits expose unavailable capacity before buffer overflow.
6. **Loss avoidance for transient congestion** — a blocked receiver applies backpressure instead of forcing immediate packet loss.
7. **Variable-sized network packets without ATM-style segmentation/reassembly** — GDP retains useful package sizes while the forwarding hardware operates on small flits.
8. **Cheap statistical multiplexing** — hop-local VCs allow many routed conversations to share links without establishing end-to-end circuits.
9. **Scalability with link speed** — required high-speed buffering grows much more slowly than in a design based on deep packet queues.
10. **Good fit for mixed traffic** — virtual channels and scheduling can isolate REALTIME/control work from blocked bulk transfers without converting the network to end-to-end circuit switching.

# Design cautions for GNet

The historical literature is equally clear about what must not be ignored:

- wormhole networks are especially vulnerable to deadlock if channel dependencies contain cycles;
- one blocked worm may occupy resources across several nodes;
- too few virtual channels permit severe head-of-line blocking;
- too many VCs add buffers, allocator state, and arbitration complexity;
- persistent overload propagates backward rather than disappearing;
- deterministic routes can produce hot spots well before aggregate network capacity is exhausted;
- priority traffic needs anti-starvation rules;
- lossless hop flow control does not replace end-to-end congestion control;
- adaptive routing must itself preserve deadlock freedom or use an escape VC/routing class.

GNet should therefore regard **VC allocation, credit accounting, deadlock freedom, scheduling fairness, and congestion control as part of the routing architecture**, not implementation details.

# Historical sources

- William J. Dally, *A VLSI Architecture for Concurrent Data Structures*, California Institute of Technology, 1986. https://thesis.caltech.edu/1122/
- William J. Dally and Charles L. Seitz, *The Torus Routing Chip*, Caltech, 1986. https://authors.library.caltech.edu/records/99gpd-5kg37
- William J. Dally and Charles L. Seitz, *Deadlock-Free Message Routing in Multiprocessor Interconnection Networks*, IEEE Transactions on Computers 36(5), 1987.
- William J. Dally, *Virtual-Channel Flow Control*, IEEE Transactions on Parallel and Distributed Systems 3(2), 1992.
- Lionel M. Ni and Philip K. McKinley, *A Survey of Wormhole Routing Techniques in Direct Networks*, IEEE Computer 26(2), 1993.
- John Y. Ngai, *A Framework for Adaptive Routing in Multicomputer Networks*, California Institute of Technology, 1989. https://thesis.library.caltech.edu/630/
- Emilio Leonardi, Fabio Neri, Mario Gerla, and P. Palnati, *Congestion Control in Asynchronous, High-Speed Wormhole Routing Networks*, IEEE Communications Magazine, 1996.
- Robert Felderman, Annette DeSchon, Danny Cohen, and Gregory Finn, *ATOMIC: A High-Speed Local Communication Architecture*, USC/ISI, 1990s.

## Bottom line

GNet's advantage is not merely that wormhole routing is "faster." It is that the network can obtain **low latency, low switch-memory cost, and immediate flow control at the same time**.

Compared with ATM, GNet avoids turning every large transfer into a stream of independently headed 53-byte cells and avoids making an end-to-end virtual circuit the basic routed abstraction. Compared with conventional buffered Ethernet/IP switching, it avoids using deep packet queues as the first response to contention.

The trade is that congestion becomes a resource-dependency problem. The network must deliberately prevent backpressure, VC occupation, and routing dependencies from turning a local bottleneck into global blockage. The classic research on virtual channels, deadlock-free routing, adaptive routing, and fair scheduling is therefore directly relevant to the GNet design.