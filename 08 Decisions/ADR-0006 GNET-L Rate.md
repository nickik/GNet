---
id: adr-0006-gnet-l-rate
title: "ADR-0006 GNET-L Rate"
aliases: ["Decision 0006"]
type: decision
status: accepted
layers: ["L1"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l1"]
parent: "[[Decisions MOC]]"
related: ["[[GNET-L]]","[[Deployment Topology]]"]
updated: 2026-09-02
---
# Decision 0006: GNET-L target is 3 Mb/s

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[GNET-L]] · [[Deployment Topology]]


Status: **ACCEPTED; supersedes earlier planning value**

Earlier GNET-L planning used 2.5 Mb/s and an initial eight-endpoint profile. The newer external GNet definition sets the local-link target at approximately 3 Mb/s. Documents and implementations should therefore use 3 Mb/s as the current target while treating exact symbol rate, coding overhead, and supported passive-hub sizes as unresolved electrical/profile questions.
