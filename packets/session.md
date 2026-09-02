# Session-control messages

Status: **SEMANTIC DRAFT; encoding OPEN**

The following operations are required for terminal, voice, and other negotiated services:

| Operation | Direction | Purpose |
|---|---|---|
| REGISTER | endpoint → registrar | publish current device/service reachability |
| LOOKUP | endpoint → directory | resolve a name or number |
| INVITE | caller → service/peer | propose a session and requested capabilities/resources |
| ALERT | peer → caller | indicate that the target is being notified |
| ACCEPT | peer → caller | accept and return session parameters |
| REJECT | peer → caller | refuse with a reason |
| CANCEL | caller → peer | abandon an unanswered invitation |
| RELEASE | either endpoint | close an established session |
| RESERVE | endpoint → network service | request bounded-delay/bandwidth resources |
| RESERVE_RESULT | network service → endpoint | accept, modify, or reject a reservation |

All requests need an idempotent transaction or invitation identifier above GDP. Session operations bind to endpoint addresses and, when present, authenticated identities. Encoding waits on the GTS identifier and security decisions.
