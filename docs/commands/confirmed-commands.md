# Confirmed device commands

This page lists device commands that have been **reproduced on real ZX909 hardware** through our own Traccar connection.

> **Use at your own risk.** Commands can change persistent tracker behaviour and may behave differently on other hardware or firmware versions. Verify the command, expected effect and recovery path before sending it.

Tested reference firmware: `ZX909_EU_V1.3.7_2025-11-17_09-43-24`

Unless stated otherwise, the binary frames below are sent as **raw hexadecimal data** over the existing Topin TCP connection.

## Command reference

| Function | Raw hex | Status | Observed effect |
| --- | --- | --- | --- |
| LED off | `7878036406000D0A` | **Confirmed** | Device LED switches off |
| LED on | `7878036406010D0A` | **Confirmed** | Device LED switches on |
| Speaker off | `787803C100000D0A` | **Confirmed** | Speaker disabled |
| Speaker on | `787803C100010D0A` | **Confirmed** | Speaker enabled |
| Moving interval: 10 s | `78780397000A0D0A` | **Confirmed** | TIMER X set to 10 seconds |
| Moving interval: 30 s | `78780397001E0D0A` | **Confirmed** | TIMER X set to 30 seconds |
| Moving interval: 10 min | `7878039702580D0A` | **Confirmed** | TIMER X set to 600 seconds |
| Moving interval: 37 min | `7878039708AC0D0A` | **Confirmed** | TIMER X set to 2220 seconds |
| Moving interval: 42 min | `7878039709D80D0A` | **Confirmed** | TIMER X set to 2520 seconds |
| Sleep | `78780397FFFF0D0A` | **Confirmed** | Device enters the observed sleep mode |
| Remote power-off | `78780248020D0A` | **Confirmed, state-dependent** | Reliable with an active tracker when the device immediately echoes the sent frame; can be ineffective in sleep/power-saving state |

## LED and speaker

The LED and speaker commands were first identified through interoperability research and then reproduced independently by sending the corresponding binary frames directly to the tracker. They therefore do not require a permanent 365GPS connection.

```text
7878036406000D0A   LED off
7878036406010D0A   LED on

787803C100000D0A   Speaker off
787803C100010D0A   Speaker on
```

## Remote power-off — message `0x48`

```text
78780248020D0A
```

On the tested hardware, remote power-off was reliable with an **active** tracker when the device immediately echoed the sent `0x48 02` frame. In sleep or power-saving state, the same downlink could remain ineffective.

The immediate echo is an observed practical success indicator in the tested setup, **not** a documented protocol acknowledgement.

## Tracking interval — message `0x97`

For normal interval values, the tested command has the form:

```text
78 78 03 97 XX XX 0D 0A
```

`XX XX` is the interval in **seconds**, encoded as an unsigned 16-bit big-endian value.

```text
000A =   10 seconds
001E =   30 seconds
0258 =  600 seconds
08AC = 2220 seconds
09D8 = 2520 seconds
```

### Controlled 30-second test

The binary meaning of `0x97` was verified with a controlled test:

1. The tracker reported `TIMER 10,600`.
2. Plain ASCII `TIMER,30,600#` was sent through the Topin TCP connection and did **not** change the setting.
3. Binary frame `78780397001E0D0A` was sent.
4. A subsequent TIMER readback reported `30,600`.

This confirms that `0x97` changes **TIMER X**, the moving tracking interval, while TIMER Y remained unchanged in this experiment.

It does **not** establish a binary command for TIMER Y / idle interval.

See [0x97 tracking interval](../protocol/0x97-tracking-interval.md) for the protocol-level documentation.

## Sleep is a special value

`78780397FFFF0D0A`

`FFFF` has been observed as the special sleep command. It should **not** be interpreted as an ordinary interval of 65,535 seconds.

## SMS TIMER syntax

At the tracker command/configuration layer, the following syntax has been observed:

```text
TIMER,X,Y#
```

where `X` is the moving interval and `Y` is the idle interval.

A TIMER readback can therefore be useful for verifying the current values.

Do not confuse this ASCII/SMS syntax with the binary Topin command. In our direct TCP experiment, sending the ASCII TIMER command as raw TCP payload did not change the setting, while the binary `0x97` frame did.

## Sending these commands with Traccar

Traccar already supports raw hexadecimal **Custom commands**. No additional ZX909-specific custom-command patch is required.

See [Sending commands with Traccar](sending-with-traccar.md) for the practical procedure.

## Not confirmed

The following items are deliberately **not** included in the confirmed command table:

- vibration control — application/API behaviour was observed, but the corresponding binary tracker command has not been reproduced
- `0x92` / `0x93` as vibration commands — investigated but not reproducible
- `FF1A` / `001A` — observed responses with unresolved semantics
- binary control of TIMER Y / idle interval — currently unknown
- changing the configured server through an already established Topin TCP connection — tested candidate binary downlinks were unsuccessful; no working binary server-setting command is confirmed

See [Open questions](../research/open-questions.md).

---

A command belongs on this page only after its effect has been reproduced on the tested hardware. Observing a function in an app or cloud API alone is not enough.
