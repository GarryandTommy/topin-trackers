# Traccar upstream status

The `0x1A` correction is based on real ZX909 traffic and a regression test. During upstream discussion, manufacturer protocol documentation was requested as supporting evidence. Suitable official documentation was not available to this project, and the change was not merged upstream.

The `0x1B` implementation has weaker evidence: code and a regression test exist, but no genuine ZX909 `0x1B` frame has validated the proposed layout.

A CRLF framing patch addressed the reproduced multiple-messages-per-TCP-read case locally, but was not retained as a safe general Topin solution and is classified **Rejected**.

Local support, passing tests and upstream acceptance are different claims and are documented separately.

Nothing here implies endorsement by Traccar, Topin, Verdant Trace or 365GPS.
