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
updated: 2026-09-11
---
# GNet Control Service and Control Message Protocol

Status: **DRAFT — architecture and base message model proposed**

GNet needs more than packet-error reporting. Every GNet node should provide a small, uniform set of diagnostics, while routers and switches should expose enough structured state to diagnose routing, queueing, credit pressure and link failures without requiring vendor-specific consoles.

**GCTL** is the umbrella name for GNet control messages carried on the normal data path. It includes bootstrap/discovery, link-local credit control, routed diagnostics and the read-only control service.

- **GCMP — GNet Control Message Protocol**: compact GCTL messages carried as GDP Type `GCTL`.
- **GCS — GNet Control Service**: the mandatory read-only status and management service implemented with GCMP request/reply messages.

---

## 1. Architectural boundary

GCTL is carried on the normal GNet data path. It is distinct from the dedicated physical control pairs used by directly attached infrastructure.

The normative separation is:

- **GLCP / physical control-pair functions** own only infrastructure-local operations such as bootstrap identification, physical capability/rate negotiation, GC3 `WANT/PERMIT`, and any GS3-local switching/path operations retained by the GS specification.
- **GCTL** owns `CREDIT_REQUEST` and `CREDIT`, bootstrap/discovery, diagnostics and management messages carried on the data path.
- **GDP** owns routed datagram metadata: Version, Type, Size Class, Hop Limit, QoS, Source and Destination.
- **GTS and higher layers** own end-to-end reliability, transport congestion behavior, sessions and authenticated administrative applications.

### 1.1 Link-local credit rule

Credits are **strictly link-local, never end-to-end**.

> **1 GNet credit = guaranteed receive capacity for exactly one physical flit at the next forwarding endpoint on the current link.**

A multi-hop path therefore has independent credit relationships at every forwarding boundary. This allows long-distance traffic to pipeline without waiting for a global destination credit round trip.

A GC3 Coupler is not a forwarding endpoint and does not maintain credits. `CREDIT_REQUEST` and `CREDIT` pass over the GC3 data medium between the actual adjacent endpoints or between a client and its local router.

A GS3 is an active forwarding endpoint. A source-to-GS3 credit relationship and the GS3-to-egress credit relationship are independent. GS3 credit advertised to an ingress describes GS3's own ingress capacity, not a mirrored copy of the destination endpoint's current credit balance.

### 1.2 Core principles

1. **Mandatory minimum diagnostics.** Every addressed GNet node implements ECHO and basic STATUS.
2. **Routers expose forwarding state.** A conforming router exposes interfaces, neighbors, routes, queues and mandatory counters.
3. **Errors are best-effort information, not reliability.** Failure to receive a GCMP error never implies successful delivery.
4. **Keep control messages bounded.** Automatic error traffic must stay small even when the network is distressed.
5. **No error storms.** Never recursively report errors about errors; rate-limit generated control traffic.
6. **No Layer-3 remote superuser.** The mandatory GCS is read-only.
7. **Congestion control is not GCMP Source Quench.** Link-local credits provide flow control; higher layers provide transport congestion behavior.
8. **Management data is structured, not formatted terminal text.**

---

## 2. GCMP carriage

GCMP is carried in an ordinary GDP datagram with GDP Type `GCTL`.

The GDP Source and Destination fields provide GCMP addressing. GDP Hop Limit applies normally. GDP Size Class defines the exact GCMP payload size; GCMP therefore does not need a total-length field.

GCMP messages normally use only:

- `ctrl32B` — 32-byte payload
- `ctrl64B` — 64-byte payload
- `msg128B` — 128-byte payload
- `msg256B` — 256-byte payload

Automatic packet-error reports SHOULD use `ctrl32B`.

GCS implementations SHOULD NOT use GDP classes larger than 256 bytes for ordinary management. Large tables are enumerated with bounded request/reply operations instead of being dumped in one packet.

Control traffic remains subject to DLP flow control and local medium/path scheduling. Implementations SHOULD reserve a small bounded network-control receive allowance so `CREDIT_REQUEST`, `CREDIT`, diagnostics and bootstrap traffic can make progress without unbounded buffering.

---

## 3. Common GCMP header

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

Initial generic flags:

| Bit | Name | Meaning |
|---:|---|---|
| 0 | `MORE` | additional records/pages remain available |
| 1 | `TRUNCATED` | response omitted information because of size/policy |
| 2..7 | Reserved | transmit zero; ignore when received |

A request chooses its Transaction ID. The reply copies it unchanged. Requests SHOULD be idempotent.

---

## 4. Base message classes

The separate [[GCTL Message Registry]] is authoritative for numeric allocation.

| Value | Message | Purpose |
|---:|---|---|
| `0x00` | RESERVED | invalid/unassigned |
| `0x01` | SOLICIT | scoped service/router discovery |
| `0x02` | ADVERTISE | discovery response |
| `0x03` | CREDIT_REQUEST | request link-local receive capacity from the next forwarding endpoint |
| `0x04` | CREDIT | advertise link-local receive capacity |
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

### 4.1 CREDIT_REQUEST / CREDIT

`CREDIT_REQUEST` and `CREDIT` always travel on the data path, on both GC3 and GS3.

On GC3, the Coupler repeats the message without interpreting it. The actual adjacent endpoint or local router responds.

On GS3, the switch consumes a credit request for the ingress link and responds with credit representing **its own local ingress receive capacity**. The switch does not first request destination credit and mirror that value to the source. Its downstream credit state is a separate relationship.

Credit-return messages MAY batch multiple credits. Exact compact field encoding and batching limits remain DRAFT.

---

## 5. Automatic error messages

The normal GCMP error is exactly one `ctrl32B` GDP payload:

```text
 8 bytes    GCMP common header
20 bytes    complete GDP header of offending packet
 4 bytes    first 32 bits of offending GDP payload
-----------------------------------------------
32 bytes    total GCMP payload
```

No full user payload is returned automatically.

### 5.1 DESTINATION_UNREACHABLE codes

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

### 5.2 HOP_LIMIT_EXCEEDED

Code `0` means Hop Limit expired in transit.

### 5.3 PARAMETER_PROBLEM codes

| Code | Meaning |
|---:|---|
| 0 | unsupported GDP version |
| 1 | malformed GDP header/field combination |
| 2 | invalid/reserved Size Class use |
| 3 | malformed destination/source address form |
| 4 | malformed GCTL control message |
| 5 | unsupported mandatory control feature |

### 5.4 CLASS_UNSUPPORTED

Because GDP never fragments packets in transit, failure to carry a Size Class is a first-class network error. The `Code` field SHOULD contain the largest lower GDP Size Class known to be accepted on the failing interface/profile (`0xFF` if unknown).

### 5.5 TRANSIT_ABORTED

| Code | Meaning |
|---:|---|
| 0 | downstream link failed/reset |
| 1 | next-hop router disappeared |
| 2 | forwarding state/resource aborted |
| 3 | local hardware/internal forwarding fault |
| 4 | packet discarded during route transition |

A router sends TRANSIT_ABORTED only when it knows that a particular routed GDP packet was discarded after forwarding had begun or after the packet had been admitted to a forwarding resource.

### 5.6 Error-generation rules

A node/router MUST NOT generate a GCMP error in response to another automatic error or EVENT_REPORT, merely because a control reply was lost, or at a rate that threatens forwarding stability.

---

## 6. ECHO

Every addressed GNet node MUST implement ECHO_REQUEST/ECHO_REPLY.

The reply copies the Transaction ID and request body, uses the same GDP payload Size Class, reverses GDP source/destination normally and does not increase the amount of returned data.

Routers MAY rate-limit ECHO replies.

---

## 7. Explicit path diagnostics

`PATH_PROBE` provides explicit route tracing. The sender transmits successive PATH_PROBE messages with GDP Hop Limits `1, 2, 3, ...` toward the destination.

- An intermediate router sends PATH_REPLY with Code `INTERMEDIATE`.
- The destination sends PATH_REPLY with Code `DESTINATION`.
- The GDP Source address identifies the responding router/node.
- The Transaction ID identifies the probe.

---

## 8. GNet Control Service (GCS)

### 8.1 Mandatory service levels

**All addressed GNet nodes:** ECHO, STATUS and basic node counters.

**Routers/switches:** interfaces, neighbors, routes, forwarding queues, GCTL/control counters, path replies and asynchronous local events to configured management sinks.

### 8.2 STATUS

A basic STATUS_REPLY SHOULD provide, subject to policy:

- node GDP address
- node role/type
- uptime
- GCTL version/capability bits
- number of active interfaces
- implementation/system name when configured

A STATUS reply MUST NOT be larger than the request's GDP Size Class unless explicitly authorized by a future authenticated mechanism.

---

## 9. Managed entity model

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

Routers SHOULD expose interface packet/flit counters, CRC/integrity errors, drops, credit-stall time/events and state-change time. Queue objects SHOULD expose occupancy, capacity, high-water marks, downstream credit availability and blocked/backpressure events.

---

## 10. Attribute record encoding

Management replies carry typed attribute records:

```text
Bytes 0..1   Attribute ID
Byte  2      Format
Byte  3      Value Length
Bytes 4..N   Value
```

Initial scalar formats include unsigned 8/16/32/64-bit values, GDP address, GDP prefix, bit set, text and opaque octets.

---

## 11. GET and bounded enumeration

GET retrieves attributes for one selected entity/instance. GET_NEXT uses an opaque 32-bit cursor for bounded enumeration of large tables such as routes and neighbors.

Cursor `0` begins enumeration; `MORE` indicates additional records; final cursor `0` means complete. Responders may invalidate stale cursors after topology changes.

---

## 12. Event reporting

`EVENT_REPORT` is a best-effort asynchronous message sent to management sinks configured by local policy.

Recommended base events include NODE_START, NODE_GOING_DOWN, INTERFACE_UP/DOWN, NEIGHBOR_UP/LOST, route-change thresholds, address conflicts, queue/drop thresholds, credit-stall thresholds and hardware forwarding faults.

EVENT_REPORT is not acknowledged by GCMP; authoritative state remains available through STATUS/GET.

---

## 13. Congestion behavior

GCTL distinguishes **congestion telemetry** from **congestion control**.

GNet congestion mechanisms belong primarily to:

1. link-local receiver credit and backpressure;
2. router queue/admission/scheduling policy;
3. end-to-end behavior in transports such as GTS.

GCMP does not define SOURCE_QUENCH. Congestion appears through queue gauges, credit-stall counters, blocked-forwarding counters, drops/aborts, optional events and routing metrics where appropriate.

---

## 14. Security and exposure

The base routed GCS is intentionally read-only. Ordinary remote nodes may receive ECHO, basic STATUS, packet-specific automatic errors and basic PATH_REPLY subject to policy.

Interface counters, queue state, neighbor identity and routing tables may be restricted to local administrative domains, configured management stations or a future authenticated management session.

Raw unauthenticated GCTL does not provide reboot, arbitrary SET, route installation/deletion, remote memory access, firmware loading, account management or cryptographic key management.

---

## 15. Bootstrap and discovery coexistence

Bootstrap/address configuration remains GCTL data-path traffic. Exact provisional GDP source/destination encoding and address-configuration field packing remain defined by their packet-format documents until separately revised/frozen.

Discovery broadcasts/multicasts MUST be scope-bounded. GCTL does not define an internet-wide broadcast discovery mechanism.

---

## 16. Recommended implementation tiers

### Tier 0 — tiny endpoint

- credit control for its local link
- receive relevant automatic errors
- ECHO
- STATUS
- NODE counters

### Tier 1 — normal host/server

Tier 0 plus interface state/counters, service status and PATH_PROBE destination reply.

### Tier 2 — router/switch

Tier 1 plus neighbor objects, route objects, queue/credit objects, GET_NEXT, intermediate path replies, events and control-plane rate limiting.

---

## 17. Resulting split

```text
Physical/GLCP     infrastructure-local bootstrap, PHY/mode, GC3 WANT/PERMIT, GS-local path control
DLP               physical flits, local transfer/integrity; no addressing
GDP               minimal routed datagram and addressing
GCTL              data-path credit control, bootstrap/discovery, errors, diagnostics, management
Routing protocol  route distribution and convergence
GTS               end-to-end reliability/congestion/session behavior
Admin management  authenticated configuration/actions above raw GCTL
```

The critical credit rule is that **GCTL carries credit messages, but credit scope remains one forwarding link only**.

---

## 18. Open items before wire freeze

- exact `CREDIT_REQUEST` / `CREDIT` compact encoding and batching limits
- exact attribute and event numeric registries
- exact GET/GET_NEXT selector packing
- bootstrap/provisional GDP source and destination encoding
- management authorization model
- PATH_REPLY optional-detail fields
- control-message rate-limit defaults
- routing-control message range

The architecture, link-local credit scope, data-path carriage of credit control, mandatory ECHO/STATUS, read-only entity management model and router queue/credit observability are the stable design direction.
