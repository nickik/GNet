# Discovery and bootstrap

Status: **ACCEPTED sequence; DRAFT packets**

## Scope rule

GNet does not broadcast discovery into the routed network. An unconfigured endpoint may issue one local SOLICIT that the attached hub or link presents to eligible local providers. Each ADVERTISE is returned directly to that endpoint/physical port. Further queries are unicast GDP packets.

## Generic discovery

SOLICIT and ADVERTISE carry a registered service type. Initial service types include Router, Directory, Terminal Server, Boot Server, Time, Identity, and Network Management. This avoids a new link mechanism for every future bootstrap service.

## Preferred boot sequence

1. Link activation and physical port identification.
2. `SOLICIT(Router)` over link-local GCTL.
3. Direct `ADVERTISE(Router)` from one or more routers.
4. Address offer, random suffix selection, claim, and confirmation.
5. Authentication/terminal registration when policy requires it.
6. `SOLICIT(Directory)` as a unicast or tightly scoped GDP control request.
7. Directory lookup for a named service such as terminal, boot, file, or RPC. Results may rank several local or remote providers by policy, load, and capability.
8. End-to-end session establishment with the selected service.

A ROM terminal may use a local terminal server directly only as a defined fallback when no directory is reachable. It must not search the whole internet for a terminal server.

## Directory hierarchy

District registrars maintain current device and local-service reachability. Metro registries identify the correct district. Ordinary name resolution is a normal routed packet service, not a permanent function of the physical control pair.

A directory service record should expose the logical service name and type, address candidates, terminal/application classes, authentication method, load/preference, location, access group, and lifetime. The endpoint asks for services visible to the current device or authenticated user.

## Failure behavior

Solicitations use a randomized retry interval. Multiple advertisements are ranked by service preference and reachability; the exact timer, preference algorithm, caching rules, and trust model remain OPEN.
