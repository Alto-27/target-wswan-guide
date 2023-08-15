# Development environment

In the first chapter of the tutorial, we're going to prepare everything necessary, or just useful, for the development of homebrew:

* the **toolchain** - a set of tools which allow compiling code for the platform of your choice, as well as libraries which provide commonly used functions; this is essential,
* an **IDE** - a tool which allows more efficient development of software by providing features such as code completion, code-aware searches and automated refactoring; this is not strictly necessary - many people have written code in Notepad - but can be very useful!
* an **emulator** - a software implementation of the hardware of the console, which allows testing homebrew software without the laborious process of re-flashing a physical cartridge. Some emulators also include a **debugger**, allowing tracing and inspection of the program's behaviour.

## Installing the toolchain

For the purposes of this guide, we'll be making use of [the Wonderful toolchain](https://wonderful.asie.pl/docs/getting-started/).
In order to use it, a recent Linux environment is required.

!!! info
    While setting up a full Linux installation is recommended, it is possible to set up a Linux virtual machine instead.
    The Windows Subsystem for Linux 2 (WSL2) is also known to work.

The installation options for the toolchain are provided on the above-linked webpage. However, some additional tools specific to
targetting the WonderSwan need to be installed. This can be done using the following command, provided the toolchain itself has
been properly configured:

```shell
    $ wf-pacman -S target-wswan
```

!!! note
    As part of this guide, the `$` character is used to denote shell commands that you should input. The `$` itself should be omitted; for example, for a line `$ hello`, you're supposed to type `hello` and press ENTER.

## Configuring an IDE

The Wonderful toolchain currently recommends using [VSCodium](https://vscodium.com/) with the [clangd plugin](https://clangd.llvm.org/installation.html)
installed.

!!! note
    Other IDEs and editors with clangd support, such as Sublime Text and vim, should also have full compatibility.
    In addition, the proprietary Visual Studio Code with Microsoft's own C/C++ extensions is supported.

To install the clangd plugin in VSCodium:

1. Select the "Extensions" tab from the left-hand menu.
2. Type "clangd" into the search bar at the top.
3. Select the project extension named "clangd" provided by "llvm-vs-code-extensions".
4. Press the "Install" button and follow any further instructions displayed by the IDE.

## Installing an emulator

Unfortunately, the WonderSwan does not currently have a fully hardware-accurate emulator, and so verification of homebrew
on hardware is still required. The following two emulators are available, but not fully recommended:

* [Ares](https://ares-emu.net/) - medium-high accuracy, but does not provide a debugger;
* [Mednafen](https://mednafen.github.io/) - medium accuracy, but *does* provide [a debugger](https://mednafen.github.io/documentation/debugger.html).
    * Due to unfixed (as of writing) emulation bugs in upstream Mednafen which prevent the execution - not just accuracy - of Wonderful-compiled homebrew, it is recommended to use the [wf-mednafen](https://github.com/WonderfulToolchain/wf-mednafen/releases) fork if you plan on making use of this emulator.

## (Optional) Testing on physical hardware

If you own a WonderSwan, you may wish to test your work on physical hardware.

Unfortunately, the WonderSwan's development hardware situation is not great, and any of the solutions listed below may be out of stock at any given time.

### Flash cartridges

A flash cartridge allows the user to load their own code and run it on physical hardware.

#### WS Flash Masta

The [WS Flash Masta](https://www.flashmasta.com/product/ws-flash-masta-usb-cartridge-for-wonderswan/) is a flash cartridge created by Flavor, providing fifteen slots of 64 megabits (8 megabytes) each for flashing your own code, as well as 512 kilobytes of save RAM.

Available for $100-$110 new when in stock, which is not very often.

If you purchase/own one, installing the [CartFriend](https://github.com/WonderfulToolchain/ws-cartfriend/releases) alternate menu is highly recommended.

#### InsideGadgets flash cartridge

InsideGadgets sells [a compatible flash cartridge](https://shop.insidegadgets.com/product/wonderswan-4mb-8mb-32kb-fram-flash-cart/). These provide 64 megabits (8 megabytes) for code, 32 KB of save RAM.

Available for $54 new when in stock, though a new customer has to spend an additional $10 for the edge adapter and $30 for the USB flasher device, for a total of $94. If you happen to also be interested in GB/GBC/GBA homebrew, said flasher might be a good investment.

!!! note
    It may be possible to use BootFriend to reflash these cartridges using just a WonderSwan console, but this remains untested as of writing.

#### WonderWitch

The [WonderWitch](http://wonderwitch.qute.co.jp/) is an official homebrew solution licensed by Bandai and developed by Qute Corporation. This cartridge provides 512 KB of NOR flash (384 KB user-accessible). However, it is not recommended for two reasons.

First of all, it is very expensive on the second-hard market, with bare cartridges going for over $100 and complete boxed sets going for over $250 as of 2023. Even if you happen to own a bare cartridge, you still need a serial port adapter to get data to the cartridge!

Second of all, the WonderWitch does not run bare WonderSwan ROM images; rather, it provides its own executable format and hardware abstraction libraries.
While this is supported by Wonderful via the "wwitch" subtarget, it is less complete than the "wswan" subtarget, which is the focus of this guide.
In addition, there is currently no free software implementation of the hardware abstraction libraries, making it not possible to run such software by a non-WonderWitch owner without potential copyright infringement.

In other words, the WonderWitch in 2023 is both more restricted and more expensive than modern solutions. However, it *does* feature a large library of freely available homebrew software from back in the day;
if that interests you as an user or collector, it may still be a worthwhile option to look into. Certainly, it is better than having *no* cartridge...

#### Legacy options

These options are no longer available for sale, but are nonetheless listed in case you run into them:

* **WonderDog** - available in 64 megabit (8MB), 32 megabit (4MB) and 4 megabit (512KB) variants.
* **WonderMagic Color** - the only contemporary unofficial flash cartridge for the system; very dated by today's standards. In case you run into one, a copy of the PC flashing software is archived [here](https://mega.nz/file/yw1lgTCJ#2-kOdqdZkmo-V1nBU9U_rx7iefz1nmJqj5t-IbJVExI).

### Serial port adapters

A serial port adapter allows the user to communicate with the WonderSwan using a PC. These are not necessary (unless you're working with the WonderWitch OS),
but can make debugging and data transfer more convenient.

#### ExtFriend

The [ExtFriend](https://github.com/WonderfulToolchain/ws-extfriend) is a firmware for the Raspberry Pi Pico board which acts as an USB serial port adapter for the WonderSwan.
In addition, it allows USB digital audio capture of the WonderSwan's headphone output.

Unfortunately, no sellers of this device are currently known to exist - some DIY is required. For the adventurous, Japanese guide to using an HDMI breakout board is provided [here](https://twitter.com/peca_port0/status/1631569109912817667)

#### WonderWitch

The WonderWitch comes with an RS-232 serial port adapter.

#### RetroOnyx link cable

The [RetroOnyx USB Link Cable](https://www.retroonyx.com/product-page/wonderswan-usb-link-cable) is a high-end modern solution, with custom-designed connectors instead of HDMI ports and an integrated USB-serial
adapter. However, it does ask for $85 in return.
