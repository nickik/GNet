---
id: gnet-copper-cabling
title: "GNet Copper Cabling"
aliases: ["GNet cable grades","GNet twisted pair"]
type: media
status: draft
layers: ["L1"]
tags: ["gnet","gnet/media","gnet/status/draft","gnet/cabling"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet PHY Profiles]]","[[GNet Modular Connector]]"]
updated: 2026-09-03
---
# GNet copper cabling

GNet is deliberately designed to exploit existing balanced telephone wiring where practical while also giving Digital a certified installation cable for predictable performance.

The grade names below are GNet engineering/installability classes, not later EIA/TIA Category-3/5/6 terminology.

| Grade | Intended plant | Required GNet-3 behavior |
|---|---|---|
| 0 | highly degraded existing UTP: poor twist, splices/bridge taps, poor balance, high noise | 0.75 Mbit/s fallback where link qualification succeeds |
| 1 | degraded but usable balanced UTP | 1.5 Mbit/s fallback where qualification succeeds |
| 2 | normal usable balanced telephone/data UTP | 3.0 Mbit/s nominal under qualified installation limits |
| 3 | Digital-certified four-pair shielded balanced data cable | reliable 3 Mbit/s baseline and preferred path toward GNet-10 qualification |

A grade does **not** promise operation at an arbitrary distance. Reach depends on insertion loss, balance, crosstalk, splices, termination, noise, and the actual transceiver.

## Grade 3 construction direction

Historically plausible construction for a late-1970s Digital-certified cable is:

- four individually twisted balanced copper pairs;
- approximately 22–24 AWG solid copper conductors as an engineering target;
- overall foil/braid shielding with drain/ground treatment defined by the installation standard;
- controlled pair geometry and twist to limit crosstalk;
- connector termination designed around [[GNet Modular Connector|GMC-8]].

The exact characteristic impedance, capacitance, attenuation mask, near/far-end crosstalk limits, shield bonding, maximum channel length, and bend/installation limits are:

> **TBD — requires PHY validation**

Digital should qualify cable from multiple suppliers against an electrical acceptance fixture rather than make interoperability depend on one vendor's construction.
