# Service discovery packets

Status: **DRAFT**

Discovery is generic: Router, Directory, Terminal Server, and later services use the same SOLICIT/ADVERTISE messages with different Service Type values.

## SOLICIT

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  GCTL Version | Message Type  |             Flags             | Flit 0
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Transaction ID                          | Flit 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Service Type          |     Scope     |  Reserved = 0 | Flit 2
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

GCTL Version is draft value 1. Message Type is SOLICIT. Flags are zero until allocated. Transaction ID is a random request value. Scope values are LINK=0, ROUTER_DOMAIN=1, DISTRICT=2, and METRO=3.

An unconfigured endpoint may send only `SOLICIT(Router, LINK)` using DLP link-local GCTL. A configured endpoint sends other solicitations as GDP/GCTL. Intermediaries must not expand the requested scope.

## ADVERTISE

```text
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  GCTL Version | Message Type  |             Flags             | Flit 0
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Transaction ID                          | Flit 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Service Type          |          Preference           | Flit 2
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                 Provider Address [63:32]                      | Flit 3
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                 Provider Address [31:0]                       | Flit 4
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Lifetime (seconds)                         | Flit 5
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Capabilities                            | Flit 6
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Message Type is ADVERTISE. Transaction ID and Service Type echo SOLICIT. Larger Preference values are preferred unless policy overrides them. Provider Address may be zero only for a router advertisement returned by physical-port identity before GDP configuration. Lifetime zero means do not cache. Capabilities is service-specific.

SOLICIT is three flits; ADVERTISE is seven flits. These layouts remain DRAFT.
