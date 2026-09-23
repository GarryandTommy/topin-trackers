# Topin protocol overview

This section documents protocol behaviour observed on real ZX909 hardware and the parts that have been reproduced or implemented during the GarryAndTommy interoperability work.

> **Use at your own risk.** This is reverse-engineered interoperability documentation based on tested hardware and observed traffic. Other Topin devices or firmware versions may use different message layouts or behaviour.

Tested reference hardware:

- Verdant Trace VT02 / Topin ZX909
- firmware `ZX909_EU_V1.3.7_2025-11-17_09-43-24`
- Topin protocol over TCP

## Evidence levels

Protocol findings in this project use four evidence labels:

- **Confirmed** — reproduced on real hardware with a directly observable or controlled result.
- **Observed** — seen in real device/software traffic, but its complete meaning has not yet been established.
- **Unknown / Hypothesis** — a possible interpretation that still requires evidence.
- **Rejected** — investigated interpretation or implementation that did not hold up in testing.

Seeing a packet in a log is not enough by itself to classify its meaning as confirmed.

## General frame shape

Many frames observed from the tested ZX909 use this outer form:

```text
78 78 ... 0D 0A
```

`78 78` acts as the observed frame prefix and `0D 0A` as the observed frame ending.

The bytes between them depend on the message type and payload.

This apparent delimiter structure must not be confused with safe TCP stream framing. Multiple complete Topin messages can arrive in a single TCP read, and treating CRLF alone as a generic frame delimiter was investigated and rejected.

See [Framing and TCP stream handling](framing.md).

## Message types documented in this project

| Message | Area | Evidence | Documentation |
| --- | --- | --- | --- |
| `0x11` | Offline/backfilled GPS | **Observed** | [0x11 offline GPS](0x11-offline-gps.md) |
| `0x1A` | LTE LBS / Wi-Fi related position data | **Confirmed** for tested LTE cell field layout | [0x1A LTE LBS](0x1a-lte-lbs.md) |
| `0x1B` | Proposed offline LTE LBS variant | **Hypothesis / test-covered implementation** | [0x1B offline LBS](0x1b-offline-lbs.md) |
| `0x64` | LED control downlink | **Confirmed** | [Confirmed commands](../commands/confirmed-commands.md) |
| `0x97` | Moving tracking interval / sleep | **Confirmed** | [0x97 tracking interval](0x97-tracking-interval.md) |
| `0xC1` | Speaker control downlink | **Confirmed** | [Confirmed commands](../commands/confirmed-commands.md) |

The table is intentionally limited to message types for which this project currently has useful evidence. It is not intended to be a complete Topin protocol specification.

## LTE cell identifiers

A central finding concerns the LTE cell information carried by the tested ZX909.

For the relevant `0x1A` layout, the device uses:

```text
TAC     4 bytes
ECI     4 bytes
Signal  1 byte
```

The ECI is treated as an unsigned 32-bit value.

An older Traccar interpretation used 16-bit reads for fields in this area. On the tested ZX909 this caused the decoder to become misaligned and could lead to incorrect position handling.

The corresponding Traccar work uses:

```text
readInt()
readUnsignedInt()
readUnsignedByte()
```

for these fields.

See [0x1A LTE LBS](0x1a-lte-lbs.md) for the packet-level evidence.

## `0x1A` and `0x1B`

The tested hardware exposed a newer LTE-oriented LBS cell layout that was not correctly represented by the existing decoder path.

`0x1A` has been reproduced with real ZX909 traffic and was used to build a regression test and decoder correction.

`0x1B` was implemented as a proposed offline LTE LBS variant by applying the LTE field structure established for `0x1A` to the corresponding offline decoder path. Decoder support and a regression test were added locally, but no real ZX909 `0x1B` frame has been observed to validate that assumption. It is therefore classified as **Unknown / Hypothesis**, not Observed or Confirmed.

## Offline data and backfill

The ZX909 can buffer position information and later send stored data after connectivity returns.

This behaviour has been observed on both tested trackers. It means that packet arrival time at the server is not necessarily the time at which the underlying position was recorded.

`0x11` traffic is relevant to the observed offline/backfill behaviour and is documented separately.

See [0x11 offline GPS](0x11-offline-gps.md).

## Downlink commands

Some device functions can be controlled with binary Topin frames sent over the existing TCP connection.

Confirmed examples include:

- LED on/off (`0x64`)
- speaker on/off (`0xC1`)
- moving tracking interval (`0x97`)
- sleep through the special `0x97` value `FFFF`

The maintained command values and their evidence are kept in [Confirmed device commands](../commands/confirmed-commands.md).

Remote power-off using `0x48` with value `02` has also been reproduced on the tested hardware, but its behaviour is state-dependent. With an active tracker, shutdown was reliable when the sent frame was immediately echoed by the device; in sleep or power-saving state, the same downlink could remain ineffective. The immediate echo is an observed practical success indicator, not a documented protocol acknowledgement.

## TCP is a byte stream

A single TCP read is not guaranteed to correspond to a single Topin message.

Real ZX909 traffic showed multiple complete messages arriving together in one TCP buffer. An initial CRLF-based decoder approach was therefore investigated, but it is not considered a safe general solution.

The intended future direction is a **protocol-aware binary frame decoder** that determines frame boundaries from the Topin message structure rather than assuming that TCP read boundaries or CRLF alone define messages.

See [Framing and TCP stream handling](framing.md).

## Cloud interoperability observations

During controlled interoperability research, communication involving 365GPS exposed additional packets and command behaviour.

Some responses associated with `0x1A`, including `FF1A` / `001A` timestamp-related packets, were observed but their semantics remain unresolved.

These observations are not promoted to confirmed protocol definitions merely because they appeared in cloud traffic.

See [365GPS interoperability research](../research/365gps-interoperability.md).

## Scope and limitations

This documentation is based primarily on two Verdant Trace VT02 devices associated with the ZX909 platform and the reference firmware listed above.

It should not be assumed that:

- every Topin tracker uses the same packet structure;
- every ZX909 firmware revision behaves identically;
- a message type has identical semantics across unrelated Topin device families;
- an observed cloud response is required for independent operation.

Where evidence is incomplete, the individual protocol pages state that explicitly.

---

This overview is a map of the protocol work, not a substitute for the packet-level pages. Those pages contain the evidence, byte layouts, tests and unresolved questions behind each finding.
