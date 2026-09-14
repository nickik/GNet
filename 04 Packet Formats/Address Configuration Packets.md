---
id: address-configuration-packets
title: "Address Configuration Packets"
aliases: ["ADDRESS_OFFER","ADDRESS_CLAIM"]
type: packet
status: frozen
layers: ["L3"]
tags: ["gnet","gnet/packet","gnet/status/frozen","gnet/layer/l3"]
parent: "[[Packet Formats MOC]]"
related: ["[[GCTL Protocol]]","[[Discovery and Bootstrap]]","[[Addressing and Routing]]","[[ADR-0019 GCTL Credit and Bootstrap Wire Profile]]"]
updated: 2026-09-14
---
# Address-configuration packets

Status: **FROZEN for GNet 0.1**

Address configuration occurs after link establishment and router discovery. GCTL carries the offer/claim/result exchange on the normal GDP data path.

All multi-byte values use network byte order. The first 8 bytes are the normal GCMP common header.

## ADDRESS_OFFER (`0x10`)

Body:

```text
Router Address    8 bytes
Prefix            8 bytes
Prefix Length     1 byte
Reserved          3 bytes = 0
Candidate Address 8 bytes
Lifetime          4 bytes
```

The Transaction ID is carried in the GCMP common header.

For GNet 0.1, Prefix Length MUST be one of `/16`, `/32`, `/48`, or `/56`.

The Prefix MUST be normalized, with all suffix bits zero. Candidate Address MUST lie inside Prefix.

The remaining host bits are router-managed endpoint space; there is no endpoint subnet delegation or separate local-suffix policy in GNet 0.1.

## ADDRESS_CLAIM (`0x11`)

Body:

```text
Candidate Address 8 bytes
Claim Nonce       8 bytes
```

The Claim Nonce is client-selected and scoped to this bootstrap exchange. It is not a permanent machine identity.

The claimed address MUST match the address supplied in the corresponding ADDRESS_OFFER Transaction ID.

## ADDRESS_ACK (`0x12`)

Body:

```text
Confirmed Address 8 bytes
Lifetime          4 bytes
```

The confirmed address MUST match the address offered and claimed for the Transaction ID.

## ADDRESS_NAK (`0x13`)

Body:

```text
Candidate Address 8 bytes
Retry Delay       4 bytes
```

The common GCMP `Code` field carries the rejection reason.

## Padding

Remaining bytes in the selected GDP Size Class are zero padding and MUST be transmitted as zero. Receivers MUST reject nonzero padding for these frozen typed messages.

## Binding and policy

Physical port identity may bind a transaction to a directly attached client but is not a global MAC address.

The following remain policy/state-machine concerns rather than changes to this packet format:

- lease persistence;
- collision policy among multiple authorities;
- multi-router coordination;
- renumbering;
- authentication binding;
- authorization policy.

The earlier four-bit-VCID / 28-carried-bit direct-DLP diagrams are obsolete.
