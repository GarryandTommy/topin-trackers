# 365GPS interoperability research

The purpose of this research was to understand behaviour of lawfully owned trackers and reproduce useful functions on owner-controlled infrastructure, not to clone the 365GPS service.

Cloud/app observations were treated as hypotheses. A function became **Confirmed** only after its effect was independently reproduced on real hardware.

This method contributed to identifying binary downlinks for LED, speaker and tracking-interval functions.

`FF1A` / `001A` timestamp-related responses were observed around `0x1A`, but their semantics remain unresolved. Vibration-related app/API behaviour was also observed, while candidate binary commands were not reproducible.

The repository should document observations and independently derived interoperability results, not redistribute proprietary APKs, decompiled source or manufacturer material without permission.

This project is independent and is not affiliated with, sponsored by, or endorsed by Topin, 365GPS or Verdant Trace.
