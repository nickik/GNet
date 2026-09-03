---
id: dlp-segment-size-classes
title: "DLP Segment Size Classes"
aliases: ["Old GNet segment classes","Old package size classes"]
type: protocol
status: superseded
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/superseded","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[GDP Datagram]]","[[ADR-0010 DLP Segment Size Classes]]"]
updated: 2026-09-03
---
# DLP segment size classes

Status: **SUPERSEDED**

An earlier GNet design put a two-bit size class and explicit payload length into DLP, with 64-, 256-, and 1,024-octet limits. That design is retained here only for historical/backlink continuity.

The current architecture keeps DLP minimal and uses the four-bit **GDP Size Class** as the normal native GNet package-size declaration. See [[GDP Datagram]].

GC/GS link-control requests carry the GDP Size Class so infrastructure can apply receiver-buffer and scheduling policy before data starts. No separate normative DLP Segment Class exists in the baseline profile.

See [[ADR-0010 DLP Segment Size Classes]] for the supersession record.
