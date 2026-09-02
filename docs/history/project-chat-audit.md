# Project-chat recovery audit

Status: **INFORMATIVE**

This audit records material recovered from earlier Project GNET discussions and how it was incorporated without silently promoting proposals to standards.

## Added as accepted architecture

- one-endpoint-per-logical-line DLP with no L2 address or session identity;
- tunnel-first GTS with a separate Reset ID required for close/reset/rebind;
- multiple independently negotiated streams within one tunnel;
- logical service names that may resolve to several local or remote providers;
- directory records carrying capability, authentication, load/preference, location, and access policy;
- GTerm multiplexing with a locally trapped SESSION key;
- local ROM/OS, cartridge/cassette, and network boot as distinct boot sources;
- signaling/media separation and direct endpoint media after GSC setup;
- external SS7/E.164 handling at gateways and databases, not GDP routing;
- home star, scheduled neighbourhood access, routed district trunks, dual-homed aggregation, and a redundant metro core;
- strict separation of PLIO, QDX/QDX-GNET, and network protocol semantics.

## Preserved as drafts

- 16-bit source/destination ports versus setup-only service codes of up to seven characters;
- 64-bit Reset ID, 16-bit Stream ID, control stream 0, and default stream 1;
- the GSC method set and numeric registry;
- provider-ranking and directory-record encodings;
- GNET Access 256, GD16, GC32, and Voice Gateway 96 product profiles;
- QDX-GNET batch/copy/queued acceleration commands;
- store-and-forward, cut-through/wormhole, credit, and hop-local-label acceleration.

## Superseded or rejected

- the earlier 2.5 Mb/s GNET-L planning value is superseded by the newer approximately 3 Mb/s target;
- an earlier 32-bit address hierarchy is superseded by the 64-bit hierarchical GDP address;
- shared district/metro cable-style channels are superseded by dedicated routed trunks and a redundant routed core;
- GPP was an earlier name for the minimal point-to-point layer; the accepted names are DLP at L2 and GDP at L3;
- QDX-GNET does not absorb routing, discovery, reliability, authentication, or RPC semantics.

## Conflicts still open

The chat history does not settle minimal DLP framing versus the current length/class draft, numeric ports versus named service selectors, exact GTS fields, packet MTUs/fragmentation, or store-and-forward versus wormhole/cut-through behavior. These remain explicit entries in [OPEN-QUESTIONS.md](../../OPEN-QUESTIONS.md).
