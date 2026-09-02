# GNET-P point-to-point trunk

Status: **ACCEPTED rate family; OPEN framing details**

GNET-P provides dedicated synchronous infrastructure links at 10, 25, and 50 Mb/s. Initial systems may use coaxial cable; later systems may use fiber without changing GDP.

Each direction is point-to-point. The link scheduler may expose service classes or reserved channels, but all user traffic remains DLP/GDP packet traffic. Clock recovery, line code, frame synchronization, keepalive, error monitoring, protection switching, and multiplexing are OPEN.

GNET-P is distinct from external GNET-L cabling and from the internal QDX bus.
