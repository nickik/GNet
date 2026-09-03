---
id: virtual-channels-and-vcids
title: "Virtual Channels and VCIDs"
aliases: ["VCID","GNet virtual channels","Virtual Channel Identifier"]
type: protocol
status: mixed
layers: ["L2"]
tags: ["gnet","gnet/protocol","gnet/status/mixed","gnet/layer/l2"]
parent: "[[Media and Links MOC]]"
related: ["[[Direct Link Protocol]]","[[DLP Segment Size Classes]]","[[32-bit Flit Format]]","[[Transport and Flows]]","[[ADR-0008 VCID in Every Flit]]"]
updated: 2026-09-02
---
# Virtual channels and VCIDs

> [!info] Knowledge graph
> **Up:** [[Media and Links MOC]] · **Related:** [[Direct Link Protocol]] · [[DLP Segment Size Classes]] · [[32-bit Flit Format]] · [[Transport and Flows]] · [[ADR-0008 VCID in Every Flit]]

Status: **FROZEN 4-bit position and hop-local meaning; ACCEPTED bounded reuse; OPEN allocation state machine**

The Virtual Channel Identifier (VCID) is the four-bit field at the start of every 32-bit GNet flit. It identifies the active DLP segment to which the remaining 28 bits belong.

## What a VCID identifies

A VCID is:

- local to one link and one direction;
- temporary and reusable after a bounded segment completes;
- repeated in the header, payload, and trailer flits of that segment;
- used to interleave flits from several active segments without confusing their reassembly or integrity state;
- replaced at every router rather than carried end to end.

A VCID is not a GDP address, GDP field, permanent circuit, application session, GTS Tunnel ID, GTS Stream ID, or persistent reservation identifier.

## Identifier hierarchy

```text
Physical port or premise channel
    identifies who may transmit or receive on this medium

VCID
    identifies one active bounded DLP segment on that link direction

Link Flow ID or reservation binding
    may associate successive segments with persistent scheduling state

GDP address and protocol type
    provide globally routed packet delivery

GTS Tunnel ID and Stream ID
    provide end-to-end transport multiplexing
```

## Why GNet needs VCIDs

Without a VCID, one packet must occupy the link continuously until its final bit or the receiver cannot distinguish interleaved packets. VCIDs allow a scheduler to alternate voice, control, and bulk-data flits while maintaining independent segment assembly and integrity state.

Bounded segments prevent a long or stalled wormhole transfer from holding a VCID and downstream resources indefinitely. [[DLP Segment Size Classes]] fixes the maximum meaningful payloads at 64, 256, and 1,024 octets. A larger GDP packet may be carried as several DLP segments; a persistent Link Flow ID may associate those segments with the same reservation without making the ephemeral VCID permanent.

## Scope on different media

- **GNET-L:** the physical star leg identifies the endpoint; the VCID multiplexes segments in each direction on that leg.
- **GNET-A:** the polling grant and premise channel identify the transmitter; the VCID namespace is interpreted within that channel/direction context.
- **GNET-P:** each point-to-point trunk direction has its own VCID namespace.

## Required state machine

An implementation maintains separate state for every active VCID: header interpretation, expected meaningful length, received carried-bit count, padding validation, integrity state, scheduling class, and any Flow ID binding.

The first flit on an inactive VCID creates that state. The expected trailer completes or rejects the segment and returns the VCID to the available pool. Unexpected payload, duplicate headers, invalid lengths, missing trailers, CRC failures, and timeouts must discard the partial segment without allowing stale state to contaminate later reuse.

## Open encoding and control questions

- whether VCID 0 is reserved for link control or all 16 values are allocatable;
- no additional base size class; `11` remains reserved;
- whether allocation is implicit on first use or explicitly granted;
- how a persistent Link Flow ID is bound and its width;
- timeout, cancellation, retry, and error-reporting rules;
- fairness and priority between active VCIDs;
- whether interleaving is mandatory on every medium or an optional profile capability.
