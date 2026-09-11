---
id: gctl-message-registry
title: "GCTL Message Registry"
aliases: ["Control message registry","GCMP message registry"]
type: registry
status: draft
tags: ["gnet","gnet/registry","gnet/status/draft"]
parent: "[[Registries MOC]]"
related: ["[[GCTL Protocol]]","[[Discovery Packets]]","[[Address Configuration Packets]]","[[GNet Link Control Protocol]]"]
updated: 2026-09-11
---
# GCTL message registry

> [!info] Knowledge graph
> **Up:** [[Registries MOC]] · **Related:** [[GCTL Protocol]] · [[Discovery Packets]] · [[Address Configuration Packets]]

Status: **DRAFT allocations**

GCTL is the suite name. GCMP is the control-message wire protocol carried on the normal GNet data path.

## Message types

| Value | Message | Purpose |
|---:|---|---|
| `0x00` | RESERVED | invalid/unassigned |
| `0x01` | SOLICIT | scoped service/router discovery |
| `0x02` | ADVERTISE | discovery response |
| `0x03` | CREDIT_REQUEST | request link-local receive credit from the adjacent forwarding endpoint |
| `0x04` | CREDIT | advertise link-local receive capacity to the adjacent forwarding endpoint |
| `0x05..0x0F` | Reserved | future bootstrap/link-adjacent control |
| `0x10` | ADDRESS_OFFER | bootstrap address offer |
| `0x11` | ADDRESS_CLAIM | client claims offered address |
| `0x12` | ADDRESS_ACK | claim accepted |
| `0x13` | ADDRESS_NAK | claim rejected |
| `0x20` | ECHO_REQUEST | reachability/round-trip probe |
| `0x21` | ECHO_REPLY | echo response |
| `0x22` | DESTINATION_UNREACHABLE | routed delivery failed |
| `0x23` | HOP_LIMIT_EXCEEDED | GDP Hop Limit expired |
| `0x24` | PARAMETER_PROBLEM | malformed/unsupported routed packet |
| `0x25` | CLASS_UNSUPPORTED | GDP Size Class cannot continue |
| `0x26` | TRANSIT_ABORTED | admitted/forwarding packet later aborted |
| `0x27` | Reserved | future automatic diagnostic |
| `0x28` | PATH_PROBE | explicit path diagnostic probe |
| `0x29` | PATH_REPLY | path diagnostic response |
| `0x30` | STATUS_REQUEST | mandatory node-status query |
| `0x31` | STATUS_REPLY | mandatory node-status response |
| `0x32` | GET_REQUEST | retrieve managed entity/attributes |
| `0x33` | GET_REPLY | GET result |
| `0x34` | GET_NEXT_REQUEST | bounded entity/table enumeration |
| `0x35` | GET_NEXT_REPLY | next record/page |
| `0x36` | EVENT_REPORT | asynchronous best-effort management event |
| `0x37..0x7F` | Reserved | future diagnostics/management |
| `0x80..0x9F` | ROUTING CONTROL | reserved for route-distribution protocol(s) |
| `0xA0..0xFE` | Reserved | future standard/extension allocation |
| `0xFF` | EXPERIMENTAL | controlled experiments |

## Credit-control semantics

`CREDIT_REQUEST` and `CREDIT` are carried on the normal data path on both GC3 and GS3. They are not GLCP control-pair operations.

Credits are strictly link-local:

> **1 credit = guaranteed receive capacity for exactly one physical flit at the next forwarding endpoint on the current link.**

A GC3 Coupler does not consume these messages or hold credit state. On a shared GC3 medium, the adjacent endpoint or local router responds.

A GS3 is an active forwarding endpoint. It consumes a `CREDIT_REQUEST` for its ingress relationship and responds with `CREDIT` representing its own receive capacity. The GS3-to-egress credit relationship is independent.

Exact compact field encoding and batching limits for `CREDIT_REQUEST` / `CREDIT` remain DRAFT.

## DESTINATION_UNREACHABLE codes

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

## HOP_LIMIT_EXCEEDED codes

| Code | Meaning |
|---:|---|
| 0 | Hop Limit expired in transit |

## PARAMETER_PROBLEM codes

| Code | Meaning |
|---:|---|
| 0 | unsupported GDP version |
| 1 | malformed GDP header/field combination |
| 2 | invalid/reserved Size Class use |
| 3 | malformed destination/source address form |
| 4 | malformed GCTL control message |
| 5 | unsupported mandatory control feature |

## CLASS_UNSUPPORTED code

For `CLASS_UNSUPPORTED`, Code carries the largest lower GDP Size Class known to be accepted by the failing interface/profile. `0xFF` means unknown.

## TRANSIT_ABORTED codes

| Code | Meaning |
|---:|---|
| 0 | downstream link failed/reset |
| 1 | next-hop router disappeared |
| 2 | forwarding state/resource aborted |
| 3 | local hardware/internal forwarding fault |
| 4 | packet discarded during route transition |

## PATH_REPLY codes

| Code | Meaning |
|---:|---|
| 0 | INTERMEDIATE — probe expired at this router as intended |
| 1 | DESTINATION — probe reached destination |
| 2 | POLICY_LIMITED — node replies but with restricted detail |

Exact GET/GET_NEXT result codes, entity-attribute registries and event-code registries remain DRAFT.
