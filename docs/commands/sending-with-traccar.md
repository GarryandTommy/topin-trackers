# Sending commands with Traccar

This page describes how confirmed ZX909 binary commands can be sent through an existing Topin TCP connection using Traccar.

> **Use at your own risk.** A command sent to a tracker can change device behaviour or persistent settings. Verify the target device and command before sending it. Some changes can cause loss of connectivity and may require SMS or local modem access to recover.

Tested reference environment:

- Verdant Trace VT02 / Topin ZX909
- firmware `ZX909_EU_V1.3.7_2025-11-17_09-43-24`
- Traccar Server 6.15.x
- Topin protocol over TCP

## No custom Traccar patch is required

Traccar's `BaseProtocol` already supports **Custom commands** containing raw hexadecimal data.

Confirmed binary downlinks can therefore be sent directly from Traccar on the tested ZX909 without adding a ZX909-specific command implementation.

The LTE/LBS decoder patches documented elsewhere in this project are unrelated to this capability.

## Before sending a command

The tracker must have an active connection to the Traccar server. Check that the correct device is selected, it is currently connected, the connection uses the Topin protocol, and the hexadecimal command is correct.

For first tests, use a command with an immediately visible and reversible effect, such as LED off/on.

## Sending a raw command

In the Traccar web interface:

1. open the command dialog for the intended device;
2. select **Custom command**;
3. enter the complete hexadecimal frame in the **Data** field;
4. send the command.

Example:

```text
7878036406000D0A
```

This is the confirmed **LED off** command. To reverse the test:

```text
7878036406010D0A
```

This is the confirmed **LED on** command.

See [Confirmed device commands](confirmed-commands.md) for the maintained list of reproduced commands.

## Queueing

Depending on the Traccar interface/version, the command dialog may offer a queueing option such as **No queue**.

For direct interoperability testing we used the command while the tracker was online. This keeps the experiment tied to the currently observed connection and avoids an old test command being delivered unexpectedly after a later reconnect.

Do not assume that every command is safe to queue for later delivery.

## Verify the result independently

A successful send action in the Traccar interface proves only that Traccar attempted to transmit the command. It does not by itself prove the intended device-side effect.

Whenever possible, verify the result independently: observe the physical LED for LED commands, verify the speaker state for speaker commands, and use TIMER readback plus subsequent device behaviour for tracking-interval changes.

This distinction is why this project labels commands **Confirmed** only after their effect has been reproduced on real hardware.

## What appears in the Traccar log

During testing, raw Topin traffic could be observed in the Traccar log. For example:

```text
7878036406000D0A
```

was visible as a server-to-device Topin frame when the LED-off command was sent.

Logs are useful evidence, but direction matters: seeing a hexadecimal frame in a log does not by itself establish what it means. Meaning is assigned only after the corresponding effect has been reproduced.

## State-dependent remote power-off

On the tested hardware, remote power-off with `0x48 02` was reliable when an active tracker immediately returned the sent frame. In sleep or power-saving state, the same command could remain ineffective. The immediate echo is an observed practical success indicator, not a protocol-specified acknowledgement.

## ASCII/SMS commands are different

Do not assume that a command normally sent by SMS can simply be pasted into a Traccar Custom command as ASCII text.

During the controlled tracking-interval experiment, sending:

```text
TIMER,30,600#
```

through the Topin TCP connection did not change the TIMER setting.

The binary `0x97` command:

```text
78780397001E0D0A
```

changed the setting successfully.

SMS commands, binary Topin downlinks and modem AT commands are therefore documented as separate command layers in this project.

## Commands that can cut off remote access

Extra care is required for commands that change the server hostname/IP, server port, APN/PDP configuration, network mode, or sleep/connectivity behaviour. A wrong value can make the tracker unreachable from Traccar.

Several candidate server-setting frames were tested over the established Topin connection without achieving a server change. This does not prove that no such binary command exists. It should not be inferred from the examples on this page.

## Troubleshooting

If a command appears to do nothing:

1. confirm that the tracker is still online;
2. confirm that the selected Traccar device is the intended physical tracker;
3. inspect the Traccar log for the outgoing frame;
4. verify the command against [Confirmed device commands](confirmed-commands.md);
5. check the result on the device or through an independent readback.

Do not repeatedly send an unverified command merely because no visible response appeared.

---

This page documents the transport procedure. The meaning and evidence status of individual binary commands is maintained separately in [Confirmed device commands](confirmed-commands.md).
