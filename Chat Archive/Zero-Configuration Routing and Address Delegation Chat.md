# Zero-Configuration Routing and Address Delegation Chat

> [!important] Historical design discussion
> This archive records an exploratory GNet discussion. Statements here are not automatically normative. In particular, the GDP header printed near the end of the discussion is **superseded by later GNet decisions**: the link is flit-oriented with a 32-bit flit carrying a 4-bit VCID and 28 data bits, GDP itself has no integrity checksum, and current packet-size classes are defined elsewhere in the knowledge base. Use current specification/decision notes for implementation.

## Discussion summary

This discussion develops the GNet router self-configuration model away from conventional `HELLO` discovery and toward a state-driven `STATUS` / `DELEGATION` exchange.

Key ideas explored:

- Routers should require essentially no preconfigured identity or topology information.
- A child router periodically sends `STATUS` upstream and also sends it immediately after meaningful state changes.
- `STATUS` reports local interface capacity/state, address demand, current delegated ranges, protocol capabilities, and optional performance/load hints.
- The upstream/parent router replies with `DELEGATION`, assigning address range(s) and providing useful information about itself and the upstream path.
- Subsequent `STATUS` messages reflect the child's changed state after delegation, link changes, address pressure, or performance changes.
- Address delegation is recursive: a router that receives a range can subdivide/re-delegate capacity to downstream routers without NAT.
- The model is intended to be zero-configuration and locally negotiated rather than dependent on DHCP servers, manually configured subnet masks, or preassigned router IDs.
- The modern Internet mechanisms closest in function are DHCPv6 Prefix Delegation, Router Advertisements, LLDP/CDP, routing protocols such as IS-IS/OSPF/RPL, and BGP, but none combines GNet's recursive delegation and state reporting into one minimal mechanism.

## Archived discussion

Yes — you're clearly building a **clean, deterministic, zero-config negotiation model**, and "HELLO" (which implies neighbor discovery) doesn't quite fit anymore. **`STATUS`** is a much better term.

Let’s build this out in detail, piece by piece.

---

## GNet `STATUS` Message (Router → Upstream Parent)

**Purpose:**

Let a child router describe its **current capabilities, needs, and state** — for the parent to make an informed delegation decision.

---

## Fields in a `STATUS` Message

| Field | Description |
| --- | --- |
| `interfaces_total` (8-bit) | Number of downstream-capable ports |
| `interfaces_up` (8-bit) | Number of ports currently linked |
| `addresses_needed` (16-bit) | Number of device addresses required |
| `addresses_max` (16-bit) | Maximum it can reasonably manage |
| `address_current_block(s)` | (Optional) Current assigned ranges |
| `load_factor` (8-bit) | Current % usage of forwarding/memory/CPU |
| `bandwidth_hint` (enum) | e.g. `LOW`, `MEDIUM`, `HIGH`, `UNLIMITED` |
| `latency_class` (enum) | e.g. `FAST`, `MEDIUM`, `SLOW` |
| `performance_rating` (8-bit) | General rating from 0–255 (optional, future use) |
| `version` / `capabilities` | Protocol version, features supported |
| `status_flags` | e.g. `RESTARTED`, `LINK_DOWN`, `NEED_HELP`, etc. |

All fields are optional but encouraged. You could even version this message format cleanly over time.

---

## When is `STATUS` Sent?

| Trigger | Why |
| --- | --- |
| **On boot** | Always sends initial status request for delegation |
| **On link change** | Interfaces up/down triggers updated view |
| **On reconfiguration** | Address block change, delegation received |
| **On performance shift** | Load passes a threshold (e.g. +20%) |
| **Periodic broadcast** | Randomized interval (e.g. every 5–15s), avoids lock-step |
| **Manual override / debug** | Triggered by admin or watchdog logic |

This ensures upstream routers **always have a recent snapshot** of what’s going on below.

---

## Parent Router Responds with `DELEGATION`

### `DELEGATION` Message Includes

| Field | Meaning |
| --- | --- |
| `assigned_range(s)` | GNet address block(s) assigned to this router |
| `lifetime` | Optional time limit / lease |
| `upstream_address` | This router's own GNet address |
| `policy_flags` | e.g. may delegate further, may reassign, must ACK |
| `cost_class` | Optional routing cost (for upstream path learning) |
| `capabilities` | Optional: version, metrics, time sync info, etc. |

---

## From There On

- Child sends updated `STATUS` if anything changes.
- Parent updates route table accordingly.
- If child goes silent, parent expires delegation.
- Parent may reclaim and redistribute blocks.

Everything is local, deterministic, and intended to avoid central configuration.

---

## Optional Backoff Mechanism

To avoid chatterstorms:

- Random delay before sending new `STATUS`.
- Minimum interval unless the change is critical.
- Optional parent request for an immediate status refresh.

---

## Modern Internet Protocols Discussed as Comparisons

The discussion compared the model with several modern mechanisms:

| Protocol | Relevant function |
| --- | --- |
| DHCP / DHCPv6 | Host/network parameter assignment |
| DHCPv6-PD | Delegates address prefixes to downstream routers |
| IPv6 Router Advertisements | Announces local routing/prefix information |
| BGP | Inter-domain reachability and policy |
| LLDP / CDP | Link-neighbor capability discovery |
| Zeroconf / mDNS | Local device/service discovery |
| NDP | IPv6 neighbor/router discovery |
| IS-IS / OSPF | Interior dynamic routing |
| RPL | Dynamic routing for constrained/low-power networks |
| AutoIP | Local self-assigned addressing |

The closest conceptual combination identified was **recursive prefix delegation plus a lightweight routing/status protocol**, with DHCPv6-PD being the closest single modern address-delegation mechanism.

---

## Historical GDP Header Printed During the Discussion

The conversation then printed the following GDP concept. It is retained here only as historical context and **must not be treated as the current normative GDP format**.

```text
Byte Offset →
00         01         02         03         04         05         06         07
+----------+----------+----------+----------+----------+----------+----------+----------+
|Ver|Type | SizeCls  |  HopLimit |         Destination Address (64 bits)              |
+----------+----------+----------+----------+----------+----------+----------+----------+
08         09         10         11         12         13         14         15
+----------+----------+----------+----------+----------+----------+----------+----------+
|                     Destination Address (continued)                               |
+----------+----------+----------+----------+----------+----------+----------+----------+
16         17         18         19         20         21         22         23
+----------+----------+----------+----------+----------+----------+----------+----------+
|                        Source Address (64 bits)                                   |
+----------+----------+----------+----------+----------+----------+----------+----------+
24         25
+----------+----------+
| Source (cont.)      |
+---------------------+
```

At that point the discussion described hierarchical addressing using the human-facing names `Top`, `Org`, `Division`, and device, and emphasized decimal-friendly presentation rather than hex for ordinary administration.

## Current relevance

The lasting design contribution of this discussion is not the historical GDP diagram. It is the **self-configuring router control model**: routers report state upward, parents delegate address capacity downward, and the topology continually reflects real state without requiring per-router identities or manual subnet configuration.
