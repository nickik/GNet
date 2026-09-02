# Service discovery packets

Status: **DRAFT**

Discovery is generic: Router, Directory, Terminal Server, and later services use the same SOLICIT/ADVERTISE messages with different Service Type values.

## SOLICIT

| Offset | Size | Field | Rules |
|---:|---:|---|---|
| 0 | 1 | GCTL version | Draft value 1. |
| 1 | 1 | Message type | SOLICIT. |
| 2 | 2 | Flags | Zero until allocated. |
| 4 | 4 | Transaction ID | Random value selected by requester. |
| 8 | 2 | Service Type | Registry value. |
| 10 | 1 | Scope | LINK=0, ROUTER_DOMAIN=1, DISTRICT=2, METRO=3. |
| 11 | 1 | Reserved | MUST be zero. |

An unconfigured endpoint may send only `SOLICIT(Router, LINK)` using DLP link-local GCTL. A configured endpoint sends other solicitations as GDP/GCTL. Intermediaries must not expand the requested scope.

## ADVERTISE

| Offset | Size | Field | Rules |
|---:|---:|---|---|
| 0 | 1 | GCTL version | Draft value 1. |
| 1 | 1 | Message type | ADVERTISE. |
| 2 | 2 | Flags | Zero until allocated. |
| 4 | 4 | Transaction ID | Echoes SOLICIT. |
| 8 | 2 | Service Type | Echoes requested service. |
| 10 | 2 | Preference | Larger value is preferred; policy may override. |
| 12 | 8 | Provider address | Zero only when router advertisement precedes address configuration and physical return is used. |
| 20 | 4 | Lifetime seconds | Zero means do not cache. |
| 24 | 4 | Capabilities | Service-specific bitmap. |

Total sizes: SOLICIT 12 octets; ADVERTISE 28 octets. These sizes and field widths remain DRAFT.
