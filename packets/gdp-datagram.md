# GDP datagram packet

Status: **DRAFT 20-octet encoding; FROZEN field set**

| Offset | Size | Field |
|---:|---:|---|
| 0 | 1 | Version |
| 1 | 1 | Type |
| 2 | 1 | Hop Limit |
| 3 | 1 | QoS |
| 4 | 8 | Source address |
| 12 | 8 | Destination address |
| 20 | remaining DLP payload | Payload |

Total GDP header: **20 octets**. The DLP payload length determines the GDP payload length.

A source of zero is permitted only during explicitly defined bootstrap exchanges. A normal router decrements Hop Limit before transmission; if the result is zero it discards the packet and may return a control error once such errors are defined.

Version 0 is reserved. The first interoperable version is expected to use Version 1. Type and QoS values are provisional registries. No padding or optional header is implied by this layout.
