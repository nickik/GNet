# Direct Link Protocol (DLP / GNET-LINK)

Status: **FROZEN role; DRAFT encoding**

DLP is the common L2 envelope used by GNET-L, GNET-A, and GNET-P. It identifies the carried protocol, delimits a payload, and detects link corruption. The medium-specific layer supplies ingress/egress port or channel identity.

## Draft frame

| Offset | Size | Field | Meaning |
|---:|---:|---|---|
| 0 | 1 | Protocol | Registry value for GDP, link-local GCTL, or a foreign protocol. |
| 1 | 1 | Link class | Medium-local scheduling class; zero is ordinary best effort. |
| 2 | 2 | Payload length | Unsigned payload octets, network byte order. |
| 4 | N | Payload | Protocol data. |
| 4+N | 1 | CRC-8 | CRC over header and payload; polynomial is OPEN. |

Preamble, synchronization, idle symbols, and request/grant signaling are physical-medium concerns and are not counted in Payload length.

DLP has no universal MAC source or destination fields. Point-to-point wiring, a hub port, a GNET-A premise channel, or a GNET-P trunk supplies local delivery identity. Link-local fanout is an explicit control operation, never learned switching.

The maximum payload, minimum frame, alignment, CRC polynomial, error counters, and foreign-protocol encapsulation details are OPEN.
