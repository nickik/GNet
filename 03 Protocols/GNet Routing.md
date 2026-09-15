# GNet Dynamic Routing

Status: draft work item on branch `dynamic-topology-routing`.

This document will specify interoperable router adjacency, topology distribution, route computation inputs, and route-origin semantics for GNet. It intentionally keeps topology/control-plane behavior separate from packet forwarding, egress scheduling, DLP VC allocation, and future deadlock-escape routing.

## Architecture

```text
GCTL router adjacency
        |
        v
Topology database
        |
        v
Shortest-path calculation
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

## Stage 1 — Spec + data model

- [ ] Specify 64-bit `RouterID`, stable across router interfaces and independent of GDP interface addresses.
- [ ] Specify `LinkID` semantics and uniqueness scope for router-to-router links.
- [ ] Specify route origins: Connected, Learned, Static, and reserved Escape.
- [ ] Specify administrative preference independently of route origin.
- [ ] Define metric as an unsigned routing cost; initial routing uses a single deterministic scalar metric.
- [ ] Define GCTL router adjacency messages and exact wire layout.
- [ ] Define topology advertisement wire layout containing at least origin RouterID, sequence, lifetime/age, links, connected prefixes, and metrics.
- [ ] Define origin rules: a router originates only its own directly known topology state and does not rewrite another origin's advertisement.
- [ ] Define update rules: higher sequence accepted; duplicate sequence idempotently ignored; lower sequence rejected; expired state withdrawn.
- [ ] Reserve registry/message values required by the final GCTL encoding.

## Stage 2 — Adjacency + topology database

- [ ] Define adjacency state machine (`Down`, discovery/seen, bidirectional `Up`; exact names may change).
- [ ] Define hello/acknowledgement behavior, timers, hold/liveness semantics, and restart behavior.
- [ ] Define how directly connected GDP prefixes are announced by a router.
- [ ] Define topology flooding rules between router adjacencies.
- [ ] Define duplicate suppression by origin+sequence.
- [ ] Define topology database replacement and expiry semantics.
- [ ] Define link/prefix withdrawal after adjacency loss.
- [ ] Specify required two-router interoperability behavior: each router learns the other router and its directly connected prefixes using GCTL.

## Stage 3 — SPF + RIB/FIB

- [ ] Specify the conceptual RIB/FIB split.
- [ ] Specify route selection precedence between connected, static, and learned routes.
- [ ] Specify deterministic shortest-path behavior over the topology graph; initial algorithm semantics correspond to single-next-hop SPF/Dijkstra.
- [ ] Specify path-cost calculation for an advertised prefix.
- [ ] Specify deterministic tie-breaking so conforming routers do not depend on map/insertion order.
- [ ] Define the information a selected FIB entry must expose to the forwarding plane.
- [ ] Keep static routes supported while allowing fully learned cross-router paths.
- [ ] Reserve Escape as a distinct route class but do not define the escape-routing algorithm in this revision.

## Stage 4 — Failure + reconvergence

- [ ] Specify adjacency-down triggers: explicit local link failure and hold/liveness timeout.
- [ ] Specify topology withdrawal/change advertisement and sequence advancement after local state changes.
- [ ] Specify stale route invalidation before/while recomputing a replacement path.
- [ ] Specify deterministic reconvergence onto an alternate path.
- [ ] Specify behavior when a failed link returns.
- [ ] Add a normative example: preferred path, link failure, topology propagation, SPF recomputation, replacement FIB, traffic restored.

## Deferred from this revision

- ECMP and unequal-cost multipath.
- Congestion-aware/adaptive routing.
- Up*/down* or another deadlock-free escape topology algorithm.
- Detailed VC0 escape forwarding behavior.
- Routing hierarchy/areas.
- Cryptographic routing authentication.

## Required implementation proof

A conforming implementation should be able to run a router-to-router topology in which routers are configured only with their own connected interfaces plus local identity/link configuration. Cross-router reachability must emerge from GCTL adjacency, topology advertisements, SPF, RIB selection, and FIB installation rather than manually configured cross-router static routes.
