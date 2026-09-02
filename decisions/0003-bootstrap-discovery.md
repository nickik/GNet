# Decision 0003: Generic, bounded discovery

Status: **ACCEPTED**

Use generic SOLICIT/ADVERTISE messages with registered service types. The initial router solicitation is the only inherent link-local fanout. Its responses are direct. After address configuration, directory and other discovery use scoped unicast/routed messages.

The standard startup path is router, address, directory, then selected service. This keeps ROM clients simple without turning the internet into a broadcast discovery domain.
