# Terminal service architecture

Status: **ACCEPTED model; DRAFT protocol**

A GNet terminal is a network endpoint, not a permanently wired extension of one minicomputer. Its ROM or local OS discovers the network, obtains an address, finds a directory, and asks for a named terminal service.

GTerm provides routable virtual terminal sessions. A terminal may multiplex several remote sessions while treating its local operating environment as another context. The server sees a GTerm endpoint and capabilities; it does not depend on Apollo, Luna, or other terminal hardware details.

A session service supports the following application operations: REGISTER, LOOKUP, INVITE, ALERT, ACCEPT, REJECT, CANCEL, and RELEASE. Terminal negotiation must eventually cover character encoding, screen/cursor model, keyboard capabilities, echo/editing policy, flow control, and optional graphics/file transfer.

The identity token may be used to request authenticated login, but terminal discovery must work before user login. A local terminal-server fallback is permitted when the directory is unavailable; selection rules remain OPEN.
