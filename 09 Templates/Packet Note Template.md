---
id: "{{title}}"
title: "{{title}}"
aliases: []
type: packet
status: draft
layers: []
tags: ["gnet", "gnet/packet", "gnet/status/draft"]
parent: "[[Packet Formats MOC]]"
related: ["[[32-bit Flit Format]]", "[[Virtual Channels and VCIDs]]"]
updated: "{{date:YYYY-MM-DD}}"
template: true
---
# {{title}}

> [!info] Knowledge graph
> **Up:** [[Packet Formats MOC]] · **Related:** [[32-bit Flit Format]] · [[Virtual Channels and VCIDs]]

```text
    Flit data — exactly 32 bits
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         Data [31:0]                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Associated link metadata — baseline VC2, not part of flit data
   +---+
   |VC |
   +---+
    2 bits
```

A PHY-specific packet/link note may additionally show phits, framing, or inline metadata encoding. Do not treat the baseline VCID as part of the 32-bit data row.

## Fields

## Validation

## State transition

## Error behavior

## Test vectors
