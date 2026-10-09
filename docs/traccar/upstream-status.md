# Traccar upstream status

The `0x1A` correction is based on real ZX909 traffic and a regression test. During upstream discussion, manufacturer protocol documentation was requested as supporting evidence. Manufacturer tracker-to-server protocol documentation has since been received directly from Topin. It is an additional reference for upstream discussion, not a replacement for live ZX909 validation.

The `0x1B` implementation has weaker evidence: code and a regression test exist, but no genuine ZX909 `0x1B` frame has validated the proposed layout.

A CRLF framing patch addressed the reproduced multiple-messages-per-TCP-read case locally, but was not retained as a safe general Topin solution and is classified **Rejected**.

Local support, passing tests and upstream acceptance are different claims and are documented separately.

Nothing here implies endorsement by Traccar, Topin, Verdant Trace or 365GPS.

## Upstream pull requests

- [#6012 — Fix LTE cell decoding for Topin protocol](https://github.com/traccar/traccar/pull/6012): LTE cell-field decoding work based on real ZX909 traffic and regression testing.
- [#6013 — Add frame decoding for Topin protocol](https://github.com/traccar/traccar/pull/6013): framing-related upstream contribution; the earlier CRLF-only approach remains **Rejected** as a general binary framing solution.

Submission does not imply merge or acceptance. Consult the linked pull requests for their current review status.
