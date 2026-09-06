---
id: minimum-gnet-3-nic
title: "Minimum GNet-3 NIC"
aliases: ["Minimum GNet NIC","GNet-3 compatibility profile"]
type: implementation
status: accepted
tags: ["gnet","gnet/implementation","gnet/status/accepted","gnet/nic"]
parent: "[[Implementation MOC]]"
related: ["[[GNet PHY Profiles]]","[[GNet Link Control Protocol]]","[[Virtual Channels and VCIDs]]","[[ADR-0012 Minimum GNet-3 Compatibility Profile]]","[[ADR-0017 32-bit Data Flit and PHY Phits]]"]
updated: 2026-09-06
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
- 3.0 Mbit/s nominal **flit-data** mode;
- mandatory 1.5 and 0.75 Mbit/s flit-data fallback modes;
- 32-bit GNet data flits;
- baseline VC2 metadata associated with every flit;
- the GNet-3 PHY-defined mapping of flits and VC metadata onto physical phits.

## Flit and VC requirements

Baseline link semantics:

```text
32 data bits per flit
2-bit associated VCID
4 wire VCIDs
no SOF bit
```

The VCID is not subtracted from the 32-bit flit data field.

The NIC MUST maintain at least **two concurrent active receive contexts**. This is the minimum needed to pause one NORMAL transfer at a grant boundary while receiving an eligible REALTIME transfer on another VC. Implementations MAY maintain contexts for all four VCIDs.

A receive context tracks at least the active VC, expected remaining transfer size, integrity state, and buffer/credit accounting.

## Flow-control requirements

The NIC tracks actual available receive capacity and advertises it through GLCP.

> **1 credit = guaranteed receive capacity for one complete 32-bit data flit and its associated link metadata.**

Credit accounting is independent of how many physical phits the PHY uses to carry that flit. The NIC MAY batch credit returns. It MUST NOT advertise capacity that is not actually reserved/available to that flow, and it MUST NOT reuse an outstanding reserved credit until that credit is returned or recovery cancels the allocation.

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
- future wider VC identifiers such as VC4 or VC8;
- in-band control;
- bonded lanes;
- implementation acceleration features.

Future wider VC modes MUST retain 32-bit flit data and are not part of Minimum GNet-3. Unknown capabilities MUST be safely ignored or rejected without breaking Minimum GNet-3 operation.
