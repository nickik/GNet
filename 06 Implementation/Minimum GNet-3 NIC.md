---
id: minimum-gnet-3-nic
title: "Minimum GNet-3 NIC"
aliases: ["Minimum GNet NIC","GNet-3 compatibility profile"]
type: implementation
status: accepted
tags: ["gnet","gnet/implementation","gnet/status/accepted","gnet/nic"]
parent: "[[Implementation MOC]]"
related: ["[[GNet PHY Profiles]]","[[GNet Link Control Protocol]]","[[Virtual Channels and VCIDs]]","[[ADR-0012 Minimum GNet-3 Compatibility Profile]]"]
updated: 2026-09-03
---
# Minimum GNet-3 NIC

Status: **ACCEPTED universal compatibility profile**

Every native GNet NIC, including future GNet-10, GNet-20, server, cluster, router, and high-performance adapters, MUST initially be able to operate as a Minimum GNet-3 NIC. Advanced capabilities are negotiated only after baseline establishment.

There is no incompatible standalone "Minimum GNet-10 NIC".

## Physical requirements

A Minimum GNet-3 NIC provides:

- one [[GNet Modular Connector|GMC-8]] four-pair attachment;
- CONTROL-UP and CONTROL-DOWN;
- DATA-UP and DATA-DOWN;
- 3.0 Mbit/s nominal data mode;
- mandatory 1.5 and 0.75 Mbit/s fallback data modes;
- 32-bit physical flits.

## Flit and VC requirements

Baseline wire format:

```text
2-bit VCID
30 carried bits
4 wire VCIDs
no SOF bit
```

The NIC MUST maintain at least **two concurrent active receive contexts**. This is the minimum needed to pause one NORMAL transfer at a grant boundary while receiving an eligible REALTIME transfer on another VC. Implementations MAY maintain contexts for all four VCIDs.

A receive context tracks at least the active VC, expected remaining transfer size, integrity state, and buffer/credit accounting.

## Flow-control requirements

The NIC tracks actual available receive capacity and advertises it through GLCP.

> **1 credit = one physical flit of guaranteed receive capacity.**

The NIC MAY batch credit returns. It MUST NOT advertise capacity that is not actually reserved/available to that flow, and it MUST NOT reuse an outstanding reserved credit until that credit is returned or recovery cancels the allocation.

## Priority

Minimum GNet-3 understands exactly:

- `NORMAL`
- `REALTIME`

REALTIME is restricted by the link profile to small latency-sensitive/control packages. It does not create a general eight-level QoS implementation requirement in the minimum NIC.

## Capability negotiation

After baseline link establishment an advanced NIC MAY advertise capabilities including:

- GNet-10;
- future GNet-20;
- larger receive buffers/credit windows;
- more simultaneous RX contexts;
- future VC4;
- in-band control;
- bonded lanes;
- implementation acceleration features.

Unknown capabilities MUST be safely ignored or rejected without breaking Minimum GNet-3 operation.
