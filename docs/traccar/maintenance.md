# Maintaining the Traccar patches

> **Use at your own risk.** Upstream changes or cherry-pick conflicts can alter behaviour. Test and keep a rollback build.

Maintained original commits:

`1d5eb6f` — confirmed `0x1A` correction  
`dfa73c4` — implemented/test-covered `0x1B` support, awaiting live validation

Do **not** include `97f932c`, the rejected CRLF framing experiment.

## Update workflow

1. Fetch the official Traccar release/tag.
2. Create `zx909-<version>` from that tag.
3. Cherry-pick `1d5eb6f`.
4. Cherry-pick `dfa73c4`.
5. Run the relevant Topin regression tests.
6. Review any upstream changes affecting `TopinProtocolDecoder` or the patched code paths.
7. Build and field-test with the tested ZX909 hardware.
8. Only after successful validation, push the version-specific branch to the project fork.

On Traccar 6.15.3 the cherry-picked commits became `ed5c046` and `58b0368`.

A clean cherry-pick is not proof of unchanged runtime semantics; tests and field validation remain necessary.
