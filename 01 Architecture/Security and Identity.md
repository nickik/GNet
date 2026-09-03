---
id: security-and-identity
title: "Security and Identity"
aliases: ["GNet security","DigitalKey"]
type: architecture
status: accepted
layers: ["L2","L5","L6"]
tags: ["gnet","gnet/architecture","gnet/status/accepted","gnet/layer/l2","gnet/layer/l5","gnet/layer/l6"]
parent: "[[Architecture MOC]]"
related: ["[[GNet Service Model]]","[[GSC Protocol]]","[[Transport and Flows]]","[[GNet-Dominant Networking Evolution]]"]
updated: 2026-09-03
---
# Security and identity

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[GNet Service Model]] · [[GSC Protocol]] · [[Transport and Flows]] · [[GNet-Dominant Networking Evolution]]

Status: **ACCEPTED architecture; OPEN mechanisms**

GNet separates three security scopes that may be deployed independently:

1. **link security** between directly connected peers;
2. **end-to-end tunnel/session security** between communicating endpoints;
3. **identity and authorization** for people, devices, roles, and services.

Routers forward addresses and QoS; they do not authenticate users on every GDP packet. Endpoint/session protocols provide end-to-end integrity, confidentiality, replay protection, and peer authentication. The Digital Identity System or another directory-backed authority maps people, devices, roles, and service permissions.

## Link security

DLP/GNET-LINK is point-to-point and may evolve optional negotiated security and operational capabilities without adding fields to GDP. Possible later profiles include:

- peer/link authentication;
- link payload encryption;
- cryptographic integrity and replay protection;
- negotiated link/security capabilities;
- link speed, capacity, duplex, and quality reporting;
- error, congestion, performance, and OAM status.

These are link-local properties and should normally be negotiated or reported through DLP/GNET-LINK control exchanges rather than repeated in every GDP packet.

Link encryption protects a single physical hop. Intermediate routers still receive the clear GDP packet and re-encrypt it for the next protected link. This makes link encryption suitable for early protection of trunks, leased circuits, radio links, financial installations, government/military networks, and other media vulnerable to tapping, but it is not a substitute for end-to-end security.

The term **GPP** was used for this basic point-to-point protocol in some project discussions. The current repository specification names the layer **DLP/GNET-LINK**; normative documents should use the current name unless an explicit naming decision changes it.

## End-to-end security

GTS tunnel/stream security protects communication across arbitrary intermediate GNet routers and operators. It may authenticate peers, negotiate keys, protect integrity and replay state, and encrypt stream/message payloads while leaving GDP routing information available to routers.

Link and session encryption may be used together:

```text
Application
    -> encrypted/authenticated GTS stream or session
        -> GTS
            -> GDP
                -> DLP/GNET-LINK with optional per-link protection
                    -> physical medium
```

The Reset ID is also capability-like: it authorizes tunnel close/reset/rebind and is deliberately absent from normal data. Call-control and media encryption use separate keys so a session server may coordinate setup without decrypting endpoint media.

## Identity

A terminal or mobile device may contain a removable **DigitalKey**, a smart-card/SIM-like security token. It may carry the user's GNet identity, telephone/loading number, protected encryption credentials, billing association, and online-service credentials. During login it proves possession of a credential to an identity or terminal service. Secret key material must not be transmitted. A device address is not itself a user identity.

Bootstrap security has three distinct authorization questions:

1. **Link admission:** may this physical device/port attach?
2. **Network configuration:** which prefix/address and router may it use?
3. **Application login:** which user may open which service?

These may share credentials, but the protocol must keep their authorization decisions separate. Cryptographic algorithms, key distribution, certificate/credential format, revocation, and viable early-1980s hardware profiles remain OPEN.
