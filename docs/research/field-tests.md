# Field tests

This page collects short, evidence-oriented field tests performed with
the project's Verdant Trace VT02 / Topin ZX909 trackers.

The purpose is not to publish complete production logs. Each case
documents a question, the controlled observation, and the limit of what
can be concluded.

Tested firmware:

`ZX909_EU_V1.3.7_2025-11-17_09-43-24`

## 1. LTE-LBS `0x1A` field validation

### Question

Does the newer ZX909 LTE-LBS packet use the wider LTE cell fields
implemented by the local Traccar patch?

### Method

Real `0x1A` traffic was captured from the trackers and compared with
independent modem cell diagnostics.

### Result

The observed TAC and ECI values matched between modem diagnostics and
packet bytes. The working layout is:

``` text
TAC     4 bytes
ECI     4 bytes, unsigned
Signal  1 byte
```

The corrected decoder was subsequently used with real `0x1A` traffic
from both project trackers.

### Conclusion

**Confirmed on the tested ZX909 firmware.**

This is the strongest live validation for the LTE-LBS decoder change.

## 2. Attempt to provoke a genuine `0x1B` offline LTE-LBS packet

### Question

Will the tested firmware emit `0x1B` when cellular service exists but
the configured server is temporarily unreachable and the tracker later
reconnects?

### Method

The tracker was tested without a usable GPS fix while cellular service
remained available. The server/port was temporarily made unreachable,
followed by reconnection and movement.

### Result

No genuine `0x1B` packet was observed. After reconnection, other traffic
was seen, including normal messages and a real `0x1A`.

### Conclusion

**Negative observation, not proof of absence.**

The local `0x1B` decoder implementation remains **Implemented / awaiting
live validation**. This experiment does not establish that the firmware
can never send `0x1B`; it only records that the chosen trigger
conditions did not produce one.

## 3. TIMER-X / `0x97` controlled cross-check

### Question

What does the binary Topin `0x97` command change?

### Method

1.  Read the current timer by SMS: `TIMER 10,600`.
2.  An ASCII timer string sent over the existing Topin TCP connection
    did not change the setting.
3.  Send binary frame:

``` text
78780397001E0D0A
```

4.  Read the timer again by SMS.

### Result

The SMS readback changed to:

``` text
TIMER 30,600
```

TIMER-X changed from 10 to 30 seconds; TIMER-Y remained 600.

Additional tested `0x97` values included 10 s, 600 s, 2220 s, 2520 s,
and the special `FFFF` sleep value.

### Conclusion

**Confirmed:** `0x97` controls TIMER-X / the movement tracking interval
on the tested firmware. `FFFF` is a special sleep value and should not
be interpreted as an ordinary 65535-second interval.

## 4. Raw binary downlink through Traccar

### Question

Is a Topin-specific Traccar custom-command patch required to send known
ZX909 binary commands?

### Method

Complete known Topin frames were sent as Traccar Custom commands over an
existing device connection.

### Result

Traccar sent the requested bytes unchanged. The tracker received them
and, for commands where echo behavior was observed, returned the frame
byte-for-byte.

### Conclusion

**Confirmed:** Traccar's existing raw Custom command path is sufficient
for the tested binary commands. A separate ZX909-specific custom-command
patch is not required.

## 5. Remote power-off: `0x48 02`

### Command

``` text
78780248020D0A
```

### Question

Why did apparently identical shutdown attempts sometimes work and
sometimes leave the tracker online?

### Observation

Repeated tests showed a state-dependent pattern.

When the tracker was **active**, successful remote power-off was
associated with the tracker immediately echoing the `0x48 02` command
back. Under these conditions the power-off function was reproducible.

When the tracker was already in a **sleep / power-saving state**, the
same downlink could be sent without producing an effective shutdown.
Earlier tests in which no immediate matching echo appeared were followed
by continued tracker traffic.

### Conclusion

**Confirmed operational behavior on the tested hardware:** remote
power-off is reliable when the tracker is active and immediately echoes
the command. In sleep/power-saving state the command can remain
ineffective.

The immediate echo is therefore a useful practical indicator in the
tested setup. It should not be generalized into a formal protocol
acknowledgement rule without manufacturer documentation.

## 6. Server change over the existing Topin TCP session

### Question

Can the tracker be moved to another server by sending a server-setting
downlink through its already established Topin TCP connection?

### Method

Several candidate binary downlinks were tested through the working
connection. The device received/echoed tested frames, after which its
configured server behavior was checked.

### Result

No tested downlink changed the configured server address as intended.

### Conclusion

**No working binary method confirmed.** None of the tested candidate
downlinks changed the configured server address. This does not establish
that no such Topin command exists.

The known configuration path remains the tracker configuration/SMS
layer.

## 7. Offline GPS / `0x11` backfill

### Observation

Both project trackers produced historical GPS data that was transmitted
later in bursts. This behavior was observed with different SIM setups.

Multiple complete `0x11` records can arrive close together and can
contribute to multiple protocol frames appearing in one TCP read.

### Conclusion

**Observed:** the tested ZX909 devices delivered historical GPS fixes
later in bursts. This behaviour was seen with different SIM setups and
should not be attributed to one SIM provider.

`0x11` messages are associated with this backfill behaviour, but that
association is not treated as a confirmed protocol meaning.

## 8. TCP coalescing

### Observation

Real server logs repeatedly contained multiple complete Topin messages
in a single TCP read.

### Conclusion

**Confirmed:** TCP read boundaries are not Topin message boundaries.

A CRLF delimiter decoder was experimentally implemented but rejected as
a general solution because Topin is a binary protocol and a delimiter
byte sequence cannot safely be assumed to be absent from payload data
without a proven framing rule.

The preferred future direction is a protocol-aware binary
`TopinFrameDecoder`.

## 9. `FACTORY#` and configuration layers

### Question

Does an application factory reset also erase the persistent modem
PDP/APN configuration?

### Method

A before/after SMS configuration snapshot was taken around `FACTORY#`,
and the device's ability to reconnect was observed.

### Result

Some application settings reset, including TIMER/LED-related values and
`MLG`. The configured server and application APN state remained in the
tested case. Most importantly, the modem-side `AT*CGDFLT` configuration
remained effective and the tracker returned online.

### Conclusion

**Observed on tested firmware:** the factory reset does not behave as a
complete erase of every persistent configuration layer. The observation supports treating tracker/application settings and
persistent modem configuration as separate diagnostic layers.

## 10. GNSS divergence

### Observation

During a joint real-world test, the two trackers sometimes reported
positions several hundred metres apart at the same device time even
though each individual track looked internally smooth. Other joint test
periods showed much smaller separation.

### Limitation

Without an independent ground-truth track, the experiment cannot
identify which receiver was wrong during the divergent period.

### Conclusion

This is a useful caution when interpreting individual GNSS points, but
not evidence of a Topin protocol or cellular decoding error.

## Publication rules for field-test material

Public examples should contain only the minimum data needed to reproduce
the technical point. Remove or replace:

-   IMEI, IMSI, ICCID and phone numbers,
-   private server hostnames and public IP history,
-   home/start/end locations,
-   unnecessary GPS coordinates,
-   complete movement profiles,
-   authentication tokens and API keys.

Raw logs remain internal evidence.
