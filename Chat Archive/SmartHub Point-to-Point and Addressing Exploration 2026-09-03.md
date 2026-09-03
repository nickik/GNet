# SmartHub Point-to-Point and Addressing Exploration

> [!WARNING]
> **Historical / outdated discussion.** This file archives an earlier GNet design exploration and a summary of the ideas discussed. It is **not** the current GNet specification and must not be used to override newer protocol, link, packet-format, routing, or hardware documentation.

**Archived:** 2026-09-03  
**Status:** Historical discussion / superseded design exploration  
**Scope:** GPP / Direct Link layering, SmartHub forwarding, shared data-line control, fan-out, and early address-assignment ideas.

---

## Summary

The discussion explored whether GNet should separate a minimal local/link protocol from the routed GNet packet layer, in a manner conceptually similar to Ethernet carrying IP, while retaining GNet's point-to-point-oriented architecture.

The principal direction explored was:

- Keep **GPP / Direct Link Protocol** as the local link mechanism.
- Carry the larger routed **GNet/GDP packet** inside it.
- Avoid Ethernet-style local source/destination addresses if the physical/logical link can remain point-to-point.
- Treat the early **SmartHub** as a controlled concentrator that makes each attached endpoint look logically point-to-point even though the high-speed data path is shared.
- Put link setup, arbitration, grants/credits, and similar scheduling information on the **control pair**, because the data line is shared.
- Do not support passive multi-drop of multiple independent computers behind a single SmartLink port in the base design.
- If fan-out is ever required, use an explicit downstream hub/router device rather than a Y-cable or hidden multi-drop bus.
- A later tree of SmartHubs could theoretically use hierarchical prefix delegation, but that was judged too complex for the earliest SmartHub generation.
- The simplified early SmartHub idea therefore restricts downstream ports to **end clients only**, avoiding recursive prefix delegation and child-hub routing logic.

These were exploratory conclusions only. Newer GNet documents take precedence.

---

## 1. Link-layer split considered

The discussion compared two approaches:

1. A small Ethernet-like local frame carrying 8-bit source and destination identifiers, with GDP inside it.
2. A genuinely point-to-point GPP / Direct Link frame with no local endpoint addresses, carrying GDP directly.

The second approach was preferred in the discussion because an 8-bit local address space would create a second addressing and forwarding domain. That would require additional discovery/mapping rules and would move GNet toward an Ethernet/IP-style two-address system.

The intended conceptual separation was:

```text
Physical/link control
    ↓
GPP / Direct Link framing
    ↓
GDP routed packet
    ↓
Higher transport/session protocols
```

The benefit was architectural separation without introducing Ethernet-style MAC addressing.

---

## 2. SmartHub as logical point-to-point concentrator

A SmartHub was treated as allowing several physical endpoint connections while preserving a point-to-point abstraction at each endpoint.

The important physical constraint raised in the discussion was that the **data path is shared between the participants of the SmartLink system**. Therefore, setup and scheduling cannot assume that every endpoint owns an independent data pair at all times.

The explored model was:

- Endpoint-specific communication with the SmartHub over the **control pair**.
- Shared high-speed data resource used only when an endpoint has been granted permission.
- The endpoint does not transmit arbitrarily onto the shared data path.
- The SmartHub coordinates access centrally.

Conceptually:

```text
            control ─── Host A
           /
SmartHub ──+── control ─── Host B
           \
            control ─── Host C

        shared / scheduled data path
```

Thus the system can be **logically point-to-point while physically sharing the expensive data resource**.

---

## 3. Control-pair responsibilities

The discussion placed the following kinds of functions on the control pair rather than on the shared data path:

- Link discovery / presence detection
- Setup and configuration
- Transmission permission / scheduling
- Flow-control or credit-related signalling
- Link reset and error state
- Potential capability reporting

An illustrative state progression discussed was:

```text
RESET → DISCOVERY → CONFIG → ACTIVE → ERROR
```

This state machine was only illustrative and was not adopted as a current specification.

The important architectural point was that **control traffic must remain available independently of access to the shared data line**.

---

## 4. Why passive fan-out was rejected

The discussion considered what would happen if two independent computers were placed downstream of one nominal point-to-point SmartLink connection.

That was judged undesirable because both machines would then share the same logical link relationship and would need some mechanism to determine which machine responds to control messages and which machine may transmit.

Supporting this would introduce several new requirements:

- Local arbitration between multiple downstream devices
- Local addressing or endpoint identifiers
- Collision avoidance or deterministic bus scheduling
- Discovery of several peers on one link
- Additional failure and contention cases

This would effectively create a small multi-drop LAN/bus protocol and undermine the intended simple point-to-point link semantics.

The resulting design rule proposed in the discussion was:

> **One SmartLink attachment corresponds to one directly attached endpoint. Passive fan-out and Y-cable multi-drop are not part of the base protocol.**

If multiple devices must share the upstream physical connection, they should sit behind an explicit active network device.

---

## 5. Explicit downstream hub idea

A possible later extension was an explicit **leaf hub** or downstream SmartHub:

```text
Router / Hub
    |
    +── SmartLink ── Leaf Hub
                       |
                       +── Host A
                       +── Host B
```

The upstream device would still see one directly connected peer: the leaf hub.

The leaf hub would terminate the upstream link and separately control its downstream endpoints. This preserves the invariant that each individual SmartLink relationship has one peer rather than silently turning a link into a multi-drop bus.

Whether such a device would route GDP, perform prefix delegation, or use some simpler forwarding model was left open.

---

## 6. Tree-of-SmartHubs prefix delegation explored

The discussion then explored how a full tree of routing-capable SmartHubs could configure itself.

The proposed conceptual mechanism was **top-down prefix delegation**:

1. A parent hub/router receives an address prefix.
2. It identifies whether a downstream peer is an end host or another hub/router.
3. It assigns a host address/small allocation to a leaf client or delegates a larger sub-prefix to a downstream hub.
4. The downstream hub subdivides that delegated prefix for its own children.
5. Each child hub uses the parent as its default route.
6. The parent automatically knows that the delegated prefix is reachable through the port on which it delegated it.

Conceptually:

```text
Parent owns P/L

Port 1 → client allocation
Port 2 → client allocation
Port 3 → delegated subtree prefix P'/L'
                    |
                    +── child hub subdivides P'/L'
```

This allows a strict tree to route largely from delegation state rather than requiring a full dynamic routing protocol at every small hub.

However, this approach implies that even small SmartHubs need logic for:

- Device-class discovery
- Prefix allocation
- Prefix tables
- Downstream routing
- Recursive delegation
- Configuration recovery after topology changes

For the earliest generation this was considered unnecessarily complex.

---

## 7. Simplified first-generation SmartHub idea

The final simplification reached in the archived discussion was to make the initial SmartHub a **leaf concentrator only**.

### Allowed topology

```text
End Host ──┐
End Host ──┼── SmartHub ── upstream router/network
End Host ──┘
```

### Explicitly not supported by that early model

```text
SmartHub ── SmartHub ── Host
```

and:

```text
SmartHub port ── passive splitter ── Host A
                              └───── Host B
```

The early SmartHub therefore does not need to understand recursive network topology or allocate sub-prefixes to downstream infrastructure.

Potential simple forwarding approaches discussed included keeping a mapping from a downstream client address to the physical SmartHub port and forwarding all non-local traffic upstream.

The exact registration/address mechanism was not finalized in the discussion and should not be treated as normative.

---

## 8. Architectural ideas worth retaining as historical context

Even though the detailed design is outdated, several architectural questions from the discussion remain useful context when evaluating newer GNet designs:

### Separate link-local and routed responsibilities

A thin local transport/framing protocol can isolate wire-specific behaviour from globally routed GDP semantics without requiring Ethernet-style MAC addressing.

### Keep control available independently of shared data bandwidth

With centrally scheduled shared media, configuration and arbitration traffic benefits from an independent control path.

### Avoid accidental multi-drop semantics

Permitting arbitrary passive fan-out has substantial protocol consequences. If GNet wants deterministic, centrally controlled links, fan-out is cleaner when represented by an explicit active network node.

### Complexity can be deferred by constraining topology

A first-generation product can support only directly connected leaf endpoints. Recursive hubs, delegated prefixes, redundant topology, and more general routing can be introduced in later generations rather than burdening the earliest hardware.

---

## 9. Supersession note

This archive records an intermediate design discussion. In particular, none of the following should be assumed to describe current GNet merely because they appear here:

- GPP frame fields
- GDP header fields
- Address width or prefix format
- SmartHub control messages or state machine
- Host registration mechanism
- Prefix delegation rules
- Routing-table format
- SmartHub forwarding behaviour
- Physical-pair assignment
- Credit/grant semantics

For current behaviour, consult the newest material under the main architecture, media/link, protocol, packet-format, registry, implementation, and decision sections of the repository.
