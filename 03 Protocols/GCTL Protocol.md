---
id: gctl-protocol
title: "GNet Control Service and Control Message Protocol"
aliases: ["GCTL","GNet Control Protocol","GCMP","GNet Control Message Protocol","GCS","GNet Control Service"]
type: protocol
status: draft
layers: ["L3"]
tags: ["gnet","gnet/protocol","gnet/status/draft","gnet/layer/l3","gnet/management","gnet/oam"]
parent: "[[Protocols MOC]]"
related: ["[[GDP Protocol]]","[[GNet Link Control Protocol]]","[[Discovery Packets]]","[[Address Configuration Packets]]","[[GCTL Message Registry]]","[[Addressing and Routing]]"]
updated: 2026-09-07
---
# GNet Control Service and Control Message Protocol

Status: **DRAFT — architecture and base message model proposed**

GNet needs more than packet-error reporting. Every GNet node should provide a small, uniform set of diagnostics, while routers and switches should expose enough structured state to diagnose routing, queueing, credit pressure and link failures without requiring vendor-specific consoles.

This specification defines one control suite with two closely related parts:

- **GCMP — GNet Control Message Protocol**: compact routed Layer-3 control/error/diagnostic messages carried as GDP Type `GCTL` (`0x01` in the current draft registry).
- **GCS — GNet Control Service**: the mandatory read-only status and management service implemented with GCMP request/reply messages.

**GCTL** remains the umbrella name for the complete GNet network-control suite, including bootstrap messages already assigned to GCTL.

The design goal is approximately: **ICMP-like packet errors + Chaosnet-like mandatory STATUS + DECnet-like structured counters/events, without making Layer 3 a remote-administration system.**

---

## 1. Architectural boundary

GCTL is network-level control associated with GDP reachability, bootstrap, diagnostics and management.

It is distinct from hop-local link control:

- **GLCP** owns HELLO, capability/rate negotiation, link credit, GRANT/CREDIT, VC allocation/release, hop-local ABORT/RESET and physical/link state.
- **GDP** owns only routed datagram metadata: Version, Type, Size Class, Hop Limit, QoS, Source and Destination.
- **GCMP/GCS** report routed failures and expose network state.
- **GTS and higher layers** own end-to-end reliability, transport congestion behavior, sessions and authenticated administrative applications.

GCMP MUST NOT become a second link-flow-control protocol. In particular, it has no CREDIT, GRANT or receive-window mechanism.

### 1.1 Core principles

1. **Mandatory minimum diagnostics.** Every addressed GNet node implements ECHO and basic STATUS.
2. **Routers expose forwarding state.** A conforming router exposes interfaces, neighbors, routes, queues and mandatory counters.
3. **Errors are best-effort information, not reliability.** Failure to receive a GCMP error never implies successful delivery.
4. **Keep control messages bounded.** Automatic error traffic must stay small even when the network is distressed.
5. **No error storms.** Never recursively report errors about errors; rate-limit generated control traffic.
6. **No Layer-3 remote superuser.** The mandatory GCS is read-only. Reboot, firmware load, memory access, route installation and arbitrary configuration changes require a separately authenticated management facility.
7. **Congestion control is not GCMP Source Quench.** GNet already has hop-local credit/backpressure and higher-layer transport behavior. GCMP may report congestion as telemetry, but endpoints MUST NOT treat an unauthenticated control message as a command to throttle a flow.
8. **Management data is structured, not formatted terminal text.** Human-readable tools render typed records returned by GCS.

---

## 2. Lessons from earlier networks

| System | Useful mechanism | GNet lesson |
|---|---|---|
| ARPANET 1822/1822L | explicit network-interface messages such as destination dead, incomplete transmission, interface reset, IMP going down; trace indication | report precise forwarding failures; provide explicit path diagnostics and planned-shutdown events |
| Pup | simple end-to-end datagram architecture; error Pup on forwarding failure/congestion; idempotent stateless request/reply services | requests should be retryable and servers should not require per-query state |
| XNS | separate well-known Echo, Error and Routing Information protocols | keep basic diagnostic/error semantics small and separately identifiable |
| IP/ICMP | destination unreachable, time exceeded, parameter problem, echo; offending-packet quotation; no recursive errors | use compact automatic errors containing enough of the original packet to identify the failed operation |
| Chaosnet | mandatory STATUS on every host; name, connected subnets and packet counters; routing-table dump facilities | make operational introspection part of the base network, not an optional afterthought |
| DECnet NICE/Event Logger/MOP | typed node/circuit/line information, counters, events and tests | use a common entity/attribute model, counters and asynchronous events; keep dangerous maintenance actions out of unauthenticated GCMP |
| AppleTalk AEP/RTMP | tiny echo protocol separated from route maintenance; route aging | keep ECHO trivial and do not overload diagnostics with route-distribution semantics |
| SNMP | very small request model over datagrams; management information defined separately from transport; table walking and asynchronous traps | separate the management object namespace from the wire protocol; support bounded enumeration and event reports |
| OSI CMIP | rich managed-object/action/event model | retain typed objects and events, but avoid association and encoding complexity in the base GNet control plane |

A later Internet lesson is also adopted explicitly: **Source Quench-style control messages are not a sound congestion-control mechanism.** Congestion telemetry is useful; unauthenticated router commands that directly alter transport sending behavior are not.

---

## 3. GCMP carriage

GCMP is carried in an ordinary GDP datagram with GDP Type `GCTL`.

The GDP Source and Destination fields provide GCMP addressing. GDP Hop Limit applies normally. GDP Size Class defines the exact GCMP payload size; GCMP therefore does not need a total-length field.

GCMP messages normally use only:

- `ctrl32B` — 32-byte payload
- `ctrl64B` — 64-byte payload
- `msg128B` — 128-byte payload
- `msg256B` — 256-byte payload

Automatic packet-error reports SHOULD use `ctrl32B`.

GCS implementations SHOULD NOT use GDP classes larger than 256 bytes for ordinary management. Large tables are enumerated with bounded request/reply operations instead of being dumped in one packet.

Control traffic remains subject to DLP/GLCP credit and scheduling rules. Implementations SHOULD reserve a small bounded network-control queue/buffer allowance so diagnostics can still make progress under heavy user traffic, but control traffic MUST NOT bypass link backpressure or be allowed to consume unbounded buffering.

---

## 4. Common GCMP header

Every GCMP payload begins with an 8-byte common header.

```text
Byte  0        Version
Byte  1        Message Type
Byte  2        Code
Byte  3        Flags
Bytes 4..7     Transaction ID
Bytes 8..N     Message body
```

| Field | Size | Meaning |
|---|---:|---|
| Version | 8 bits | GCMP version; initial value `1` |
| Message Type | 8 bits | operation/error type from the GCTL registry |
| Code | 8 bits | type-specific reason/result/subtype |
| Flags | 8 bits | generic message flags |
| Transaction ID | 32 bits | request/reply correlation or event sequence |

### 4.1 Flags

Initial generic flags:

| Bit | Name | Meaning |
|---:|---|---|
| 0 | `MORE` | additional records/pages remain available |
| 1 | `TRUNCATED` | response omitted information because of size/policy |
| 2..7 | Reserved | transmit zero; ignore when received |

A request chooses its Transaction ID. The reply copies it unchanged. A Transaction ID has meaning only to the requester and need not be globally unique.

Requests SHOULD be idempotent. A requester may retransmit the same request and Transaction ID after a timeout. A responder is not required to retain transaction state, although it may cache recent replies.

---

## 5. Base message classes

The following allocation is the proposed base registry. The separate [[GCTL Message Registry]] is authoritative for numeric allocation once frozen.

| Value | Message | Purpose |
|---:|---|---|
| `0x00` | RESERVED | invalid/unassigned |
| `0x01` | SOLICIT | scoped service/router discovery |
| `0x02` | ADVERTISE | discovery response |
| `0x10` | ADDRESS_OFFER | bootstrap address offer |
| `0x11` | ADDRESS_CLAIM | client claims offered address |
| `0x12` | ADDRESS_ACK | claim accepted |
| `0x13` | ADDRESS_NAK | claim rejected |
| `0x20` | ECHO_REQUEST | reachability/round-trip probe |
| `0x21` | ECHO_REPLY | echo response |
| `0x22` | DESTINATION_UNREACHABLE | routed delivery failed |
| `0x23` | HOP_LIMIT_EXCEEDED | GDP Hop Limit expired |
| `0x24` | PARAMETER_PROBLEM | malformed/unsupported routed packet |
| `0x25` | CLASS_UNSUPPORTED | GDP Size Class cannot continue on the selected path/profile |
| `0x26` | TRANSIT_ABORTED | packet was accepted into forwarding but later aborted |
| `0x28` | PATH_PROBE | explicit path diagnostic probe |
| `0x29` | PATH_REPLY | path diagnostic response |
| `0x30` | STATUS_REQUEST | mandatory node-status query |
| `0x31` | STATUS_REPLY | mandatory node-status response |
| `0x32` | GET_REQUEST | retrieve one managed object/attribute set |
| `0x33` | GET_REPLY | GET result |
| `0x34` | GET_NEXT_REQUEST | bounded table/entity enumeration |
| `0x35` | GET_NEXT_REPLY | next record/page |
| `0x36` | EVENT_REPORT | asynchronous best-effort management event |
| `0x80..0x9F` | ROUTING CONTROL | reserved for GNet route-distribution protocols |
| `0xFF` | EXPERIMENTAL | controlled experiments |

Routing-table **inspection** is part of GCS. Routing-table **distribution and modification** are separate routing-control functions and are not defined by this document.

---

## 6. Automatic error messages

### 6.1 Compact error body

The normal GCMP error is exactly one `ctrl32B` GDP payload:

```text
 8 bytes    GCMP common header
20 bytes    complete GDP header of offending packet
 4 bytes    first 32 bits of offending GDP payload
-----------------------------------------------
32 bytes    total GCMP payload
```

This follows the useful ICMP principle of returning enough of the failed packet for the sender to identify the operation, while exploiting GNet's fixed 20-byte GDP header and 32-byte control class.

Protocols that expect to consume GCMP errors SHOULD place a useful connection, tunnel, flow, service or transaction discriminator in the first 32 bits of their GDP payload.

No full user payload is returned automatically.

### 6.2 DESTINATION_UNREACHABLE codes

Proposed codes:

| Code | Meaning |
|---:|---|
| 0 | no route to destination prefix |
| 1 | destination address unknown/unreachable |
| 2 | destination does not implement GDP payload type |
| 3 | requested local service unavailable |
| 4 | administratively prohibited |
| 5 | address/scope violation |
| 6 | forwarding resource unavailable |
| 7 | routing loop/invalid route state detected |

### 6.3 HOP_LIMIT_EXCEEDED codes

| Code | Meaning |
|---:|---|
| 0 | Hop Limit expired in transit |

GNet does not perform GDP fragmentation, so no fragment-reassembly timeout subtype is required.

### 6.4 PARAMETER_PROBLEM codes

| Code | Meaning |
|---:|---|
| 0 | unsupported GDP version |
| 1 | malformed GDP header/field combination |
| 2 | invalid/reserved Size Class use |
| 3 | malformed destination/source address form |
| 4 | malformed GCTL control message |
| 5 | unsupported mandatory control feature |

### 6.5 CLASS_UNSUPPORTED

Because GDP never fragments packets in transit, failure to carry a Size Class is a first-class network error.

`CLASS_UNSUPPORTED` means the selected outgoing link, adaptation profile or destination cannot accept the offending GDP Size Class.

The `Code` field SHOULD contain the largest lower GDP Size Class known to be accepted on the failing interface/profile (`0xFF` if unknown). The quoted original packet still provides correlation.

The source may select a smaller application/transport package if its protocol permits. GCMP itself does not fragment or retransmit the packet.

### 6.6 TRANSIT_ABORTED

This is inspired by the useful distinction in ARPANET between "destination dead" and an **incomplete transmission** after the network had already accepted a message.

Proposed codes:

| Code | Meaning |
|---:|---|
| 0 | downstream link failed/reset |
| 1 | next-hop router disappeared |
| 2 | forwarding state/resource aborted |
| 3 | local hardware/internal forwarding fault |
| 4 | packet discarded during route transition |

A router sends TRANSIT_ABORTED only when it knows that a particular routed GDP packet was discarded after forwarding had begun or after the packet had been admitted to a forwarding resource.

### 6.7 Error-generation rules

A node/router MUST NOT generate a GCMP error:

- in response to another GCMP automatic error;
- in response to EVENT_REPORT;
- when the original source address is not valid for a routed unicast reply;
- merely because a control reply itself was lost;
- at a rate that threatens forwarding stability.

Errors are best effort and MUST be rate-limited. Under severe congestion, a router may suppress an error and increment a local `control-errors-suppressed` counter instead.

No GNet transport may assume that absence of a GCMP error proves delivery.

---

## 7. ECHO

Every addressed GNet node MUST implement ECHO_REQUEST/ECHO_REPLY.

The reply:

- copies the Transaction ID;
- copies the request body without modification;
- uses the same GDP payload Size Class as the request;
- reverses GDP source/destination in the normal manner;
- does not increase the amount of returned data.

This provides reachability, packet-integrity-through-the-path testing and round-trip timing without invoking the richer management service.

Routers MAY rate-limit ECHO replies.

---

## 8. Explicit path diagnostics

GNet should not require operators to turn an expected Hop Limit failure into an accidental path-tracing mechanism.

`PATH_PROBE` provides an explicit equivalent.

The sender transmits successive PATH_PROBE messages with GDP Hop Limits `1, 2, 3, ...` toward the destination.

- If a router would expire a PATH_PROBE, it sends PATH_REPLY with Code `INTERMEDIATE` rather than HOP_LIMIT_EXCEEDED.
- If the destination receives PATH_PROBE normally, it sends PATH_REPLY with Code `DESTINATION`.
- The GDP Source address of PATH_REPLY identifies the responding router/node.
- The Transaction ID identifies the individual probe.

An authorized/locally permitted PATH_REPLY may additionally include ingress interface, selected egress interface, queue-pressure summary and node name as typed attributes. Basic unauthenticated replies need reveal only the responding GDP address and result.

This gives GNet a native route-trace operation while keeping ordinary Hop Limit semantics unchanged.

---

## 9. GNet Control Service (GCS)

### 9.1 Mandatory service levels

**All addressed GNet nodes:**

- ECHO
- STATUS
- basic node counters

**Routers/switches:**

- interfaces
- neighbors
- routes
- forwarding queues
- GCTL/control counters
- path replies
- asynchronous local events to configured management sinks

A very small terminal or embedded endpoint may omit router-only entity classes.

### 9.2 STATUS

STATUS is deliberately simple and mandatory, following the strongest operational idea in Chaosnet.

A basic STATUS_REPLY SHOULD provide, subject to policy:

- node GDP address
- node role/type
- uptime
- GCTL version/capability bits
- number of active interfaces
- implementation/system name when configured

A requester asks for optional detail through an attribute bitmap or attribute records. The responder may set `TRUNCATED` or omit policy-restricted fields.

A STATUS reply MUST NOT be larger than the request's GDP Size Class unless the request explicitly carries a future authenticated larger-reply authorization. The baseline protocol therefore has no reflection-amplification behavior.

---

## 10. Managed entity model

GCS exposes typed **entities** with typed **attributes**. Management tools should never have to parse vendor-formatted text.

Initial entity classes:

| Class | Entity | Typical instances |
|---:|---|---|
| `0x01` | NODE | local GNet node |
| `0x02` | INTERFACE | physical/logical GNet interfaces |
| `0x03` | NEIGHBOR | directly known GNet neighbors |
| `0x04` | ROUTE | hierarchical prefix routes |
| `0x05` | QUEUE | forwarding/QoS queues |
| `0x06` | SERVICE | locally advertised GNet services |
| `0x07` | CONTROL | GCMP/GCTL engine itself |

Exact attribute numeric allocation should live in a separate management-object registry once the model is frozen.

### 10.1 Mandatory NODE attributes

- Address
- Role
- Uptime
- GCTL capabilities
- Interface count
- System/name string when configured

### 10.2 Mandatory INTERFACE attributes for routers

- local interface identifier
- administrative/operational state
- attached neighbor or link identity when known
- nominal link rate/profile
- packages received/transmitted
- flits received/transmitted when available
- integrity/CRC errors reported by the link layer
- retransmit/recovery events when applicable
- dropped packages
- credit-stall time/events
- last state-change time

### 10.3 Mandatory NEIGHBOR attributes

- neighbor GDP address
- interface identifier
- neighbor state
- route/link cost
- age since last valid control/routing contact

### 10.4 Mandatory ROUTE attributes

- destination prefix
- prefix length
- next-hop GDP address
- outgoing interface
- route metric/cost
- route age
- route source/class

GCS reads route state; it does not install or remove routes.

### 10.5 Mandatory QUEUE attributes

Queue visibility is especially important for wormhole/credit-based GNet.

Routers SHOULD expose:

- interface/output identifier
- QoS/traffic class
- current occupancy
- configured capacity
- high-water mark
- current downstream credit availability
- cumulative credit-stall time/events
- blocked-wormhole events/time when measurable
- drops/aborts by reason

This is a major deliberate improvement over ICMP-style diagnostics: an operator can determine **where forwarding pressure is accumulating**, not merely that a destination stopped responding.

### 10.6 Mandatory CONTROL attributes

- GCMP messages received/sent
- malformed GCMP messages
- automatic errors generated
- automatic errors suppressed/rate-limited
- ECHO requests/replies
- STATUS/GET requests
- policy-denied management requests

Counters SHOULD be represented as monotonically increasing 64-bit values on the wire. Small implementations may maintain them internally as multiple machine words.

---

## 11. Attribute record encoding

Management replies carry a sequence of typed attribute records.

```text
Bytes 0..1   Attribute ID
Byte  2      Format
Byte  3      Value Length
Bytes 4..N   Value
```

Initial scalar formats:

| Format | Meaning |
|---:|---|
| `0x01` | unsigned 8-bit |
| `0x02` | unsigned 16-bit |
| `0x03` | unsigned 32-bit |
| `0x04` | unsigned 64-bit / counter |
| `0x05` | 64-bit GDP address |
| `0x06` | GDP prefix + prefix length |
| `0x07` | bit set |
| `0x08` | text/display string |
| `0x09` | opaque octets |

Unknown optional attributes are ignored. Attribute semantics, units and access class are defined by the management-object registry rather than by application-specific text.

---

## 12. GET and bounded enumeration

### 12.1 GET_REQUEST / GET_REPLY

GET retrieves attributes for one selected entity/instance.

The request identifies:

- Entity Class
- Instance/Handle
- requested Attribute ID or attribute group
- optional selector/filter records

GET_REPLY returns the same selector plus zero or more attribute records. The Code field reports success, no-such-entity, no-such-attribute, policy-denied, unsupported or temporarily-busy.

### 12.2 GET_NEXT_REQUEST / GET_NEXT_REPLY

Large tables such as routes and neighbors are never returned as an unbounded dump.

GET_NEXT uses an opaque 32-bit cursor:

- Cursor `0` begins enumeration.
- The reply returns one bounded record/page plus a cursor for the next request.
- `MORE` indicates that additional records may exist.
- Cursor `0` in a successful final reply means enumeration is complete.

A responder may implement cursors without persistent per-client state. If topology changes invalidate a cursor, it may return `STALE_CURSOR` and require the requester to restart.

This provides the useful table-walking property of later management protocols without requiring a large object-identifier encoding or a long-lived management connection.

---

## 13. Event reporting

`EVENT_REPORT` is a best-effort asynchronous message sent to management sinks configured by local policy.

The Transaction ID is used as a monotonically advancing local event sequence where practical. Sequence gaps tell a manager to poll current state; events themselves are not a reliable log.

Recommended base events for routers:

- NODE_START
- NODE_GOING_DOWN
- INTERFACE_UP
- INTERFACE_DOWN
- NEIGHBOR_UP
- NEIGHBOR_LOST
- ROUTE_CHANGE_THRESHOLD
- ADDRESS_CONFLICT / ADDRESS_CONFIGURATION_FAILURE
- QUEUE_DROP_THRESHOLD
- CREDIT_STALL_THRESHOLD
- CONTROL_RATE_LIMIT_THRESHOLD
- HARDWARE_FORWARDING_FAULT

An event identifies Entity Class, instance, event code, severity and optional attribute records containing current values.

EVENT_REPORT is never acknowledged by GCMP. Critical systems may repeat important events a small bounded number of times, but the authoritative state remains available through STATUS/GET.

This combines the useful "IMP going down" idea from ARPANET with the structured event-logging model used by DECnet, while avoiding a reliable event protocol inside Layer 3.

---

## 14. Congestion behavior

GCTL distinguishes **congestion telemetry** from **congestion control**.

GNet congestion mechanisms belong primarily to:

1. hop-local DLP/GLCP credit and backpressure;
2. router queue/admission/scheduling policy;
3. end-to-end behavior in transports such as GTS.

GCMP does **not** define SOURCE_QUENCH and an endpoint MUST NOT reduce a transport rate merely because an unauthenticated GCMP packet says a router was congested.

Instead, congestion appears through:

- QUEUE gauges and high-water marks;
- credit-stall counters;
- blocked-wormhole counters;
- drop/abort counters;
- optional threshold EVENT_REPORT messages;
- routing metrics where the routing protocol chooses to incorporate sustained congestion.

During acute overload, forwarding user/control traffic and preserving network stability take priority over generating diagnostic errors about every discarded packet.

---

## 15. Security and exposure

The base routed GCS is intentionally **read-only**.

### 15.1 Unauthenticated baseline

Subject to local policy, an ordinary remote node may receive:

- ECHO
- basic STATUS
- packet-specific automatic errors
- basic PATH_REPLY

### 15.2 Restricted operational detail

Interface counters, queue state, neighbor identity and routing tables may reveal topology or traffic information. Routers MAY restrict these to:

- directly attached/local administrative domains;
- configured management stations;
- a future authenticated management session.

A denied request returns a bounded policy-denied result or is silently discarded according to local policy.

### 15.3 Not part of raw GCMP

The following are deliberately excluded from unauthenticated GCMP/GCS:

- reboot/shutdown command
- arbitrary SET of configuration
- install/delete route
- reset remote interface
- read/write memory or registers
- firmware/software load or dump
- user/account management
- cryptographic key management

Where remote administration is required, the same entity/attribute vocabulary may be reused by a separately authenticated higher-layer management protocol.

---

## 16. Bootstrap and discovery coexistence

Existing GCTL bootstrap semantics remain:

1. establish the physical/link relationship with GLCP;
2. use provisional/link-local GDP/GCTL rules for SOLICIT;
3. receive ADVERTISE from an authorized router/service;
4. perform ADDRESS_OFFER / ADDRESS_CLAIM / ADDRESS_ACK or NAK;
5. transition to ordinary routed GDP operation.

The common GCMP header provides the Version, Message Type, Flags and Transaction ID needed by those exchanges. Exact bootstrap addressing and address-configuration field packing remain defined by their packet-format documents until separately frozen.

Discovery broadcasts/multicasts MUST be scope-bounded. GCTL does not define an internet-wide broadcast discovery mechanism.

---

## 17. Recommended implementation tiers

### Tier 0 — tiny endpoint

Required:

- receive automatic errors relevant to its traffic
- ECHO
- STATUS
- NODE counters

### Tier 1 — normal host/server

Tier 0 plus:

- interface state/counters
- service status
- PATH_PROBE destination reply

### Tier 2 — router/switch

Tier 1 plus:

- neighbor objects
- route objects
- queue/credit objects
- GET_NEXT enumeration
- path intermediate replies
- event generation
- control-plane rate limiting and suppression counters

The wire protocol is identical across tiers.

---

## 18. Why this design fits GNet

GNet's forwarding system has information that traditional best-effort internetworks often hid from operators: link credits, output queues, blocked wormholes, class/QoS state and explicit hierarchical routes.

The control plane should expose that information without contaminating GDP's deliberately minimal forwarding header.

The resulting split is:

```text
GLCP              hop-local link state, credits, VC control
GDP               minimal routed datagram
GCMP              routed errors, echo, path probes, control requests/replies
GCS               structured read-only node/router management model
Routing protocol  route distribution and convergence
GTS               end-to-end reliability/congestion/session behavior
Admin management  authenticated configuration/actions above the raw control plane
```

This preserves the fast GNet forwarding path while making the network unusually observable and diagnosable.

---

## 19. Prior-art references

- J. Postel, **Internet Control Message Protocol**, RFC 792, September 1981: https://www.rfc-editor.org/rfc/rfc792.html
- A. Malis, **ARPANET 1822L Host Access Protocol**, RFC 878, December 1983: https://www.rfc-editor.org/rfc/rfc878.html
- D. Boggs, J. Shoch, E. Taft, R. Metcalfe, **Pup: An Internetwork Architecture**, Xerox PARC CSL-79-10, 1979: https://xeroxalto.computerhistory.org/_cd8_/pup/puppaper.dm!1_/.Pup12.bravo.html
- **Chaosnet protocol — STATUS and routing**, Chaosnet documentation: https://chaosnet.net/protocol
- Xerox Network Systems well-known protocol types, including Routing Information, Echo and Error: https://www.iana.org/assignments/xns-protocol-types/xns-protocol-types.xhtml
- Digital Equipment Corporation, **DECnet Phase IV Network Management Functional Specification**, July 1983, AA-X437A-TK (Bitsavers)
- Apple, **Inside AppleTalk — Routing Table Maintenance Protocol / AppleTalk Echo Protocol**
- J. Case et al., **A Simple Network Management Protocol**, RFC 1067, August 1988: https://www.rfc-editor.org/rfc/rfc1067.html
- M. Rose, K. McCloghrie, **Structure and Identification of Management Information**, RFC 1155, May 1990
- F. Gont, **Deprecation of ICMP Source Quench Messages**, RFC 6633, May 2012 — retrospective congestion-control lesson

---

## 20. Open items before wire freeze

- exact attribute and event numeric registries
- exact GET/GET_NEXT fixed selector packing
- bootstrap/provisional GDP source and destination encoding
- management authorization model above the unauthenticated baseline
- PATH_REPLY optional-detail fields
- precise per-node/per-interface control-message rate-limit defaults
- routing-control message range and its relationship to the eventual hierarchical routing protocol

The **architecture**, 8-byte common header, compact 32-byte automatic-error form, mandatory ECHO/STATUS concept, read-only entity management model, explicit PATH_PROBE, no Source Quench, and router queue/credit observability are proposed as the stable design direction.
