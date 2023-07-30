# Hello, Display!

In this lesson, we're going to learn the basics of compiling and testing a WonderSwan homebrew project; by the end of it, we will write basic code to display something on the screen.

First, set up a new project:

```shell
    $ mkdir 1-hellodisplay/ # (1)!
    $ wf-wswantool project new 1-hellodisplay/ # (2)!
```

1. Create the directory "1-hellodisplay" in the terminal's current working directory.
2. Create a new wswan targetting project with the directory and name "1-hellodisplay".

Next, compile the project:

```shell
    $ cd 1-hellodisplay/
    $ make
```

This will output the following lines:

```shell
  CC      src/main.c # (1)!
  ROMLINK 1-hellodisplay.wsc # (2)!
romlink: program size is 64 bytes, load at FFFB:0000
  MERGE   compile_commands.json # (3)!
```

1. The C compiler is compiling the source file `src/main.c`.
2. The ROM file `1-hellodisplay.wsc` is being linked.
3. The `compile_commands.json` file is being generated. This is used by your IDE to provide syntax highlighting; as a consequence, it is a good idea to run `make` after adding a new `.c` file to the project.

Finally, you can run the ROM file using your emulator of choice. However, it's not displaying anything - after all, there's no code written yet.

Let's examine the `src/main.c` file:

```c
// SPDX-License-Identifier: CC0-1.0
//
// SPDX-FileContributor: Adrian "asie" Siekierka, 2023
// (1)!
#include <wonderful.h> // (2)!
#include <ws.h> // (3)!

void main(void) {
	while(1);
}
```

1. The default copyright header for the template file. As per the [Introduction](../index.md), this is required to inform you that you can use the code contained without restrictions. However, before writing your own code, if you wish to put different terms on your code, you should remove it.
2. The `wonderful.h` include contains common CPU/target definitions. It's a good idea to put it in essentially every C file.
3. The `ws.h` include contains hardware definitions specific to the WonderSwan, as well as many lower-level hardware abstraction functions.

!!! warning
    Pay special attention to the `while(1);` at the end - this is an infinite loop.
    
    You may be used to ending `main()` with a `return;` of some type. On bare metal, however, one should not return from main, as there's no operating system to return to. Removing it will cause the system to jump to arbitrary code with unforeseen consequences!

With only an infinite loop in `main`, it's clear that the code won't do anything. Let's make it do something!

For now, we're going to assume a "mono" program - we'll introduce the color mode in a later chapter. On the "mono" WonderSwan, the palette pipeline that converts tile information to display shades consists of two parts: the *palette*, mapping each of the four possible palette *indexes* to one of eight color values; and a *shade look-up table*, mapping each of the color values to one of the sixteen total shades which the panel can display.

``` mermaid
graph LR
A[Palette index<br><i>one of four colors</i>] -->|palette<br><i>one of sixteen</i>| B[Color value<br><i>one of eight values</i>] 
B -->|shade look-up table<br><i>global</i>| C[Shade<br><i>one of sixteen values</i>]
```

First, we need to initialize the shade look-up table. At the beginning of `main`, add the following function call:

```c
    ws_display_set_shade_lut(SHADE_LUT_DEFAULT);
```

This sets a reasonable default table, with the color value `0` corresponding to the brightest shade (`0`) and `7` to the darkest shade (`15`).

!!! tip
    As the "mono" WonderSwan's panel is a monochrome LCD, it's shades with *higher* values which are darker; the opposite is true for the WonderSwan Color's RGB values.

Next, we need to enable the display. We're going to enable the "screen 1" layer - as the WonderSwan's memory is zeroed by default, this should not cause anything to be drawn to the screen, but will enable viewing the background color - which, by default, is the palette index `0` of the palette `0`:

```c
    outportw(IO_DISPLAY_CTRL, DISPLAY_SCR1_ENABLE);
```

Finally, we're going to do some changes to the display. Edit the `while(1)` loop as follows to cycle the first palette:

```c
    while (1) {
		outportw(IO_SCR_PAL_0, inportw(IO_SCR_PAL_0) + 1); // (1)!
		ws_busywait(65535); // (2)!
    }
```

1. This loads the first palette, adds one to it, and saves it. As the lowest bits constitute the color value for the first palette index, this will cause the background color to change on every execution.
2. As we don't have any interrupts set up yet, the only thing we can do is busy-wait - that is, stall the CPU for a specified number of microseconds. While this is not recommended for production code, it's simple enough to prevent rapid flicker in the demonstration.

That's all! Compile the code using `make` and run it in your emulator of choice. If done correctly, you should see the display slowly change colors from brightest to darkest and back again.
