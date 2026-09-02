---
id: security-and-identity
title: "Security and Identity"
aliases: ["GNet security","DigitalKey"]
type: architecture
status: accepted
layers: ["L5","L6"]
tags: ["gnet","gnet/architecture","gnet/status/accepted","gnet/layer/l5","gnet/layer/l6"]
parent: "[[Architecture MOC]]"
related: ["[[GNet Service Model]]","[[GSC Protocol]]"]
updated: 2026-09-02
---
# Security and identity

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[GNet Service Model]] · [[GSC Protocol]]


Status: **ACCEPTED architecture; OPEN mechanisms**

Routers forward addresses and QoS; they do not authenticate users on every packet. Endpoint/session protocols provide integrity, confidentiality, replay protection, and peer authentication. The Digital Identity System or another directory-backed authority maps people, devices, roles, and service permissions.

A terminal or mobile device may contain a removable **DigitalKey**, a smart-card/SIM-like security token. It may carry the user's GNet identity, telephone/loading number, protected encryption credentials, billing association, and online-service credentials. During login it proves possession of a credential to an identity or terminal service. Secret key material must not be transmitted. A device address is not itself a user identity.

The Reset ID is also capability-like: it authorizes tunnel close/reset/rebind and is deliberately absent from normal data. Call-control and media encryption use separate keys so a session server may coordinate setup without decrypting endpoint media.

Bootstrap security has three distinct questions:

1. **Link admission:** may this physical device/port attach?
2. **Network configuration:** which prefix/address and router may it use?
3. **Application login:** which user may open which service?

These may share credentials, but the protocol must keep their authorization decisions separate. Cryptographic algorithms, key distribution, certificate/credential format, revocation, and viable early-1980s hardware profiles remain OPEN.
