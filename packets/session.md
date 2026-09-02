# Session-control messages

Status: **DRAFT common header; message bodies OPEN**

## Common GSC envelope

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  GSC Version  | Message Type  |             Flags             | Flit 0
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Transaction ID                          | Flit 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         Dialog ID                             | Flit 2
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Message-specific body                      | Flit 3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                              ...                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Transaction ID makes requests repeatable and matches immediate responses. Dialog ID correlates messages belonging to an established or establishing session; it is zero when no dialog exists. The enclosing DLP frame supplies the exact total length. Body encoding remains OPEN.

## Required operations

- **REGISTER:** endpoint to registrar; publish current device/service reachability.
- **LOOKUP:** endpoint to directory; resolve a name or number.
- **INVITE:** caller to service/peer; propose a session and requested capabilities/resources.
- **OFFER:** either endpoint to peer; describe media, codecs, security, and resources.
- **ALERT:** peer to caller; indicate that the target is being notified.
- **ACCEPT:** peer to caller; accept and return session parameters.
- **REJECT:** peer to caller; refuse with a reason.
- **CANCEL:** caller to peer; abandon an unanswered invitation.
- **RELEASE:** either endpoint; close an established session.
- **UPDATE:** either endpoint; renegotiate media, keys, or reservations.
- **TRANSFER:** endpoint/service to peer; redirect or transfer a participant.
- **KEEPALIVE:** either direction; maintain registration or session soft state.
- **RESERVE:** endpoint to network service; request bounded-delay/bandwidth resources.
- **RESERVE_RESULT:** network service to endpoint; accept, modify, or reject a reservation.

All requests need an idempotent transaction or invitation identifier above GDP. Session operations bind to endpoint addresses and, when present, authenticated identities. Encoding waits on the GTS identifier and security decisions.

Session signaling is not the media path. Once accepted, media/data packets normally flow directly between endpoints. Internal calls bypass a PSTN gateway; E.164 and SS7 are handled by directory/session services and external gateways rather than GDP routing.
