# USB and modem access

> **Use at your own risk.** Direct modem commands can change persistent
> cellular settings and make a tracker unreachable. Read and record
> original values before changing them.

The tested finished VT02 provides useful modem access through USB-C;
opening the tracker was not required for the diagnostic work. After
installing the appropriate MeiG USB driver on Windows, a stable AT
interface was available as `MEIG Modem Device` (observed driver
`musbser.inf`, version `1.0.0.3`).

Keep three command layers separate:

-   SMS/tracker commands address tracker firmware;
-   binary Topin downlinks use the tracker/server connection;
-   AT commands address the cellular modem.

## Read-first diagnostic set

The following commands were used during the project:

``` text
AT+CPIN?
AT+CIMI
AT+CSQ
AT+CEREG?
AT+CGATT?
AT+COPS?
AT+CGDCONT?
AT*CGDFLT?
AT*REJCAUSE=2
AT+CELLINFO
```

| Command | Purpose | Changes state? | Tested finding |
| --- | --- | --- | --- |
| `AT+CPIN?` | SIM/PIN state | No | Confirmed diagnostic |
| `AT+CIMI` | Read IMSI | No | Confirmed; redact output |
| `AT+CSQ` | Radio signal indication | No | Confirmed diagnostic |
| `AT+CEREG?` | LTE/EPS registration and cell information | No | Confirmed diagnostic |
| `AT+CGATT?` | Packet-domain attach state | No | Confirmed diagnostic |
| `AT+COPS?` | Current operator information | No | Confirmed diagnostic |
| `AT+CGDCONT?` | PDP context configuration | No | Confirmed diagnostic; did not by itself expose or fix the persistent state relevant to the recovery case |
| `AT*CGDFLT?` | Attempt to read the persistent default PDP context | No | Readback unsupported in the tested case: returned `CME ERROR: 4`; the write form nevertheless worked in the documented recovery case |
| `AT*REJCAUSE=2` | Expose registration/session rejection information | Diagnostic request | Confirmed diagnostic |
| `AT+CELLINFO` | Cell diagnostics | No | Used to cross-check TAC/ECI against real `0x1A` traffic; exact output may vary |

Use `AT+CIMI` only when needed and redact the returned IMSI from public
logs. Cell information and timestamps can also reveal location context.

`AT*CGDFLT?` returned `CME ERROR: 4` on the tested modem even though
writing `AT*CGDFLT=...` worked. Do not infer that a failed readback
means the write form is unsupported.

For a tracker that has radio signal but cannot establish data, begin
with the read-only registration and attach queries. Persistent APN/PDP
changes are a later recovery step, not a first-line configuration
method.

AT cell diagnostics were also used as an independent cross-check of
TAC/ECI values found in real `0x1A` packets.

See [APN and persistent PDP context](../modem/apn-and-pdp-context.md)
for the recovery case and [0x1A LTE LBS](../protocol/0x1a-lte-lbs.md)
for the packet evidence.

This page does not prescribe undocumented PCB pinouts or flashing
interfaces.
