# Changelog

All notable changes to KEEL. Versions are the single-file `keel.html`.

## v1.4.0 — 2026-08-11

### Fixed
- **The boot-log reader called a dead radio a working one.** Meshtastic writes
  "No RF95 radio", not "no radio found", so every failure pattern missed — and
  the success pattern matched the substring "RF95 init" inside the line
  `RF95 init result -2`. A board with nothing soldered to it would have been
  told its SPI wiring was fine. Rewritten around RadioLib result codes: nothing
  is a success unless a result of `0` says so, every attempted interface is
  counted, and the code is decoded (`-2` chip not found, `-16` SPI timeout, and
  the rest), with unknown codes admitted as unknown.
- ANSI escape sequences are stripped before matching and before display, so the
  log is readable and the patterns see the text rather than the colour codes.

### Added
- The reader now reports firmware version and variant from the `S:B:` line, and
  checks the SPI pins the firmware actually opened against the diy-v1 set.
- A real captured boot log — ESP32, `meshtastic-diy-v1` 2.7.26, no module
  fitted — is now a test fixture, escapes and all. 95 assertions.

## v1.3.0 — 2026-08-11

### Added
- **Build the plan by hand.** Station 04 offers "Ignore the script and build the
  plan myself": an editable table of offset and file, seeded from whatever the
  script did resolve. Add rows, remove rows, type offsets. Every check still
  runs — sector alignment, overlap, end of chip, and digest verification on
  every region. The run record notes that the plan was hand-built.
  The script parser is a convenience and must never be the only way to reach the
  hardware. Three releases in a row blocked a real flash on a parsing question;
  that is the wrong thing to be blocking on.

## v1.2.0 — 2026-08-11

### Added
- "Write nothing here" as an explicit choice on any script write. Skipped writes
  are listed in station 04 as the operator's decision rather than the script's,
  and recorded in the run record.

### Fixed
- The skipped-writes note stacked up a copy on every plan refresh.

## v1.1.0 — 2026-08-11

Found on first contact with a real Meshtastic release archive on Windows.

### Fixed
- **Any write naming a file that is not in the archive now gets a file picker.**
  Previously only tokens matching a known variable syntax were bindable and
  everything else was a hard stop with no way forward. From the operator's side
  "this is a variable" and "I cannot find this file" are the same situation, and
  only one of them was survivable. The parser recognising a syntax is now a
  convenience, not the safety net.
- Recognise cmd.exe delayed expansion (`!NAME!`) alongside `$NAME`, `${NAME}`
  and `%NAME%`. `device-install.bat` uses it and KEEL was reading `!FILENAME!`
  as a literal filename.
- One function, `K.resolveEntry`, now decides which archive entry a script token
  means, so station 03 and the plan builder cannot disagree about it.

### Added
- Station 03 states plainly that scripts pass the firmware in as an argument, so
  a pick is expected on the first pass rather than a sign of something wrong.
- The install script is viewable verbatim in station 03, under the parsed table.
- Station 04 links back to station 03 when a pick is outstanding.
- Bound files are marked "you chose this" so an operator decision is never
  mistaken later for something the script said.
- Three more assertions: cmd variable syntaxes, a CRLF `.bat`, and the full
  bind path from unmatched name to verified write. 82 total.

## v1.0.0 — 2026-08-11

First build. Review stop.

### Added
- Eight stations: Wire, Board, Image, Plan, Burn, Proof, Handoff, Simulator.
- THE MAP: to-scale address-space drawing with per-region state and fill.
- ESP32 ROM bootloader driver over Web Serial — reset sequence, sync, chip
  identification by magic register, eFuse MAC, SPI attach and parameters,
  compressed and uncompressed writes, baud change, `SPI_FLASH_MD5` verification.
- Hand-rolled SLIP codec, MD5, inflate, zlib stored-block deflate, adler32,
  and a zip reader — so the archive opens and the digests compute with no
  dependency on the browser having any particular compression API.
- Release archive reader: contents, image headers including the extended-header
  chip id, and `device-install.sh` / `.bat` parsing for offsets.
- Write plan with sector-alignment, overlap and end-of-chip checks, per-region
  expected digests, and an explicit approval gate.
- Variant gate: refuses an image whose declared chip disagrees with the chip on
  the wire, behind a typed override.
- Verification record exportable as JSON and text.
- Boot-log reader that names "no radio found" as a wiring problem rather than a
  firmware one.
- Simulated ROM bootloader with six faults: no sync, ROM chatter between frames,
  a refused data block, answers stopping mid-write, the wire dying, and a
  reported digest that does not match.
- Self-test, 79 assertions, in-page and offline.

### Positions taken
- ROM loader only, no stub: no whole-chip erase, no flash readback, no backup —
  and the interface says so rather than offering controls that do nothing.
- Offsets come from the archive's own install script; fallbacks are shown as
  assumptions in a different colour.
- Flash size is declared by the operator, not detected.
- Image flash-mode and size/frequency bytes are left exactly as the vendor
  shipped them.

### Known and deliberate
- ESP32 classic only. Later chips are identified and refused.
- Chip revision is not decoded.
- `AdminMessage`-style configuration is out of scope; that is WATCHFIRE's.
