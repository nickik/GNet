Exactly. That makes the bootstrap traffic much less concerning.

At the **GNet protocol level**, the only broadcast required is:

```
```

```
HOST → LOCAL BROADCAST

ROUTER_SOLICIT
    host_id = ...
    preferred_node = 17   ; optional
```

Every router attached to that local arbitration domain may hear it. Ordinary hosts ignore it.

The response is directed specifically back to the requesting station:

```
```

```
ROUTER → HOST

ROUTER_ADVERTISEMENT
    prefix = 4.27
    router = .1
```

So there is **no reason for periodic router advertisements at all** in the minimal 1978 protocol.

### We can simplify even further

If address assignment is part of discovery, I would probably collapse the two stages:

```
```

```
HOST → broadcast

NETWORK_SOLICIT
    host_id = 0x...
    preferred_node = 17
```

Router answers directly:

```
```

```
ROUTER → requesting attachment

NETWORK_ASSIGN
    prefix      = 4.27
    node        = 17
    router      = 1
    config      = ...
```

And you're done:

```
```

```
plug in
   ↓
hub gives temporary attachment = 23
   ↓
NETWORK_SOLICIT              ← only broadcast
   ↓
NETWORK_ASSIGN               ← directed reply
   ↓
normal address = 4.27.17
```

That is probably better than having separate:

```
```

```
ROUTER_SOLICIT
ROUTER_ADVERTISEMENT
ADDRESS_REQUEST
ADDRESS_ASSIGN
```

for the initial version.

## The temporary attachment number makes this especially easy

Before `.17` exists, the reply can be addressed as:

```
```

```
destination:
    LINK-ATTACHMENT 23
```

The hub already knows how to identify its physical port 23.

After assignment, normal packets use:

```
```

```
destination:
    GNET 4.27.17
```

So there are two very clean addressing regimes:

```
```

```
BOOTSTRAP
    physical attachment number

NORMAL OPERATION
    logical GNet address
```

The physical attachment never leaks into the routed address.

## And this isn't really an ARP-flood situation

Even the solicitation broadcast has tightly bounded scope:

```
```

```
GNet arbitration domain
        │
        ├── host
        ├── host
        ├── host
        └── router

NETWORK_SOLICIT
        X
        │
        └── NEVER forwarded by router
```

One machine booting therefore causes:

- one 3-Mbit/s local frame;
- perhaps one or several directed router replies;
- **zero propagation into neighboring GNet segments**.

There is no recursive amplification.

Even if 32 machines all boot simultaneously, that's roughly 32 solicitation packets—not an ongoing broadcast protocol.

### Retries

If a host gets no response:

```
```

```
solicit
wait
solicit
wait longer
solicit
```

Use a small pseudo-random delay so that a room full of machines powered on simultaneously doesn't synchronize its retries.

That's enough.

## There may be an even more extreme optimization later

If a GNet hub has a **defined router/uplink socket**, for example:

```
```

```
ports 0–31 = stations
port R     = router
```

then theoretically the hub could know:

> bootstrap traffic goes to `R`.

Then even `NETWORK_SOLICIT` wouldn't need to be broadcast electrically.

But I wouldn't require this in the base protocol. It starts giving the hub knowledge about packet classes.

The beautifully simple baseline is:

> **One
> &#x20;local broadcast when a node needs network configuration. Everything&#x20;
> after that is directed. Routers never forward that broadcast.**

That is so little broadcast traffic that I would not consider flooding a meaningful architectural concern.

why not make a generic discovery protocol and have predefined number for different types, like router, directory and potentially others.

Yes. I think that is better than inventing separate `ROUTER_SOLICIT`, `DIRECTORY_SOLICIT`, `BOOT_SOLICIT`, etc.

Make **GNet Discovery** a tiny generic link-local protocol with a registered **service type number**.

## Generic form

A host sends:

```
```

```
DISCOVER
    service_type = ROUTER
    transaction  = 0x37
```

to the local GNet broadcast destination.

Only devices offering that service respond:

```
```

```
DISCOVER_REPLY
    service_type = ROUTER
    transaction  = 0x37

    provider      = <temporary/link-local destination>
    data          = ...
```

The response is **directed back to the requester**, never broadcast.

And:

> `DISCOVER` packets are never routed.

So discovery remains strictly an arbitration-domain mechanism.

---

## Predefined service types

I would reserve a small numeric registry from the beginning.

For example:

| TypeService |                                |
| ----------- | ------------------------------ |
| `0`         | reserved                       |
| `1`         | Router / network configuration |
| `2`         | Directory                      |
| `3`         | Boot service                   |
| `4`         | Time service                   |
| `5`         | Configuration service          |
| `6`         | Name service                   |
| `7`         | Authentication service         |
| `8`         | Printer service                |
| `9`         | File service                   |
| `10`        | Terminal/login service         |
| `11`        | Management/diagnostics         |
| `12–31`     | future GNet standard services  |
| `32–127`    | registered services            |
| `128–255`   | experimental/vendor/local      |

Although I would probably **not** put every application service here. Discovery should primarily find **infrastructure services**, especially things needed before the normal directory system is available.

Otherwise it turns into an enormous LAN-wide service advertisement mechanism.

---

# Router discovery becomes just service #1

Startup:

```
```

```
Hub:
    YOU_ARE_ATTACHMENT 23
```

Host:

```
```

```
DISCOVER
    type = ROUTER
```

Router responds directly:

```
```

```
DISCOVER_REPLY
    type = ROUTER

    prefix          = 4.27
    assigned_node   = 17
    router_address  = 4.27.1
    directory       = 4.27.2
```

Potentially the router can assign the address in that same reply.

So:

```
```

```
plug cable
    ↓
attachment 23
    ↓
DISCOVER ROUTER        [one local broadcast]
    ↓
direct response
    ↓
GNet address 4.27.17
```

Very small.

---

# Then the directory becomes the second-level discovery mechanism

This is where I would avoid making GNet Discovery too ambitious.

Once a machine knows:

```
```

```
directory = 4.27.2
```

it can ask:

```
```

```
LOOKUP
    "accounting-printer"
```

or:

```
```

```
LOOKUP
    service = FILE
    organization = ENGINEERING
```

without broadcasting.

So the hierarchy becomes:

```
```

```
             GNet Discovery
          link-local / broadcast
                   │
          ┌────────┴────────┐
          │                 │
       ROUTER           DIRECTORY
          │                 │
          │                 └───── discovers arbitrary services
          │
          └──── establishes routed networking
```

That is much more scalable.

---

## Generic discovery is particularly valuable for diskless machines

Consider a diskless Telos workstation.

It comes online knowing essentially nothing.

It can do:

```
```

```
DISCOVER ROUTER
        ↓
network address acquired

DISCOVER BOOT
        ↓
boot server acquired

DISCOVER DIRECTORY
        ↓
directory acquired
```

Or the router may tell it where the directory/configuration service is, reducing this to one broadcast.

Then:

```
```

```
BOOT_REQUEST
    machine_type = TELOS
    host_id      = ...
```

Everything else is directed traffic.

This makes truly zero-configuration boot practical.

---

# Discovery should support multiple replies

That's important.

Suppose:

```
```

```
DISCOVER
    type = ROUTER
```

and there are:

```
```

```
Router A
Router B
```

Both should be able to respond.

Likewise:

```
```

```
DISCOVER
    type = DIRECTORY
```

might return:

```
```

```
Directory A
Directory B
Directory C
```

The request should therefore include a transaction number:

```
```

```
DISCOVER
    transaction = 0x42
    type        = DIRECTORY
```

and replies:

```
```

```
DISCOVER_REPLY
    transaction = 0x42
    type        = DIRECTORY
    priority    = 10
```

Host waits a very short period and chooses one.

This gives us redundancy without changing the protocol.

---

# Add `class` and a tiny qualifier field

I wouldn't make the request just:

```
```

```
DISCOVER 7
```

Give it enough extensibility to survive.

Perhaps:

```
```

```
DISCOVER

    version          0
    transaction      8 bits
    service          8 bits
    qualifier        16 bits
```

The qualifier could mean something specific to each service.

For example:

```
```

```
DISCOVER ROUTER
    qualifier = normal-network
```

versus perhaps:

```
```

```
DISCOVER ROUTER
    qualifier = management-network
```

Or:

```
```

```
DISCOVER BOOT
    qualifier = architecture RAX
```

But keep the generic header tiny.

Service-specific payload follows.

---

# I would probably use 16-bit service numbers

Eight bits looks attractive, but service numbers cost almost nothing.

A discovery packet might be:

```
```

```
+0   opcode          8
+1   version         8
+2   transaction     16
+4   service_type    16
+6   payload_length  16
+8   service-specific data...
```

That's only an 8-byte common header.

Then we can reserve:

```
```

```
0x0000            invalid
0x0001–0x00FF     core GNet
0x0100–0x0FFF     DEC/GNet registered
0x1000–0x7FFF     public registered
0x8000–0xFFFF     private/experimental
```

Given that we're designing something intended to last decades, **16 bits is worth it**.

---

# One especially useful service: `CONFIG`

I would have:

```
```

```
ROUTER
DIRECTORY
CONFIG
BOOT
TIME
```

as the foundational set.

`CONFIG` can answer questions such as:

```
```

```
machine name
directory server
time server
boot server
default filesystem
printer defaults
software repository
site parameters
```

Then the router itself remains narrow.

The router discovery reply doesn't become a giant DHCP-like option bag.

Instead:

```
```

```
DISCOVER ROUTER
        ↓
address + router + config-service

CONFIG_REQUEST
        ↓
everything else
```

That separation is architecturally cleaner.

---

## And this reinforces the dumb-hub principle

The complete division becomes:

```
```

```
HUB
    physical attachment
    request/grant
    electrical fanout

GNET DISCOVERY
    find local infrastructure

ROUTER
    assign network locator
    route packets

DIRECTORY
    find named machines/services

CONFIG
    configure systems
```

The hub has **zero knowledge of any discovery service number**. A `DISCOVER` frame is just another frame as far as it is concerned.

That is probably the right GNet design: **one
&#x20;universal, link-local discovery primitive with registered service&#x20;
numbers, used only for finding foundational infrastructure; once the&#x20;
directory is found, ordinary service discovery becomes directed rather&#x20;
than broadcast.**

as DEC we want to connect terminals to minicomputers. Increasingly our terminals should be connected with gnet. So booting from rom and then asking for a terminal server seem reasonable.  But arguably we should ask for a router first, then a directory and that might then configure the terminal, instead of just broadcasting for a terminal server.  But maybe you could first check directories, if not there then check local terminal servers.  ideally terminals also have built in security, like a smart card. And when we request access to a terminal server we can transmit our idenity for login based on a pluged in card of some kind.  arguably all of this should potentially operate on a higher levle, not with the link local. But then you can't just broadcast into the internet and ask for directories anymore.   whats the right balance between local low level and higher level interent stuf .  How did terminals usually do that sort of thing?

Yes. I would **not** make the terminal's normal operation depend on `DISCOVER TERMINAL_SERVER` broadcasts. The better architecture is a two-level system:

> **link-local discovery bootstraps the GNet infrastructure; the routed directory discovers actual services.**

That gives you the plug-and-play behavior you want without turning the entire GNet internet into a broadcast discovery domain.

## Historically, terminals were much simpler

A VT100-class DEC terminal normally&#x20;
had no concept of finding a computer. It was physically attached to a&#x20;
host serial port through EIA/RS-232 or optionally a 20 mA current loop;&#x20;
the host knew that physical terminal line and ran the login/session&#x20;
machinery behind it. 

IBM took a more centralized version&#x20;
of the same idea. 3270 terminals sat behind control units, and IBM&#x20;
communications could use polled lines: the host/controller&#x20;
infrastructure knew which terminal was at which attachment. The terminal
&#x20;itself was not discovering network services. 

DEC eventually moved very close to what you are proposing with **DECserver + LAT**:
&#x20;terminals connected to a terminal server, while LAT let that server&#x20;
connect users logically to service nodes over a LAN. LAT was&#x20;
deliberately LAN-local rather than routed, and DEC systems advertised&#x20;
LAT services on the LAN. 

Your GNet design can take the useful part of LAT **several years earlier**, while fixing the big limitation: service discovery should be capable of reaching the whole routed GNet network.

---

# I would define three levels

```
```

```
LEVEL 0        Physical/link
               ----------------
               attachment number
               request/grant
               local broadcast

LEVEL 1        GNet bootstrap
               ----------------
               DISCOVER ROUTER
               DISCOVER DIRECTORY
               network-address assignment

LEVEL 2        GNet services
               ----------------
               directory lookup
               terminal/session service
               boot
               file service
               authentication
               printing
               etc.
```

Only Levels 0–1 use local broadcast discovery.

Level 2 normally does **not**.

---

# Terminal startup

Consider a ROM-based GNet VT terminal.

It boots:

```
```

```
GNet VT
   │
   │ physical link comes up
   ▼
Hub says:
    ATTACHMENT = 23
```

Then:

```
```

```
DISCOVER ROUTER
```

This is a local broadcast.

Router replies directly:

```
```

```
NETWORK_ASSIGN

    address     = 12.8.17
    router      = 12.8.1
    directory   = 12.8.2
```

Now the terminal has joined the actual GNet internet.

At this point, **stop doing broadcasts**.

---

# Then ask the directory

The terminal sends a normal routed request:

```
```

```
12.8.17 → 12.8.2

DIRECTORY_QUERY
    service = TERMINAL
```

The directory can answer:

```
```

```
TERMINAL services:

    Corporate-VMS      12.14.7
    Engineering-RSTS   12.8.35
    Accounting         19.2.6
    Development        7.81.12
```

Notice that some of these can be:

- in the room;
- elsewhere in the building;
- elsewhere on campus;
- across the country.

The terminal doesn't care.

That's the crucial improvement over something like LAT.

---

## The directory can also configure the terminal

I would go further than merely returning addresses.

The directory knows:

```
```

```
terminal identity
user identity
location/network
organization
available services
administrative policy
```

So after boot:

```
```

```
DIRECTORY_LOGIN / CONFIGURE

terminal = VT-GNET-18372
identity = Alice
```

Directory might answer:

```
```

```
PROFILE

default-service = ENGINEERING
allowed-services:
    ENGINEERING
    CORPORATE
    DOCUMENTATION

printer = ENG-3F-PRINT
home-service = USERS-NORTH
time-service = ...
```

The user's terminal could therefore present:

```
```

```
Welcome Alice

1  Engineering
2  Corporate VMS
3  Documentation
4  Other...
```

That is vastly more sophisticated than a fixed terminal-server connection.

---

# Where `DISCOVER TERMINAL` is still useful

I would **keep the service number**, but make it fallback/local-emergency functionality.

Startup logic:

```
```

```
DISCOVER ROUTER
       │
       ├─ success
       │      ↓
       │  acquire GNet address
       │      ↓
       │  contact DIRECTORY
       │      ↓
       │  find TERMINAL service through directory
       │
       └─ failure
              ↓
       DISCOVER DIRECTORY locally
              │
              ├─ success → use it
              │
              └─ failure
                     ↓
              DISCOVER TERMINAL locally
```

That gives extremely good degraded operation.

For example:

```
```

```
WAN/router is broken
```

but the office still contains:

```
```

```
GNet hub
local PDP-11
local terminal service
20 terminals
```

They can still operate locally.

That's valuable.

---

# I would actually reverse the first two probes slightly

There are two legitimate strategies:

### Router-first

```
```

```
DISCOVER ROUTER
      ↓
network configuration
      ↓
directory supplied by router
```

This minimizes broadcasts and is the cleanest normal boot path.

### Directory-first

```
```

```
DISCOVER DIRECTORY
      ↓
perhaps directory supplies everything
```

But that makes the directory responsible for low-level network bootstrap.

I don't like that coupling.

**Router first should be normative.**

The router reply should include:

```
```

```
your GNet address
default router
directory address
```

No separate `DISCOVER DIRECTORY` is necessary in the normal case.

So most terminals generate exactly **one broadcast when they boot**.

---

# Smart-card authentication fits extremely well here

And critically, authentication belongs at **Level 2**, not in GNet link discovery.

The card shouldn't determine:

```
```

```
GNet address
hub attachment
router
```

Those identify the machine/network attachment, not the human.

Instead:

```
```

```
Terminal
   │
   ├── terminal identity
   │
   └── Smart Card
          │
          └── USER identity + secret
```

Then when the user wants a session:

```
```

```
Terminal → Authentication/Directory Service

LOGIN_BEGIN
    card_identity = NICK-14278
```

Server returns:

```
```

```
CHALLENGE
    0x83A17C...
```

Card computes something based on its secret:

```
```

```
response = AUTH(secret, challenge)
```

and the terminal sends:

```
```

```
AUTH_RESPONSE
```

The secret itself **never leaves the card**.

That is the architecture I would aim for.

Microprocessor smart-card&#x20;
technology was appearing right at the end of the 1970s: Michel&#x20;
Ugon/Bull's microprocessor-card work dates to 1977–78, so a DEC program&#x20;
started around this period is technologically aggressive but credible.&#x20;
&#x20;And DES became a U.S. federal encryption standard in 1977, giving DEC a
&#x20;contemporary standardized cryptographic primitive to build around. 

For the very first cheap terminals, DEC could make the reader **optional** while designing the ROM/session protocol around authenticated credentials from the beginning.

---

# Don't send the card credential to the terminal server as proof

This is one change I would make to your proposal.

Rather than:

```
```

```
CARD
  ↓
Terminal
  ↓
Terminal Server

"Here is Nick's credential."
```

prefer:

```
```

```
                  Authentication
                      Service
                    /         \
                   /           \
              Terminal       Host/service
```

The authentication service verifies the card and creates a short-lived session credential:

```
```

```
SESSION_TICKET

user       = Nick
service    = Engineering-VMS
expires    = ...
sessionkey = ...
signature/MAC = ...
```

Terminal then gives that **ticket** to the chosen terminal service.

The actual permanent card secret is never revealed to:

- terminal servers;
- minicomputers;
- routers;
- hubs.

That becomes especially valuable once GNet spans an entire corporation.

---

# And "terminal server" starts becoming the wrong abstraction

Since these terminals contain GNet NICs themselves, they don't really need a historical terminal server whose job is:

```
```

```
serial terminals
      ↓
terminal server
      ↓
network
```

Your terminal is already:

```
```

```
terminal
   ↓
GNet
```

What we actually need is a **Terminal Session Service** on the host.

Conceptually:

```
```

```
GNet VT
    │
    │ GNet terminal/session protocol
    ▼
┌────────────────────┐
│ RAX / PDP-11 / VAX │
│ Terminal Service   │
│                    │
│ session 1 → TTY 34 │
│ session 2 → TTY 35 │
└────────────────────┘
```

For an older PDP-11 that knows nothing about GNet, you can have a box that really is a terminal server:

```
```

```
GNet terminals
      │
      ▼
GNet Terminal Gateway
      │
      ├── serial → PDP-11
      ├── serial → PDP-10
      └── serial → other host
```

But newer DEC operating systems should implement the terminal-session protocol directly.

That removes an entire piece of hardware.

---

# This gives a very strong migration strategy

### 1978 legacy installation

```
```

```
GNet VT
   │
GNet
   │
Terminal Gateway
   │
RS-232
   │
PDP-11
```

### GNet-aware PDP-11/RAX

```
```

```
GNet VT
   │
GNet
   │
PDP-11/RAX
```

### Large corporate network

```
```

```
                    GNet Directory
                         │
        ┌────────────────┼────────────────┐
        │                │                │
      RAX              PDP-11          Mainframe
   terminal svc      terminal svc       gateway
        ▲                ▲                ▲
        └────────────────┼────────────────┘
                         │
                      routers
                         │
                       hub
                         │
                      GNet VT
                         │
                     smart card
```

The user inserts the card and can potentially access **any authorized computing service in the whole GNet organization**, not just whichever computer happens to have a wire going to that desk.

## So I would make this the architectural rule

**Link-local GNet Discovery answers only:**

```
```

```
How do I join this network?
Where is my bootstrap infrastructure?
```

**The routed GNet Directory answers:**

```
```

```
What services exist?
Which ones may I use?
Where are they?
```

**Authentication answers:**

```
```

```
Who is this person?
What are they authorized to do?
```

And **Terminal Session Protocol answers:**

```
```

```
How do I create and operate an interactive session?
```

That
&#x20;separation is considerably better than either extreme—neither&#x20;
broadcasting every service request locally like later LAT, nor requiring
&#x20;a completely configured routed internet before a ROM terminal can do&#x20;
anything. The common case needs only **one local broadcast (****`DISCOVER ROUTER`****) at boot**, after which essentially the whole terminal experience can be routed, directory-driven, and authenticated.
