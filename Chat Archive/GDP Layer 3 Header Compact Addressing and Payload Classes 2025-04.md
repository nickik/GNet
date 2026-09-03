# GDP Layer 3 Header, Compact Addressing, and Payload Classes — April 2025

> [!important] Discussion archive
> This file archives the supplied GDP design discussion and summarizes the decisions reached in that discussion. No other GNet specification or design document is changed by this commit.

## Discussion summary

The discussion refines GDP (Global Data Protocol), GNet's Layer 3 protocol, toward a compact format suitable for both small serial/LAN networks and globally routed WANs.

Key decisions and design points:

- GDP provides Layer 3 routing, addressing, traffic classification, hop limiting, and a reserved flow-control identifier.
- The original header checksum is removed. Integrity is not performed by the GDP header itself.
- The Payload Size Class field is expanded to 4 bits, allowing 16 standardized packet payload classes.
- QoS terminology is replaced by an 8-bit `Traffic Class` field.
- The packed fixed portion of the header is 6 bytes:
  - Version: 4 bits
  - Type: 4 bits
  - Payload Size Class: 4 bits
  - Source Address Mode: 1 bit
  - Destination Address Mode: 1 bit
  - Reserved: 2 bits
  - Traffic Class: 8 bits
  - Hop Limit: 8 bits
  - Flow Control ID: 16 bits
- Source and destination addresses independently support two forms:
  - short: 8-bit local identifier
  - full: 64-bit global GNet address
- Header size therefore ranges from 8 bytes (short source + short destination) to 22 bytes (full source + full destination). Mixed forms are also possible.
- The 64-bit GNet address is hierarchically structured as eight 8-bit routing levels. Routers can route on these byte-sized levels rather than treating the address as a flat identifier.
- Short addresses exploit topology knowledge. On a local network, the router or bridge already knows the prefix represented by the local segment and can reconstruct/attach the full source or destination address when traffic leaves that segment.
- Short addressing is useful in both directions. If source and destination are in the same local 8-bit group, both may be transmitted in short form.
- ArcNet is a strong fit: GDP software can use ArcNet's 8-bit local node identity, while an ArcNet/GNet bridge supplies the appropriate higher-level GNet prefix when forwarding beyond the local network.
- DNP is described in this discussion as a **Point-to-Point based Network connection**. A DNP-attached client receives an 8-bit identifier when connected, allowing very small GDP headers on serial switched networks.
- Flow Control ID and future authentication/reserved functionality may simply be ignored by early routers. Their presence does not require the first-generation hardware to implement the later optimization mechanisms.
- GDP performs no Layer 3 fragmentation. Payload size is represented by a standard 4-bit size-class code.
- Payload size classes selected in this discussion:

| Class | Payload |
| ---: | ---: |
| 0 | 0 bytes |
| 1 | 64 bytes |
| 2 | 256 bytes |
| 3 | 512 bytes |
| 4 | 1500 bytes |
| 5 | 4096 bytes |
| 6 | 8192 bytes |
| 7 | 16384 bytes |
| 8-15 | Reserved |

- The 1500-byte class is mandatory for every GDP router. Classes above 1500 bytes are best-effort capabilities and may be unavailable on a particular link or router.
- The 512-byte class is retained for slower early serial links, where transmission latency, retransmission cost, and limited buffering can make a 1500-byte packet undesirable.
- 8192 rather than a later conventional 9000-byte jumbo size is preferred because GNet is being designed without that legacy constraint and can use clean binary-oriented sizes. 16384 provides another defined growth step.
- The design is intended to be highly suitable for almost-pure-hardware routing: no fragmentation, compact/packed fields, byte-oriented hierarchical routing, simple hop-limit processing, and the ability for early hardware to ignore future optimization fields.
- GDP is intended as an open standard and as a common Layer 3 protocol over many underlying LAN and WAN technologies rather than being tied to one physical or data-link technology.

## Archived discussion

Apr 7, 2025 at 6:59 PM

This is about the Layer 3 protocolLayer 3: GDP (Global Data Protocol)  Purpose: Routing, addressing, QoS, and flow control      Header: 18 bytes total          Version (2b), Type (4b), Payload Size Class (2b)          Hop Limit (8b), QoS (8b), Header Checksum (8b)          Flow Control ID (16b)          Source & Destination Address (64b each)      Supports fixed payload sizes: 64, 128, 256, 512 bytes      No fragmentation      Header Checksum: 8-bit additive over header only  Print full asci package   how hard would an almost pure hardware router implemntation for this be? 

Reasoned about hardware router implementation difficulty for 47 seconds

Below
&#x20;is an example of a “full ASCII” (textual/hex) representation of a&#x20;
notional GDP packet header, followed by a brief discussion on how&#x20;
challenging a nearly all-hardware router implementation might be.

---

## 1) Example “Full ASCII”/Hex Representation of an 18-Byte GDP Header

The fields, as given, are:

- **Version** (2 bits)
- **Type** (4 bits)
- **Payload Size Class** (2 bits)
- **Hop Limit** (8 bits)
- **QoS** (8 bits)
- **Header Checksum** (8 bits)
- **Flow Control ID** (16 bits)
- **Source Address** (64 bits)
- **Destination Address** (64 bits)

> **Important Note:**
> &#x20;Strictly adding these bit-sizes up suggests more than 18 bytes if you&#x20;
> store each field in standard byte boundaries (especially the 64-bit&#x20;
> addresses alone use 16 bytes). In many real-world protocols, small&#x20;
> multi-bit fields are packed together, and there can be alignment or&#x20;
> other constraints that push the header size above 18 bytes.
>
> For demonstration, we’ll assume&#x20;
> you’ve packed the first three fields (Version, Type, Payload Size Class)
> &#x20;into a single byte, and that the rest can be arranged to stay within&#x20;
> your stated total. Below is an example layout.

### Example Packed Layout (Hypothetical)

| Byte OffsetBits UsedFieldExample ValueNotes |                                                                    |                                      |                           |                                    |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------ | ------------------------- | ---------------------------------- |
| **0**                                       | 2 bits for Version, 4 bits for Type, 2 bits for Payload Size Class | V=01, T=0010, PSC=10 (binary) → 0x4A | `0x4A`                    | Combined 8 bits total (2+4+2)      |
| **1**                                       | 8 bits                                                             | Hop Limit = 0xFF                     | `0xFF`                    | Example: 255                       |
| **2**                                       | 8 bits                                                             | QoS = 0x08                           | `0x08`                    | Example QoS                        |
| **3**                                       | 8 bits                                                             | Header Checksum = 0xD2               | `0xD2`                    | 8-bit additive over header         |
| **4–5**                                     | 16 bits                                                            | Flow Control ID = 0x1234             | `0x12 0x34`               | Two bytes                          |
| **6–9**                                     | 32 bits (partial Source)                                           | Source Address (lower bits)          | `0xAB CD EF 99` (example) | Part of 64 bits for Source Address |
| **10–13**                                   | 32 bits (rest of Source)                                           | Source Address (upper bits)          | `0x01 23 45 67`           | 64 bits total for Source           |
| **14–17**                                   | 32 bits (partial Destination)                                      | Destination Address (lower bits)     | `0x9A BC DE F0`           | Part of 64 bits for Destination    |
| *(… etc.)*                                  | *(the rest must fit…)*                                             | Destination Address (upper bits)     | —                         | 64 bits total for Destination      |

If you literally tried to stay at&#x20;
“18 bytes total,” you can see there’s a mismatch: 64 bits + 64 bits for&#x20;
addresses alone is already 16 bytes. Once you add even a few more&#x20;
fields, you exceed 18 bytes. In practice, you might need at least 22–24&#x20;
bytes. But for the sake of illustrating a textual/hex packet, let’s show
&#x20;a single-line hex dump of the 18 (or so) bytes implied above:

```
```

```
Byte Index:  0   1   2   3   4   5   6   7   8   9   10  11  12  13  14  15  16  17 ...
Hex Value:  4A  FF  08  D2  12  34  AB  CD  EF  99  01  23  45  67  9A  BC  DE  F0 ...
```

(Where `4A` is your packed Version/Type/Payload Size Class, `FF`
&#x20;is Hop Limit, etc. This example necessarily glosses over some exact&#x20;
bit-packing and total-length details but illustrates how you might show a
&#x20;textual/hex “full ASCII” dump.)

---

## 2) Difficulty of an Almost Pure Hardware Router Implementation

### Simplicity Factors

1. **No Fragmentation**

   Having fixed payload sizes (64, 128, 256, 512) and no fragmentation is a
   &#x20;big simplification. You don’t need reassembly buffers or complicated&#x20;
   fragment-handling logic.
2. **Small, 8-bit Header Checksum**

   You only have to compute an 8-bit additive checksum over the header.&#x20;
   This can be done quickly in hardware as each packet arrives.
3. **Straightforward Header Fields**

   The fields (hop limit, QoS, flow-control ID, etc.) are individually easy
   &#x20;to parse in hardware, so a pipeline-based design can quickly parse and&#x20;
   then route.

### Potential Challenges

1. **64-bit Addressing**

   Each Source and Destination is 64 bits. That’s a large address space, so
   &#x20;any routing table in hardware (e.g., a CAM or specialized lookup&#x20;
   structure) could become sizable if you plan for direct exact-match&#x20;
   routing on 64-bit addresses. Many real protocols use hierarchical or&#x20;
   subnet-based addressing (like IPv4/IPv6) to keep the hardware table size
   &#x20;more manageable.
   - If GDP uses hierarchical routing or
     &#x20;prefix matching, you’ll need hardware that can handle variable-length&#x20;
     prefixes or big lumps of address space. This is doable (similar to an&#x20;
     IPv6 router in concept), but not trivial.
2. **Flow Control ID**

   The presence of a 16-bit Flow Control ID might imply some form of&#x20;
   per-flow logic or shaping. Pure hardware-based flow control typically&#x20;
   needs additional buffering and logic to handle congested flows or to&#x20;
   apply QoS rules. The hardware complexity depends on how sophisticated&#x20;
   the flow control needs to be.
3. **Pipeline Depth and Speed**

   If your target data rates are high (e.g., multi-gigabit or beyond), your
   &#x20;pipeline has to parse the header in a single pass, perform address&#x20;
   lookup, apply QoS decisions, etc., all within a few nanoseconds per&#x20;
   packet. That is doable with modern FPGAs or ASICs (similar to high-end&#x20;
   Ethernet switching silicon), but it’s not a beginner-level design task.
4. **Integration with Larger Network**

   Real deployments might require bridging with other protocols or&#x20;
   additional overhead. If GDP is the only protocol on an enclosed network,
   &#x20;hardware routing is simpler. If you need encapsulation or translation&#x20;
   to/from IPv4/IPv6, that adds extra logic.

### Overall Feasibility

- Implementing a basic GDP router pipeline in pure or almost pure hardware is **absolutely feasible**
  &#x20;if the environment is relatively controlled (e.g., an internal network&#x20;
  or a specialized application). The simplicity of fixed payload sizes and
  &#x20;an 8-bit header checksum helps.
- The main hardware complexity&#x20;
  typically arises from building and maintaining a fast, efficient 64-bit&#x20;
  address lookup table and handling any advanced QoS/flow-control&#x20;
  features. For a minimal router that just decrements hop limit and&#x20;
  switches packets by destination, it’s no harder than implementing an&#x20;
  IPv6-like pipeline in an FPGA/ASIC, just smaller scale.

In
&#x20;summary, a pure-hardware (or mostly hardware) GDP router is quite&#x20;
doable—its design would be on par with building other custom protocol&#x20;
routers in FPGA/ASIC form. The biggest considerations will be the large&#x20;
address fields (64 bits) and any advanced QoS or flow-control logic you&#x20;
plan to enforce.

Apr 9, 2025 at 5:25 PM

Just print normal text into the chat window

Reasoned about output formatting for 34 seconds

Below
&#x20;is the complete, plain-text specification for GDP. You can copy and&#x20;
paste this text into your preferred text editor and save it as a file&#x20;
(for example, “gdp-specification.txt” or “gdp-specification.md”).

---

## GDP (Global Data Protocol) – Full Technical Specification

1. Full Technical Specification

1.1 Introduction

GDP is a layer‑3 protocol within the&#x20;
GNet project. It uses flexible addressing (either 8‑bit or 64‑bit) and&#x20;
supports multiple fixed payload sizes. GDP is designed to work&#x20;
efficiently on small local networks (such as ArcNet or DNP&#x20;
point‑to‑point links) and also scale to larger hierarchical setups.

1.2 GNet Hierarchical Addressing

From earlier GNet documentation, each node is assigned a 64‑bit global address, subdivided into 8 levels of 8 bits each:

Level 0: (8 bits)

Level 1: (8 bits)

Level 2: (8 bits)

Level 3: (8 bits)

Level 4: (8 bits)

Level 5: (8 bits)

Level 6: (8 bits)

Level 7: (8 bits)

Routers forward packets by&#x20;
comparing the relevant 8‑bit levels to decide the next hop. When only a&#x20;
short distance or a single link is involved (for example, on ArcNet or a
&#x20;small serial ring), nodes can use a short 8‑bit address. In this case,&#x20;
the router or bridge attaches the higher‑level prefix when the packet&#x20;
leaves the local segment.

1.3 GDP Header Format

GDP’s header size is variable—from 8
&#x20;bytes up to 22 bytes—depending on whether the source and destination&#x20;
addresses are in short (8‑bit) mode or full (64‑bit) mode.

The header is structured as follows:

---

## Bytes 0–3 (Fixed Header):  Byte 0:  [Version (4 bits) | Type (4 bits)]  Byte 1:  [Payload Size Class (4 bits) | Src/Dst Mode (2 bits) | Reserved (2 bits)]  Byte 2:  [Service Class (8 bits)]  Byte 3:  [Hop Limit (8 bits)]  Bytes 4–5:  Flow Control ID (16 bits) – reserved for future flow-control features.  Variable fields:  Source Address: 1 byte if in short mode, or 8 bytes if in full mode.  Destination Address: 1 byte if in short mode, or 8 bytes if in full mode.

Field Summary:

– Version (4 bits): GDP protocol version.

– Type (4 bits): Packet type (for example, data, control).

– Payload Size Class (4 bits): Indicates the payload size category.

– Src/Dst Mode (2 bits): Two flags; one indicates the source address&#x20;
mode (0 = short 8-bit, 1 = full 64-bit) and the other indicates the&#x20;
destination address mode (0 = short 8-bit, 1 = full 64-bit).

– Reserved (2 bits): Reserved for future use.

– Service Class (8 bits): Used for priority or traffic class marking.

– Hop Limit (8 bits): Decremented by each router; if it reaches 0, the packet is dropped.

– Flow Control ID (16 bits): Reserved for potential future enhancements.

– Source Address: 1 byte (if short) or 8 bytes (if full).

– Destination Address: 1 byte (if short) or 8 bytes (if full).

Minimum header size = 6 bytes (fixed portion) + 1 byte (source) + 1 byte (destination) = 8 bytes.

Maximum header size = 6 bytes + 8 bytes (source) + 8 bytes (destination) = 22 bytes.

1.4 Payload Size Classes

GDP does not support fragmentation.
&#x20;Each packet’s payload must fit exactly in one of the predefined size&#x20;
classes. (All routers must support the 1500-byte class.)

## Payload Size Classes:

## Class Code | Payload Size (bytes) | Description

## 0      |     0                | Control/keepalive (no payload)  1      |     64               | Minimal overhead (e.g., tunnel requests)  2      |     256              | Optimized for ArcNet segments  3      |     512              | Suited for serial links (e.g., DNP, PPP-like)  4      |     1500             | Standard LAN frame size (mandatory)  5      |     4096             | Larger frame size for capable links  6      |     8192             | Jumbo frame for high-performance links  7      |     16384            | Future doubling of jumbo frame size  8–15    |   Reserved           | For future use

1.5 Operational Notes

– Addressing Modes:

  • Use short (8‑bit) addresses on local segments (such as ArcNet or DNP).

  • Use full (64‑bit) addresses when packets traverse multiple network domains.

– Hierarchical Routing:

  Routers examine the 8‑bit segments of a 64‑bit address (one level at a time) to determine the next hop.

– Hop Limit:

  The hop limit is decremented at each router; if it reaches 0, the packet is discarded.

– Flow Control:

  The 16‑bit Flow Control ID is reserved for future enhancements and is ignored in simple router implementations.

– No Fragmentation:

  Packets must be transmitted as a single frame matching one of the fixed payload size classes.

---

2. GDP in LAN and WAN Use Cases

---

2.1 Open Standard, Universal Fit

GDP is an open standard designed&#x20;
for both local area networks (LANs) and wide area networks (WANs). Its&#x20;
flexible addressing allows:

  • Short 8‑bit addresses for small, fixed networks (such as ArcNet or DNP point‑to‑point links).

  • Full 64‑bit addresses for routed, hierarchical networks.

2.2 Integration with DNP (Point-to-Point Serial Links)

• Short Address Mode:

 Devices connected via DNP receive an 8‑bit identifier, which significantly reduces header overhead.

• Router-Attached Prefix:

 When packets leave the local DNP network, the router automatically&#x20;
appends the appropriate 64‑bit prefix to the short address for proper&#x20;
global routing.

• Minimal Overhead:

 Ideal for the low-bandwidth and deterministic conditions of early serial networks.

2.3 LAN Use with ArcNet

• Native Compatibility:

 ArcNet devices typically use 8‑bit addresses. GDP’s short addressing fits seamlessly with this model.

• Payload Matching:

 GDP’s 1500‑byte payload class matches the typical LAN frame sizes of the era, avoiding the need for fragmentation.

• Bridging:

 ArcNet–GNet bridges can map short addresses to full 64‑bit addresses when forwarding packets between network segments.

2.4 Comparison with IPv4 (Mid‑1980s Context)

Feature Comparison:

 – Address Length: IPv4 uses 32 bits; GDP supports 64 bits or an 8‑bit short mode.

 – Fragmentation: IPv4 supports fragmentation (with added complexity); GDP does not support fragmentation.

 – Routing Hierarchy: IPv4 uses a mostly flat addressing scheme, while GDP employs an 8‑level hierarchical routing system.

 – Header Overhead: IPv4’s header is larger and fixed; GDP’s header is minimal (from 8 to 22 bytes).

 – Extensibility: IPv4 is limited in extensibility; GDP reserves bits for future expansion.

2.5 Why Choose GDP?

– Low Overhead:

 GDP’s optional 8‑bit addressing minimizes header size on small or point‑to‑point links.

– Scalable Addressing:

 Full 64‑bit addressing provides a larger space for hierarchical routing.

– Simplicity:

 A fixed, compact header with reserved fields makes routing and hardware implementation straightforward.

– Open Standard:

 GDP is designed to be implemented on any LAN or point‑to‑point technology of our era without proprietary restrictions.

---

3. Conclusion

---

GDP
&#x20;(Global Data Protocol) provides a unified network layer for both local&#x20;
and wide area networks in the mid‑1980s. Its design accommodates the

Do Markdown, only print minimal amount into asci tables. Just simple size and simple name.  Split the Markdown into multiple sections separated by non Markdown normal text. 

Reasoned about Markdown structure and tables for 17 seconds

Below
&#x20;is the complete GDP specification in Markdown. Each major section is&#x20;
separated by plain (unformatted) text lines. The ASCII table is kept&#x20;
minimal with just two columns ("Name" and "Size").

---

# GDP (Global Data Protocol)

---

## 1. Introduction

GDP is a layer‑3 protocol developed&#x20;
within the GNet project. It uses flexible addressing (supporting either&#x20;
8‑bit or 64‑bit addresses) and fixed payload sizes. Designed for small&#x20;
local networks (like ArcNet or DNP point‑to‑point links) and for large&#x20;
hierarchical networks, GDP offers a simple and efficient method for data
&#x20;transport without fragmentation.

---

## 2. Hierarchical Addressing

Each node in the GNet system is&#x20;
assigned a 64‑bit global address that is conceptually divided into 8&#x20;
levels of 8 bits each. Routers compare these 8‑bit levels to decide the&#x20;
next hop. For local segments, a short 8‑bit address is used; in such&#x20;
cases, the router or bridge attaches the higher‑level prefix.

---

## 3. Header Format

The GDP header is variable in size:&#x20;
it is 8 bytes at minimum and may grow to 22 bytes if full (64‑bit)&#x20;
addresses are used for both the source and the destination.

**Fixed fields (6 bytes total):**

- **Byte 0:** Version (4 bits) and Type (4 bits)
- **Byte 1:** Payload Size Class (4 bits), Src/Dst Mode flags (2 bits: one for source, one for destination), Reserved (2 bits)
- **Byte 2:** Service Class (8 bits)
- **Byte 3:** Hop Limit (8 bits)
- **Bytes 4–5:** Flow Control ID (16 bits, reserved for future enhancements)

**Address fields (variable):**

- **Source Address:** 1 byte if short, 8 bytes if full.
- **Destination Address:** 1 byte if short, 8 bytes if full.

*Minimum header size:* 6 (fixed) + 1 (source) + 1 (destination) = 8 bytes

*Maximum header size:* 6 (fixed) + 8 (source) + 8 (destination) = 22 bytes

---

## 4. Payload Size Classes

Each GDP packet must fit a&#x20;
predetermined payload size class. Routers must support the 1500-byte&#x20;
class, while other classes are optional. Below is a simple ASCII table&#x20;
showing minimal payload sizes and their names.

```
```

```
+----------+--------+
| Name     | Size   |
+----------+--------+
| Control  | 0      |
| Tunnel   | 64     |
| ArcNet   | 256    |
| Serial   | 512    |
| LAN      | 1500   |
| Medium   | 4096   |
| Jumbo    | 8192   |
| Future   | 16384  |
+----------+--------+
```

---

## 5. Operational Notes

- **Address Modes:**

  Use short (8‑bit) addresses on local segments (e.g., ArcNet, DNP). Use&#x20;
  full (64‑bit) addresses for packets that traverse multiple network&#x20;
  domains.
- **Hierarchical Routing:**

  Routers inspect each 8‑bit segment of a 64‑bit address to determine the appropriate next hop.
- **Hop Limit:**

  This field is decremented by each router; if it reaches zero, the packet is discarded.
- **Flow Control:**

  The Flow Control ID is reserved for future enhancements and is ignored in simple implementations.
- **No Fragmentation:**

  Packets must match one of the fixed payload size classes exactly.

---

## 6. Use Cases

### 6.1 LAN Applications

GDP is well-suited for local area&#x20;
networks. The mandatory 1500-byte class fits the frame sizes of many&#x20;
existing LAN technologies. In environments like ArcNet, native 8‑bit&#x20;
addressing minimizes overhead, and simple bridging can be used when&#x20;
extending networks.

### 6.2 DNP (Point-to-Point Serial) Networks

For DNP serial links, devices&#x20;
receive an 8‑bit identifier to reduce header overhead. When traffic&#x20;
leaves the serial segment, routers append the necessary 64‑bit prefix to
&#x20;allow for hierarchical routing without additional overhead on the local
&#x20;link.

---

## 7. Conclusion

GDP provides a unified network&#x20;
layer that is scalable and efficient for both small local networks and&#x20;
larger hierarchical systems. Its flexible addressing—supporting both&#x20;
8‑bit and 64‑bit modes—combined with fixed payload classes (from 0 to&#x20;
16,384 bytes) allows GDP to offer minimal overhead where required and&#x20;
robustness where needed. This open, simple approach sets GDP apart as a&#x20;
future-proof alternative in our era.

---

Feel free to copy this text into your Markdown editor and adjust as necessary.
