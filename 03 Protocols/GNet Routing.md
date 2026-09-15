# GNet Dynamic Routing

Status: draft on branch `dynamic-topology-routing`.

This revision deliberately defines a small forward-route-exchange protocol. It does **not** define a topology database, link-state flooding, or SPF/Dijkstra. Those may be introduced by a later revision if needed.

## Architecture

```text
GCTL neighbor discovery
        |
        v
forward route exchange
        |
        v
RIB (connected / learned / static / future escape)
        |
        v
FIB
        |
        v
GDP forwarding
```

The forwarding plane, router egress scheduler, and DLP VC allocation remain separate from routing control.

## RouterID

`RouterID` is an unsigned 64-bit value.

- A router generates its RouterID from a high-quality random source.
- `0x0000000000000000` is invalid and MUST NOT be used.
- The generated RouterID SHOULD be persisted and reused across normal restarts.
- RouterID is independent of all GDP interface addresses.
- A router has one RouterID even if it has multiple interfaces or GDP prefixes.

Random generation is used to avoid requiring a central RouterID authority in the first routing revision.

## LinkID

`LinkID` is an unsigned 64-bit non-zero identifier for a local router-to-router attachment.

A LinkID identifies the attachment advertised by the sending router. The first revision does not require both ends of a physical link to use the same LinkID.

## Route origin

The routing data model defines these route origins:

| Wire value | Origin | Initial administrative preference |
| ---: | --- | ---: |
| `0` | Connected | `0` |
| `1` | Learned | `100` |
| `2` | Static | `10` |
| `3` | Escape | `255` |

Lower administrative preference is preferred. `Escape` is reserved for future deadlock-escape routing and is not active normal-route selection in this revision.

Administrative preference is local route-selection policy. It is distinct from the route metric carried by the routing protocol.

## Route metric

The initial metric is one unsigned 32-bit scalar cost.

- Lower metric is better.
- Default link metric is `100`.
- When learned routes are forwarded in Stage 3, the forwarding router adds the local link metric using saturating arithmetic.
- This revision does not derive metric automatically from bandwidth or latency.

## GCTL routing message assignments

This revision reserves three GCTL message type values:

| Type | Name |
| ---: | --- |
| `0x40` | `ROUTER_HELLO` |
| `0x41` | `ROUTER_HELLO_ACK` |
| `0x42` | `ROUTE_ADVERTISE` |

All three use the normal 8-byte GCTL v1 header:

```text
0               1               2               3
+---------------+---------------+---------------+---------------+
| Version = 1   | Message Type  | Code = 0      | Flags = 0     |
+---------------+---------------+---------------+---------------+
|                  Transaction ID (32 bits)                     |
+---------------------------------------------------------------+
```

Multi-byte integers are encoded in network byte order (big-endian).

## ROUTER_HELLO / ROUTER_HELLO_ACK

Both messages use the same 24-byte body:

```text
+---------------------------------------------------------------+
|                    RouterID (64 bits)                         |
+---------------------------------------------------------------+
|                    LinkID (64 bits)                           |
+---------------------------------------------------------------+
| Hold Time milliseconds (32) | Link Metric (32)                |
+---------------------------------------------------------------+
```

The complete encoded GCTL message is therefore 32 bytes.

Fields:

- `RouterID`: sender's non-zero 64-bit random persistent identity.
- `LinkID`: sender's non-zero local attachment identifier.
- `Hold Time`: requested neighbor liveness hold time in milliseconds.
- `Link Metric`: scalar cost for reaching the sender over this attachment.

Stage 1 defines the wire representation only. The Stage 2 adjacency state machine defines when HELLO and HELLO_ACK are transmitted and when a neighbor is declared down.

## ROUTE_ADVERTISE

`ROUTE_ADVERTISE` uses this 24-byte body:

```text
+---------------------------------------------------------------+
|                Advertising RouterID (64 bits)                 |
+---------------------------------------------------------------+
|                 GDP Prefix Network (64 bits)                  |
+---------------------------------------------------------------+
| Prefix Len (8)  | Origin (8)    | Reserved = 0 (16)           |
+---------------------------------------------------------------+
|                       Metric (32 bits)                        |
+---------------------------------------------------------------+
```

The complete encoded GCTL message is 32 bytes.

Rules:

- `Advertising RouterID` MUST be non-zero.
- GDP prefix lengths are `0..64`.
- The GDP prefix network MUST be canonical: all host bits after `Prefix Len` are zero.
- `Origin` uses the route-origin table above.
- The 16 reserved bits MUST be zero when sent and MUST be rejected when non-zero in this revision.
- `Metric` is an unsigned 32-bit scalar cost.

The first revision advertises one prefix per message. Route aggregation or multiple-prefix packing is deliberately deferred.

## Stage 2 — Neighbor discovery

The next stage remains intentionally small:

- `DOWN -> UP` neighbor state only.
- Send `ROUTER_HELLO` on router links.
- Return `ROUTER_HELLO_ACK`.
- Store RouterID, local port, local/remote LinkID, neighbor GDP address, metric, and hold timer.
- Expire the neighbor when the hold timer elapses.

No topology database is required.

## Stage 3 — Forward route exchange + RIB/FIB

Routers exchange routes directly rather than exchanging a complete topology graph.

- Advertise connected routes.
- Forward learned routes to other neighbors.
- Add the outgoing link metric before forwarding.
- Apply split horizon: a learned route is not advertised back toward the neighbor from which it was learned.
- Store the neighbor RouterID from which each learned route was received.
- Prefer lower total metric among otherwise equivalent learned routes.
- Break equal learned-route metric ties by lower RouterID for deterministic behavior.
- Maintain competing Connected, Static, and Learned routes in the RIB.
- Install only selected routes in the FIB.
- Keep static routes available as policy overrides/fallbacks.

A later message assignment may add explicit `ROUTE_WITHDRAW`; Stage 4 should define it before failure/reconvergence is implemented.

## Stage 4 — Failure + reconvergence

The target behavior is:

```text
neighbor/link failure
        |
        v
neighbor DOWN
        |
        v
routes learned from neighbor removed
        |
        v
withdrawal propagated
        |
        v
RIB/FIB reselected
        |
        v
traffic uses alternate route
```

The exact withdrawal wire format is deferred until Stage 4.

## Deferred

- Link-state topology database.
- SPF/Dijkstra.
- ECMP and unequal-cost multipath.
- Congestion-aware/adaptive routing.
- Up*/down* or another deadlock-free escape topology algorithm.
- Detailed VC0 escape forwarding behavior.
- Routing hierarchy/areas.
- Cryptographic routing authentication.

## Stage 1 conformance requirement

An implementation of Stage 1 must demonstrate:

1. non-zero randomly generated 64-bit RouterIDs;
2. stable RouterID, LinkID, route-origin, and metric data types;
3. exact 32-byte HELLO, HELLO_ACK, and ROUTE_ADVERTISE wire images;
4. encode/decode round trips;
5. rejection of malformed lengths, zero IDs, non-canonical prefixes, unknown route origins, and non-zero reserved bits.
