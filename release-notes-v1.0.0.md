# KEEL v1.0.0 — laying the first plank on a bare ESP32

*Draft for Gears of Resistance*

There is a particular kind of nothing that a brand-new ESP32 development board
is. Three of them turn up in a bag for less than the price of one finished
radio, and they are genuinely blank: no firmware, no identity, no idea that a
mesh exists. Meshtastic's own `diy-v1` variant is built for exactly this board.
Bolt a LoRa module to it and you have a node for a fraction of the money.

Getting the firmware onto it is a solved problem — `esptool` has solved it for a
decade. But solved is not the same as *legible*. When you run `esptool
write_flash`, a great deal happens that you do not see, and at the end you are
told it worked. KEEL is the same job done where you can watch it.

## What it does

Open one HTML file. It walks the board into its ROM bootloader one labelled step
at a time, tells you what chip actually answered, reads the release archive you
dropped on it, shows you every byte it intends to write and where, waits for you
to approve that, and then writes — and then reads the chip's own MD5 back for
every region and compares it to ours before it will call anything done.

No install. No server. No network at any point. The firmware comes off your
disk; nothing here downloads anything.

## THE MAP

Every instrument in the fleet gets one element it is remembered by. KEEL's is a
to-scale drawing of the chip's whole address space, with each region drawn as a
band on it. Bands fill as bytes land and only go solid when the digest agrees.
Empty space stays empty, so you can see at a glance how much of the chip a run
is *not* touching — which, on the ROM loader, is most of it.

When something fails, the map is left exactly where it stopped. Bands above the
red one are on the chip and verified. The red one is not, and neither is
anything below it. That is more useful than a modal that clears the screen and
says "error".

## Three decisions worth arguing with

**It uses the ROM bootloader and nothing else.** Standard practice is to upload
Espressif's stub loader into RAM first, which is faster and unlocks whole-chip
erase and flash readback. KEEL does not. That costs real capability — there is
no backup, because reading flash out is a stub feature — but it means the
program is one file you can read end to end rather than one file plus an opaque
binary blob it pushes into your chip's memory. Where that limit bites, the
interface says so plainly instead of offering a control that quietly does
nothing. A backup button that does not back anything up is worse than no button.

**Offsets come out of your archive, not out of my code.** Meshtastic has moved
its ESP32 partition layout before and will again. Any table of addresses I typed
into this program would be correct until it silently was not. So KEEL finds
`device-install.sh` inside the release zip, reads the addresses *it* names, and
shows you the script line each one came from. Where a release has no script, the
fallback offsets are drawn in brass everywhere they appear and labelled as
assumptions.

**Flash size is declared, not detected.** Reading the SPI flash ID on the ROM
loader means driving the flash peripheral by hand through register writes. I
could have done it and been mostly right. Mostly right, about the size of the
chip you are about to write, is not a thing worth having. You pick it, and every
screen that uses it says where the number came from.

## The mistake this program is built around not making

The ESP32 ROM returns **two** status bytes on a command response. Every chip
Espressif made afterward returns four. If you read a four-byte status as two,
the failure code lands in the wrong place and a refused write parses as a
success — you get a green screen and a brick.

So the response parser refuses any frame too short to carry a status rather than
guessing, and the self-test contains an assertion whose only job is to prove
that reading the wrong status length does not silently pass. That is the whole
character of the thing: a flasher that says it worked when it did not is worse
than one that will not run.

## Verified is not working

The last screen does not congratulate anyone. An MD5 match proves the bytes are
on the chip. It says nothing about whether SCK is on GPIO 5 where `diy-v1`
expects it, whether there is an antenna on the module, or whether the 915 MHz
part you soldered matches the region you are about to set. Meshtastic ships with
the region deliberately unset and holds the transmitter off until someone sets
it — which is not a fault to debug, and is the first thing a DIY builder
misreads.

So KEEL ends by handing off: reset into the application, watch the boot log, and
if the firmware says it cannot find a radio, that is wiring, not firmware. Then
release the port, because WATCHFIRE wants it next.

---

*KEEL is FI-221. Single file, no network, 79 assertions. Built alongside
WATCHFIRE, which does everything after the bootloader.*
