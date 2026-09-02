# Open questions and specification backlog

Priority meanings: **P0** blocks interoperable prototype work; **P1** blocks a complete network; **P2** can follow an initial implementation.

## P0 — wire compatibility

1. **DLP framing:** confirm header fields, maximum payload, alignment, inter-frame gap, CRC-8 polynomial/initial value/reflection, and error handling.
2. **GNET-L electrical layer:** choose line code, clock recovery, voltage/termination, connector pinout, cable category and reach, isolation, grant timing, and failure detection.
3. **GNET-L scale:** reconcile the simple initial electrical design with 16/32/64-port passive hubs; define when repeating or active stages are mandatory.
4. **Pair 4:** choose reserved use—clock, power/control, protection, or future bandwidth—or require it to remain unused.
5. **GDP encoding:** accept or replace the draft 8/8/8/8/64/64-bit, 20-octet header; define Version 1, byte order, malformed-packet behavior, and QoS bit semantics.
6. **Path payload limit:** choose link MTUs and the endpoint fragmentation/reassembly method without adding fields to GDP.
7. **Address configuration:** define prefix-offer authority, random-suffix collision probability, claim timers, persistent leases, multi-router coordination, and renumbering.
8. **Transport redesign:** resolve the duplicate Receive Window, half-octet CONNECT/ACK sizes, sequence/ack fields, ports, checksum, handles, and state machines.
9. **Conformance:** add canonical hexadecimal frames, state-machine tests, and validation/error cases for every packet.

## P1 — complete network behavior

10. **Address hierarchy:** allocate or explicitly make variable the Region/Metro/District/Facility/Customer/Device fields; define organizational addressing and aggregation boundaries.
11. **Intra/inter-domain routing:** specify neighbor discovery, route exchange, metrics, policy, authentication, loop prevention, convergence, and failure recovery for the proposed G-RIB/GNet-PGP.
12. **Control errors and OAM:** define unreachable, hop-limit, echo, trace, link-state monitoring, counters, alarms, and rate limiting.
13. **Directory:** define namespace syntax, service records, registrar/metro replication, cache and lifetime rules, consistency, updates, and failure modes.
14. **Discovery selection:** define retry/randomization, preference, multiple providers, scope forwarding, fallback terminal servers, and denial-of-service limits.
15. **Bootstrap trust:** decide how a client trusts a router and directory before ordinary user authentication.
16. **Identity and smart cards:** choose credentials, challenge/response, key storage, revocation, delegation, and an implementable early-1980s cryptographic profile.
17. **Transport algorithms:** define retransmission timers, congestion behavior, receive-window units/scaling, duplicate suppression, keepalive, reset, and simultaneous open.
18. **Reservations:** define requested bandwidth/delay/loss parameters, admission control, soft-state refresh, preemption, QoS mapping, and teardown.
19. **Mobility:** choose stable versus locator addresses, home/district registrar roles, handover, stale-session recovery, and security.
20. **GNET-A:** define medium/modulation, polling and ranging, privacy, premise isolation, reserved-channel scheduling, reach, and failure domains.
21. **GNET-P:** define synchronous framing, line codes, clock hierarchy, keepalives, channel scheduling, protection/failover, and coax-to-fiber compatibility.

## P2 — applications, operations, and governance

22. **GTerm:** define character sets, terminal capabilities, echo/editing, screen model, keyboard events, flow control, multiplexing, and graphics/file extensions.
23. **Boot:** define image naming, discovery, transfer, verification, fallback, and configuration delivery for ROM clients.
24. **RPC and Universal Digital Bytecode:** define naming, schemas/capabilities, error model, cancellation, deadlines, retries, and safe module upload.
25. **Gateways:** specify DECnet, TCP/IP, XNS, and SNA encapsulation/gateway behavior and prevent accidental layer leakage.
26. **Management and accounting:** define configuration, telemetry, software loading, access control, usage records, and fault isolation while keeping accounting out of forwarding.
27. **Implementation profiles:** define minimal terminal, workstation, local router, district hub, high-port-count switch, and high-speed trunk profiles.
28. **Versioning and registries:** decide allocation authority, experimental/private ranges, extension compatibility, deprecation, and negotiation.
29. **Licensing and standards governance:** select repository license, patent policy, change process, implementer review, and ownership of the open standard.
30. **Reference implementation:** choose language, simulator/emulator targets, capture format, diagnostics, and reproducible interoperability lab.

## Recommended next decisions

Work in this order: (1) freeze the 20-octet GDP header; (2) freeze DLP framing and CRC; (3) settle GNET-L signaling/pinout/timing; (4) repair GTS CONNECT and define the complete transport state machine; (5) publish golden packet vectors. Those five decisions create the smallest coherent implementable profile.
