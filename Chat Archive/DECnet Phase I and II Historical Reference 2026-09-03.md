# DECnet Phase I and II historical reference — 2026-09-03

> [!note] HISTORICAL REFERENCE
> This file archives a research discussion about early DECnet for comparison with GNet. It does **not** define or modify the GNet protocol suite. Current GNet specification documents remain canonical.

## Summary

This discussion examines **DECnet Phase I and Phase II** as historical networking systems, concentrating on how functions familiar from later IP networks were handled before DECnet gained general routing.

The central distinction is that both phases were still fundamentally **direct-link systems**:

- **Phase I (1974–1975)** was essentially a two-node point-to-point network. DDCMP supplied reliable framing, sequencing, CRC checking, acknowledgements and retransmission, but there was no routed network layer, network-wide numeric addressing, TTL/hop count, ARP-like resolution, or multi-hop forwarding.
- **Phase II (1975–1976)** expanded DECnet to small multi-node networks, commonly described as supporting roughly 32 nodes, but communication was still between directly connected adjacent systems rather than through general-purpose routers. Routing became a Phase III feature.

Phase II added much of the higher-level machinery that makes DECnet recognizable as a network architecture:

- **NSP (Network Services Protocol)** provided connection-oriented task-to-task communication above DDCMP, including connection establishment, sequencing, acknowledgements, segmentation/reassembly, flow control, interrupt data and orderly disconnect.
- Services were selected with **network object numbers**, broadly analogous to later transport ports. FAL used object 17 and customer applications commonly occupied higher-numbered objects.
- **DAP/FAL** provided remote file access and transfer.
- **MOP** used DDCMP maintenance operation for functions such as downline loading and diagnostics.
- **NCP** supplied node/network configuration and management functions.

### Addressing model

Phase II is particularly interesting because its addressing model sits between a pure point-to-point link and a later routed network.

Human-facing node references used **unique alphanumeric node names**. During adjacent-node initialization, however, systems also exchanged a numeric **NODEADDR** field together with **NODENAME**, supported functions, requested functions, protocol sizes and related startup information. The archived discussion notes the Phase II constraint that NODEADDR be greater than 1 and less than 241 and unique in the local network.

This numeric value should not be confused with the routed DECnet address architecture introduced later. Phase II still lacked a general routed numeric network layer.

### Comparison with familiar IP concepts

| Later IP-network concept | DECnet Phase I / II analogue |
|---|---|
| Link framing | **DDCMP** |
| Link error detection | CRC-16 |
| Link reliability | DDCMP sequence, ACK/NAK and retransmission |
| IP routed address | None in Phase I; Phase II primarily used node names plus startup NODEADDR, but no general routed address architecture |
| Router / next hop | None for normal Phase I/II communication |
| TTL / hop limit | None because normal communication was not multi-hop routed |
| ARP | Not needed on ordinary point-to-point DDCMP links |
| TCP-like connection | **NSP** |
| TCP port | **Network object number** |
| MSS-like negotiation | NSP **SEGSIZE** during connection setup |
| DNS-like global naming | None; node names were configured rather than resolved through a global distributed service |
| FTP/NFS-like file service | **DAP/FAL** |
| Bootstrap/maintenance | **MOP** |

### DDCMP wire format

The discussion records DDCMP as defining **Data**, **Control**, and **Maintenance** message families. A numbered data message was represented as:

```text
SOH | COUNT | FLAGS | RESP | NUM | ADDR | BLKCK1 | DATA (COUNT bytes) | BLKCK2
```

Important fields include:

- `COUNT` — payload byte count.
- `RESP` — acknowledgement state returned toward the peer.
- `NUM` — transmitted message sequence number, modulo 256.
- `ADDR` — station address; normally 1 on point-to-point links and usable for tributary addressing on multipoint DDCMP.
- `BLKCK1` / `BLKCK2` — CRC-16 protection for header and data.

The archived discussion cites a maximum DDCMP data block of **16,383 bytes**, while noting the CRC's strongest guaranteed properties over smaller blocks.

An example control-message form is:

```text
ENQ | ACKTYPE | ACKSUB | FLAGS | RESP | FILL | ADDR | BLKCK3
```

Control messages included functions such as ACK, NAK, REP, START and STACK.

Maintenance-mode framing was represented as:

```text
DLE | COUNT | FLAGS | FILL | FILL | ADDR | BLKCK1 | DATA | BLKCK2
```

MOP could use this maintenance path for diagnostics and downline loading without the normal numbered-data reliability procedure.

### NSP wire model

A common Phase II NSP message skeleton was described as:

```text
[optional RTHDR] | MSGFLG | DSTADDR | SRCADDR | ACKNUM | type-specific fields
```

Connection establishment used **Connect Initiate (CI)** and **Connect Confirm (CC)** messages. These could negotiate services and maximum segment size (`SEGSIZE`) and carry limited setup information. NSP then supplied sequenced data, ACK/NAK handling, interrupt messages, flow control and disconnect messages.

Adjacent systems first performed a **Node Initialization** exchange containing node identity and capability information. If configured, a subsequent **Node Verification** exchange could carry a password before the link became operational.

### Relevance to GNet research

The useful historical lesson for GNet is architectural rather than a direct protocol recommendation. DECnet shows an evolutionary path in which DEC first built a highly reliable point-to-point link protocol, then added a connection-oriented task/service layer, and only later generalized routed addressing and multi-hop forwarding. It also demonstrates that early networking stacks did not necessarily reproduce the later Ethernet + IP + TCP split: reliability, bootstrap, naming and service selection could reside at very different layers.

No GNet design decision is changed by this archive.

---

## Archived chat transcript

### User

explain Phase I and Phase II of DECNET in some detail, what could it do. How did the typical things we know from IP work.

address asigment, wire format and all the detail you can find.

### Assistant

here’s a focused deep-dive on **DECnet Phase I and Phase II**—what they could do, how “IP-like” things mapped in those eras, and the key **addressing + wire formats** you asked for.

# Phase I (1974–1975): the seed network

- **Scope & topology:** essentially a *two-node* network, PDP-11 to PDP-11 (and small PDP-8 deployments), over **point-to-point DDCMP** links; no routing.
- **Stack shape:** DDCMP (data-link) provided reliable framing, sequencing, CRC, and retransmission. Higher-level services were minimal; the richer application and management pieces (FAL/DAP/MOP/etc.) arrive with Phase II and later.

### “How IP-like things worked” in Phase I

- **Addressing:** no network-wide numeric addressing; you just brought up a *single* point-to-point link to exactly one neighbor.
- **Routing, TTL, ARP:** none—there’s nowhere to route. DDCMP is a reliable point-to-point pipe; there’s no L2/L3 address resolution problem to solve.

---

# Phase II (1975–1976): small multi-node, still hop-by-hop only

- **Scope & topology:** supports **multi-node networks (up to ~32 nodes)**, but **still only over direct links**—you can talk to *adjacent* nodes; **no routing through intermediates** yet. (Routing arrives in Phase III.)
- **Key additions vs Phase I:**
  - **NSP (Network Services Protocol)**: a connection-oriented, reliable transport/session that sits above DDCMP. It does connects, segmentation/reassembly, flow control, interrupts, and orderly disconnect.
  - **Network objects** (think “ports”): services are addressed by **object numbers 1–255**; **FAL (File Access Listener)** is **object 17**; numbers ≥200 are for customer apps.
  - **DAP (Data Access Protocol)** & **FAL**: remote file access and file transfer.
  - **MOP (Maintenance Operations Protocol)**: downline loading/maintenance using DDCMP “maintenance mode”.
  - **NCP-based management** begins to take shape (per-node config; no dynamic address service).

## Addressing & “port” model in Phase II

- **Node addressing used in practice:**
  - **Human-visible addressing is by node *name*** in Phase II (numeric system-wide addresses are a Phase III thing). DEC’s own Phase III intro manual states: “In Phase II networks, nodes are addressed by unique alphanumeric names rather than by numbers”.
  - During **adjacent-node startup** (NSP link bring-up), peers exchange a **Node Initialization** message that carries **NODENAME** *and* a **NODEADDR** field. The spec requires `NODEADDR > 1 and < 241` and uniqueness within the local network. This numeric “node address” is part of the Phase II handshake and management, but the user-facing way you targeted a peer was by name.
- **Service selection (ports):** connections target a **node** plus an **object number** (e.g., FAL=17). Customer apps typically use object numbers in the 200+ range. This is the direct analogue of **TCP ports**.

## What “IP-like” pieces looked like in Phase II

- **L2 framing & reliability:** **DDCMP** does what HDLC/PPP do in IP stacks—framing, sequence numbers, **CRC-16**, ACK/NAK, retransmission, and even pipeline up to **255 outstanding** messages. **Max block** up to **16,383 B** (CRC most effective ≤ 4093 B).
- **L3 (routing):** **none.** There is no TTL/hop-count in Phase II because nodes only talk to directly-attached neighbors. (An **RTHDR** field exists in NSP messages but is a special-purpose header used by *intercept* nodes; ordinary Phase II traffic between neighbors doesn’t carry it.)
- **Transport/session (vs TCP):** **NSP** provides:
  - **Connect** (CI)/**Confirm** (CC) handshake carrying requested **services**, **max segment size (SEGSIZE)**, and optional **user data**—very much like TCP’s option negotiation + MSS.
  - **Sequenced data**, **ACK/NAK** with segment numbers, **interrupt** messages (urgent-like), and **orderly disconnect**.
  - **Flow control:** via request counts/segment counts in the negotiated services; SEGSIZE in CC is the min of local constraints—again, very TCP-ish in spirit.
- **Name/addr resolution (vs ARP/DNS):** no ARP—links are **DDCMP point-to-point**. **Node name** is configured (per-node) and exchanged during **Node Initialization**; there’s no network-wide name service yet.
- **Authentication:** optional **Node Verification** message with a **PASSWORD** right after Node Initialization if verification is requested.

---

# Wire formats (what’s on the wire)

## DDCMP (data-link) message formats

DDCMP defines three families: **Data**, **Control**, and **Maintenance**. All include CRC protection; multi-point links use a **station address** field; point-to-point uses address **1**.

**Numbered Data Message (payload carrier)**

```text
SOH | COUNT | FLAGS | RESP | NUM | ADDR | BLKCK1 | DATA (COUNT bytes) | BLKCK2
```

- **COUNT**: bytes in DATA (≠0).
- **FLAGS**: link control (sync/ownership).
- **RESP/NUM**: receiver ACK state & sender sequence number (mod 256).
- **ADDR**: station address (1 on p2p; tributary on multipoint).
- **BLKCK1/2**: CRC-16 over header and data respectively.
- **Max DATA**: up to **16,383 bytes** (CRC strongest ≤ 4093).

**Control Messages (ACK/NAK/REP/START/STACK, etc.)** share fields and also carry **ADDR** plus **BLKCK3** (CRC over the control header). Example (ACK):

```text
ENQ | ACKTYPE | ACKSUB | FLAGS | RESP | FILL | ADDR | BLKCK3
```

(START/STACK are used to establish link numbering & sync).

**Maintenance Message** (used by **MOP**):

```text
DLE(220o) | COUNT | FLAGS(=3) | FILL | FILL | ADDR | BLKCK1 | DATA | BLKCK2
```

Maintenance mode is “best-effort” CRC-checked (no sequencing/ACK), intended for diagnostics and **downline load**; **MOP** runs here.

> **CRC:** DDCMP uses **CRC-16** (x¹⁶+x¹⁵+x²+1) for header/data checks; the spec details computation and detection properties.

## NSP (Phase II) message skeletons

Many NSP messages share a common front matter:

```text
[optional RTHDR] | MSGFLG | DSTADDR | SRCADDR | ACKNUM | (…type-specific fields…)
```

Key ones:

- **Connect Initiate (CI)** → **Connect Confirm (CC)** negotiate **SERVICES**, **SEGSIZE**, optional **INFO / DATA-CTL** (user data). **DSTADDR** may be 0 in CI (ask peer to assign link address).
- **ACK/NAK** include **ACKNUM** with qualifiers (ACK/NAK).
- **Disconnect Initiate/Confirm** carry a **REASON** code and optional data.

### Link bring-up (adjacent nodes)

On each physical link, Phase II NSP uses **Node Initialization** (and optionally **Node Verification**) before allowing user traffic:

1. **Node Initialization** message includes (among others):
   - **STARTTYPE** (startup flavor)
   - **NODEADDR** (must be >1 and <241; unique within the network)
   - **NODENAME**
   - **FUNCTIONS** supported & **REQUESTS** (features you need)
   - **Version/size** info (e.g., **NSPSIZE**, **BLKSIZE**)
   - **No RTHDR** is sent on these.
2. If requested, the peer replies with **Node Verification** carrying **PASSWORD**.
3. Link state transitions through **OFF → STARTING → INITIALIZE → VERIFY → ON** depending on whether verification is required.

> Note: DEC’s Phase III intro explicitly contrasts **Phase II names** with **Phase III numbers**—Phase II *user* addressing is by **unique node name**; the **numeric** field in Node Init exists but the network didn’t yet use a global numeric addressing plan for end-user targeting.

---

# What Phase II *could do* day-to-day

- **Task-to-task (program-to-program) links** over NSP; your app listens on an **object number** (customer apps often used ≥200).
- **Remote file access and transfer** via **DAP/FAL (object 17)**—think “DEC’s NFS/FTP-ish” combo.
- **Downline load & maintenance** with **MOP** over DDCMP maintenance frames.
- **Node & line management** with early **NCP** tooling (configure node name, passwords, objects, known nodes, etc.).

---

## Quick mapping cheat-sheet (IP vs DECnet Phase II)

| IP world | DECnet Phase II analogue |
|---|---|
| MAC framing (HDLC/PPP) | **DDCMP** data/control/maintenance frames with CRC-16, seq/ACK, and station address. |
| IP addressing | **Node names** (user-visible), plus a local **NODEADDR** exchanged at link start; no routed numeric plan yet. |
| TTL/hops/routing | **None** (adjacent-only); routing starts with Phase III. |
| ARP | **Not needed**—links are point-to-point; identity exchange via **Node Initialization**. |
| TCP ports | **DECnet object numbers** (e.g., FAL=17; 200–255 for customer use). |
| TCP handshake/options/MSS | **NSP CI/CC** negotiate services and **SEGSIZE**; provide **ACK/NAK**, segment numbers, orderly disconnect. |

---

## Sources noted in the discussion

- **DDCMP 4.0 specification (March 1978)** — message formats, CRC-16, block sizes, station-address use, maintenance mode/MOP.
- **NSP Phase II specification (March 1978)** — CI/CC/ACK/NAK formats, Node Initialization/Verification, flow control, RTHDR and state machines.
- **DEC Introduction to DECnet Phase III (May 1982)** — contrast between Phase II node-name addressing and Phase III numeric addressing; network-object model.
- Contemporary DECnet histories/overviews for the broad phase capabilities and limits.
