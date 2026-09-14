---
id: adr-0019-gctl-credit-bootstrap-wire
title: "ADR-0019 GCTL Credit and Bootstrap Wire Profile"
aliases: ["Decision 0019","GCTL bootstrap wire profile","GCTL credit wire profile"]
type: decision
status: accepted
layers: ["L3"]
tags: ["gnet","gnet/decision","gnet/status/accepted","gnet/layer/l3","gnet/bootstrap","gnet/credit"]
parent: "[[Decisions MOC]]"
related: ["[[GCTL Protocol]]","[[Discovery Packets]]","[[Address Configuration Packets]]","[[Link-Local Addressing]]"]
updated: 2026-09-14
---
# Decision 0019: GCTL credit and bootstrap wire profile

Status: **ACCEPTED**

## Decision

Freeze the GNet 0.1 wire bodies for `CREDIT_REQUEST`, `CREDIT`, `SOLICIT`, `ADVERTISE`, `ADDRESS_OFFER`, `ADDRESS_CLAIM`, `ADDRESS_ACK`, and `ADDRESS_NAK`.

All messages retain the existing 8-byte GCMP common header:

```text
Byte  0        Version
Byte  1        Message Type
Byte  2        Code
Byte  3        Flags
Bytes 4..7     Transaction ID
Bytes 8..N     Message body
```

Multi-byte body values use network byte order. Unused bytes in the selected GDP Size Class MUST be transmitted as zero and MUST be ignored only when explicitly reserved; typed-message padding defined below MUST be zero and receivers MUST reject nonzero padding.

## CREDIT_REQUEST

```text
Requested Flits   4 bytes
```

`Requested Flits = 0` means: advertise whatever receive capacity is currently grantable.

## CREDIT

```text
Granted Flits     4 bytes
```

One credit represents guaranteed receive capacity for exactly one physical flit at the adjacent forwarding endpoint.

A receiver MUST NOT advertise more credit than its actual free receive capacity after subtracting credit already advertised but not yet consumed.

Credits are aggregate link-local receive capacity. They are not GTS receive credits and do not imply physical-medium/path permission.

## SOLICIT

```text
Service Type      2 bytes
Scope             1 byte
Reserved          1 byte = 0
```

Initial scope values:

```text
0   directly attached / link scope
```

Other values are reserved.

## ADVERTISE

```text
Service Type      2 bytes
Preference        1 byte
Reserved          1 byte = 0
Provider Address  8 bytes
Lifetime          4 bytes
Capabilities      4 bytes
```

`Service Type = 0x0001` identifies Router. `Lifetime = 0` means the advertisement MUST NOT be cached beyond the immediate exchange.

## ADDRESS_OFFER

```text
Router Address    8 bytes
Prefix            8 bytes
Prefix Length     1 byte
Reserved          3 bytes = 0
Candidate Address 8 bytes
Lifetime          4 bytes
```

For GNet 0.1, Prefix Length MUST be one of `/16`, `/32`, `/48`, or `/56`. Prefix MUST be normalized and Candidate Address MUST lie inside the offered prefix.

## ADDRESS_CLAIM

```text
Candidate Address 8 bytes
Claim Nonce       8 bytes
```

The claim nonce is client-selected and scoped to the bootstrap exchange. It is not a permanent node identity.

## ADDRESS_ACK

```text
Confirmed Address 8 bytes
Lifetime          4 bytes
```

The confirmed address MUST match the address offered and claimed for the Transaction ID.

## ADDRESS_NAK

```text
Candidate Address 8 bytes
Retry Delay       4 bytes
```

The common GCMP `Code` field carries the rejection reason.

## Bootstrap destination

Before a router/provider address is known, GNet 0.1 reserves the link-local value:

```text
FE80:0000:0000:0000
```

as the point-to-point/link-scoped bootstrap discovery destination.

Ordinary generated link-local endpoint addresses use `FE80/16` with a nonzero 48-bit suffix, so this value is never a normal endpoint address.

An initial `SOLICIT(Router)` may be sent to this reserved destination. Once `ADVERTISE` or `ADDRESS_OFFER` identifies the provider, subsequent bootstrap messages use actual endpoint/provider addresses.

This destination is link-scoped and MUST NOT be routed.

## Bootstrap sequence

The normal router/address-assignment exchange is:

```text
SOLICIT(Router)
    -> ADVERTISE(Router)
    -> ADDRESS_OFFER
    -> ADDRESS_CLAIM
    -> ADDRESS_ACK or ADDRESS_NAK
```

`ADVERTISE` and `ADDRESS_OFFER` may be sent back-to-back for the same Transaction ID.

## Receive-buffer relationship

Implementations MUST derive advertised link credit from real bounded receive capacity.

Conceptually:

```text
free = receive capacity - receive storage currently occupied

grantable = free - credit already advertised but not yet consumed
```

When a received packet is removed from link receive storage and passed upward, the freed flit capacity becomes grantable again and MAY be returned immediately or in a bounded batch with `CREDIT`.

A sender consumes one link credit for each physical data flit transmitted.

## Control progress

A native link profile MUST provide a bounded mechanism by which `CREDIT_REQUEST`, `CREDIT`, bootstrap, and mandatory control traffic can make progress even when ordinary advertised data credit is exhausted.

The exact physical realization may differ by link profile; the baseline software/direct-link profile may reserve a small control VC/window. This does not change the GCTL wire format and does not make the control traffic a separate physical-control-pair protocol.

## Rationale

The semantics and field set had stabilized, and leaving packing open forced implementations to carry private bootstrap profiles. Freezing these compact bodies now makes GCTL bootstrap and credit handling interoperable while keeping router policy, lease persistence, multi-router coordination, renumbering, and authentication available for later independent decisions.
