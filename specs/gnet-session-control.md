# GNet Session Control (GSC)

Status: **DRAFT protocol; ACCEPTED signaling/media separation**

GSC is a SIP-like control protocol for arbitrary interactive sessions including terminal, voice, video, and collaborative services. Session servers participate in setup and policy; accepted media or data flows travel directly between endpoints through GNet routers.

## Message set

| Message | Purpose |
|---|---|
| REGISTER | associate a person or service name with current devices/endpoints |
| LOOKUP | resolve a service name, person, or telephone number |
| INVITE | request a new session |
| OFFER | describe media, codecs, encryption, capacity, and other resources |
| ALERT | report ringing or user notification |
| ACCEPT | accept and return media addresses/flow parameters |
| REJECT | refuse as busy, unavailable, unauthorized, or unsupported |
| CANCEL | withdraw an unanswered invitation |
| RELEASE | end an active session and release resources |
| UPDATE | change media, codec, keys, or reservations |
| TRANSFER | redirect or transfer a participant |
| KEEPALIVE | maintain registration or session soft state |

GSC negotiation may cover encoding, packet interval, GDP addresses, flow identifiers, bandwidth/delay needs, optional media, security, and charging policy.

## Voice and gateway behavior

For an internal GNet-to-GNet call, signaling may traverse district GSC services while voice packets do not. Admission control confirms capacity across the local link, neighbourhood access segment, district trunk, and inter-district route before acceptance.

External telephony terminates at a voice/PSTN gateway. The gateway maps GSC to external signaling, converts packet voice to or from 64-kbit/s PCM when required, and maps E.164 numbers through an Address Mapping (AM) database. E.164 numbers are directory/session-control data, not core GDP routing-table entries. SS7 remains an external gateway protocol.

On RELEASE, the system releases external trunks and GNet reservations, closes media security state, and finalizes accounting outside the forwarding path.

Call-control keys and media keys are separate. A district service may assist key establishment without receiving or decrypting internal media.
