# Decision 0005: Separate tunnels, reset authority, and streams

Status: **ACCEPTED model; DRAFT widths and encoding**

GTS is tunnel-first. A tunnel has a Tunnel ID and a separate Reset ID that authorizes destructive control operations. The Reset ID must be presented again for close, reset, or rebind, but ordinary data omits it; possession therefore depends on observing or participating in the handshake.

A tunnel may contain multiple streams. Each stream negotiates its own reliable/unreliable, ordered/sequenced, message/byte, encryption, and compression profile. Service selection occurs during tunnel or stream setup and is not repeated in normal data packets.

The working proposal uses a 64-bit Reset ID, 16-bit Stream ID, stream 0 as control, and stream 1 as the default data stream. Those widths and reserved values are not frozen.
