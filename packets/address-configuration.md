# Address-configuration packets

Status: **DRAFT**

These link-local GCTL messages run after router discovery. Responses are sent directly using physical port/link identity.

## ADDRESS_OFFER

| Offset | Size | Field |
|---:|---:|---|
| 0 | 1 | GCTL version |
| 1 | 1 | Message type |
| 2 | 2 | Flags |
| 4 | 4 | Transaction ID |
| 8 | 8 | Router address |
| 16 | 8 | Offered prefix, host bits zero |
| 24 | 1 | Prefix length |
| 25 | 1 | Minimum random suffix bits, MUST be at least 8 for customer service |
| 26 | 2 | Reserved, zero |
| 28 | 4 | Lifetime seconds |

## ADDRESS_CLAIM

| Offset | Size | Field |
|---:|---:|---|
| 0 | 1 | GCTL version |
| 1 | 1 | Message type |
| 2 | 2 | Flags |
| 4 | 4 | Transaction ID |
| 8 | 8 | Candidate GDP address |
| 16 | 8 | Client nonce/token |

## ADDRESS_ACK / ADDRESS_NAK

| Offset | Size | Field |
|---:|---:|---|
| 0 | 1 | GCTL version |
| 1 | 1 | Message type |
| 2 | 2 | Result/reason |
| 4 | 4 | Transaction ID |
| 8 | 8 | Candidate GDP address |
| 16 | 4 | Confirmed lifetime seconds |
| 20 | 4 | Retry delay milliseconds; zero on ACK |

The exact collision table, lease persistence, multi-router coordination, renumbering, and authentication binding are OPEN.
