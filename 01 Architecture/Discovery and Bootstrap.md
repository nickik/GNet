---
id: discovery-and-bootstrap
title: "Discovery and Bootstrap"
aliases: ["GNet bootstrap"]
type: architecture
status: mixed
layers: ["L1","L3","L6"]
tags: ["gnet","gnet/architecture","gnet/status/mixed"]
parent: "[[Architecture MOC]]"
related: ["[[GNet Link Control Protocol]]","[[Discovery Packets]]","[[Address Configuration Packets]]","[[GNet Service Model]]"]
updated: 2026-09-03
---
# Discovery and bootstrap

Status: **ACCEPTED sequence; DRAFT network-control encodings/trust**

GNet separates link bootstrap from network discovery.

## Link bootstrap

GLCP first establishes:

1. physical/link synchronization;
2. Minimum GNet-3 compatibility;
3. endpoint/infrastructure presence;
4. capability and rate selection;
5. usable control/data state.

REQUEST/CREDIT/GRANT and other link-control operations remain GLCP and never become GDP discovery messages.

## Network bootstrap

After link establishment, an unconfigured endpoint uses a tightly scoped provisional/link-local GDP/GCTL bootstrap exchange to discover an authorized router and obtain/delegate network addressing information. Exact provisional GDP addressing is still DRAFT.

Preferred sequence:

1. GLCP link activation and physical port identity.
2. GDP/GCTL `SOLICIT(Router)` with link-local/provisional scope.
3. Direct/tightly scoped `ADVERTISE(Router)` responses.
4. prefix/address offer, claim, and confirmation.
5. authentication/terminal registration when policy requires it.
6. ordinary routed GDP/GCTL or directory queries for Directory, Terminal Server, Boot, Time, Identity, and other services.
7. end-to-end session establishment with the selected service.

GNet does not flood discovery into the routed network. Scope may not be silently expanded by intermediaries.

## Directory boundary

Directory services map logical names to providers after basic network reachability exists. A ROM terminal may use a local terminal server as a defined fallback, but this does not turn the physical control pair into a directory protocol.

## Failure/trust

Solicitations use randomized retry. Multiple advertisements are ranked by policy/preference/reachability. Router bootstrap trust, exact timers, caching, and multi-router coordination remain OPEN.
