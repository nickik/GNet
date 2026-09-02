# GNet Control Protocol (GCTL)

Status: **DRAFT**

GCTL carries bootstrap, discovery, address configuration, and network diagnostic messages. Before address assignment it is carried directly by DLP as link-local GCTL. After assignment it is normally a GDP payload.

GCTL does not provide ordinary directory name resolution, transport reliability, or user login. A transaction identifier allows a requester to match direct responses to a solicitation. Each request must be safe to repeat because initial delivery is unreliable.

Message families are registered in [registry/control-messages.md](../registry/control-messages.md). Discovery and address configuration are defined separately in [packets/discovery.md](../packets/discovery.md) and [packets/address-configuration.md](../packets/address-configuration.md).
