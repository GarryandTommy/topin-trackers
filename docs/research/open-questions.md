# Open questions

This page tracks questions that remain unresolved after the current hardware tests, protocol work and Traccar experiments. A question stays open when the project has useful evidence but not enough to promote an interpretation to a confirmed protocol fact.

## 1. Protocol-aware Topin framing

**Known:** Multiple complete Topin frames can arrive in one TCP read. A generic CRLF delimiter was investigated and rejected as a generally safe binary framing rule.

**Unknown:** The complete protocol-aware rules needed to split and reassemble all relevant Topin frame variants safely.

**What would close it:** Representative protocol evidence sufficient to implement and regression-test a robust binary `TopinFrameDecoder`, ideally supported by manufacturer documentation.

## 2. Live validation of `0x1B`

**Known:** A local decoder implementation and regression test exist. A controlled attempt to provoke a genuine ZX909 `0x1B` did not produce one.

**Unknown:** Whether the tested firmware emits `0x1B` and whether a real frame matches the assumed offline LTE/LBS layout.

**What would close it:** A genuine ZX909-generated `0x1B` frame captured under known conditions and compared against the implemented layout.

## 3. Binary vibration control

**Known:** Vibration-related behaviour was observed at the application/API level. Candidate binary commands investigated so far were not reproducible.

**Unknown:** The binary tracker command, if any, that directly controls vibration on the tested hardware.

**What would close it:** A command/effect pair independently reproduced on the physical tracker.

## 4. Meaning of `FF1A` / `001A`

**Known:** Short `FF1A` / `001A` timestamp-related responses were observed around `0x1A` traffic.

**Unknown:** Their semantics and whether they represent acknowledgements, requests, responses, or another protocol function.

**What would close it:** Controlled correlation with device behaviour or authoritative protocol documentation. Until then they should not be promoted to commands or acknowledgements.

## 5. Server configuration over an existing Topin TCP session

**Known:** Several candidate binary downlinks were tested through an established Topin connection; none changed the configured server address.

**Unknown:** Whether a supported binary server-configuration command exists and, if so, its frame format and persistence/restart behaviour.

**What would close it:** A reproducible binary command that changes the configured server and survives the expected restart/persistence cycle, or authoritative documentation defining the mechanism. The failed candidates do not establish that no such command exists.

## 6. Hardware / PCBA / debug interfaces

**Known:** The finished tested VT02 exposes useful modem AT access over USB-C.

**Unknown:** Reliable ZX909/VT02 PCBA documentation and the purpose of any USB/UART/test/flash interfaces on a bare board.

**What would close it:** Board-specific documentation or reproducible electrical/interface identification on a verified matching PCBA. Visually similar boards are not sufficient evidence.

## 7. Additional Topin models

**Known:** The confirmed findings in this repository come from the tested Verdant Trace VT02 / Topin ZX909 hardware and firmware.

**Unknown:** Which additional Topin trackers share compatible hardware, firmware behaviour and protocol details.

**What would close it:** Reproducible testing on identified hardware/firmware before extending project scope. Product names, housings or reseller descriptions alone are not enough.

## 8. `0x48` state dependence and echo semantics

**Known:** Remote power-off using `0x48 02` is confirmed on the tested hardware. With an active tracker, shutdown was reliable when the device immediately echoed the sent frame; in sleep/power-saving state, the same downlink could remain ineffective.

**Unknown:** Why device state affects the result and what the immediate echo formally means in the protocol.

**What would close it:** Controlled state-specific testing that identifies the relevant device conditions and/or manufacturer documentation defining the echo semantics. The observed echo should not be called a formal acknowledgement without such evidence.

## 9. Tracker APN vs persistent modem PDP

**Known:** In the tested recovery case, persistent modem configuration through `AT*CGDFLT` was decisive, while tracker/application APN state could differ. A later `FACTORY#` experiment did not simply erase every relevant persistent modem setting.

**Unknown:** How tracker/application APN state and persistent modem PDP state are stored, synchronized, prioritized or owned internally.

**What would close it:** Controlled configuration/reset experiments that establish the relationship, or authoritative firmware/modem documentation. The current observations do not establish an internal architecture.

---

An open question is closed only by reproducible hardware evidence, authoritative protocol documentation, or both. A plausible interpretation, passing synthetic regression test, echoed command, or unsuccessful experiment alone is not sufficient.
