# Packet definitions

Packet layouts use RFC-style 32-bit diagrams. Every diagram row is one transmitted flit; multi-octet integers use network byte order. The common notation, padding, and DLP packaging rules are defined in [flit-format.md](flit-format.md).

| Packet family | Status |
|---|---|
| [GDP datagram](gdp-datagram.md) | fixed field set, draft widths |
| [Discovery](discovery.md) | draft |
| [Address configuration](address-configuration.md) | draft |
| [Session control](session.md) | draft common header; bodies open |
| [Transport](transport.md) | requirements preserved; encoding open |

Every future normative packet needs field validation rules, state-machine transitions, error behavior, and hexadecimal test vectors.
