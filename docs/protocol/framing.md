# Framing and TCP stream handling

This page documents a transport-layer issue observed with the tested ZX909 hardware: **one TCP read does not necessarily contain exactly one Topin message**.

> **Use at your own risk.** The framing discussion here is based on observed ZX909 traffic and local Traccar experiments. A framing change affects how every message on a connection is delivered to the protocol decoder, so it should be tested against representative traffic before deployment.

Tested reference hardware:

- Verdant Trace VT02 / Topin ZX909
- firmware `ZX909_EU_V1.3.7_2025-11-17_09-43-24`
- Topin protocol over TCP

## The observed problem

During real tracker operation, multiple complete Topin messages were observed arriving together in a single TCP buffer.

Conceptually, a read can therefore look like:

```text
[Topin frame 1][Topin frame 2][Topin frame 3]
```

rather than:

```text
TCP read 1 -> Topin frame 1
TCP read 2 -> Topin frame 2
TCP read 3 -> Topin frame 3
```

This is normal TCP behaviour. TCP provides an ordered byte stream; application-message boundaries are not preserved by the transport.

A protocol decoder must therefore identify Topin frame boundaries itself.

## Why this mattered in Traccar

Without suitable framing, a decoder can receive several concatenated messages as one buffer.

In the ZX909 testing, this exposed a case where only the first complete message in such a buffer was processed as intended. Subsequent complete messages in the same TCP read could be missed.

A regression test was created with three concatenated real-style Topin messages to reproduce the issue.

## The first attempted fix: CRLF framing

Observed ZX909 messages commonly end with:

```text
0D 0A
```

which corresponds to CRLF.

This suggested an initially simple solution: split the TCP stream at every CRLF terminator before passing frames to the Topin protocol decoder.

A local implementation using a character-delimiter frame decoder successfully addressed the reproduced concatenated-message case.

However, that does **not** make CRLF a sufficiently proven general framing rule for the Topin protocol.

## Why the CRLF approach was rejected

The CRLF solution was not retained as the intended upstream fix.

The important distinction is:

```text
Observed frames end in 0D 0A
```

does not automatically prove:

```text
Every occurrence of 0D 0A safely defines a Topin frame boundary
```

A delimiter-only decoder assumes that the delimiter cannot occur in a way that would incorrectly split a valid binary frame and that all relevant Topin variants follow the same framing rule.

The evidence available to this project is not sufficient to make that protocol-wide assumption.

For that reason, the earlier CRLF framing patch is classified as **Rejected** for upstream use.

## What the rejected patch still taught us

Rejecting the implementation did not invalidate the underlying bug.

Two separate findings should be kept apart:

1. **Confirmed observation:** multiple complete Topin messages can arrive in one TCP read.
2. **Rejected solution:** splitting all Topin traffic solely on CRLF is not sufficiently justified as a general protocol framing strategy.

The regression case remains useful evidence for the first finding.

## Preferred direction: protocol-aware binary framing

The safer future approach is a dedicated **Topin frame decoder** that understands enough of the binary protocol structure to determine when a complete message has been received.

Conceptually:

```text
TCP byte stream
      |
      v
TopinFrameDecoder
      |
      +--> complete frame 1
      +--> complete frame 2
      +--> complete frame 3
      |
      v
TopinProtocolDecoder
```

Such a decoder should derive frame boundaries from protocol structure rather than assuming that one TCP read equals one message or that CRLF alone is always sufficient.

The exact implementation still requires protocol evidence covering the relevant Topin frame variants.

## Evidence status

**Confirmed / reproduced:**

- the tested ZX909 uses Topin over TCP;
- multiple complete Topin messages can arrive in a single TCP read;
- the condition can be reproduced with concatenated messages;
- handling only the first message can lose subsequent messages from that buffer.

**Rejected as a general solution:**

- a generic CRLF-only frame decoder for Topin.

**Still open:**

- the complete protocol-aware framing rules needed for a robust `TopinFrameDecoder`;
- whether all relevant Topin device families use identical outer framing;
- which length or structural fields can safely be used across the message variants supported by Traccar.

## Implications for packet analysis

When inspecting raw Traccar logs, do not assume that one logged TCP payload represents one protocol message.

A payload may contain:

```text
7878...0D0A7878...0D0A7878...0D0A
```

and must be interpreted as multiple candidate frames rather than one unusually long message.

Likewise, TCP itself is free to split a protocol message across reads. A robust stream decoder should therefore handle both coalescing and fragmentation.

## Upstream status

The original local CRLF framing patch is intentionally **not** part of the maintained ZX909 patch set.

The maintained local work currently consists of the LTE/LBS decoding changes for `0x1A` and `0x1B`.

A future framing contribution should be based on protocol-aware binary framing and supported by representative regression tests and, where possible, manufacturer protocol documentation.

---

The important result of this investigation is not “Topin uses CRLF framing.” It is narrower and better supported: **TCP read boundaries cannot be treated as Topin message boundaries, and the tested ZX909 can deliver multiple complete messages in one read.**
