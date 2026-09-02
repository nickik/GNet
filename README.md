# GNet

GNet is a clean-slate, hardware-efficient network architecture designed as if Digital Equipment Corporation had begun deploying an integrated local, metropolitan, and wide-area network in the late 1970s and early 1980s.

This repository is the working standards tree. It separates settled architectural constraints from provisional wire formats and unresolved design work.

## Design goals

- simple hardware forwarding and fixed-format headers;
- point-to-point or centrally controlled links, without dependence on a shared MAC or global broadcast;
- one routed datagram format across local, access, and trunk media;
- hierarchical 64-bit global addresses and end-to-end reachability without NAT;
- endpoint-owned sessions, reliability, flow control, fragmentation, and encryption;
- explicit support for real-time reserved flows, mobility, terminals, and service discovery;
- protocol neutrality at the link layer so DECnet, TCP/IP, XNS, SNA gateways, and later protocols can coexist.

## Specification map

| Area | Document |
|---|---|
| Normative status and terminology | [STATUS.md](STATUS.md), [GLOSSARY.md](GLOSSARY.md) |
| Layer model | [docs/architecture/layer-model.md](docs/architecture/layer-model.md) |
| Addressing and routing | [docs/architecture/addressing-routing.md](docs/architecture/addressing-routing.md) |
| Bootstrap and discovery | [docs/architecture/discovery-bootstrap.md](docs/architecture/discovery-bootstrap.md) |
| Names and services | [docs/architecture/service-model.md](docs/architecture/service-model.md) |
| Transport and reserved flows | [docs/architecture/transport-flows.md](docs/architecture/transport-flows.md) |
| Identity and security | [docs/architecture/security-identity.md](docs/architecture/security-identity.md) |
| Terminal service | [docs/architecture/terminal-services.md](docs/architecture/terminal-services.md) |
| Deployment and implementation | [docs/architecture/deployment-topology.md](docs/architecture/deployment-topology.md), [docs/architecture/implementation-boundary.md](docs/architecture/implementation-boundary.md) |
| Link family | [specs/gnet-link.md](specs/gnet-link.md), [specs/gnet-l.md](specs/gnet-l.md), [specs/gnet-a.md](specs/gnet-a.md), [specs/gnet-p.md](specs/gnet-p.md) |
| Routed datagram | [specs/gdp.md](specs/gdp.md), [packets/gdp-datagram.md](packets/gdp-datagram.md) |
| Control and discovery | [specs/gnet-control.md](specs/gnet-control.md), [packets/discovery.md](packets/discovery.md), [packets/address-configuration.md](packets/address-configuration.md) |
| Sessions and transport | [specs/gnet-transport.md](specs/gnet-transport.md), [packets/session.md](packets/session.md), [packets/transport.md](packets/transport.md) |
| Session applications | [specs/gnet-session-control.md](specs/gnet-session-control.md), [specs/gterm.md](specs/gterm.md) |
| Registries | [registry/protocol-types.md](registry/protocol-types.md), [registry/service-types.md](registry/service-types.md), [registry/control-messages.md](registry/control-messages.md), [registry/session-messages.md](registry/session-messages.md) |
| Recovered project history | [docs/history/project-chat-audit.md](docs/history/project-chat-audit.md) |
| Unresolved work | [OPEN-QUESTIONS.md](OPEN-QUESTIONS.md) |

## Status notation

Documents use **FROZEN**, **ACCEPTED**, **DRAFT**, and **OPEN**. Only FROZEN statements are architectural constraints. Numeric values marked DRAFT are working allocations and may change before a 1.0 wire specification.

## Repository scope

GNet defines protocols, not a particular router chassis. QDX may implement and accelerate a GNet switch internally, but QDX is not a network layer and does not own routing, discovery, authentication, or transport semantics.

## License

This repository is licensed under the [Mozilla Public License 2.0](LICENSE).
