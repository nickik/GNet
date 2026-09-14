---
id: canonical-service-selector
title: "Canonical Service Selector"
aliases: ["CSS","Service Selector","Canonical Service Selector (CSS)"]
type: protocol
status: frozen
layers: ["L5","L7"]
tags: ["gnet","gnet/protocol","gnet/status/frozen","gnet/service"]
parent: "[[Protocols MOC]]"
related: ["[[GTS Protocol]]","[[GTS Transport Packets]]","[[GNet Service Model]]","[[CSS Registered Service Registry]]"]
updated: 2026-09-14
---
# Canonical Service Selector (CSS)

Status: **FROZEN canonical namespace, wire representations, and presentation syntax**

The Canonical Service Selector (CSS) identifies the logical service selected when a GTS tunnel is established.

There is exactly **one 128-bit CSS namespace**. GTS may transmit a CSS using an 8-bit registered representation, a 32-bit short representation, or the full 128-bit representation. These are not separate namespaces: all three forms expand to one canonical 128-bit value before service lookup.

```text
Registered-8
    |
    | registry expansion
    v
Short-32
    |
    | append 96 zero bits
    v
Canonical CSS128
```

A service therefore has one identity even when a shorter wire representation is available.

## 1. Canonical value

The canonical form is an unsigned 128-bit value transmitted and displayed most-significant byte first.

The all-zero CSS value is reserved and MUST NOT identify a service.

Implementations SHOULD normalize a received selector to CSS128 before performing service lookup, authorization, logging, comparison, or directory matching.

## 2. Representation field

GTS CONNECT carries a one-byte `CSS Representation/Reserved` field. The high two bits select the representation and the low six bits are reserved and transmitted as zero.

```text
bits 7..6   CSS Representation
bits 5..0   Reserved = 0
```

| Bits | Name | Wire selector field | Meaning |
|---:|---|---:|---|
| `00` | Registered-8 | 8 bits | registered compressed representation |
| `01` | Short-32 | 32 bits | four-character short representation |
| `10` | Full-128 | 128 bits | complete canonical CSS value |
| `11` | Reserved | — | reserved |

The selector field immediately follows the representation byte.

## 3. Short-32 representation

Short-32 is exactly four ASCII characters encoded as four bytes, most-significant byte first.

The initial Short-32 alphabet is:

```text
A-Z
0-9
```

All four characters are significant. There is no NUL termination, case folding, whitespace padding, or variable-length form.

Example:

```text
#CAPI

ASCII bytes       43 41 50 49
Short-32          0x43415049
CSS128            0x43415049000000000000000000000000
```

Short-32 maps into the **most-significant 32 bits** of CSS128. The remaining low 96 bits are zero.

Formally:

```text
CSS128 = Short32 << 96
```

Another example:

```text
#MYEP
Short-32  = 0x4D594550
CSS128    = 0x4D594550000000000000000000000000
```

A Short-32 value that is assigned a Registered-8 representation MUST NOT be transmitted using Short-32. The registered representation is canonical on the wire.

## 4. Registered-8 representation

Registered-8 is an 8-bit code from [[CSS Registered Service Registry]]. Each registered code maps to exactly one four-character Short-32 mnemonic and therefore to exactly one CSS128 value.

The 8-bit number is a compressed representation of that service identity; it is not a second service namespace.

For example, with the initial registry assignments:

```text
0x01 -> FILE
0x02 -> GRPC
```

`FILE` expands as:

```text
Registered-8      0x01
Short-32          0x46494C45        ASCII "FILE"
CSS128            0x46494C45000000000000000000000000
Presentation      -FILE
```

`GRPC` expands as:

```text
Registered-8      0x02
Short-32          0x47525043        ASCII "GRPC"
CSS128            0x47525043000000000000000000000000
Presentation      -GRPC
```

The textual `-FILE` form means "the registered service named FILE" and requests its Registered-8 representation. `FILE` itself is the service mnemonic; the hyphen is presentation syntax, not part of the canonical CSS value.

Because `FILE` has a Registered-8 assignment, `#FILE` is not a valid canonical selector spelling and Short-32 `FILE` is not a valid canonical wire encoding.

## 5. Full-128 representation

Full-128 carries the complete 128-bit CSS value directly as 16 bytes, most-significant byte first.

It is intended for service identities that cannot be represented canonically by Registered-8 or Short-32.

Example:

```text
0123456789ABCDEF0123456789ABCDEF
```

is the CSS128 value:

```text
0x0123456789ABCDEF0123456789ABCDEF
```

A Full-128 value whose lower 96 bits are zero and whose upper 32 bits form a valid Short-32 value is non-canonical and MUST NOT be transmitted as Full-128. It must use Short-32, or Registered-8 if that four-character value is registered.

## 6. Shortest canonical wire representation

For every CSS value, the shortest available representation is the canonical GTS wire encoding:

```text
registered mnemonic exists?
    yes -> Registered-8
    no  -> upper 32 bits valid Short-32 and lower 96 bits zero?
              yes -> Short-32
              no  -> Full-128
```

A receiver MUST expand all accepted forms to CSS128 before comparing service identity.

A sender MUST use the shortest canonical representation. A receiver SHOULD reject a non-canonical longer representation as malformed rather than allowing multiple wire encodings for one service identity.

This rule guarantees that one service has one canonical wire representation.

## 7. Presentation syntax

The canonical human-readable selector forms are:

```text
-FILE                                   registered service
-GRPC                                   registered service
#CAPI                                   Short-32 service
#MYEP                                   Short-32 service
0123456789ABCDEF0123456789ABCDEF        Full-128 service
```

Full-128 presentation is exactly 32 hexadecimal digits, conventionally uppercase, with no `0x` prefix.

If a CSS128 value has a shorter canonical representation, software SHOULD display that shorter form rather than the full hexadecimal value.

## 8. Address plus service syntax

A GDP endpoint and service may be written as:

```text
<GDP-address>:-FILE
<GDP-address>:-GRPC
<GDP-address>:#CAPI
<GDP-address>:#MYEP
<GDP-address>:0123456789ABCDEF0123456789ABCDEF
```

The GDP address identifies **where** to connect. CSS identifies **which service at that endpoint** is requested.

This notation is presentation syntax. GDP packets carry only GDP addresses; CSS is carried by GTS CONNECT during tunnel establishment.

## 9. GTS binding rule

A GTS CONNECT carries exactly one CSS and selects exactly one service.

If CONNECT is accepted:

- the resulting tunnel is bound to that canonical CSS;
- Stream 0 belongs to that selected service;
- later STREAM_OPEN operations create additional streams inside the same service-bound tunnel;
- STREAM_OPEN does not carry a CSS and cannot change the tunnel's selected service;
- selecting a different service requires a separate GTS tunnel.

Ordinary DATA packets therefore do not repeat CSS. They carry the receiver-local Tunnel ID and tunnel-scoped Stream ID.

## 10. Directory use

A directory or discovery system may return:

```text
GDP address + CSS
```

or a higher-level service name that resolves to one or more such endpoint/service pairs.

Directory naming and CSS are intentionally separate. A long human-readable directory name may resolve to a compact registered CSS such as `-FILE`, a Short-32 value such as `#CAPI`, or an opaque Full-128 value.

## 11. Security and enumeration

Registered-8 and Short-32 values are intentionally easy to enumerate and are suitable for public/common services.

A sparse Full-128 CSS can make blind enumeration impractical if the value is unpredictable, but CSS is not an authentication or encryption mechanism. Anyone who observes or learns a CSS may attempt to use it. Access control belongs to the selected service or a higher security layer.
