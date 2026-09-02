# Security and identity

Status: **ACCEPTED architecture; OPEN mechanisms**

Routers forward addresses and QoS; they do not authenticate users on every packet. Endpoint/session protocols provide integrity, confidentiality, replay protection, and peer authentication. The Digital Identity System or another directory-backed authority maps people, devices, roles, and service permissions.

A terminal may contain a removable smart card or equivalent security token. During login it proves possession of a credential to an identity or terminal service. The token's secret must not be transmitted. A device address is not itself a user identity.

Bootstrap security has three distinct questions:

1. **Link admission:** may this physical device/port attach?
2. **Network configuration:** which prefix/address and router may it use?
3. **Application login:** which user may open which service?

These may share credentials, but the protocol must keep their authorization decisions separate. Cryptographic algorithms, key distribution, certificate/credential format, revocation, and viable early-1980s hardware profiles remain OPEN.
