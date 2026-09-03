---
id: open-questions
title: "Open Questions"
aliases: ["Specification backlog"]
type: backlog
status: open
tags: ["gnet","gnet/backlog","gnet/status/open"]
parent: "[[GNet Home]]"
related: ["[[Specification Status]]","[[Decisions MOC]]"]
updated: 2026-09-03
---
# Open questions and specification backlog

Priority meanings: **P0** blocks interoperable native-link prototypes; **P1** blocks a complete routed network; **P2** can follow the first implementation.

## P0 — native-link interoperability

1. **DLP integrity:** freeze CRC width/polynomial, initialization/reflection, trailer packing, final partial carried-region handling, and CRC-failure recovery.
2. **GLCP wire encoding:** assign opcodes and compact field layouts for HELLO/capabilities/REQUEST/RX_REQUEST/CREDIT/GRANT/RELEASE/ABORT/RESET; validate the current ~1 Mbit/s control-channel target.
3. **Copper electrical spec:** freeze differential levels, impedance/termination, isolation, line code, clock recovery, attenuation/crosstalk/noise masks, reach qualification, and failure detection.
4. **GMC-8:** freeze pair-to-pin mapping, mechanical dimensions/keying, contact/shield rules, latch protection, and installation tooling.
5. **GC3 conformance:** validate 8-flit NORMAL scheduling quantum, exact REALTIME anti-starvation rule, timeout/recovery, and the current class-0..7 / realtime-1..3 package policy.
6. **VC state machine:** freeze allocation/reuse, timeout, abort/reset, stale-credit recovery, and malformed/early/late flit behavior for four baseline VCIDs.
7. **Golden vectors:** publish canonical GLCP sequences, GDP/DLP bit packing, error cases, and timing traces.

## P1 — complete network behavior

8. **GDP packing:** finalize Version/Type/QoS registries and malformed-packet rules for the 20-octet header candidate while preserving the frozen minimal field set.
9. **GCTL/bootstrap:** specify provisional/link-local GDP addressing, router discovery, prefix delegation/claim, retry/randomization, and authorization.
10. **Routing:** define route exchange, metrics, prefix delegation, policy, authentication, loop prevention, convergence, and failure recovery. Any authorized capable host may implement the router role.
11. **Address hierarchy:** freeze bit allocation/variable-prefix policy beneath the human-facing `Top / Org / Division / ... / Device` terminology.
12. **GNET-A:** define medium/modulation, polling/ranging, privacy, premise isolation, reservation scheduling, reach, and failure domains.
13. **GNET-P:** define synchronous framing, line codes, control carriage, keepalives, protection/failover, and coax/fiber profiles; resolve commercial naming versus LAN GNet-10.
14. **GNet-20:** define bonded-lane mode transition, in-band control encoding, lane synchronization, and recovery—or defer it to a later revision.
15. **Transport:** freeze GTS state machines, identifiers, retransmission, congestion behavior, receive-window units, fragmentation/reassembly, end-to-end integrity, reset/rebind, and stream profiles.

## P2 — products, applications, governance

16. **GC3-32:** determine whether electrical loading, scheduler complexity, and economics justify a 32-port shared Coupler or make GS3 the mandatory next step.
17. **Advanced VC4:** decide what future switched/cluster profile justifies wider VCIDs and how capability negotiation gates it.
18. **QDX-GNET ABI:** freeze queue descriptors/completions and optional acceleration features without leaking private semantics into GNet.
19. **Applications/services:** continue GTerm, boot, directory, RPC, file, voice, and Universal Server protocols.
20. **Management/security:** define telemetry, software loading, access control, link protection, routing trust, and key management.
21. **Standards governance:** define interoperability certification, extension/private ranges, deprecation/version rules, and patent/specification commitments.
22. **Reference implementation:** choose simulator/emulator targets, capture format, diagnostics, and reproducible interoperability tests.

## Recommended next decisions

Work next on: (1) GLCP encoding/state machine; (2) DLP CRC/trailer; (3) copper/GMC-8 electrical qualification; (4) golden GC3 traces including realtime preemption; (5) GCTL bootstrap and routing delegation.
