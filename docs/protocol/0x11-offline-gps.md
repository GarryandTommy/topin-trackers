# `0x11` offline GPS and backfill

This page documents offline GPS/backfill behaviour observed on real ZX909 hardware.

> **Use at your own risk.** The interpretation here is based on observed traffic from the tested devices and firmware. Do not assume that every Topin device or firmware revision uses `0x11` identically.

Tested reference hardware:

- Verdant Trace VT02 / Topin ZX909
- firmware `ZX909_EU_V1.3.7_2025-11-17_09-43-24`
- Topin protocol over TCP

## What was observed

The tested ZX909 can retain position information while normal server delivery is interrupted and transmit stored positions after connectivity returns.

This creates a characteristic sequence:

```text
position recorded
      |
      | connectivity interrupted
      v
stored on tracker
      |
      | connection returns
      v
older position delivered to server
```

The server receive time can therefore be later than the timestamp contained in the position itself.

This behaviour has been observed on both tested trackers.

## `0x11` in the observed traffic

`0x11` messages appeared in traffic associated with offline/stored GPS positions and subsequent backfill.

For this project, `0x11` is therefore documented as:

**Observed — offline/backfilled GPS message**

This wording is deliberate. The project has real traffic showing the relationship between `0x11` and stored position delivery, but this page does not claim a complete manufacturer specification for every byte of the message.

## Why timestamps matter

Backfilled positions must not be interpreted solely by their arrival order.

For example:

```text
20:10   position A recorded
20:11   connection unavailable
20:12   position B recorded/stored
20:13   position C recorded/stored
20:14   connection restored
20:14   position B arrives
20:14   position C arrives
```

Positions B and C are not new 20:14 positions merely because the server received them then.

The device timestamp is essential for reconstructing the actual movement history.

## Gap, reconnect and burst

During field testing, a tracker could show a gap in live server traffic followed by a reconnect and a burst of stored data.

That pattern should not automatically be interpreted as missing GPS measurements.

It can instead represent:

1. positions being recorded by the tracker;
2. temporary loss or interruption of server connectivity;
3. local buffering;
4. later delivery of the stored positions.

Historical data showed this buffering/backfill behaviour on both test devices.

## What this does not prove

The observations do **not** establish that:

- every connectivity gap is backfilled;
- every missing server position was successfully stored on the tracker;
- `0x11` is the only message type involved in offline data;
- all Topin models use the same offline format;
- buffering capacity or retention limits are known;
- delayed delivery guarantees that the original GPS fix was accurate.

Those questions require separate evidence.

## Relationship to TCP framing

Backfill can cause several stored messages to be transmitted in quick succession.

This is one situation in which multiple complete Topin frames may appear together in a single TCP read.

The two findings are related operationally but should not be confused:

- **backfill** explains why a tracker may send several stored positions rapidly;
- **TCP coalescing** explains why several application frames may be presented together in one TCP buffer.

See [Framing and TCP stream handling](framing.md).

## Evidence handling

When analysing a suspected backfill sequence, retain at least:

- the raw Topin frame;
- the position/device timestamp;
- the server receive timestamp if available;
- connection/disconnection events around the sequence;
- the decoded coordinates and fix information.

Public examples should be sanitized so that they do not reveal private routes, home locations, device identifiers or SIM information.

A sanitized example is planned under:

[`examples/0x11-backfill.txt`](../../examples/0x11-backfill.txt)

## Current status

| Finding | Status |
| --- | --- |
| ZX909 can deliver stored positions after reconnect | **Observed** |
| Behaviour seen on both tested trackers | **Observed** |
| `0x11` associated with offline/backfilled GPS traffic | **Observed** |
| Complete byte-level `0x11` specification | **Not yet documented** |
| Buffer size / retention limits | **Unknown** |

---

The practical lesson is simple: **arrival time is not necessarily measurement time**. When a ZX909 reconnects and sends stored data, position timestamps must be preserved and interpreted as part of the movement history.
