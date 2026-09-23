# Getting started

This guide is the practical entry point for owners of a **Topin ZX909-based tracker**, currently tested with the **Verdant Trace VT02**, who want to operate the device with their **own SIM and their own Traccar server** instead of depending on the 365GPS cloud.

The project is based on real devices running the observed firmware:

`ZX909_EU_V1.3.7_2025-11-17_09-43-24`

Other firmware versions or Topin models may behave differently. See [Device identification](hardware/device-identification.md) before assuming compatibility.

> **Use at your own risk.** The procedures documented here can change persistent tracker, modem and server settings. Incorrect configuration may cause loss of connectivity or require local access to recover the device. Verify commands and parameters for your hardware and firmware before applying them.

## What you need

For the basic setup you need:

- a compatible ZX909-based tracker
- a working SIM with mobile data
- the correct APN for that SIM
- a reachable Traccar server
- the **Topin** protocol enabled on a TCP port
- a way to configure the tracker initially

You do **not** need a permanent 365GPS connection for normal operation once the tracker is configured for your own server.

## 1. Prepare Traccar

Create the tracker as a device in Traccar using its device identifier/IMEI.

Enable a TCP port for the **Topin** protocol. In our test installation we use port `5093`, but the port number itself is not special: the important part is that the tracker can reach the selected TCP port and that Traccar handles it as Topin.

Example:

```text
tracker.example.net:5093
```

Do not copy the private server address from somebody else's setup. Use a hostname or public address that resolves to your own Traccar installation.

### Stock Traccar and the ZX909

Basic Topin communication works with Traccar, but newer ZX909 LTE/LBS packets exposed decoder differences in the versions we tested.

This project currently maintains two small decoder changes:

- `0x1A` — corrected decoding of the newer LTE cell structure
- `0x1B` — support for the corresponding offline LTE/LBS message

The `0x1A` change has been validated with real ZX909 traffic. The `0x1B` implementation has regression-test coverage but still awaits equivalent live-device validation.

You can start with normal GPS tracking before dealing with LBS. The patches and maintenance procedure are documented under [Traccar integration](traccar/integration.md).

## 2. Configure the SIM and APN

The tracker needs a data-capable SIM and the APN required by its provider.

A useful lesson from the tested ZX909 is that there can be **two different configuration layers**:

1. the APN known to the tracker application/firmware
2. the persistent default PDP/APN configuration of the cellular modem

During our testing, changing only the application-level APN was not always sufficient. The modem could see a network while still failing to register or attach correctly.

USB modem diagnostics showed that `AT+CGDCONT` alone did not necessarily represent the persistent configuration that mattered after restart. The persistent modem state exposed by `AT*CGDFLT` was decisive in the tested case.

Do not change modem parameters merely because this guide mentions them. If normal APN configuration works, leave the modem alone.

If the tracker has signal but cannot establish mobile data, continue with:

- [USB modem access](hardware/usb-modem-access.md)
- [APN and persistent PDP context](modem/apn-and-pdp-context.md)

Those pages contain the diagnostic path before any persistent modem setting is changed.

## 3. Point the tracker to your server

The tracker must be configured with your own server hostname/address and Topin TCP port.

The exact configuration method depends on what access you currently have to the device. This project documents only methods that have been observed or reproduced on the tested hardware.

A previously documented Topin configuration path uses a server-setting command of the form:

```text
SERVER#hostname#port#
```

Treat this as a device-configuration operation: verify the hostname and port carefully before sending it. A wrong server address can make remote access to the tracker inconvenient or impossible until another configuration path is available.

### Important: changing the server over the existing Topin TCP connection

Several candidate binary downlinks for changing the server configuration have been tested through an established Topin TCP session, but none changed the configured server address. No working binary server-setting command is therefore confirmed, and this method is not part of the getting-started procedure.

## 4. Verify the first connection

After the tracker is configured, check the Traccar server log.

A successful setup should show the device connecting on the **Topin** protocol and sending binary packets beginning with the observed `78 78` framing.

Do not judge success only by whether a marker appears immediately on the map. ZX909 devices can buffer GPS records and send them later, and cellular/LBS messages are a separate path from normal GPS fixes.

Useful checks are:

- the connection is handled as `topin`
- the expected device identifier is recognized
- fresh GPS positions arrive
- the device reconnects after a restart
- device timestamps are reasonably current

## 5. Test a harmless downlink

Once the tracker has a stable Topin TCP connection, Traccar can send raw hexadecimal data using its existing **Custom command** support. No ZX909-specific custom-command patch is required for this.

For example, the following command has been reproduced on the tested ZX909 hardware:

```text
7878036406000D0A
```

**Confirmed effect:** LED off.

The corresponding confirmed LED-on command is:

```text
7878036406010D0A
```

This is a useful first end-to-end test because the result is directly observable on the device.

See [Confirmed device commands](commands/confirmed-commands.md) and [Sending commands with Traccar](commands/sending-with-traccar.md) before experimenting with other frames.

## 6. Tracking interval

The tested firmware also accepts a binary `0x97` command for the moving tracking interval.

Example:

```text
78780397001E0D0A
```

`0x001E` is 30 seconds as an unsigned 16-bit big-endian value.

In a controlled test, sending this frame changed the moving interval from 10 seconds to 30 seconds while the idle interval remained at 600 seconds. The new `30,600` setting was independently verified through the tracker's TIMER readback.

This confirms the binary mapping for **TIMER X / moving interval**. It does **not** establish the binary setter for TIMER Y / idle interval.

`FFFF` has been observed as the special sleep value and should not be treated as an ordinary 65535-second interval.

See [0x97 tracking interval](protocol/0x97-tracking-interval.md) for the evidence and packet format.

## 7. What works without 365GPS

On the tested ZX909 hardware, the project has reproduced direct control through its own Traccar connection for:

- LED on/off
- speaker on/off
- several tracking/power-saving intervals
- sleep mode

Remote power-off through `0x48 02` has also been reproduced, but its reliability is state-dependent; see [Confirmed commands](commands/confirmed-commands.md) for the tested conditions and limitations.

The goal is not to clone the 365GPS service. It is to document enough interoperable behaviour to operate lawfully acquired hardware on infrastructure controlled by its owner.

Some candidate functions discovered during interoperability research are deliberately **not** listed here because they have not yet been reproduced reliably.

## 8. Known limitations

Before treating the tracker as fully understood, be aware of the current boundaries:

- `0x1B` LTE/LBS decoding still needs live-device validation.
- Vibration control has been observed at the application/API level, but a corresponding binary tracker command is not confirmed.
- Short `FF1A` / `001A` responses have been observed, but their semantics remain unknown.
- Multiple complete Topin frames can arrive in one TCP read. A simple CRLF delimiter was investigated and rejected as a generally safe binary framing solution.
- Cell/LBS decoding does not itself calculate a location. Traccar needs an external geolocation provider to turn cell information into an estimated position.
- Compatibility with additional Topin models must be verified rather than inferred from similar names or housings.

Open items are tracked in [Open questions](research/open-questions.md).

## Where to go next

If your tracker is now reporting to your own Traccar server, choose the part that matches what you want to do next:

- **Control the tracker:** [Confirmed commands](commands/confirmed-commands.md)
- **Understand the packets:** [Protocol overview](protocol/overview.md)
- **Fix or inspect cellular connectivity:** [APN and PDP context](modem/apn-and-pdp-context.md)
- **Use LTE/LBS:** [Traccar geolocation](lbs/traccar-geolocation.md)
- **Run the patched server on Home Assistant OS:** [Home Assistant deployment](deployment/home-assistant.md)
- **Work on Traccar support:** [Traccar integration](traccar/integration.md)
- **See what is not solved yet:** [Open questions](research/open-questions.md)

---

This documentation intentionally separates **Confirmed**, **Observed**, **Unknown / Hypothesis**, and **Rejected** findings. If your hardware behaves differently, firmware version and exact device identification are useful information when reporting the result.
