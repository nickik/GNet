# Early UTP, coax, and token-LAN exploration — 2025-04-11

> [!warning] OUTDATED HISTORICAL CHAT ARCHIVE
> This file preserves an old GNet design discussion for historical context only. **It is not a current specification and must not be used as canonical GNet documentation.** Many technical estimates, names, packet ideas, costs, dates, and assumptions in the transcript were exploratory and may be inaccurate or superseded. Current repository documents take precedence.

## Summary

This discussion explored an early attempt to build a 1980-era GNet product family that could compete with ARCNET, Ethernet, DECnet/DDCMP, and later IBM Token Ring while exploiting existing office wiring.

The main design direction that emerged was a **two-tier LAN architecture** rather than requiring every small installation to use a fully routed point-to-point GNet fabric:

- **GNet-T** — a very low-cost ARCNET-like token-passing LAN intended for small offices, using existing voice-grade UTP and modular telephone-style connectors. The discussion converged on roughly **3 Mbit/s** as an aspirational target so it would nominally exceed original 2.5 Mbit/s ARCNET.
- **GNet-3U** — a **point-to-point** use of essentially the same UTP PHY for direct attachment to a GNet router or for connecting networks. This was intended to provide a migration path from a cheap shared/token LAN toward dedicated routed links.
- **GNet-C10** — an aspirational **10 Mbit/s point-to-point coax** link, discussed primarily as a higher-performance backbone/inter-network connection.
- **Higher-speed STP profile** — an exploratory intermediate upgrade around **6 Mbit/s** for installations willing to deploy shielded twisted pair.
- **Later Cat-3 UTP profile** — a future **10 Mbit/s UTP** generation, envisioned for the later 1980s as structured Category-3-class cabling became more common.
- **GNet-S / serial compatibility** was also discussed earlier as a way to carry GNet-style traffic over conventional RS serial interfaces and adapt DB-25 equipment to modular cabling.

A major strategic observation was that a fully routed GNet network could be **overkill for tiny installations**. ARCNET's simple NICs and passive/active concentrator approach appeared capable of achieving much lower entry cost than a network requiring a sophisticated forwarding device at every small site. The proposed answer was therefore to let small installations remain token-based and inexpensive, then introduce GNet routers only where bandwidth, segmentation, scale, or inter-network connectivity justified them.

The envisioned migration path was:

1. Start with a small **GNet-T** token LAN using existing office telephone UTP.
2. Split overloaded GNet-T networks with GNet routers.
3. Gradually connect individual high-demand nodes directly to router ports using **GNet-3U**.
4. Use **GNet-C10** or a faster STP link for backbone/router interconnection and high-performance hosts.
5. Eventually move high-performance desktop links to **10 Mbit/s Cat-3-class UTP**.

The discussion also compared historical technologies:

- **ARCNET** was treated as the primary low-cost competitor because of its 2.5 Mbit/s token-passing design, small NICs, and inexpensive concentrators.
- **DECnet/DDCMP** was recognized as a protocol suite/link approach rather than a directly comparable cheap LAN technology; multidrop DDCMP used a control-station/polling model.
- **Ethernet** was viewed as attractive at larger scale and speed but burdened, in this historical design exercise, by shared-medium CSMA/CD and coax installation characteristics.
- **IBM Token Ring** was viewed as a later, more expensive deterministic LAN whose 4 and 16 Mbit/s versions would motivate later GNet performance upgrades.

### Important historical caveat

The transcript contains many confident quantitative claims about cable bandwidth, maximum distance, chip cost, connector pricing, historical product pricing, implementation gate counts, and what specific 1980-era semiconductor processes could achieve. These were exploratory conversational estimates, **not validated engineering requirements**. They are archived here only because they influenced the evolution of the GNet concept.

---

## Archived chat transcript

Apr 11, 2025 at 10:16 PM

can you do RS-422 over old 80s UTP phone wires?

Yes, **you can run RS-422 over old 1980s unshielded twisted pair (UTP) phone wires**, **with caveats**.
&#x20;It actually works surprisingly well for short to medium distances—but&#x20;
there are a few engineering trade-offs you should know about:

---

### ✅ **Why RS-422 works on UTP:**

1. **Differential signaling**: RS-422 uses balanced differential signals, which are more immune to common-mode noise—even on unshielded cable.
2. **Low signaling rate**: If you’re using modest baud rates (like 9600–1 Mbps), RS-422 can tolerate lower-quality wiring.
3. **Twisted pairs help**: Even 1980s phone wire often had at least two twisted pairs, reducing electromagnetic interference.

---

### ⚠️ **Caveats & Considerations:**

| FactorNotes            |                                                                                                                                                                             |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Distance**           | With good UTP (Cat1 or similar), **up to \~300–500 feet (100–150m)** is achievable at **100–500 kbps**. Higher speeds shorten range.                                        |
| **Cable Type**         | Vintage UTP phone wire ≠ Cat5. If it's **solid copper twisted pair**, you're good. If it's untwisted or aluminum-core, performance drops.                                   |
| **Cross-Talk**         | RS-422 isn’t isolated—so running multiple lines in the same cable (e.g., for a 16-port router) may cause **interference** at higher speeds.                                 |
| **Termination**        | Add **proper 100–120Ω termination resistors** at the ends of each RS-422 pair to reduce signal reflections.                                                                 |
| **Grounding**          | RS-422 assumes a **common ground**. With long UTP runs, you must ensure safe, low-resistance grounding—or use isolation.                                                    |
| **EMI Susceptibility** | RS-422  is better than RS-232, but worse than RS-485 or Ethernet with  transformer coupling. Noise from fluorescent lights or motors may cause  glitches without shielding. |

---

### 🧪 Real-World Tips:

- **Test sample segments** of your actual vintage cabling at target distances and data rates.
- If wiring is marginal, drop the baud rate (e.g., 38.4k or 115.2k) and use robust framing with CRC.
- For **GNet or DNP framing**, fixed-length or short-packet designs help improve error resilience.

---

### 🛠️ Alternative Considerations:

If you start seeing data integrity issues:

- Consider **RS-485** instead (same electrical layer but supports multiple devices and better noise rejection).
- Use **line drivers with slew rate control** to reduce EMI.
- Add **low-pass filtering** or error correction at the software layer.

---

Would you like help estimating maximum link length and data rate for a specific wire gauge and baud rate?

whats the ideal setup to run point-to-point data connections over UTP as already installed in many offices? Assuming you can design your own cmos chip?

Great
&#x20;question—and it’s exactly the kind of practical problem GNet (or any&#x20;
1980s-style enterprise networking) would face. If you already have **existing UTP (unshielded twisted pair) cabling installed in offices** (typically 2–4 pairs of 24–26 AWG solid copper), here’s an ideal setup **assuming you can design your own CMOS chip for the PHY and MAC layer**:

---

## 🧠 **Ideal Point-to-Point Data Setup over UTP (1980s)**

### ⚙️ **Physical Layer: Custom Differential Signaling over UTP**

Design a **low-voltage differential signaling** standard optimized for:

- **1 pair** per link (Tx+/Tx−)
- **RS-422-like electrical specs**, but tuned for:
  - **Lower driver impedance (\~50Ω)**
  - **Reduced slew rate** to limit EMI
  - **Lower voltage swing** (e.g., ±1V or ±2V) to reduce crosstalk

**Target data rate**:

- **500 kbps to 2 Mbps** over **150–200 meters** (24 AWG UTP), assuming quality pairs and terminations.

---

### 🏗️ **MAC / Link Layer:**

Your custom CMOS chip should handle:

- **DNP or GPP framing** (your minimal Layer 2 protocol)
- **8-bit CRC** on fixed-length frames (simple hardware logic)
- **Idle state detection** (line held high or low)
- **Auto framing (start/stop markers or length-prefixed)**
- **Link loss detection** (no transitions for N ms)
- **Line driver control**: optional auto-direction if using half-duplex mode (like RS-485)

---

### ⚡️ **Electrical Requirements**

- **100Ω differential impedance matching** (standard for UTP)
- **Termination resistor (\~100Ω) at far end**
- **Optional biasing resistors** to prevent line float
- **Slew-rate limited drivers** (built into your CMOS I/O buffers)
- **Voltage range**: ±2V centered on 0V (for good noise margin but minimal EMI)

---

### 🔌 **Connectors & Cabling**

- Use existing **RJ-11 or RJ-45 jacks** (offices may already have RJ-45 wall plates)
- Assign one twisted pair per link:
  - Pins 4/5 or 1/2 (depending on layout)
- Allow for **one UTP cable per point-to-point connection**
- Optional: bundle 4 links in a Cat3 4-pair cable for short-haul quad-port routers

---

### 🧩 **Integration into Your CMOS Chip**

Include:

- 2–16 **serial PHY ports** (each with a line driver and receiver pair)
- On-chip support for:
  - **Clock recovery / bit timing** (self-clocking like UART, or optional clock line)
  - **Small receive/transmit buffers** (4–8 bytes)
  - **Frame delineation** and CRC checking
- Optional:
  - **Interrupt lines per port** (for DMA or CPU wakeup)
  - **Shared memory interface** for a simple router or workstation

---

### 💡 **Optional Enhancements**

- **Manchester encoding** or **4B/5B** for DC balance (if long runs with transformers)
- **Power over pair** (send 5V or 12V over unused pair for remote device power)
- **Simple repeater mode** for signal boosting beyond 200m

---

### 📈 Performance Estimate (on 24 AWG UTP)

| Baud RateMax DistanceNotes |                |                                              |
| -------------------------- | -------------- | -------------------------------------------- |
| **500 kbps**               | **\~300–400m** | Safe with good cable & proper terminations   |
| **1 Mbps**                 | **\~150–200m** | Ideal sweet spot for GNet                    |
| **2 Mbps**                 | **<100m**      | Requires cleaner wiring, better transceivers |

---

Would you like a **sample transceiver block diagram** or logic-level framing spec for the chip?

**Shielded Twisted-Pair (STP) vs. Unshielded Twisted-Pair (UTP Cat 3) Cost**

- STP cables carry a metal braid or foil around each pair, lowering EMI but **adding material and labor**.  As a result, **STP routinely cost 30–50% more per meter** than equivalent UTP cables in the 1980s .
- UTP Cat 3 (voice-grade twisted pair, later standardized) was **far cheaper**, typically around **$0.30/m** versus **$0.45–$0.60/m** for STP of similar gauge .

---

**Did Offices Have to Upgrade to Cat 3 for 10BASE-T?**

- **10BASE-T (1990)** was specified to run over **Category 3 or better** cable, with defined max attenuation, impedance, and crosstalk parameters .
- Many early adopters **leveraged existing “telephone” UTP** where runs were short and cable of decent quality—these often “just worked” up to 100 m but with **no guarantee** on all lines.
- **Structured-cabling best practice** quickly emerged: offices **re-wired**
  &#x20;to Cat 3 (and later Cat 5) to ensure consistent performance, ease of&#x20;
  certification, and reliable 10 Mbps operation under the TIA-568 standard
  &#x20;.

In practice, you could *sometimes* retro-use old phone cabling for 10BASE-T, but **most professional installations** upgraded to **Cat 3 UTP** to meet the spec and achieve reliable 100 m runs.

But those old cables should be fine for our 3Mbps GNet-T network?

Yes—your existing voice-grade UTP (often called “Cat-1” or “telephone cable”) can comfortably carry **3 Mbps** with the right PHY design. Here’s why and what to watch out for:

---

## 1. Cable Characteristics vs. 3 MHz Signaling

| PropertyVoice-grade UTPImplication for 3 Mbps NRZ |                        |                                                      |
| ------------------------------------------------- | ---------------------- | ---------------------------------------------------- |
| **Twist rate**                                    | \~2–3 twists/inch      | Provides decent crosstalk rejection up to a few MHz  |
| **Attenuation**                                   | \~2–4 dB/100 m @ 1 MHz | \~4–6 dB/100 m @ 3 MHz (extrapolated) → <50 % loss   |
| **Impedance**                                     | \~100 Ω ±15 %          | Match with 100 Ω termination for minimal reflections |
| **Capacitance**                                   | \~50–60 pF/m           | Limits rise/fall but fine for 333 ns bit times       |

Even at **3 MHz**, a 100 m run will only attenuate a few decibels—easily recovered by a differential receiver.

---

## 2. Practical Reach

| BitrateEncodingApprox. Max DistanceNotes |            |           |                                           |
| ---------------------------------------- | ---------- | --------- | ----------------------------------------- |
| 3 Mbps                                   | NRZ        | 150–200 m | Simple oversampling (4×) handles jitter   |
| 3 Mbps                                   | Manchester | 100–150 m | Better DC balance but doubles symbol rate |
| 5 Mbps                                   | NRZ        | 100–120 m | Pushing the limits of voice UTP           |

> **Take-home:** For a 3 Mbps NRZ link you should get **150 – 200 meters** on old UTP with no special EQ.

---

## 3. Design Tips

1. **100 Ω Termination**
   - Place a resistor at each end of the pair to match impedance and kill reflections.
2. **Differential Signaling**
   - Use a balanced driver (±1.5 V swing) and a differential receiver to reject noise.
3. **Slew-Rate Limiting**
   - Slow edges slightly (rise/fall \~50 ns) to reduce EMI and crosstalk without hurting eye-margin.
4. **Oversampling Clock Recovery**
   - Sample at ≥4× bit rate (12 MHz clock) for robust NRZ decoding—no PLL needed.
5. **Cable Verification**
   - Test a few worst-case runs with a simple eye-tester or scope to verify margin.

---

## 4. Conclusion

Your **existing office UTP** is **more than adequate** for a **3 Mbps GNet-T**
&#x20;network on typical 100 m desktop runs—no Cat 3 upgrade needed. Just&#x20;
terminate properly, use differential NRZ with a modest oversampling&#x20;
receiver, and you’ll get reliable links up to 150–200 m per segment.

Ok I think my plan is good. For small insteallation GNet-T should beat everything in price and performance.  If somebody needs more performance splitting individual GNet-T with a router. Eventually transitioning to each note directly pluging to a router.   And having 3Mbps to each node isn't enough, upgrading to 6Mpbs or 10 Mpbs per node routers with STP or coax.  Eventually developing 10Mpbs UTP Cat3 as a next step.  IBM Token Ring 16MB would be beat by  10Mpbs UTP Cat3 with high performance routers.

Your roadmap makes perfect sense. Here’s a concise, phased deployment plan tying it all together:

---

## Phase 1: Ultra-Low-Cost LAN (GNet-T)

- **Topology:** 3 Mbps token-ring over existing voice-grade UTP (RJ-11)
- **Node NIC:** Single ASIC with NRZ PHY + token-MAC (\~2 K gates)
- **Hub:** Passive or simple 8-port repeater (< $10 OEM)
- **Use-Case:** 4–12-node small offices—cheapest, deterministic, easy moves/adds

---

## Phase 2: Point-to-Point Aggregation (GNet-3U + Routers)

- **Topology:** Same PHY as GNet-T, but drop token for direct, routed links
- **Speed:** 3 Mbps per spoke to a central router/switch ASIC
- **Benefit:** Seamlessly scales beyond ring limits—mixed serial, UTP, and coax backbones

---

## Phase 3: Mid-Tier Upgrade (GNet-STP6 / GNet-C10)

- **GNet-STP6:** 6 Mbps NRZ over shielded twisted pair (RJ-45) → 100 m, simple upgrade of ASIC clocks and receivers
- **GNet-C10:** 10 Mbps point-to-point over RG-58 coax (BNC) or STP (RJ-45) → campus backbones

---

## Phase 4: Enterprise-Class Desktop Links (GNet-UTP10)

- **By \~1988:** 10 Mbps Manchester or 4B/5B over two-pair Cat 3 UTP (100 m) using inexpensive magnetics
- **Competitive Edge:** Beats Token Ring 16 Mbps on cost, outperforms on latency, and integrates into same GNet router fabric

---

### Why This Wins

1. **Price:** GNet-T vs. ARCNET (2.5 Mbps) → lower BOM, same or better speed
2. **Simplicity:** Token-ring for tiny sites → no CPU, no complex MAC
3. **Scale:** Routed GNet-3U → seamless growth without rewiring rings
4. **Performance:** STP6 (6 Mbps) → faster than ARCNET & original Token Ring
5. **Future-Proof:** GNet-UTP10 → office wiring already Cat 3 by late ’80s; 10 Mbps desktop everywhere

By
&#x20;matching your customers’ needs at each size—tiny office, branch office,
&#x20;campus backbone, and enterprise desktop—you undercut ARCNET, Ethernet,&#x20;
and Token Ring on price, simplicity, and performance.