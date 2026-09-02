# Decision 0002: Minimal GDP header

Status: **FROZEN field set; DRAFT widths**

GDP contains exactly Version, Type, Hop Limit, QoS, Source Address, and Destination Address. Payload length is known from DLP. Integrity, fragmentation, options, reliability, flow/session identification, and encryption are intentionally excluded.

The working encoding assigns one octet to each control field and eight octets to each address, producing exactly five 32-bit flits (20 octets). This allocation can change before version 1 without changing the architectural decision.
