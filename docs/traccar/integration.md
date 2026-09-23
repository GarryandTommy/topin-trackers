# Traccar integration

> **Use at your own risk.** Local decoder or server changes can affect position processing and connectivity. Keep a known-good rollback path.

The tested ZX909 trackers communicate with Traccar using **Topin over TCP**. This project uses port `5093`; the port number itself is not a ZX909 protocol requirement.

Stock Traccar supplies the Topin protocol and raw Custom-command capability. Project-specific work addresses LTE/LBS decoding.

Maintained local commits:

- `1d5eb6f` — confirmed `0x1A` LTE cell decoding correction.
- `dfa73c4` — implemented/test-covered `0x1B` support, awaiting live validation.

On the Traccar 6.15.3 branch, these patches were applied as `ed5c046` (`0x1A`) and `58b0368` (`0x1B`). The original commit IDs above identify the reusable project patches independently of a particular Traccar release branch.

The rejected CRLF framing experiment is not part of the maintained patch set.

Correct LTE identifiers do not themselves produce coordinates; LBS requires a separate cell-geolocation service.

See [Maintaining the Traccar patches](maintenance.md) for the update workflow and [Upstream status](upstream-status.md) for the distinction between local validation and upstream acceptance.
