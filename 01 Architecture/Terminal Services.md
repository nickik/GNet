---
id: terminal-services
title: "Terminal Services"
aliases: ["GNet terminal architecture"]
type: architecture
status: accepted
layers: ["L7"]
tags: ["gnet","gnet/architecture","gnet/status/accepted","gnet/layer/l7"]
parent: "[[Architecture MOC]]"
related: ["[[GTerm Protocol]]","[[Discovery and Bootstrap]]"]
updated: 2026-09-02
---
# Terminal service architecture

> [!info] Knowledge graph
> **Up:** [[Architecture MOC]] · **Related:** [[GTerm Protocol]] · [[Discovery and Bootstrap]]


Status: **ACCEPTED model; DRAFT protocol**

A GNet terminal is a network endpoint, not a permanently wired extension of one minicomputer. Its ROM or local OS discovers the network, obtains an address, finds a directory, and asks for a named terminal service.

GTerm provides routable virtual terminal sessions. A terminal may multiplex several remote sessions while treating its local operating environment as another context. The server sees a GTerm endpoint and capabilities; it does not depend on Apollo, Luna, or other terminal hardware details.

A dedicated SESSION key is always handled by trusted local firmware/software and cannot be captured by a remote host. It opens the selector for local and remote session contexts.

A session service supports the following application operations: REGISTER, LOOKUP, INVITE, ALERT, ACCEPT, REJECT, CANCEL, and RELEASE. Terminal negotiation must eventually cover character encoding, screen/cursor model, keyboard capabilities, echo/editing policy, flow control, and optional graphics/file transfer.

The identity token may be used to request authenticated login, but terminal discovery must work before user login. A local terminal-server fallback is permitted when the directory is unavailable; selection rules remain OPEN.

Boot and terminal service are distinct. Supported boot sources are system ROM/local OS, a cartridge or cassette image, and GNet network boot. The boot chooser and fallback order remain a product-profile question.
