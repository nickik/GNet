# Service Names, RPC, DNS, and Internet Plumbing — 2026-09-03

> [!important] Historical design discussion
> This archive records an exploratory GNet discussion. It is **not normative**. Where the discussion conflicts with accepted ADRs or [[Specification Status]], the accepted/current documentation wins.
>
> In particular, several intermediate statements in the conversation are superseded by the current architecture: GDP is a minimal routed datagram and carries **no checksum, Flow Control ID, receive window, session ID, or transport state**; DLP credits/VCIDs are hop-local and do **not** replace end-to-end GTS sequence/ACK/retransmission semantics. The current baseline native flit profile is **32 bits = 2-bit VCID + 30 carried bits**; VC4 (`4 + 28`) is a future negotiable profile.

## Discussion summary

This discussion explores how the layers above GDP should expose services and how GNet should replace the collection of supporting protocols that make an Internet-scale network operational.

### Human-readable service names instead of numeric ports

The core proposal is to replace user-visible numeric port numbers with short service names during connection or stream establishment.

Examples:

```text
localhost:ssh
host:sysinfo
router:dns
```

The proposed service name is at most seven characters in the original idea. The name is carried only during `CONNECT` / `STREAM_OPEN`-style setup and is then replaced by the established tunnel/stream state, so the additional setup bytes do not become per-packet overhead.

The important architectural interpretation is that these are **service selectors**, not conventional TCP/UDP port numbers rendered as text. A receiving host maps a service name to a local service endpoint/process/interface. Standard names can be registered globally, while hosts may expose additional names.

This gives GNet a naturally discoverable service model: a host can expose standard services such as `sysinfo`, and a discovery interface can enumerate the services and RPC interfaces available on that host.

### Layering clarified

The conversation converged on the following conceptual stack:

1. **DLP / link layer (L2)** — hop-local physical/link transfer, flits, VCIDs, credits, grants/on-off signaling, framing and hop-local accidental-error detection.
2. **GDP / network layer (L3)** — global addressing, routing, Hop Limit, QoS, packet size class, and best-effort routed datagrams. It deliberately contains no transport/session state.
3. **GTS / transport + session (L4/L5)** — tunnel/session establishment, service selection, streams, reliability modes, sequencing/ACK/retransmission, receive flow control, end-to-end integrity, optional encryption, and reserved real-time flows.
4. **RPC / invocation layer (roughly L6)** — common method/interface invocation and structured request/response semantics over GTS streams.
5. **Application/service layer (L7)** — actual named services such as system information, name resolution, directory, mail, file service, terminal service, etc.

The earlier discussion referred to L4/L5 as one combined GNet session/stream protocol because GNet's tunnel establishment and transport behavior are intentionally designed together rather than reproducing an OSI-style standalone session protocol. The current repository names this transport/session family **GTS**.

### DNS/name resolution as a native GNet service

A key idea is that Internet-style name resolution does not need to be embedded into GDP. It can be a standard GNet Layer-7 service over the RPC layer, analogous in architectural placement to DNS-over-HTTP but native to GNet rather than tunneled through HTTP.

For example:

```text
router:dns
```

could expose RPC operations such as resolving a host/service name, returning one or more GNet addresses, TTL/cache information, aliases, and potentially service metadata.

This preserves the useful properties of DNS — hierarchical naming, delegation, caching and globally distributed administration — without requiring the historical DNS wire protocol to become part of the GNet network layer.

### Service discovery

The short-name system naturally extends into remote service discovery. A host could expose a standard discovery/sysinfo RPC interface that answers questions such as:

- which service names are present;
- which RPC interfaces/methods they implement;
- protocol/interface versions;
- capability and authentication requirements;
- optional human-readable description;
- whether a service is local, proxied, or delegated elsewhere.

This makes a remote machine more self-describing without requiring every application to invent a separate discovery protocol.

### Shared Flow ID / Tunnel ID proposal

During the conversation an optimization was proposed: instead of carrying both an L3 Flow Control ID and an L4 Tunnel ID, use the same identifier and increase it to roughly **27 bits** to reduce collision/reuse risk.

That proposal is retained as historical design exploration, but it does **not** match the current accepted GDP baseline. Current GDP intentionally has no Flow Control ID or session identity; routers should not require ordinary GTS transport state for forwarding.

If the idea is revisited, the useful part is the general principle — avoid redundant identifiers across adjacent layers — but it should be evaluated within GTS or an explicitly defined reservation/QoS mechanism rather than silently adding transport identity back into baseline GDP.

### Internet-operational functions GNet still needs

The discussion distinguished high-level applications from the protocols/functions fundamental to operating a global routed network. GNet needs native answers for the following broad functions:

| Function | GNet direction |
| --- | --- |
| Bootstrap and address delegation | GCTL-style local control, recursive delegation and status exchange |
| Route computation/exchange inside an administrative domain | GNet IGP/control protocol, conceptually comparable to IS-IS/OSPF but designed for GNet addressing/topology |
| Inter-domain routing and policy | GNet exterior route/policy exchange, conceptually comparable to BGP |
| Error/control reporting | GDP/GCTL control messages for unreachable, Hop Limit expiration, malformed/unsupported traffic, etc. |
| Multicast/group membership | Native group membership and routed multicast semantics if multicast is adopted |
| Name resolution | Layer-7 `dns`/name service over RPC/GTS |
| Time synchronization | Standard time service over GTS/RPC; exact protocol remains open |
| Authentication/encryption/key establishment | Primarily GTS/session and higher-layer security rather than GDP baseline |
| Management/telemetry | Standard RPC services such as `sysinfo`, routing status and device management |

Several direct Internet analogies from the exploratory discussion should **not** be copied mechanically. GNet point-to-point links do not require Ethernet ARP/MAC-learning semantics. GDP already has explicit size classes and no IP-style fragmentation, so IP PMTU machinery is not a direct fit. GNet's own bootstrap/delegation design is intended to replace DHCP-style configuration rather than reproduce DHCP.

### Wormhole-style link flow control

The conversation then considered GNet's wormhole/cut-through direction with hop-local VCIDs, receiver credits, infrastructure grants and on/off backpressure.

The durable distinction is:

- **DLP credit** means downstream buffer capacity is available for a physical flit.
- **GRANT/scheduling** controls when reserved credit may be consumed on shared infrastructure.
- **VCID** is hop-local and may be replaced at every forwarding node.
- These mechanisms prevent local buffer overrun and support cut-through forwarding.
- They do **not** by themselves prove end-to-end delivery across a multi-hop path.
- Reliable GTS streams therefore still need end-to-end sequencing, acknowledgments/retransmission, integrity and appropriate receive/congestion control.

This is especially important because an earlier assistant response incorrectly suggested removing GTS sequence/ACK/window behavior merely because DLP has credits. The current GNet architecture explicitly keeps those concerns separate.

## Design ideas worth carrying forward

- Treat short human-readable service names as first-class **GTS service selectors**, not aliases for legacy numeric ports.
- Carry the service name only during setup; ordinary stream data should use compact established identifiers.
- Standardize a minimal registry of globally meaningful service names, while allowing local/private names.
- Put a common RPC/interface-description layer above GTS so `sysinfo`, name resolution, management and future distributed services use one invocation model.
- Define GNet name resolution as a native standard service over GTS/RPC rather than putting DNS semantics into GDP.
- Keep DLP's VCID/credit/grant machinery strictly hop-local.
- Keep GDP minimal and stateless with respect to transport.
- Design GTS reliability/session behavior independently of DLP backpressure, while exploiting DLP signals where useful for performance.
- Avoid duplicated identifiers when possible, but do not collapse layer boundaries merely to save a few header bits.

## Open questions created by this discussion

1. **Service-name encoding:** fixed 7-byte field, length-prefixed 1–7 byte name, packed character alphabet, or registry-assigned compact ID after negotiation?
2. **Namespace:** who owns standard names, are names case-sensitive, and how are vendor/private names represented?
3. **Binding point:** is the service selected when opening a tunnel, when opening an individual stream, or both?
4. **Discovery:** should service enumeration be part of a general `sysinfo` service, a dedicated discovery service, or a standard RPC reflection mechanism?
5. **RPC layer:** exact interface-description, method-ID, error model and version-negotiation semantics remain to be designed.
6. **Name service:** exact GNet DNS replacement/compatibility model, record types, delegation, caching and bootstrapping remain open.
7. **Routing protocols:** GCTL bootstrap/delegation is not by itself the complete IGP/EGP design; global route exchange and policy still need explicit specifications.
8. **GTS identifiers:** the current 64-bit Tunnel ID / Reset ID / stream-width proposals remain draft; the 27-bit shared-ID idea should be evaluated against collision probability, reboot/reuse safety and mobility/rebinding requirements.
9. **Congestion control:** hop-local credits provide backpressure, but a global routed network still needs a policy for persistent multi-hop congestion, fairness and avoiding head-of-line/backpressure propagation pathologies.

## Relationship to current repository

Current accepted architecture relevant to this discussion:

- [[Direct Link Protocol]] / [[Virtual Channels and VCIDs]] — hop-local flits, VCIDs, credit/grant behavior.
- [[GDP Datagram]] / [[ADR-0015 Restore Minimal GDP Header]] — minimal stateless routed L3 package.
- [[Transport and Flows]] / [[GTS Protocol]] — tunnel-first endpoint transport/session layer above GDP.
- [[Specification Status]] — precedence and current frozen baseline.

The service-name, RPC reflection/discovery and native name-service ideas should remain proposals until they are promoted into dedicated architecture/protocol notes or ADRs.

## Archived discussion outline

The conversation progressed through these stages:

1. Proposed replacing numeric ports with <=7-character service names such as `localhost:ssh` and `host:sysinfo`, with the name present only during setup.
2. Proposed using those names to expose standard RPC interfaces and discover services on remote hosts.
3. Clarified that RPC is a separate layer above GNet transport/session, not part of GDP.
4. Revisited the GNet stack and the reason transport/session are treated together as L4/L5.
5. Proposed reusing an L3 flow identifier as the transport Tunnel ID, perhaps widened to 27 bits.
6. Proposed implementing DNS/name resolution as a native Layer-7 GNet service over the RPC layer.
7. Surveyed the Internet's operational plumbing — routing, control/error reporting, configuration, multicast, security, time and management — and considered which functions GNet needs native equivalents for.
8. Noted that GNet's wormhole-style links already use on/off and credit-based signaling.
9. Corrected the resulting overreach: hop-local credits/VCIDs help link flow control but do not eliminate end-to-end GTS reliability and congestion responsibilities.

The later portion of the source conversation is preserved from the attached project chat export; it includes exploratory comparisons with DNS, DHCP, ICMP, OSPF/IS-IS/BGP, multicast, IPsec, ECN and LLDP. Those comparisons are useful as requirements prompts, not as prescriptions to copy the Internet protocols directly.
