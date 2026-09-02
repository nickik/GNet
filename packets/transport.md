# GTS transport packets

Status: **OPEN; historical field requirements preserved**

## Current semantic packet family

The tunnel/stream model requires encodings for:

- **CONNECT** proposes Tunnel ID, reset authority, service selector, security/profile, and initial flow-control values.
- **CONNECT_ACK** accepts and returns a local Tunnel Handle and negotiated values.
- **STREAM_OPEN** requests a Stream ID and delivery/security/compression profile.
- **STREAM_ACCEPT** accepts or modifies the stream profile.
- **DATA** carries Stream ID, data/fragment position, and only the state required by that profile.
- **ACK** acknowledges reliable/sequenced delivery and advertises flow-control state.
- **STREAM_CLOSE** ends one stream without destroying the tunnel.
- **TUNNEL_CLOSE, RESET, and REBIND** are destructive controls that MUST prove the Reset ID.

Normal DATA MUST NOT contain the Reset ID. Service names, ports, or service codes appear during CONNECT/STREAM_OPEN and are not repeated on DATA.

## Mechanical flit packing of the historical CONNECT proposal

The latest recorded field sequence totals 164 meaningful bits. Packed without implicit alignment, it occupies six flits and needs 28 final padding bits:

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Type  | Rsvd  | Recv Window  |       Destination Port         | Flit 0
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Receive Window (12) |          Window Scale [23:4]            | Flit 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |WS[3:0]|                 Tunnel ID [63:36]                     | Flit 2
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Tunnel ID [35:4]                           | Flit 3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |TID[3:0]|       Source Port       |      Checksum [15:4]       | Flit 4
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Sum[3:0]|                  Padding = 0                         | Flit 5
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

This diagram exposes rather than repairs the problems: two different Receive Window fields and several fields split across flit boundaries. It is **not** an accepted wire encoding.

## Mechanical flit packing of the historical CONNECT_ACK proposal

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Type  |    Reserved (12)     |           Checksum             | Flit 0
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Receive Window (12) |          Window Scale [23:4]            | Flit 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |WS[3:0]|                 Tunnel ID [63:36]                     | Flit 2
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Tunnel ID [35:4]                           | Flit 3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |TID[3:0]|              Tunnel Handle [63:36]                   | Flit 4
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                  Tunnel Handle [35:4]                         | Flit 5
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |TH [3:0]|                  Padding = 0                         | Flit 6
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

CONNECT_ACK contains 196 meaningful bits, occupies seven flits, and also needs 28 final padding bits. It is **not** an accepted wire encoding.

Earlier requirements also called for 32-bit tunnel sequence and acknowledgment numbers. The next transport decision must define CONNECT, CONNECT_ACK, DATA, ACK, RESET, and CLOSE together, with exact state machines and test vectors, rather than repairing these layouts in isolation.
