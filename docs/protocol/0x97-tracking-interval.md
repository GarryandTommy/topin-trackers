# `0x97` tracking interval and sleep

This page documents the binary Topin `0x97` downlink behaviour reproduced on the tested ZX909 hardware.

> **Use at your own risk.** `0x97` changes tracker behaviour. In particular, `FFFF` is a special sleep value and must not be treated as an ordinary interval. Verify the target device and command before sending it.

Tested reference hardware:

- Verdant Trace VT02 / Topin ZX909
- firmware `ZX909_EU_V1.3.7_2025-11-17_09-43-24`
- Topin protocol over TCP

## Confirmed frame format

For normal tested interval values:

```text
78 78 03 97 XX XX 0D 0A
```

`XX XX` is the moving tracking interval in **seconds**, encoded as an unsigned 16-bit big-endian value.

Examples reproduced during the project:

| Interval | Value | Complete frame |
| --- | --- | --- |
| 10 s | `000A` | `78780397000A0D0A` |
| 30 s | `001E` | `78780397001E0D0A` |
| 10 min / 600 s | `0258` | `7878039702580D0A` |
| 37 min / 2220 s | `08AC` | `7878039708AC0D0A` |
| 42 min / 2520 s | `09D8` | `7878039709D80D0A` |

## What `0x97` changes

The tracker exposes a TIMER configuration with two values:

```text
TIMER,X,Y#
```

For the tested device:

- `X` is the moving tracking interval;
- `Y` is the idle interval.

The binary `0x97` frame has been shown to change **TIMER X**.

A binary setter for TIMER Y has not been established.

## Controlled 30-second experiment

The meaning of the two-byte `0x97` value was verified with a controlled before/after test.

Initial TIMER state:

```text
10,600
```

An ASCII tracker command was first sent as raw data over the existing Topin TCP connection:

```text
TIMER,30,600#
```

It did not change the TIMER setting.

The binary frame was then sent:

```text
78780397001E0D0A
```

A subsequent TIMER readback showed:

```text
30,600
```

This demonstrated two things:

1. the binary value `001E` set TIMER X to 30 seconds;
2. TIMER Y remained at 600 seconds.

The result was also consistent with subsequent tracker behaviour.

## Big-endian interval encoding

The tested normal values follow straightforward unsigned 16-bit big-endian encoding.

For example:

```text
30 decimal = 0x001E
```

so the frame becomes:

```text
78 78 03 97 00 1E 0D 0A
```

Likewise:

```text
600 decimal = 0x0258
```

produces:

```text
78 78 03 97 02 58 0D 0A
```

This relationship was tested with multiple interval values rather than inferred from a single example.

## Special value `FFFF`

The frame:

```text
78780397FFFF0D0A
```

has been reproduced as a **sleep command**.

Therefore:

```text
FFFF
```

must not be interpreted as an ordinary interval of 65,535 seconds.

It is a protocol-level special value in the observed `0x97` behaviour.

## Binary Topin command versus TIMER command

Three command layers should remain distinct:

```text
TIMER,X,Y#                  tracker command / SMS-style syntax
78780397XXXX0D0A            binary Topin downlink
AT...                       cellular modem command
```

They are not interchangeable merely because they can affect related device settings.

In particular, the controlled experiment showed that sending the ASCII TIMER command as raw Topin TCP data did not perform the same operation as sending the binary `0x97` frame.

## Current everyday setting

During project testing, a moving interval of 30 seconds with an idle interval of 600 seconds was used:

```text
TIMER 30,600
```

The confirmed binary command:

```text
78780397001E0D0A
```

sets the moving side (`X`) to 30 seconds. It does not establish how to set the idle side (`Y`) through a binary Topin downlink.

## Evidence status

| Finding | Status |
| --- | --- |
| `0x97` controls TIMER X / moving interval | **Confirmed** |
| Normal value is seconds | **Confirmed** |
| Normal value uses unsigned 16-bit big-endian encoding | **Confirmed** |
| `001E` sets 30 seconds | **Confirmed** |
| Multiple other interval values follow the same encoding | **Confirmed** |
| TIMER Y remains unchanged by the tested `0x97` setter | **Confirmed in controlled test** |
| `FFFF` invokes sleep behaviour | **Confirmed** |
| `FFFF` means 65,535-second interval | **Rejected** |
| Binary setter for TIMER Y | **Unknown** |

## Related documentation

The maintained ready-to-send frames are listed in [Confirmed device commands](../commands/confirmed-commands.md).

For the practical Traccar procedure, see [Sending commands with Traccar](../commands/sending-with-traccar.md).

The confirmed SMS-style TIMER syntax is summarized in [Confirmed device commands](../commands/confirmed-commands.md); it remains separate from the binary protocol described here.

---

The central finding is reproducible: **for ordinary values, `0x97` carries the moving interval as a two-byte big-endian number of seconds; `FFFF` is a special sleep value.**
