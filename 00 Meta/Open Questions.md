---
id: open-questions
title: "Open Questions"
aliases: ["Specification backlog"]
type: backlog
status: open
tags: ["gnet","gnet/backlog","gnet/status/open"]
parent: "[[GNet Home]]"
related: ["[[Specification Status]]","[[Decisions MOC]]"]
updated: 2026-09-02
---
# Open questions and specification backlog

> [!info] Knowledge graph
> **Up:** [[GNet Home]] · **Related:** [[Specification Status]] · [[Decisions MOC]]


Priority meanings: **P0** blocks interoperable prototype work; **P1** blocks a complete network; **P2** can follow an initial implementation.

## P0 — wire compatibility

1. **DLP segment framing:** settle the 8-bit Protocol, 2-bit Link Class, 2-bit Segment Class, and 16-bit Payload Length carried header; define inter-segment gap, padding validation, integrity-trailer width/algorithm/polynomial, and malformed-segment handling. The base payload-size classes are already fixed at 64, 256, and 1,024 octets.
2. **VCID state machine:** allocate or reserve the 16 VCID values; define implicit versus explicit allocation, per-direction reuse, trailer completion, cancellation, timeout, CRC failure, stale-state prevention, interleaving fairness, and any persistent Link Flow ID binding. The four-bit field and 28-bit carried region are already frozen.
3. **GNET-L electrical layer:** choose line code, clock recovery, voltage/termination, connector pinout, cable category and reach, isolation, grant timing, and failure detection.
4. **GNET-L scale:** reconcile the simple initial electrical design with 16/32/64-port passive hubs; define when repeating or active stages are mandatory.
5. **Pair 4:** choose reserved use—clock, power/control, protection, or future bandwidth—or require it to remain unused.
6. **GDP encoding:** accept or replace the draft 8/8/8/8/64/64-bit, 20-octet header; define Version 1, malformed-packet behavior, and QoS bit semantics. GDP integrity is not open: GDP has no integrity field.
7. **Path payload limit:** choose link MTUs and the endpoint fragmentation/reassembly method without adding fields to GDP.
8. **Address configuration:** define prefix-offer authority, random-suffix collision probability, claim timers, persistent leases, multi-router coordination, and renumbering.
9. **Transport redesign:** resolve the duplicate Receive Window, half-octet CONNECT/ACK sizes, sequence/ack fields, end-to-end checksum, handles, stream packets, Reset ID proof, and state machines.
10. **Service selection:** choose 16-bit ports, setup-only short service codes, directory service identifiers, or a defined combination; settle which fields appear in CONNECT versus STREAM_OPEN.
11. **Conformance:** add canonical hexadecimal frames, state-machine tests, and validation/error cases for every packet.

## P1 — complete network behavior

12. **Address hierarchy:** allocate or explicitly make variable the Region/Metro/District/Facility/Customer/Device fields; define organizational addressing and aggregation boundaries.
13. **Intra/inter-domain routing:** specify neighbor discovery, route exchange, metrics, policy, authentication, loop prevention, convergence, and failure recovery for the proposed G-RIB/GNet-PGP.
14. **Control errors and OAM:** define unreachable, hop-limit, echo, trace, link-state monitoring, counters, alarms, and rate limiting.
15. **Directory:** define namespace syntax, the recovered service-record fields, registrar/metro replication, provider ranking, cache and lifetime rules, consistency, updates, and failure modes.
16. **Discovery selection:** define retry/randomization, preference, multiple providers, scope forwarding, fallback terminal servers, and denial-of-service limits.
17. **Bootstrap trust:** decide how a client trusts a router and directory before ordinary user authentication.
18. **Identity and DigitalKey:** choose credentials, challenge/response, key storage, revocation, delegation, billing/telephone associations, and an implementable early-1980s cryptographic profile.
19. **Transport algorithms:** define retransmission timers, congestion behavior, receive-window units/scaling, duplicate suppression, keepalive, reset, rebind, simultaneous open, and per-stream profiles.
20. **Reservations:** define requested bandwidth/delay/loss parameters, admission control across every link in the path, soft-state refresh, preemption, QoS mapping, and teardown.
21. **Mobility:** choose stable versus locator addresses, home/district registrar roles, handover, stale-session recovery, tunnel rebind, and security.
22. **GSC encoding:** define the common envelope, transaction/dialog identifiers, directory bindings, error vocabulary, retransmission, and negotiation representation.
23. **GNET-A:** define medium/modulation, polling and ranging, privacy, premise isolation, reserved-channel scheduling, reach, and failure domains.
24. **GNET-P:** define synchronous framing, line codes, clock hierarchy, keepalives, channel scheduling, protection/failover, and coax-to-fiber compatibility.

## P2 — applications, operations, and governance

25. **GTerm:** define character sets, terminal capabilities, echo/editing, screen model, keyboard events, flow control, multiplexing, SESSION-key behavior, and graphics/file extensions.
26. **Boot:** define image naming, discovery, transfer, verification, local/cartridge/network source selection, fallback, and configuration delivery for ROM clients.
27. **RPC and Universal Digital Bytecode:** define naming, schemas/capabilities, error model, cancellation, deadlines, retries, and safe module upload.
28. **Gateways:** specify DECnet, TCP/IP, XNS, and SNA encapsulation/gateway behavior and prevent accidental layer leakage.
29. **Telephony gateways:** define AM/E.164 records, SS7 mapping, PCM conversion, release/accounting events, and failure recovery without leaking telephone numbers into GDP routing.
30. **Management and accounting:** define configuration, telemetry, software loading, access control, usage records, and fault isolation while keeping accounting out of forwarding.
31. **Implementation profiles:** define minimal terminal, workstation, local router, GNET Access 256, GD16, GC32, high-port-count switch, high-speed trunk, and voice-gateway profiles.
32. **QDX-GNET ABI:** freeze base queue descriptors/completions and define which batch/copy/route accelerators are optional.
33. **Forwarding mode:** decide where store-and-forward is mandatory and whether cut-through, wormhole, credits, or hop-local labels are compatible optional profiles.
34. **Versioning and registries:** decide allocation authority, experimental/private ranges, extension compatibility, deprecation, and negotiation.
35. **Standards governance:** confirm how the existing MPL 2.0 applies to specification text, and define any additional patent commitment, change process, implementer review, and stewardship of the open standard.
36. **Reference implementation:** choose language, simulator/emulator targets, capture format, diagnostics, and reproducible interoperability lab.

## Recommended next decisions

Work in this order: (1) freeze the DLP carried header, integrity trailer, and VCID state machine; (2) freeze the 20-octet GDP field encoding, with no GDP integrity field; (3) settle GNET-L signaling/pinout/timing; (4) choose the service selector and repair GTS CONNECT around the Tunnel/Reset/Stream model; (5) define the complete transport state machine and its end-to-end integrity; (6) publish golden packet vectors. Those decisions create the smallest coherent implementable profile.
