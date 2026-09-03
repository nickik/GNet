# Early GNet WAN Strategy and Routed Network Manifesto — 2026-09-03

> [!important] Historical design discussion
> This file archives an exploratory GNet strategy discussion. It does not modify or supersede current protocol specifications unless separately adopted elsewhere in the repository.

## Summary

This discussion explored how GNet could be launched as an open global routed network beginning in 1984, using leased telecommunications capacity rather than requiring ownership of the underlying transmission infrastructure.

Key points:

- Early fiber and WAN markets were examined through FDDI, direct optical links, Sprint, MCI, UUNET, WorldCom, WilTel, Brooks Fiber, ANS, SONET, T1/T3, DDS and microwave-carrier capacity.
- The central strategic model is to make GNet independent of the physical transport: use whichever point-to-point link is economically attractive, including serial lines, DDS, T1/T3, microwave carrier services, fiber, ARCNET, Ethernet and other LAN technologies.
- A plausible early U.S. backbone concept is a small number of major routing centers linked by leased high-capacity circuits, with lower-cost DDS or similar links feeding secondary cities.
- Chicago was identified as a natural central hub because of its historic role as a railroad and telecommunications crossroads, with East Coast and West Coast nodes completing the national structure.
- The discussion considered an early backbone such as Sunnyvale/Los Angeles → Chicago → New York, with additional southern routes through Dallas, Atlanta and Florida.
- A major strategic opportunity would be a close relationship with a low-cost long-distance carrier such as Sprint, allowing GNet to buy transport while concentrating on packet networking, services and customer growth.
- The network should be seeded with inexpensive GNet router/server systems deployed in major cities and sold into ordinary LAN environments. The same system can act as a LAN server, GNet router and WAN gateway.
- Free and open C implementations of GDP routing and parsing are important to adoption. Existing computers should be able to join GNet without purchasing GNet hardware.
- The strongest bootstrap proposition is not any single application, but immediate participation in a global routed network. Once attached, a user or organization can reach remote systems and can publish new services to the entire GNet community.
- The GNet Protocol Suite should scale from a single serially connected client through very large LANs and ultimately a global network with billions of users.
- DNP (Direct Network Protocol) is intended for direct links, while GNet should also operate over existing LAN technologies such as ARCNET and Ethernet.
- GNet should provide ample address space and straightforward gateways to existing public, private, government and research networks, including IP networks.
- A simple file-distribution service was discussed as one potentially attractive early application, but it is secondary to the larger routed-network strategy.

## Routed-network manifesto direction

The revised manifesto should emphasize:

1. A global routed network rather than a particular application or piece of hardware.
2. Free and open tools and specifications that anyone can use, modify and implement.
3. Permissionless growth: anyone can attach a node, extend the network or create a new service.
4. Immediate global reach for newly created services once connected to GNet.
5. Independence from the underlying transmission technology.
6. Deployment ranging from one serial-connected machine to LANs containing hundreds of thousands of nodes.
7. Performance, reliability, large-scale routing and address-space decisions made for future global growth.
8. Interconnection with existing networks rather than requiring their replacement.

## Revised 1984 manifesto draft

**The GNet Manifesto (1984)**  
**The Routed, Open Network for Everyone**

### Introduction

In 1984, communication networks are either rigid, centralized, or entirely disconnected. We believe in a different future: a global, routed network that is open, decentralized, and accessible to all.

**GNet** is that future. It is a protocol suite and architecture that allows anyone—from a single user to global institutions—to participate in a scalable, resilient, and self-extending global network.

### The Core Idea

- **A routed network**, where messages find their way automatically across any number of links
- **Free and open tools** anyone can run, modify, or build upon
- **Designed for scale**, from a lone serial-connected client to a LAN of hundreds of thousands of nodes
- **Protocols designed for performance and growth**
- **Every node is equal** — anyone can join, route, or offer services

GNet is not a proprietary system, not a research project, and not a closed club. It is the beginning of a **global public data network**, built by its users.

### What Makes GNet Unique

- Uses the **GNet Protocol Suite**, including:
  - **GDP** for packet routing
  - **DNP** (Direct Network Protocol) for simple, point-to-point framing
- Operates over any data link:
  - Serial lines
  - DDS
  - ARCNET, Ethernet, and others
- Works on any system with minimal requirements

From the smallest microcomputer to a campus-wide system, GNet grows with you.

### The Power of Openness

- **Anyone can run a node**
- **Anyone can create a new service**, and instantly offer it to the entire global network
- **Anyone can extend the protocol**
- The tools and specifications are available to all, freely and openly

No fees. No gatekeepers. No central control.

### A Network That Grows With You

- Connect a single terminal via serial line, and join the world
- Extend your LAN using GNet routing and addressing
- Interconnect diverse systems over long distances
- Bridge to or from existing private or public networks, including government or IP-based systems

Wherever you start, you can grow.

### Built to Scale

- GNet Protocol supports **large address space**
- **Efficient routing** for small systems and major backbones alike
- Designed for **reliable delivery**, even across heterogeneous networks
- Will scale to **billions of users**

This is not just a patchwork of links. It is a **global architecture**, a foundation for future generations.

### The Call to Connect

- To the curious: you can build your own node
- To the developers: you can define the next generation of services
- To the world: you can join this new network, today

We are not building a tool. We are building **a living, growing, routed network** for humanity.

**GNet, 1984.**

## Source transcript

The remainder is the archived source material from the discussion.

---

[Generated image omitted from archive]

The discussion covered early GNet router/server economics, possible use of inexpensive hardware-assisted routing, deployment in major U.S. cities, leased T1/DDS/microwave/fiber transport, historical comparisons to Sprint, MCI, UUNET and related carriers, and the evolution of the GNet launch manifesto.

The final user direction for the manifesto was:

> Don't mention terms that didn't exist in 1984. Don't focus so much on file sharing. Don't talk about hardware. Focus primary on the idea of a global routed network with free and open tools anybody can use. Anybody can extend the network and develop new services, and instantly make them accessible to the whole world. System can be trivially deployed your LAN used over serial lines, use the Direct Network Protocol (DNP) that is part of the GNET Protocol suite or existing LAN technologies, like ARCNET or Ethernet. From a single client, to your own LAN of hundreds of thousands of nodes. GNet Protocol is built for the future. It allows for reliable communication across the network. Its design is for performance and will scale up to billions of users. Lots of address space, easily possible to route to or from all existing networks, government or private networks, like IP.
