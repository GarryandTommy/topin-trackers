# APN and persistent PDP context

> **Use at your own risk.** A wrong persistent APN/PDP configuration can
> remove mobile data connectivity. If normal tracker APN configuration
> works, leave the modem alone.

The project encountered two distinguishable configuration layers:

1.  tracker/application APN state;
2.  the cellular modem's persistent default PDP/APN state.

This distinction became visible during recovery from an attach failure.
The tracker could have network/signal visibility while packet attachment
still failed. Changing `AT+CGDCONT` alone did not provide the persistent
recovery obtained with `AT*CGDFLT`.

## Confirmed recovery path

**Do not copy these commands unless the APN is correct for your SIM/provider. `AT*CGDFLT` changes persistent modem configuration; an incorrect value can prevent the tracker from reconnecting after reset.**

Provider-specific values used during testing were:

``` text
AT*CGDFLT=1,"IP","internet.telekom",,,,,,,,,,,,,,,,,,1
AT+RESET
```

and:

``` text
AT*CGDFLT=1,"IP","sensor.net",,,,,,,,,,,,,,,,,,1
AT+RESET
```

These are examples for the tested SIMs, not universal defaults.

After the correct persistent default PDP/APN was set and the modem
reset, LTE registration/data attachment recovered; `AT+CGATT?` again
showed an attached state.

## Why `APN#` is not enough

The tracker's SMS/application `APN#` response is not proof of the
modem's effective persistent PDP configuration. During testing,
application-level APN state and the working modem configuration could
differ.

A later `FACTORY#` experiment reinforced this separation. Some
tracker/application settings reset, but the modem-side configuration
established with `AT*CGDFLT` remained effective and the tracker returned
online.

This does not establish the firmware's internal ownership or
synchronization rules between the two layers; those remain an open
question.

## Diagnostic rule

Use the normal tracker APN mechanism first. Use direct modem access only
when diagnosis shows it is necessary. Record existing values and retain
a recovery path before changing persistent modem state.

See [USB and modem access](../hardware/usb-modem-access.md).
