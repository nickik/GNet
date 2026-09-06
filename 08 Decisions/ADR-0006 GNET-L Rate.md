---
id: adr-0006-gnet-l-rate
title: "ADR-0006 GNET-L Rate"
aliases: ["Decision 0006"]
type: decision
status: accepted
layers: ["L1"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l1"]
parent: "[[Decisions MOC]]"
related: ["[[GNET-L]]","[[Deployment Topology]]","[[GNet PHY Profiles]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
---
# Decision 0006: GNET-L target is 3 Mb/s

> [!info] Knowledge graph
> **Up:** [[Decisions MOC]] · **Related:** [[GNET-L]] · [[Deployment Topology]]

Status: **ACCEPTED; supersedes earlier planning value**

Earlier GNET-L planning used 2.5 Mb/s and an initial eight-endpoint profile. The newer external GNet definition sets the local-link target at approximately **3 Mbit/s of 32-bit flit data**.

The named data rate excludes hop-local VC metadata and PHY framing/line-code overhead. Under the baseline VC2 model, a simple inline representation carries 34 logical link bits per 32-bit flit; sustaining 3.0 Mbit/s of flit data therefore requires 3.1875 Mbit/s of link-bit capacity before any additional coding overhead.

Documents and implementations should use 3 Mbit/s as the current GNet-3 flit-data target while treating exact symbol rate, coding overhead, and supported Coupler sizes as unresolved electrical/profile questions.
