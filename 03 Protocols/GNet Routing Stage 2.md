# GNet Dynamic Routing — Stage 2 Neighbor Discovery

Status: draft, normative for Stage 2 implementation on branch `dynamic-topology-routing`.

This stage defines only point-to-point router neighbor discovery and liveness. Route propagation remains Stage 3.

## Adjacency model

Each router maintains one adjacency object for each local router-facing link/port. The local port association is implementation-local and is not encoded in GCTL.

The first revision has only two externally meaningful states:

```text
DOWN -> UP
UP   -> DOWN
```

A router starts `DOWN`.

## HELLO transmission

A router-facing adjacency may transmit `ROUTER_HELLO` using the Stage 1 wire format. The body contains the sender's:

- non-zero `RouterID`;
- non-zero local `LinkID`;
- non-zero hold time in milliseconds;
- scalar link metric.

A hold time of zero is invalid and MUST be rejected.

## HELLO reception

When a valid `ROUTER_HELLO` is received from a RouterID different from the local RouterID, the receiver:

1. records the remote RouterID, remote LinkID, advertised metric, and advertised hold time;
2. records the current receive time as `last_seen`;
3. transitions the adjacency to `UP`;
4. replies with `ROUTER_HELLO_ACK` using the same transaction ID.

The ACK body contains the receiver's own local RouterID, local LinkID, hold time, and metric.

A HELLO containing the receiver's own RouterID is invalid on that adjacency and MUST be rejected.

## HELLO_ACK reception

When a valid `ROUTER_HELLO_ACK` is received from a RouterID different from the local RouterID, the receiver records the peer information exactly as for HELLO reception, refreshes `last_seen`, and transitions the adjacency to `UP`.

No additional intermediate state is required in this revision. Transaction matching beyond echoing the HELLO transaction ID in the ACK is deferred.

## Liveness

The peer's advertised hold time controls expiry of that peer record.

If:

```text
now - last_seen >= peer_hold_time
```

then an `UP` adjacency transitions to `DOWN`.

A later valid HELLO or HELLO_ACK from the peer may transition it back to `UP`.

Receiving another HELLO or HELLO_ACK before expiry refreshes `last_seen` and replaces the stored remote LinkID, metric, and hold time with the newly advertised values.

## LinkID scope

The local and remote LinkID values are independent. The protocol does not require both ends of one physical link to use the same LinkID.

## Not in Stage 2

Stage 2 does not:

- exchange or install routes;
- process `ROUTE_ADVERTISE` as routing state;
- maintain a topology database;
- run SPF/Dijkstra;
- implement route withdrawals;
- implement ECMP or escape routing.

## Conformance proof

A Stage 2 implementation must demonstrate at least:

1. A sends HELLO to B; B records A as `UP` and returns an ACK with the same transaction ID.
2. A receives B's ACK and records B as `UP`.
3. Each side stores the peer RouterID, remote LinkID, metric, hold time, and last-seen time.
4. A repeated HELLO refreshes liveness and updates peer parameters.
5. The peer transitions to `DOWN` when its hold time elapses.
6. Zero hold time and self-RouterID HELLO/ACK messages are rejected.
7. Stage 1 wire-format tests and existing GCTL/network regression tests remain green.
