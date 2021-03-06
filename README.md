# up5k_basic
A small 6502 system with MS BASIC in ROM. This system
includes the following features:

* Up to 52kB SRAM with optional write protect (using two of the four available SPRAM cores)
* 8 bits input, 8 bits output
* 115200bps serial I/O port
* NTSC monochrom composite video with text/glyph and graphic modes,
  32kB video RAM (4 8kB pages) and original OSI 2kB character ROM
* 2kB ROM for startup and I/O support
* 8kB Ohio Scientific C1P Microsoft BASIC loaded from spi flash into protected RAM
* SPI port with access to 4MB flash memory
* LED PWM driver
* PS/2 Keyboard port
* 4-voice sound generator with 1-bit sigma-delta output

![screenshot](screenshot.png)

## prerequisites
To build this you will need the following FPGA tools

* Icestorm - ice40 FPGA tools
* Yosys - Synthesis
* Nextpnr - Place and Route (newer than Mar 23 2019)

Info on these can be found at http://www.clifford.at/icestorm/

You will also need the following 6502 tools to build the startup ROM:

* cc65 6502 C compiler (for default option) https://github.com/cc65/cc65

## Building

	git clone https://github.com/emeb/up5k_basic.git
	cd up5k_basic
	git submodule update --init
	cd icestorm
	make

## Loading

I built this system on a custom up5k board and programmed it with a custom
USB->SPI board that I built so you will definitely need to tweak the programming
target of the Makefile in the icestorm directory to match your own hardware.
Note that the 8kB BASIC ROM must now be loaded into the SPI configuration
flash memory starting at offset 0x40000 in order for BASIC to run correctly.
You can find a link to the ROM data at the end of this document.

## Running BASIC

You will need to connect a PS/2 keyboard to the ps2_clk/dat pins, or a
115200bps serial terminal port to the TX/RX pins of the FPGA - the data input
routines can take characters from either or both.

Load the bitstream an you'll see the boot prompt:

    D/C/W/M?

This is asking if you're doing a cold or warm start. Hit "C" (must be
uppercase) and then BASIC will start running. It will prompt you:

    MEMORY SIZE?

to which you answer with 'enter' to let it use all memory. It then prompts
with:

    TERMINAL WIDTH?

Again, hit 'enter' to use the default. It then prints a welcome message and
is ready to accept BASIC commands and code. You can find out more about
how to use this version of BASIC here: https://www.pcjs.org/docs/c1pjs/

## C'MON

Answering "M" to the boot prompt will start the C'MON monitor which allows
examining and editing memory contents as well as executing machine code.

## Boot ROM

The 2kB ROM located at $f800 - $ffff contains the various reset/interrupt
vectors, initialization code, the C'MON monitor and I/O routines needed to
support BASIC. It's extremely stripped-down and just handles character input,
output and Control-C checking - the load and save vectors are currently stubbed
out.

You can revise this ROM with your own additional support code, or to reduce the
size of the ROM to free up resources for more RAM - you'll need to edit the
linker script to change the memory sizes, as well as modify the rom_monitor.s
file with code changes. The cc65 assembler and linker are required to put it
all together into the final .hex file needed by the FPGA build.

## Video

This is a simple NTSC composite video generator which
is based on the original Ohio Scientific C1P system. The luma and sync output
bits should be combined by running sync thru a 560 ohm resistor and luma thru a
330 ohm resistor to a common node driving a 75 ohm baseband composite video
load. This will generate a 262-line 60fps progressive-scanned NTSC-compatible
signal which should work with most modern NTSC-capable video displays.

The video generation has been upgraded from the OSI 24x24 display

* Memory arbitration to prevent glitching when the 6502 accesses video RAM
* 256 x 224 B/W pixel graphics mode
* 4 pages text or graphics
* 32 characters wide by 27 lines high plus overscan using the original OSI
character generator, complete with all the unique gaming glyphs like tanks,
cars and spaceships as shown in this rendering:

![characters](chargen1x.png)

## SPI

The iCE40 Ultra Plus features two SPI and two I2C ports as hard IP cores that
are accessible through a "system bus" that's similar to the popular Wishbone
standard. I've added a 6502 to Wishbone bridge mapped to addresses $F100-$F1FF
which provides access to all four cores. Currently only the SPI core at
addresses $F106-$F10F is connected but I've tested it and added some bare-bones
access routines in the support ROM which can communicate with the SPI flash
memory on my test PCB. This requires features that were added to nextpnr on 
March 23, 2019 so make sure you've got the latest from git.

## LED PWM

Many FPGAs in the iCE40 family provide hard IP cores for driving RGB LEDs. A
simple interface to this is provided so the 6502 may control the LED driver.

## Sound Generator

A 4-voice sound generator is provided which supports pitch from 0-32kHz in 
roughly 0.5Hz steps, choice of waveform (saw/square/triangle/noise) and
volume control on each voice. Output is via a 1-bit sigma-delta process
which requires a simple 1-pole RC filter (100ohm + 0.1uf) lowpass filter
to smooth the digital pulse waveform down to analog audio.

## Simulating

Simulation is supported and requires the following prerequisites:

* Icarus Verilog simulator http://iverilog.icarus.com/
* GTKWave waveform viewer http://gtkwave.sourceforge.net/

To simulate, use the following commands

	cd icarus
	make
	make wave

This will build the simulation executable, run it and then view the output.

## Copyright

There have been questions raised about the copyright status of the MS BASIC
provided in this project. To the best of my knowledge, the contents of the file
src/basic_8k.hex is still property of Microsoft and is used here for educational
purposes only. The full source code for this can be found at:

https://github.com/brajeshwar/Microsoft-BASIC-for-6502-Original-Source-Code-1978

The ROM files from which I created the .hex file are also available in many
places - I used this archive: http://www.osiweb.org/misc/OSI600_RAM_ROM.zip

## Thanks

Thanks to the developers of all the tools used for this, as well as the authors
of the IP cores I snagged for the 6502 and UART. I've added those as submodules
so you'll know where to get them and who to give credit to.
