---
id: gnet-modular-connector
title: "GNet Modular Connector"
aliases: ["GMC-8","GNet connector","8P8C-style GNet connector"]
type: media
status: draft
layers: ["L1"]
tags: ["gnet","gnet/media","gnet/status/draft","gnet/connector"]
parent: "[[Media and Links MOC]]"
related: ["[[GNet Copper Cabling]]","[[GNet PHY Profiles]]"]
updated: 2026-09-03
---
# GNet Modular Connector

The formal Digital name is **GNet Modular Connector, 8-contact (GMC-8)**.

Names considered during definition were:

- **GMC-8** — GNet Modular Connector, 8-contact;
- **DMC-8** — Digital Modular Connector, 8-contact;
- **DGC-8** — Digital GNet Connector, 8-contact.

`GMC-8` is preferred because it identifies the network family while remaining descriptive and vendor-neutral enough for an interoperability standard.

## Mechanical/electrical envelope

GMC-8 has:

```text
8 contacts
8 conductors
4 balanced pairs
```

Its contact arrangement is in the general eight-position modular-telephone connector family. Modern explanatory material may call the shape **8P8C/RJ45-like**, but `RJ45` is not the normative GNet name and the connector is never described as "8-pair".

## Retention requirement

Digital's plug/jack specification SHOULD protect the flexible retaining latch better than the common fully exposed modular plug. The preferred mechanical direction combines:

- recessed latch travel;
- side shoulders/latch guards on the plug or boot;
- strain relief that transfers cable pull to the plug body rather than the latch;
- a boot geometry that prevents normal cable bending from loading the retaining tab.

Exact dimensions, contact plating, insertion-cycle rating, pair-to-pin assignment, shield termination, and keying remain **DRAFT — requires connector/tooling validation**.
