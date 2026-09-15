# GNet Dynamic Routing

Status: draft on branch `dynamic-topology-routing`.

This revision deliberately defines a small forward-route-exchange protocol. It does **not** define a topology database, link-state flooding, or SPF/Dijkstra. Routers discover direct neighbors with GCTL, exchange reachability, retain competing learned routes, and explicitly withdraw reachability after failure.

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

Random generation avoids requiring a central RouterID authority in this revision.

## LinkID

`LinkID` is an unsigned 64-bit non-zero identifier for a local router-to-router attachment.

A LinkID identifies the attachment advertised by the sending router. This revision does not require both ends of a physical link to use the same LinkID.

## Route origin

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
- When a route is advertised on a router link, the sender adds that outgoing link metric using saturating arithmetic.
- This revision does not derive metric automatically from bandwidth or latency.

## GCTL routing message assignments

| Type | Name |
| ---: | --- |
| `0x40` | `ROUTER_HELLO` |
| `0x41` | `ROUTER_HELLO_ACK` |
| `0x42` | `ROUTE_ADVERTISE` |
| `0x43` | `ROUTE_WITHDRAW` |

All messages use the normal 8-byte GCTL v1 header:

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

The complete encoded GCTL message is 32 bytes.

Fields:

- `RouterID`: sender's non-zero 64-bit random persistent identity.
- `LinkID`: sender's non-zero local attachment identifier.
- `Hold Time`: requested neighbor liveness hold time in milliseconds.
- `Link Metric`: scalar cost for reaching the sender over this attachment.

A valid HELLO or HELLO_ACK establishes or refreshes the direct adjacency. When no valid refresh is seen for the peer-advertised hold time, the adjacency becomes `DOWN`.

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

Rules:

- `Advertising RouterID` MUST be the RouterID of the direct neighbor that transmitted this message and MUST be non-zero.
- GDP prefix lengths are `0..64`.
- The GDP prefix network MUST be canonical: all host bits after `Prefix Len` are zero.
- `Origin` uses the route-origin table above.
- Static and Escape routes MUST NOT be imported from another router in this revision.
- The 16 reserved bits MUST be zero when sent and MUST be rejected when non-zero.
- `Metric` is the sender's advertised total scalar cost to the prefix, including the sender's outgoing-link cost toward the receiver.

The first revision advertises one prefix per message.

## ROUTE_WITHDRAW

`ROUTE_WITHDRAW` removes reachability previously learned from the sending neighbor. It also uses a fixed 24-byte body:

```text
+---------------------------------------------------------------+
|               Withdrawing RouterID (64 bits)                  |
+---------------------------------------------------------------+
|                 GDP Prefix Network (64 bits)                  |
+---------------------------------------------------------------+
| Prefix Len (8)  |             Reserved = 0 (56)               |
+---------------------------------------------------------------+
```

The complete encoded GCTL message is 32 bytes.

Rules:

- `Withdrawing RouterID` MUST be the RouterID of the direct neighbor that transmitted the message and MUST be non-zero.
- The GDP prefix MUST be canonical.
- All seven reserved bytes MUST be zero when sent and MUST be rejected when non-zero.
- A withdrawal removes only the learned candidate for that prefix whose `learned_from` RouterID equals the sender. It MUST NOT remove connected, static, or candidates learned from other neighbors.
- Receiving a withdrawal for a route that is already absent is idempotent and MUST NOT create an error condition.

## Neighbor discovery

The initial adjacency state remains intentionally small:

- `DOWN -> UP` only.
- Send `ROUTER_HELLO` on router links.
- Return `ROUTER_HELLO_ACK` using the same transaction ID.
- Store RouterID, local/remote LinkID association, metric, peer hold time, and last-seen time.
- Expire the neighbor when its hold timer elapses.
- A later valid HELLO/ACK may bring the adjacency back `UP`.

No topology database is required.

## Forward route exchange + RIB/FIB

Routers exchange routes directly rather than exchanging a complete topology graph.

- Advertise connected routes.
- Forward selected learned routes to other neighbors.
- Add the outgoing link metric before forwarding.
- Apply split horizon: a learned route is not advertised back toward the neighbor from which it was learned.
- Store the neighbor RouterID from which each learned route was received.
- Keep competing learned candidates from different neighbors so an alternate route can already be present when the preferred path fails.
- Prefer lower total metric among otherwise equivalent learned routes.
- Break equal learned-route metric ties by lower RouterID for deterministic behavior.
- Maintain competing Connected, Static, and Learned routes in the RIB.
- Install only selected routes in the FIB.
- Keep static routes available as policy overrides/fallbacks.

## Failure and reconvergence

When a direct adjacency goes `DOWN`:

1. Remove every learned route whose `learned_from` is that neighbor.
2. Re-select the local RIB/FIB immediately; a retained alternate learned route may become active without waiting for new discovery.
3. For each prefix whose candidate set changed, generate a triggered update toward each remaining neighbor:
   - if the currently selected route may be advertised to that neighbor, send a fresh `ROUTE_ADVERTISE`;
   - otherwise send `ROUTE_WITHDRAW`.
4. Apply the same rule when a `ROUTE_WITHDRAW` is received from a neighbor.
5. Rebuild/program the forwarding table before forwarding subsequent traffic using the changed route.

This gives the failure chain:

```text
link/hold failure
      |
      v
neighbor DOWN
      |
      v
remove routes learned from neighbor
      |
      v
RIB/FIB select retained alternate (if any)
      |
      v
trigger ADVERTISE or WITHDRAW to remaining neighbors
      |
      v
traffic follows replacement route
```

Split horizon still applies to triggered advertisements. A triggered withdrawal may be sent where the current selected route is not advertisable to that neighbor, which explicitly clears reachability that neighbor may have learned earlier.

When a failed adjacency returns `UP`, normal advertisements are exchanged again. Deterministic metric/RouterID selection therefore converges back to the preferred route when it is again the best candidate.

## Deferred

- Link-state topology database.
- SPF/Dijkstra.
- ECMP and unequal-cost multipath.
- Hold-down timers, route poisoning, and poisoned reverse.
- Congestion-aware/adaptive routing.
- Up*/down* or another deadlock-free escape topology algorithm.
- Detailed VC0 escape forwarding behavior.
- Routing hierarchy/areas.
- Cryptographic routing authentication.
