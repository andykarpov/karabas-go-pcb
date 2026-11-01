# Hotfix: DSKCHG of the floppy connector (all revisions A..C)

The floppy connector J8 does not use pin 34 (DSKCHG, "disk changed"), so the cores could only guess
when a disk was taken out or put in. This hotfix takes pin 34 to the RP2040; the firmware passes it
to the core (`CMD_FDD_CHNG`, karabas-go-firmware `docs/protocol.md`). Boards without the hotfix keep
working as before.

The schematic of every revision from A to C is the same here: U18 (74LVC14, SO-14) has one free
inverter (input pin 5 on GND, output pin 6 not connected), and TP15 (RP2040 GPIO28) is a bare test
point.

## Steps

1. U18 pin 5: lift the leg off its pad (the pad is GND). Do not leave it floating.
2. Connect J8 pin 34 to the lifted U18 pin 5.
3. A 1k resistor from U18 pin 5 to VCC (+5 V), as R23..R26 for INDEX / TR00 / WPRT / RDATA. The
   drive output is open collector: the pull-up belongs to the board, not to the drive. The 74LVC
   input takes 5 V.
4. Connect U18 pin 6 to TP15 (GPIO28).

Through the inverter GPIO28 is high when the selected drive reports a change.

## Check

With `debug=1` in `karabas.ini` the firmware log says at start:

- `Floppy DSKCHG line on GPIO28 (hotfix)`: the wire is there;
- `No floppy DSKCHG line`: GPIO28 is a bare test point (no hotfix, or no contact at U18 pin 6).

The firmware reads GPIO28 with its pull-up and with its pull-down: a bare pad follows the pull, the
gate drives it.

## Drives

PC 3.5" drives give DSKCHG on pin 34. Some drives can be jumpered to give READY there instead (DC /
RDY jumper): set it to DC. Amiga drives have _CHNG on pin 2 and _RDY on pin 34, they need a PC
drive or an adapter.

## Next revision

Route U18 pin 6 to GPIO28 as well (the same firmware and bitstreams then work on every board).
