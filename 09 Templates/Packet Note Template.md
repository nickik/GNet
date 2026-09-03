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
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | VCID  |                  Carried bits [27:0]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Fields

## Validation

## State transition

## Error behavior

## Test vectors
