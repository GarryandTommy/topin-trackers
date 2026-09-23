# `0x1A` LTE LBS decoding

This page documents the LTE cell layout observed in ZX909 `0x1A` messages and the corresponding Traccar decoder correction.

> **Use at your own risk.** This layout was reproduced with the tested ZX909 hardware and firmware. Do not assume that every Topin model or firmware revision uses the same `0x1A` structure.

Tested reference hardware:

- Verdant Trace VT02 / Topin ZX909
- firmware `ZX909_EU_V1.3.7_2025-11-17_09-43-24`
- Topin protocol over TCP

## Summary

The tested ZX909 uses the following LTE cell-field layout in the relevant `0x1A` message:

```text
TAC     4 bytes
ECI     4 bytes
Signal  1 byte
```

The ECI is an **unsigned 32-bit value**.

This differs from the field widths previously assumed by the affected Traccar decoder path. Reading the LTE fields as 16-bit values caused the remaining payload to become misaligned.

## Why the old decoding failed

A binary decoder must consume exactly the number of bytes actually present in each field.

If a four-byte field is read as only two bytes, the next read starts in the middle of that field:

```text
Actual payload:

[TAC: 4 bytes] [ECI: 4 bytes] [Signal: 1 byte]
 ^^^^^^^^^^^^   ^^^^^^^^^^^^   ^^^^^^^^^^^^^^

Incorrect 16-bit interpretation:

[TAC: 2] [next read starts here ...]
```

Once this happens, later values are decoded from the wrong byte positions.

For the tested ZX909 LTE message, this was the source of the decoding problem.

## Correct field reads

The local Traccar correction uses:

```java
network.addCellTower(CellTower.from(
        buf.readUnsignedShort(),
        buf.readUnsignedByte(),
        buf.readInt(),
        buf.readUnsignedInt(),
        buf.readUnsignedByte()));
```

The relevant part is:

```text
readInt()          -> TAC, 4 bytes
readUnsignedInt()  -> ECI, 4 bytes
readUnsignedByte() -> signal, 1 byte
```

The preceding values in the `CellTower.from(...)` call are the mobile country and network identifiers handled by the surrounding decoder logic.

## TAC

For this observed layout, the Tracking Area Code is read as:

```java
buf.readInt()
```

That consumes four bytes.

The important finding here is the **field width in the ZX909 packet**. It should not be generalized into a claim that all LTE protocol representations encode TAC this way.

## ECI

The E-UTRAN Cell Identifier is read as:

```java
buf.readUnsignedInt()
```

This consumes four bytes while preserving the full unsigned 32-bit value.

Using a 16-bit read would both truncate the identifier and shift all subsequent parsing by two bytes.

## Signal

The signal field following the ECI is one byte:

```java
buf.readUnsignedByte()
```

With TAC and ECI consumed at their observed widths, this byte is read from the correct position.

## Independent modem cross-check

During testing, the LTE identifiers decoded from a real `0x1A` packet were cross-checked against cellular modem diagnostics from the same physical ZX909. The modem reported TAC `119f` and ECI `01e91105`; the corresponding packet contained `0000119f` and `01e91105`. This independently supports the interpretation of the four-byte fields as TAC and ECI.

## Regression test

The correction was accompanied by a regression test built from a real ZX909 packet.

The purpose of the test is not merely to verify that the decoder accepts a packet. It protects the byte alignment and resulting LTE cell interpretation against later regressions.

The maintained local patch corresponds to the original project commit:

```text
1d5eb6f  Fix LTE cell decoding for Topin LBS WiFi 2 (0x1A)
```

When applied to the Traccar 6.15.3 source branch, the cherry-picked commit became:

```text
ed5c046
```

The original commit identifier is retained in the project documentation because it identifies the reusable project patch independently of a particular Traccar release branch.

## Patch maintenance

The `0x1A` correction is one of the two patches intentionally retained for the local ZX909 Traccar build.

For a new Traccar release, the current workflow is:

```text
official Traccar release tag
        |
        v
new zx909-<version> branch
        |
        v
cherry-pick 1d5eb6f
        |
        v
run regression tests
        |
        v
build and field-test
```

The separate rejected CRLF framing commit is **not** part of this maintained patch set.

See [Traccar maintenance](../traccar/maintenance.md).

## Upstream status

The decoder correction was prepared as upstream-quality work with a regression test.

During upstream discussion, manufacturer protocol documentation was requested as supporting evidence for the packet layout. The project did not have suitable official documentation to provide, so the change was not merged upstream.

That does not invalidate the hardware observation or local regression test. It does mean that this repository should distinguish clearly between:

- behaviour reproduced on the tested hardware;
- local Traccar support;
- official manufacturer documentation;
- code accepted by upstream Traccar.

See [Upstream status](../traccar/upstream-status.md).

## Evidence status

| Finding | Status |
| --- | --- |
| Relevant ZX909 `0x1A` LTE TAC field occupies 4 bytes | **Confirmed** |
| Relevant ZX909 `0x1A` LTE ECI field occupies 4 bytes | **Confirmed** |
| ECI must be handled as unsigned 32-bit value | **Confirmed** |
| Signal field following ECI occupies 1 byte | **Confirmed** |
| Old 16-bit interpretation causes payload misalignment | **Confirmed** |
| Regression test with real ZX909 packet | **Confirmed** |
| Same layout applies to every Topin device/firmware | **Not established** |
| Manufacturer protocol documentation for this layout | **Not available to this project** |

## LBS is not geolocation by itself

Decoding TAC and ECI correctly gives Traccar usable cellular network information. It does not by itself turn those identifiers into latitude and longitude.

An external cell-geolocation provider can use the decoded network identifiers to estimate a position.

That part of the system is documented separately in [Traccar geolocation](../lbs/traccar-geolocation.md).

---

The central `0x1A` finding is therefore narrow and reproducible: **on the tested ZX909 packet layout, TAC is 4 bytes, ECI is an unsigned 4-byte value, and the following signal value is 1 byte. Reading those fields at smaller widths misaligns the remainder of the message.**
