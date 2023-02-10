# Game Blaster (CT-1300)

Here's the partially reverse engineered Game Blaster/Creative Music System card. Some information is still missing, particularly for the ID chip, but also the values of a few capacitors. Some guesses were made about traces on the top layer hidden from view by chips or other components. That's right, this is based on a few photos of the card I was able to find.

This isn't a complete design. Don't expect to be able to fab it out and have a working card. I'm putting this up in the hopes that someone might find it a useful reference for further reverse engineering work.

![Game Blaster layout photo](https://github.com/schlae/game-blaster/blob/main/GameBlaster.png)

[Schematic](https://github.com/schlae/game-blaster/blob/main/GameBlaster.pdf)


## Unknowns

Here's a list of everything that is not currently known about the card.

* Values of C14, C21, C22, the bodge capacitor on the back of the board, C23, C24, C25, C26, C27, C28, C36, zener diode Z1
* Traces on the top layer underneath IC4, IC8, and IC9 are mostly guesses
* The identity of the mystery chip, IC4.
* Differences between revision A and revision B.

## Mystery ID chip

### Identifiers

This chip is a 40-pin DIP marked as follows:

```
8827
CT 1302 A
CTPL 8708
```

```
CT1302A
CTPL8708
28840HA05
```

```
CT1302A
CTPL8708
28844HA05
```

Some cards have a chip that has been sanded down and labeled with a plain paper sticker:

```
C.M.S.
CT1302
```

If we believe the markings, we might assume that this is an OTP gate array device or potentially a mask-programmed gate array. "CTPL" might stand for "Creative Technology Programmable Logic" and "8708" might refer to the year 1987 and the work week 8, which implies that this is when Creative released the data files to the company who manufactured the chip.

The remaining lines on the chip, "28844HA05" or "28840HA05" or "8827" seem to imply several different manufacture dates for the chips, all in 1988, work weeks 27, 40, and 44.

Of course, since chip markings are easily customizable, this might all be designed to throw competitors off the trail, and it's an off-the-shelf device that has been repurposed.

### Functionality

The chip provides several different functions:

* 8-bit data bus connection (bidirectional). This is wired directly to the ISA data bus.
* Two 8-bit ports. One each is wired to the SAA1099 sound generator chips. Each port has a control line that seems to be used to latch data into the port from the ISA data bus.
* Card ID register. When writing port 226h or reading port 22Ah, external logic selects some sort of internal 8-bit latch in the chip. Basic on the connections of AND gate IC8D, this latch may be shared with port B, which implies that the values latched in each port can be read back.
* Some type of shift register. It is not known if the shift register is coming from one of the existing port registers or is standalone. This is accessed by reading from port 224h and examining bit 7 (the MSB). Plenty of software assumes this is always 7Fh, but the CMSDRV.COM driver program checks for a pattern of 1s and 0s on the MSB when performing 8 consecutive read operations from this port.

### Partial Pin Description

| Pin | Direction | Description |
|-----|-----------|-------------|
| 1   |           | Gnd         |
| 2   | Input, active low | Latches the data bus on port A |
| 3-10 | Output   | Port A, output only? |
| 11 | Input, active low | Latches the data bus on port B |
| 12-19 | Output  | Port B, output only? |
| 20 | | VCC |
| 21 | | Gnd |
| 22 | Input | Unknown, strapped to VCC. Chip select? |
| 23 | Input | Unknown, strapped to VCC. Chip select? |
| 24, 25 | Inputs, active low | One could be a R\/W# line and the other a register select. | 
| 26 | Output | Unknown, output of a shift register? |
| 27 | Input, active low | Unknown, asserted to read the contents of port B |
| 28 | Input, active low | Unknown, asserted to reset the shift register? |
| 29 | Input, active low | Selects the shift register for access, or perhaps acts as the shift clock for the shift register |
| 30, 31 | | Unknown, likely no connect pins |
| 32-39 | Bidirectional | Connection to ISA data bus |
| 40 | | VCC

### CMSDRV.COM Code

I've reverse engineered parts of the CMS driver provided with the card, and learned a few things.

* The program performs a partial checksum of itself, writes the value out to the card ID register, and reads it back later on. The value is expected to be A0h.
* The program attempts to disguise the results of card detection by "encrypting" the card not detected string. The string is just XORed with 07h.
* The program uses self-modifying code in a few places, presumably to hamper reverse engineering efforts. Typically it just overwrites literal values that are part of MOV instructions. This is used to change the INT 21h from a 4Ch command (exit) to a 31h command (terminate and stay resident).
* After initializing the SAA1099 registers, the program reads from 224h eight times and checks bit 7. the first four values are ignored, but the last four must match the pattern `1100`.
* Right before terminating (and staying resident) the program stores a magic number at 228h.

## Next Steps

We need someone who has a card to examine it and provide photos from different angles so we can read the values of a bunch of the capacitors. Photos at extreme angles next to the column of chips on the left side of the board could help us follow the top layer traces. As an alternative, someone with a multimeter/continuity tester could help by measuring a few points to confirm connectivity.

Learning more about the mystery ID chip has a few paths forward:
* Someone out there may recognize the chip from the partial pinout
* Connect a logic analyzer to every pin of the chip, start CMSDRV.COM, and observe the waveforms
* Desolder the chip and experiment with unused functionality (strapped pins, unconnected pins, and so on)
* Destructively decapping the chip would be the simplest, but I doubt anyone would want to sacrifice a card for that. :)


