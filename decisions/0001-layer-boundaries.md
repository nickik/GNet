# Decision 0001: Keep routers below sessions

Status: **FROZEN**

DLP provides link framing; GDP provides routed delivery; GTS and applications provide sessions, reliability, fragmentation, encryption, and identity. This boundary keeps router hardware small, fixed-format, and independent of application evolution.

QDX may accelerate link and GDP work but does not become a protocol layer. Accounting and user authentication stay outside the forwarding fast path.
