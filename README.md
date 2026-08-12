# KEEL

**Field Instrument FI-221 (proposed — number not confirmed).** A single-file
browser bench that writes a Meshtastic release onto a bare ESP32 over the ROM
serial bootloader, and proves afterward that the bytes landed.

Sibling to WATCHFIRE (FI-220). KEEL lays down what the board is built on;
WATCHFIRE sails it.

    Make. Hack. Learn. Share. Repeat.

---

## What it is for

One job: an ESP-WROOM-32 devkit — the 30-pin CP2102 or CH340 class of board —
with a bare SPI LoRa module wired to it, being turned into a Meshtastic node
for the first time. That is the hardware the stock `diy-v1` variant targets.

It is not a general-purpose `esptool` port and does not try to be. It knows one
chip family, one bootloader, and one shape of release archive, and it keeps a
record of what it did.

## Running it

Open `keel.html`. That is the whole install. It runs from `file://` with no
server, no build step, no CDN, and no network access at any point.

Web Serial is needed for a real board: Chrome, Edge or Opera on desktop. Every
other part of the instrument — including a complete simulated flash run — works
in any browser.

## The eight stations

| | | |
|---|---|---|
| 01 | **Wire** | Open the port, walk the board into its bootloader one labelled step at a time, and identify the chip. |
| 02 | **Board** | The `diy-v1` pin map in DevKit V1 silkscreen labels, the module pin each goes to, and the pre-flight checks worth making before anything is powered. |
| 03 | **Image** | Drop the release archive. KEEL reads its contents, its install script, and every image header. |
| 04 | **Plan** | Every region, offset, length and expected digest, as a document to approve before anything is sent. |
| 05 | **Burn** | Execution, with THE MAP as the main view. |
| 06 | **Proof** | The verification record, exportable as JSON or text. |
| 07 | **Handoff** | The region warning, the boot log, and the pass to WATCHFIRE. |
| 08 | **Simulator** | A synthetic bootloader with six switchable faults, plus the self-test. |

## THE MAP

The signature element. A to-scale drawing of the chip's address space with each
region as a band on it. Bands go **planned → erasing → writing → verified**, or
**failed**, and the fill grows with the byte count. Unwritten space is left
visibly empty, so you can see at a glance how much of the chip a run is *not*
touching. When a run stops, the map is left exactly where it stopped.

## Positions this instrument takes

**ROM only. No stub loader.** KEEL does not push Espressif's stub into RAM. That
costs speed, whole-chip erase and flash readback. It buys a program you can read
end to end. If the stub is ever added it will be loaded from disk by the
operator, never embedded.

**No backup, and no pretending otherwise.** Reading flash out needs the stub, so
KEEL will not offer a backup button that silently does nothing. Whatever is in
the regions being written is gone.

**Nothing is reported as written until it is read back.** Every region is
verified with `SPI_FLASH_MD5` against a digest computed locally from the same
bytes. No digest, no green.

**The archive is the authority on offsets.** KEEL parses `device-install.sh` (or
the `.bat`) out of the release and uses the addresses *it* names. Where a release
carries no script, the fallback offsets are shown in brass wherever they appear,
labelled as assumptions. Meshtastic has repartitioned before and will again.

**Flash size is declared, not detected.** Reading the SPI flash ID on the ROM
loader means driving the SPI peripheral by hand. A guessed size is worse than an
honest one, so you select it and the interface says so everywhere it matters.

**A verified write is not a working node.** The digest proves the bytes landed.
It says nothing about wiring, antenna, band or region. The success screen does
not congratulate; it hands off.

## Before you flash a DIY board

The `diy-v1` SPI pins are **not** the ESP32 defaults. On a DevKit V1 silkscreen:
SCK `D5`, MOSI `D27`, MISO `D19`, CS `D18`, RESET `D23`, DIO0 `D26`, DIO1 `D33`,
BUSY `D32`. Wiring from habit puts SCK and CS on each other's pins and the radio
never answers. Station 02 has the full table and both header columns.

GPIO12 is the MTDI strapping pin. Held high at boot on a 3.3 V-flash WROOM-32 the
chip will not start — and `diy-v1` puts GPS_RX on it.

Never key a LoRa module without an antenna. Match the module's band to the region
you intend to set: a 902–928 MHz part for `US_915`.

## Self-test

Station 08, in the page, offline, against the simulator: 79 assertions covering
the SLIP codec, checksums, response parsing at both status lengths, MD5 against
RFC 1321 vectors, inflate against fixed and dynamic Huffman vectors, the archive
reader, image headers, the install-script parser, the plan builder, the map
geometry, and complete runs — clean, refused mid-write, gone quiet, and reporting
a digest that does not match.

A headless harness (jsdom) runs the same `K.selfTest()` against the same file.

## Protocol notes worth keeping

- Status bytes are **two** on the ESP32 ROM and four on every chip after it.
  Read that wrong and every failure parses as a success. The parser refuses
  frames too short to carry a status rather than guessing.
- The checksum is `0xEF`-seeded and applies only to `MEM_DATA`, `FLASH_DATA`
  and `FLASH_DEFL_DATA`.
- On the ROM loader, the size field of `FLASH_DEFL_BEGIN` is the *erase* size
  rounded up to whole write blocks, not the compressed length.
- The ROM write block is 1024 bytes. The stub's 16384 does not apply here.
- The ROM answers a sync eight times; the extra frames are drained.

## Still open

Meshtastic's current ESP32 partition offsets are read from your archive rather
than asserted here — check them once against a real release. Whether the
filesystem image should be written by default is a judgement call KEEL leaves to
you (writing it wipes stored config), and the checkbox in station 04 is how you
make it.

Not certified for anything. It is a bench instrument, and it is your board.
