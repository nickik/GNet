# Decision 0004: Direct links have no network identity

Status: **FROZEN**

DLP assumes one directly attached endpoint per logical line. It provides byte/frame delimiting and CRC-8 but no source/destination address and no session state. Physical port or channel identity determines local delivery.

An earlier name, GPP (GNet Point-to-Point), described the same design direction. The accepted layer names are DLP at L2, GDP at L3, and GNet transport/session protocols above GDP. GNet may also be carried over other link technologies through an appropriate DLP/encapsulation profile.
