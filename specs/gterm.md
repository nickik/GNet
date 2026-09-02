# GTerm virtual terminal protocol

Status: **ACCEPTED behavior; OPEN encoding**

GTerm provides routable virtual terminal service. One underlying GTS tunnel may carry multiple simultaneous terminal sessions. Local operation is presented as another session context rather than a different user model.

The terminal has a dedicated **SESSION** key. Firmware or the trusted local environment always intercepts it; a remote host cannot capture or disable it. SESSION opens the local session selector and permits switching among local and remote contexts.

Each session needs:

- a session identifier within the GTerm connection;
- terminal-class and capability negotiation;
- input, output, control, resize/status, and close operations;
- explicit encoding and flow-control behavior;
- an authentication binding when required.

The earlier conceptual packet was connection ID, session ID, payload; the connection is now expected to map to a GTS tunnel/stream arrangement. Exact identifiers and framing remain OPEN.

GTerm discovery is directory-based and routable. The server sees GTerm capabilities rather than Apollo/Luna-specific hardware.
