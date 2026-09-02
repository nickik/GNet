# GTS transport packets

Status: **OPEN; historical field requirements preserved**

## Previously corrected CONNECT field list

The latest recorded CONNECT proposal contained, in order:

| Field | Bits |
|---|---:|
| Type | 4 |
| Reserved | 4 |
| Receive Window | 8 |
| Destination Port | 16 |
| Receive Window | 12 |
| Window Scale | 24 |
| Tunnel ID | 64 |
| Source Port | 16 |
| Checksum | 16 |

Total: **164 bits (20.5 octets)**. The duplicated Receive Window and half-octet total must be resolved before encoding.

## Previously corrected CONNECT_ACK field list

| Field | Bits |
|---|---:|
| Type | 4 |
| Reserved | 12 |
| Checksum | 16 |
| Receive Window | 12 |
| Receive Window Scale | 24 |
| Tunnel ID | 64 |
| Tunnel Handle | 64 |

Total: **196 bits (24.5 octets)**. This also requires alignment correction.

Earlier requirements also called for 32-bit tunnel sequence and acknowledgment numbers. The next transport decision must define CONNECT, CONNECT_ACK, DATA, ACK, RESET, and CLOSE together, with exact state machines and test vectors, rather than repairing these layouts in isolation.
