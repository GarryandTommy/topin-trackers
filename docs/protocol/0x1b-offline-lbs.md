# `0x1B` proposed offline LTE LBS decoding

This page documents a **proposed** ZX909 interpretation and local Traccar implementation for message type `0x1B`.

> **Use at your own risk.** Unlike the `0x1A` LTE layout, this `0x1B` interpretation has **not been validated with a real `0x1B` frame from the tested ZX909 hardware**. It is an implementation hypothesis backed by code and a regression test, not a confirmed protocol finding.

Tested reference platform for the related work:

- Verdant Trace VT02 / Topin ZX909
- firmware `ZX909_EU_V1.3.7_2025-11-17_09-43-24`
- Topin protocol over TCP

## Evidence status

**Implemented / awaiting live validation**

The assumed `0x1B` packet layout remains **Unknown / Hypothesis** until validated against a genuine ZX909 `0x1B` frame.

No real ZX909 `0x1B` packet has been observed in the project data so far.

The implementation should therefore not be cited as proof that the ZX909 sends `0x1B`, nor that a real `0x1B` packet necessarily has the assumed layout.

## Where the idea came from

The `0x1B` work did not originate from a captured ZX909 `0x1B` packet.

It followed from two pieces of information already present during the decoder work:

1. the existing Topin decoder contained a corresponding offline LBS/Wi-Fi decoding path;
2. real ZX909 `0x1A` traffic demonstrated that the newer LTE cell representation uses wider TAC and ECI fields than the affected decoder logic expected.

The local implementation therefore explored the corresponding offline variant using the same LTE cell-field interpretation established for `0x1A`.

That is a reasonable implementation hypothesis, but it remains a hypothesis until real hardware traffic validates it.

## Proposed message type

The local decoder work introduced:

```java
private static final int MSG_LBS_WIFI_OFFLINE_2 = 0x1B;
```

and handled it alongside the related LTE LBS decoding logic.

The proposed LTE cell fields are:

```text
TAC     4 bytes
ECI     4 bytes, unsigned
Signal  1 byte
```

These widths are **confirmed for the tested `0x1A` layout**, but only **assumed for `0x1B`**.

See [`0x1A` LTE LBS decoding](0x1a-lte-lbs.md) for the hardware evidence behind that field structure.

## Local Traccar implementation

A separate local patch added `0x1B` support and a regression test.

Original project commit:

```text
dfa73c4  Add offline LTE cell decoding for Topin (0x1B)
```

When cherry-picked onto the Traccar 6.15.3 project branch, it became:

```text
58b0368
```

The patch remains part of the local ZX909 build so that the proposed decoder path can be tested if suitable traffic is encountered.

## What the regression test proves

The regression test proves that the **implemented decoder behaves as intended for the test packet supplied to it**.

It does not prove that:

- the test packet was captured from a real ZX909;
- the tested ZX909 firmware actually emits message type `0x1B`;
- the proposed byte layout matches a real device-generated `0x1B` packet;
- other Topin models use the same structure.

This distinction is important: a passing test can verify code behaviour without validating the underlying protocol assumption.

## What would confirm `0x1B`

A useful hardware validation would require a genuine `0x1B` message captured from a ZX909 under known conditions.

Ideally, validation would include:

1. the complete raw frame;
2. evidence that it was transmitted by the tracker;
3. the circumstances under which it appeared;
4. independently known or verifiable cellular identifiers where possible;
5. successful decoding using the proposed field layout.

If such evidence becomes available, the status of this page can be upgraded from **Unknown / Hypothesis** to the appropriate evidence level.

## Why the patch is still useful

Keeping the implementation does not require pretending that the protocol assumption is confirmed.

It provides:

- a concrete decoder candidate ready for real traffic;
- a regression test for the proposed interpretation;
- a reproducible starting point for future validation;
- a clear record of how the hypothesis was derived.

The evidence label prevents the implementation itself from being mistaken for protocol documentation.

## Upstream status

The `0x1B` change was not merged into upstream Traccar.

As with the related `0x1A` work, manufacturer protocol documentation would provide much stronger support for an upstream protocol change. For `0x1B`, the evidence is currently weaker because this project additionally lacks a genuine ZX909 packet validating the proposed message layout.

See [Upstream status](../traccar/upstream-status.md).

## Current status

| Claim | Status |
| --- | --- |
| ZX909 `0x1A` uses the documented wider LTE cell fields | **Confirmed** |
| Local Traccar code supports proposed message type `0x1B` | **Confirmed** |
| Regression test exists for the proposed `0x1B` implementation | **Confirmed** |
| Real ZX909 `0x1B` frame captured | **No** |
| ZX909 actually emits `0x1B` | **Unknown** |
| Real ZX909 `0x1B` uses the proposed LTE layout | **Unknown / Hypothesis** |

---

The key distinction is: **`0x1B` is implemented, but not yet validated as a ZX909 protocol finding.** The repository keeps the code and the hypothesis while making the missing hardware evidence explicit.
