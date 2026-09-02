# Packet definitions

Packet tables use network byte order for all multi-octet integers. Bit 7 is the most significant bit of an octet unless a document says otherwise.

| Packet family | Status |
|---|---|
| [GDP datagram](gdp-datagram.md) | fixed field set, draft widths |
| [Discovery](discovery.md) | draft |
| [Address configuration](address-configuration.md) | draft |
| [Session control](session.md) | semantic draft |
| [Transport](transport.md) | requirements preserved; encoding open |

Every future normative packet needs field validation rules, state-machine transitions, error behavior, and hexadecimal test vectors.
