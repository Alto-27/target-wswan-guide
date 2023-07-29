# Brief hardware overview

Before trying to design homebrew - be it a game, a demo or a tool - for a platform, it is a good idea to be aware of its general features and constraints.

The WonderSwan is a handheld console released by Bandai in March 1999, with an enhanced Color model released in December 2000. This part of the tutorial serves as a brief overview of its capabilities.

## CPU

The CPU is an NEC V30MZ CPU, clocked at 3.072 MHz. This CPU is compatible with the Intel 80186, an expanded version of the 16-bit 8086 CPU whose distant grandchildren power PCs to this day.
However, its timing characteristics are very different - in a good way! - to those of the early 80s chip; the V30MZ was designed in the 1990s, and as such features much better IPC (instructions per clock) than the original Intel part. The CPU also has little to do with the older NEC V20/V30.

The CPU exposes a 20-bit (capable of addressing 1 megabyte) bus, albeit segmentation means it is accessed in terms of 64 KB areas; cartridges can be larger than that, which adds banking on top of the list of concerns.

Note that while NEC provides their own opcode and register names, the Wonderful toolchain uses the Intel names. As such, you are better off using tutorials and guides targetting the 8086/80186.

## Memory

The WonderSwan features 16 kilobytes of internal memory. The WonderSwan Color bumps that to a total of 64 kilobytes.

Unlike most 8/16-bit platforms, the WonderSwan has a unified memory architecture - CPU, video and audio data are all stored in the same internal memory at different locations.

The WonderSwan Color also adds general DMA, capable of efficiently copying data from the internal or cartridge memory to the internal memory.

## Graphics

The WonderSwan features a 224x144 display, capable of simultaneously[^1] displaying up to 8 out of 16 shades of gray, or 241 out of 4096 colors on the WonderSwan Color through:

[^1]: Without mid-line register manipulation tricks.

* a background color,
* two overlapping 32x32 tile layers ("screens"),
* up to 128 8x8 sprites, 32 per line.

Unusually, the display is clocked at 75.47 Hz.

The default mode for tile storage is to use 2-bit-per-pixel data plus one of sixteen palettes to select the shades or colors. The palettes are configured as follows:

* Palettes 0-15 are available for the screens, while only 8-15 are available for spirtes.
* Palettes 0-3 and 8-11 feature an opaque color "zero", while 4-7 and 12-15 assume it to be transparent.

The WonderSwan Color augments this with two additional modes, which allow displaying 4-bit-per-pixel tile data, albeit with color "zero" always transparent.

In addition to the display, there are also six user-controlled segments to the right or bottom of the console, which can be used as indicators.

Additional hardware features include:

* displaying sprites both beneath and on top of the second screen,
* limiting the second screen to be drawn inside or outside of a specified rectangle,
* limiting the sprites to be drawn inside a specified rectangle,
* hardware screen scrolling,
* hardware tile horizontal and vertical flipping.

## Sound

The WonderSwan features four channels of audio, each capable of playing 32 x 4-bit wavetable samples with a specified frequency and separate left/right volume.
The audio circuit is digital, outputting samples at 24000 Hz; the built-in speaker is mono, whereas the headphone output allows for stereo.

Channel 2 can be optionally configured to play 8-bit unsigned PCM samples, while channel 4 can be optionally configured to play pseudorandom noise.

The WonderSwan Color augments this with:

* "Hyper Voice", a fifth channel capable of playing stereo PCM samples,
* Sound DMA, allowing for relatively inexpensive copying of sample data from ROM or RAM to channel 2 or Hyper Voice.

## Input

The WonderSwan features eleven keypad inputs: the two directional diamonds on the left (Y1-Y4, X1-X4), as well as the A, B and Start buttons.

Notably, the arrangement of the directional diamonds allows an alternate layout, where the WonderSwan is held vertically.

## Cartridge

The cartridge is capable of anything, depending on the mapper chip. For the purposes of this guide, we're going to discuss the features of the official Bandai mapper chips - the 2001 and 2003.

The Bandai 2001 allows up to 16 megabytes of ROM data, while the 2003 allows up to 64 megabytes.

!!! warning
    Even though the theoretical limit of Bandai's chips is 64 MB, there are no commercial games larger than 16 MB, and no broadly available flash cartridges capable of playing games larger than 8 MB. *Caveat emptor.*

Memory banking is provided by splitting the address space into four areas:

* `0x10000 - 0x1FFFF` - SRAM
* `0x20000 - 0x2FFFF` - ROM, bank 0
* `0x30000 - 0x3FFFF` - ROM, bank 1
* `0x40000 - 0xFFFFF` - ROM, "linear" bank 2

Banks 0 and 1 can point to any 64KB part of the ROM. Bank 2 can point to any 1MB part of the ROM, but only its top[^2] 768KB is exposed.

[^2]: When discussing memory, "bottom" typically refers to the *first* address of memory, while "top" refers to the *last* address of memory.

Optionally, a cartridge can provide EEPROM, as well as a real-time clock.

## Other

Additional hardware features include:

* 128 bytes (extended to 2 kilobytes in the Color) of internal EEPROM, of which a small amount of bytes can be used by user programs,
* A 96000/38400 baud serial port, usable for connecting to a PC, another WonderSwan or a peripheral.

## More information

More detailed hardware documentation is listed as part of [awesome-wsdev](https://github.com/WonderfulToolchain/awesome-wsdev).
