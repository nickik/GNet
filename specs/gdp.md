# GNet Datagram Protocol (GDP)

Status: **FROZEN field set; DRAFT encoding**

GDP is the common routed L3 protocol. Its header is fixed and deliberately contains only six fields.

## Required semantics

- **Version** selects the GDP wire version.
- **Type** identifies the payload protocol.
- **Hop Limit** is decremented at each GDP router; a packet reaching zero is discarded.
- **QoS** selects forwarding and scheduling behavior, subject to local policy.
- **Source** and **Destination** are 64-bit GDP addresses.

GDP MUST NOT acquire length, checksum/hash, fragmentation, options, flow/session ID, sequence numbers, acknowledgments, or encryption metadata. DLP supplies frame length and link CRC; endpoints supply the rest above GDP.

The current encoding is exactly five 32-bit flits (20 octets) and is defined in [packets/gdp-datagram.md](../packets/gdp-datagram.md). Protocol Type and QoS allocations are DRAFT.
