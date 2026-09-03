---
id: gnet-dominant-networking-evolution
title: "GNet-Dominant Networking Evolution"
aliases: ["GNet alternate networking history","GNet vs Ethernet IP evolution"]
type: history
status: working
layers: ["L2","L3","L4","L5"]
tags: ["gnet","gnet/history","gnet/architecture","gnet/qos","gnet/mobility","gnet/security","gnet/telephony"]
parent: "[[History MOC]]"
related: ["[[GNet Architecture Overview]]","[[Addressing and Routing]]","[[Transport and Flows]]","[[Security and Identity]]"]
updated: 2026-09-03
---
# GNet-dominant networking evolution

> [!info] Knowledge graph
> **Up:** [[History MOC]] · **Related:** [[GNet Architecture Overview]] · [[Addressing and Routing]] · [[Transport and Flows]] · [[Security and Identity]]

This note records the architectural consequences discussed for a history in which GNet, rather than Ethernet/IP, becomes the dominant general-purpose networking architecture. It is a historical/strategic interpretation, not a frozen wire specification.

## Core divergence

GNet starts from point-to-point links and routed network-layer forwarding rather than shared Ethernet segments and transparent bridges. There is therefore no requirement for a large bridged Layer-2 domain as the normal LAN abstraction.

Consequences:

- no Ethernet-style MAC-learning fabric as the foundation of the network;
- no Spanning Tree Protocol requirement to suppress forwarding loops in bridged LANs;
- no need for VLANs as the primary mechanism for carving one physical bridged fabric into separate logical LANs;
- no VTP-style VLAN database distribution problem;
- redundancy and path choice belong to routing and the GNet control plane;
- access switches are naturally multi-port GNet forwarding/router nodes rather than transparent bridges.

By the 1990s, enterprise and campus networks therefore resemble an all-routed fabric much earlier than equivalent Ethernet/IP networks.

## VLAN, STP, and VTP in the non-GNet history

These technologies solve problems created by large Ethernet Layer-2 domains:

- **VLAN (802.1Q):** tags frames so one switched Ethernet fabric can contain multiple logical broadcast domains.
- **STP:** disables selected bridge paths so a physical graph containing loops behaves as a loop-free tree and does not produce persistent frame/broadcast loops.
- **VTP:** Cisco control protocol for distributing VLAN definitions among switches.

GNet does not need direct equivalents for ordinary networking because segmentation, isolation, redundancy, and reachability are Layer-3/control-plane functions from the beginning.

## Telephony as a first-class workload

A major intended GNet advantage is early support for large numbers of simultaneous telephone and other low-latency real-time connections.

Early shared Ethernet is poorly matched to this workload because CSMA/CD contention and collision backoff create variable access delay, while relatively large frames can impose serialization delay on latency-sensitive traffic. Basic Ethernet also did not begin with strong admission control or deterministic scheduling for telephone traffic.

GNet instead combines several architectural choices:

- point-to-point or centrally scheduled links rather than uncontrolled shared-medium contention;
- bounded packet/link size classes, allowing small real-time transfers without waiting behind very large bulk transfers;
- explicit QoS/priority information;
- reserved real-time flows and admission control at higher layers;
- VCID-based interleaving and preemption on links where supported;
- routing and scheduling that can distinguish telephone/interactive traffic from bulk transfer.

This makes integrated voice/data networking plausible much earlier. A GNet telephone is simply a network endpoint using a reserved low-latency flow rather than a device requiring a parallel TDM network or a later VoIP retrofit.

## MPLS comparison

MPLS in the IP world adds short labels that routers swap hop-by-hop. Operators use it for predictable forwarding, explicit or constrained paths, traffic engineering, fast reroute, and large carrier VPN services.

A simplified MPLS path is:

```text
IP packet
  -> ingress classifies packet and pushes label
  -> core routers forward/swap using label
  -> egress removes label and resumes ordinary forwarding
```

Many of the motivations behind MPLS are already natural GNet control-plane concerns: path selection, QoS, reserved flows, traffic engineering, and regular hardware forwarding. GNet may still develop local path identifiers or label-like forwarding optimizations, but it does not require an MPLS layer merely to escape the limitations of a pure best-effort IP forwarding model.

The important distinction is that GNet should not blindly copy MPLS. Any label/path mechanism should remain an optimization or control-plane tool and should not replace the normal GDP address model.

## SDN evolution

In Ethernet/IP history, software-defined networking frequently had to manage or work around Layer-2 constructs such as VLANs, bridge domains, spanning trees, overlays, and virtual switching.

In GNet, SDN-like control is narrower and more direct:

- collect topology, capacity, delay, fault, and utilization information;
- compute routes and alternate paths;
- perform traffic engineering;
- establish or modify reserved flows;
- apply organizational and security policy;
- update routes when mobile endpoints change attachment point.

Thus GNet SDN is primarily a programmable routing/traffic-engineering control plane rather than a mechanism for creating a more useful network on top of a bridged Ethernet substrate.

## Mobile clients

Mobility is intended to be a first-class architectural concern rather than an afterthought tied to a bridged LAN.

The desired separation is:

```text
identity / session
        !=
current physical attachment point
```

A terminal, portable computer, telephone, or later radio device may move to another GNet edge without requiring the logical session to be defined solely by the old topological address. The transport/session layer can maintain persistent tunnel/session identity and authorize rebind, while the network/control plane updates reachability toward the endpoint's new attachment point.

Wireless access points therefore need not behave as Ethernet bridges. A radio interface can be another GNet access link, and handoff is fundamentally an attachment/routing update.

This architecture should reduce the need for later Mobile-IP-style home-agent tunnelling and Layer-2 roaming tricks, although exact mobility mechanisms remain protocol work.

## VPN evolution

VPN remains useful, but its meaning changes.

In Ethernet/IP systems VPN products often combine several unrelated jobs: encryption, IP tunnelling, Layer-2 emulation, private-address handling, NAT traversal, and creation of virtual routing domains.

In GNet these concerns can be separated:

1. **link protection** — authenticate and optionally encrypt a directly connected point-to-point link;
2. **end-to-end/session protection** — authenticate peers and encrypt an application/transport session across arbitrary routers;
3. **routing/policy membership** — decide which Org/Division/services an authenticated endpoint or site may reach.

A site-to-site GNet VPN can therefore be a secure association between edge routers plus route/policy exchange rather than a fake Ethernet segment. Remote access becomes authenticated membership in an organization's routing/policy domain rather than pretending that a remote machine is plugged into an office LAN.

## Link-layer security evolution

The discussion used the name **GPP** for the basic point-to-point link protocol. The current repository specification calls this layer **DLP/GNET-LINK**; this note follows the current repository terminology.

Because DLP operates only between directly connected peers, it is a natural place to evolve optional link-local capabilities without changing GDP:

- peer/link authentication;
- encryption of link payloads;
- integrity/replay protection;
- negotiated link capabilities;
- richer speed/duplex/capacity reporting;
- error, congestion, and performance statistics;
- operational and maintenance control messages.

These should normally be negotiated or carried in control messages rather than enlarging every data packet.

Early link encryption is historically plausible first on valuable point-to-point trunks, leased circuits, military networks, banks, and similar installations. It protects against tapping of the individual medium, but routers still decrypt and re-encrypt at every hop.

## End-to-end encryption

Later GNet systems add encryption at the transport/session level. This is distinct from link encryption:

```text
Application
    -> session security / encrypted GTS stream
        -> GTS
            -> GDP
                -> DLP with optional link security
                    -> physical link
```

Link encryption protects one hop. Session encryption protects the communicating endpoints across all intermediate routers and operators. Both can operate simultaneously.

GTS already provides the appropriate architectural home for optional end-to-end encryption and persistent tunnel/session identity. Cryptographic algorithm selection, credentials, key exchange, and historical implementation profiles remain separate design questions.

## Expected 1990s and early-2000s outcome

If GNet wins broadly, several technologies that appear as later corrections in the Ethernet/IP world either disappear or develop differently:

| Ethernet/IP history | Likely GNet outcome |
|---|---|
| Large bridged LAN | Routed point-to-point access fabric |
| STP/RSTP/MSTP | Not required for normal forwarding |
| VLANs | Address/routing/policy domains |
| VTP | No direct equivalent required |
| ATM promoted for deterministic voice/data convergence | Less compelling because GNet natively supports small, prioritized/reserved traffic |
| MPLS introduced for labels, TE, VPNs | Some functions native to GNet control plane; optional path/label optimization may still appear |
| Mobile IP and L2 roaming workarounds | Session identity plus routing/rebind mobility |
| VPN as tunnelling plus virtual LAN/IP machinery | Secure membership, routing policy, and optional encrypted paths/sessions |
| SDN overlays over Ethernet | Direct programmable GNet routing, reservation, and policy control |

## Design principle

The important historical claim is not that GNet eliminates the need for advanced network control. It eliminates much of the need to reconstruct advanced network behavior on top of a transparent bridged LAN.

The resulting trajectory is:

> **routed point-to-point fabric first; QoS, telephony, mobility, security, and programmable control evolve directly on that fabric.**
