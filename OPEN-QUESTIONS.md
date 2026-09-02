# Open questions and specification backlog

Priority meanings: **P0** blocks interoperable prototype work; **P1** blocks a complete network; **P2** can follow an initial implementation.

## P0 — wire compatibility

1. **DLP framing:** decide whether minimal DLP is only a protocol discriminator plus CRC or uses the current class/length draft; confirm maximum payload, alignment, inter-frame gap, CRC-8 polynomial/initial value/reflection, and error handling.
2. **GNET-L electrical layer:** choose line code, clock recovery, voltage/termination, connector pinout, cable category and reach, isolation, grant timing, and failure detection.
3. **GNET-L scale:** reconcile the simple initial electrical design with 16/32/64-port passive hubs; define when repeating or active stages are mandatory.
4. **Pair 4:** choose reserved use—clock, power/control, protection, or future bandwidth—or require it to remain unused.
5. **GDP encoding:** accept or replace the draft 8/8/8/8/64/64-bit, five-flit header; define Version 1, malformed-packet behavior, and QoS bit semantics.
6. **Path payload limit:** choose link MTUs and the endpoint fragmentation/reassembly method without adding fields to GDP.
7. **Address configuration:** define prefix-offer authority, random-suffix collision probability, claim timers, persistent leases, multi-router coordination, and renumbering.
8. **Transport redesign:** resolve the duplicate Receive Window, half-octet CONNECT/ACK sizes, sequence/ack fields, checksum, handles, stream packets, Reset ID proof, and state machines.
9. **Service selection:** choose 16-bit ports, setup-only short service codes, directory service identifiers, or a defined combination; settle which fields appear in CONNECT versus STREAM_OPEN.
10. **Conformance:** add canonical hexadecimal frames, state-machine tests, and validation/error cases for every packet.

## P1 — complete network behavior

11. **Address hierarchy:** allocate or explicitly make variable the Region/Metro/District/Facility/Customer/Device fields; define organizational addressing and aggregation boundaries.
12. **Intra/inter-domain routing:** specify neighbor discovery, route exchange, metrics, policy, authentication, loop prevention, convergence, and failure recovery for the proposed G-RIB/GNet-PGP.
13. **Control errors and OAM:** define unreachable, hop-limit, echo, trace, link-state monitoring, counters, alarms, and rate limiting.
14. **Directory:** define namespace syntax, the recovered service-record fields, registrar/metro replication, provider ranking, cache and lifetime rules, consistency, updates, and failure modes.
15. **Discovery selection:** define retry/randomization, preference, multiple providers, scope forwarding, fallback terminal servers, and denial-of-service limits.
16. **Bootstrap trust:** decide how a client trusts a router and directory before ordinary user authentication.
17. **Identity and DigitalKey:** choose credentials, challenge/response, key storage, revocation, delegation, billing/telephone associations, and an implementable early-1980s cryptographic profile.
18. **Transport algorithms:** define retransmission timers, congestion behavior, receive-window units/scaling, duplicate suppression, keepalive, reset, rebind, simultaneous open, and per-stream profiles.
19. **Reservations:** define requested bandwidth/delay/loss parameters, admission control across every link in the path, soft-state refresh, preemption, QoS mapping, and teardown.
20. **Mobility:** choose stable versus locator addresses, home/district registrar roles, handover, stale-session recovery, tunnel rebind, and security.
21. **GSC encoding:** define the common envelope, transaction/dialog identifiers, directory bindings, error vocabulary, retransmission, and negotiation representation.
22. **GNET-A:** define medium/modulation, polling and ranging, privacy, premise isolation, reserved-channel scheduling, reach, and failure domains.
23. **GNET-P:** define synchronous framing, line codes, clock hierarchy, keepalives, channel scheduling, protection/failover, and coax-to-fiber compatibility.

## P2 — applications, operations, and governance

24. **GTerm:** define character sets, terminal capabilities, echo/editing, screen model, keyboard events, flow control, multiplexing, SESSION-key behavior, and graphics/file extensions.
25. **Boot:** define image naming, discovery, transfer, verification, local/cartridge/network source selection, fallback, and configuration delivery for ROM clients.
26. **RPC and Universal Digital Bytecode:** define naming, schemas/capabilities, error model, cancellation, deadlines, retries, and safe module upload.
27. **Gateways:** specify DECnet, TCP/IP, XNS, and SNA encapsulation/gateway behavior and prevent accidental layer leakage.
28. **Telephony gateways:** define AM/E.164 records, SS7 mapping, PCM conversion, release/accounting events, and failure recovery without leaking telephone numbers into GDP routing.
29. **Management and accounting:** define configuration, telemetry, software loading, access control, usage records, and fault isolation while keeping accounting out of forwarding.
30. **Implementation profiles:** define minimal terminal, workstation, local router, GNET Access 256, GD16, GC32, high-port-count switch, high-speed trunk, and voice-gateway profiles.
31. **QDX-GNET ABI:** freeze base queue descriptors/completions and define which batch/copy/route accelerators are optional.
32. **Forwarding mode:** decide where store-and-forward is mandatory and whether cut-through, wormhole, credits, or hop-local labels are compatible optional profiles.
33. **Versioning and registries:** decide allocation authority, experimental/private ranges, extension compatibility, deprecation, and negotiation.
34. **Standards governance:** confirm how the existing MPL 2.0 applies to specification text, and define any additional patent commitment, change process, implementer review, and stewardship of the open standard.
35. **Reference implementation:** choose language, simulator/emulator targets, capture format, diagnostics, and reproducible interoperability lab.

## Recommended next decisions

Work in this order: (1) freeze the 20-octet GDP header; (2) freeze DLP framing and CRC; (3) settle GNET-L signaling/pinout/timing; (4) choose the service selector and repair GTS CONNECT around the Tunnel/Reset/Stream model; (5) define the complete transport state machine; (6) publish golden packet vectors. Those decisions create the smallest coherent implementable profile.
