# Topin Trackers (ZX909 Family)

Open-source interoperability research, protocol documentation and
practical integration work for Topin GPS/LTE trackers, currently focused
on the **ZX909 platform** and the **Verdant Trace VT02**.

> **Project status:** Active research. Findings are based on real
> hardware and are labelled by evidence level. This project is
> independent and is not affiliated with, sponsored by, or endorsed by Topin, 365GPS or Verdant Trace.

> **Use at your own risk.** This project documents experimental interoperability work on real hardware. Commands, patches and configuration changes may behave differently on other devices or firmware versions.

## What this project is for

Small LTE/GPS trackers are useful hardware, but operating them outside
the manufacturer's cloud can require knowledge that is difficult to
find: server configuration, modem state, binary protocol details, device
commands and tracker-server integration.

This repository collects what we have learned from real ZX909 hardware
so that the work is reproducible and useful to other owners, developers
and manufacturers.

The current goals are to:

-   operate compatible trackers with an **own SIM and own tracking
    server**
-   document the observed **Topin binary protocol**
-   provide confirmed **device commands** without requiring the
    manufacturer cloud
-   improve and document **Traccar integration**
-   document modem, LTE/LBS and deployment behaviour
-   distinguish verified results from hypotheses and open questions

## Tested hardware

Current primary test devices:

-   **Verdant Trace VT02**
-   platform/protocol association: **Topin ZX909**
-   observed firmware: `ZX909_EU_V1.3.7_2025-11-17_09-43-24`

### Manufacturer information and upcoming hardware

Direct contact with Topin has provided manufacturer documentation for the
365GPS/Topin tracker-to-server communication protocol. This material is
used as a reference alongside independently captured traffic and device
tests; manufacturer descriptions are not automatically classified as
confirmed behaviour on our tested ZX909 firmware. The manufacturer
document itself is not redistributed in this repository.

One G63, one G67 and a bare ZX910 PCBA have been ordered and shipped for
future investigation. According to Topin, the supplied G63 and G67 use
the ZX910 platform. They have **not yet been tested by this project**;
no ZX909-to-ZX910 compatibility is claimed.

The title "ZX909 Family" describes the current scope of this research.
It is **not** intended as an official Topin product-family
classification. Other Topin models will only be listed as compatible
after they have been technically verified.

## Quick start

If you own a compatible tracker and want to run it on your own
infrastructure, start with:

1.  [Getting started](docs/getting-started.md)
2.  [Confirmed device commands](docs/commands/confirmed-commands.md)
3.  [Sending raw commands with
    Traccar](docs/commands/sending-with-traccar.md)
4.  [Traccar integration](docs/traccar/integration.md)

The repository also documents USB/modem access and persistent APN/PDP
configuration for cases where changing the application-level APN alone
is not sufficient.

## Project findings

Some of the strongest results currently documented by this project
include:

-   corrected decoding of newer ZX909 LTE cell data in Topin message
    `0x1A`
-   local, regression-tested support work for offline LTE/LBS message `0x1B`,
    awaiting live-device validation
-   direct LED and speaker control through the existing Topin TCP
    connection
-   binary `0x97` tracking-interval control, including a controlled
    30-second TIMER-X experiment
-   raw hexadecimal downlinks through Traccar without a proprietary
    cloud
-   observed offline GPS buffering/backfill
-   repeated evidence that multiple complete Topin frames can arrive in
    a single TCP read

See the protocol and command documentation for the exact evidence level
of each result.

## Evidence levels

We use four labels throughout the project:

  -----------------------------------------------------------------------
  Status                              Meaning
  ----------------------------------- -----------------------------------
  **Confirmed**                       Reproduced on real hardware in a
                                      controlled or directly observable
                                      test

  **Observed**                        Seen in real traffic or software
                                      behaviour, but semantics are not
                                      fully proven

  **Unknown / Hypothesis**            Plausible interpretation or open
                                      research question

  **Rejected**                        Investigated explanation or
                                      implementation that did not hold up
  -----------------------------------------------------------------------

This distinction is intentional. For example, a function observed in the
365GPS ecosystem is not automatically treated as a confirmed binary
tracker command.

## Documentation

### Use it

Practical setup, confirmed commands, Traccar usage and deployment.

### Understand it

Hardware identification, USB/modem access, APN/PDP behaviour, protocol
messages and LTE/LBS.

### Develop it

Traccar decoder changes, regression tests, patch maintenance and
protocol framing work.

### Research it

Interoperability methodology, selected anonymized field tests, rejected
approaches and open questions.

## Traccar work

The project maintains a deliberately small delta against upstream
Traccar.

The two current ZX909 decoder changes address:

-   newer LTE cell-field decoding for `0x1A`
-   offline LTE/LBS handling for `0x1B`

The `0x1A` correction has been validated with real packets on the
patched server. `0x1B` is implemented but still awaits equivalent live
validation.

A previous CRLF delimiter approach for splitting multiple Topin messages
was rejected as an unsafe general solution for a binary protocol. A
future framing solution should be protocol-aware.

The research has led to concrete upstream Traccar contributions:

- [Traccar PR #6012 — Fix LTE cell decoding for Topin protocol](https://github.com/traccar/traccar/pull/6012)
- [Traccar PR #6013 — Add frame decoding for Topin protocol](https://github.com/traccar/traccar/pull/6013)

These are upstream contributions, not a claim that the changes have been
merged. Manufacturer protocol documentation has since been received
from Topin and can inform further review and validation.

## Why this matters

Topin GPS/LTE trackers are commonly operated through the manufacturer's
365GPS ecosystem. This project investigates interoperability with
independently operated infrastructure and publishes reproducible findings
from real devices. Where appropriate, the work is contributed upstream
to Traccar so others can benefit without assuming that untested models
or firmware versions are compatible.

## Interoperability research

Some device commands were discovered by observing behaviour in the
existing 365GPS ecosystem and then testing candidate commands
independently over our own Traccar connection.

The important part is the second step:

**vendor-system observation → hypothesis → direct test on our own
infrastructure → evidence status**

We publish our own observations and reproducible results, not
proprietary application source code, APKs, credentials or private
production traffic.

## Privacy and sample data

Public examples are minimized and anonymized. Full production logs,
IMEIs, SIM identifiers, private server addresses, API keys and real
movement histories are not part of this repository.

## Current open questions

Among the remaining research topics are:

-   a robust protocol-aware Topin frame decoder
-   live validation of `0x1B`
-   the binary command corresponding to vibration control
-   semantics of observed `FF1A` / `001A` responses
-   additional server-control commands over an existing Topin TCP
    session
-   ZX910 PCBA access, debug/test points and firmware tooling
-   validation of additional Topin tracker models

## About GarryAndTommy

**GarryAndTommy** is an independent open-source GPS/IoT project built
around real hardware, reproducible experiments and practical
interoperability.

The project started with two small cat trackers. The cats provided the
requirement; the trackers provided the protocol problem.

------------------------------------------------------------------------

Topin, ZX909, 365GPS, Verdant Trace and Traccar are names or trademarks
of their respective owners. References are used only to identify tested
hardware, software and protocols.
