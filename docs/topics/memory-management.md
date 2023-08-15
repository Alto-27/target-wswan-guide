# Memory management

The WonderSwan poses unique challenges for implementing a robust memory management system:

* The unified memory architecture places display and audio data in shared RAM with the CPU, however its restrictions lead to harsh alignment requirements, and thus the need for non-contiguous memory allocation in RAM.
* The combined segmentation and banking benefits from a ROM layouting scheme which takes it into account.

## Layouting

By default, the following types of code and data will be placed into the following locations:

* Code (`.fartext` section) and read-only data marked `__wf_rom` (`.farrodata` section) - the last 768 KiB "linear" bank of ROM,
* Data not marked `__wf_rom` (`.rodata`, `.data` section) - console RAM.

To change this, one can use the C `section` attribute combined with Wonderful's extensions to section naming. For example:

```c
__attribute__((section(".iramx_2bpp_2000.tiles")))
ws_tile_t tile_2bpp[512];
```

This block of code will allocate 512 tiles worth of memory in:

* internal console RAM (`.iram`),
* without zeroing the content on startup (`x`),
* with constraints expected of 2BPP tile data - the memory area `0x2000` - `0x5FFF`, aligned to a multiple of 16 bytes (`2bpp`),
* at location 0x2000 in memory (`2000`).

The following prefixes exist:

| Prefix | Optional arguments | Description |
| - | - | - |
| `.iram[x]` | offset | Mono internal RAM (0x0000 - 0x3FFF) |
| `.iramc[x]` | offset | Mono/Color internal RAM (0x0000 - 0xFFFF) |
| `.iramC[x]` | offset | Color internal RAM (0x4000 - 0xFFFF) |
| `.iram[...]_screen` | offset | Screen data (aligned to 0x800 bytes) |
| `.iram[...]_sprite` | offset | Sprite table (aligned to 0x200 bytes) |
| `.iram[...]_2bpp` | offset | 2bpp tile data (aligned to one tile) |
| `.iram[...]_4bpp` | offset | 4bpp tile data (aligned to one tile) |
| `.iram[...]_wave` | offset | Audio wavetable data (aligned to 0x40 bytes) |
| `.iram[...]_palette` | offset | Color palette data (aligned to 0x20 bytes) |
| `.rom0` | bank index, offset | ROM bank 0 |
| `.rom1` | bank index, offset | ROM bank 1 |
| `.romL` | bank index, offset | ROM linear bank |

with the following optional arguments:

* **bank index** - the index of a bank, padded to the top of ROM.
* **offset** - the absolute offset within a bank or memory location.

It is good practice to ensure every section name is unique; one can use `.` after the prefix to specify further naming information.
However, variables in a section with the same name are guaranteed to end up in the same bank, which can be useful.

## Layouting in wf-process

wf-process supports an easy syntax for moving data to a specific ROM bank, index, or offset:

```lua
process.emit_symbol("gfx_title_screen", tilemap_color, {
    ["bank"] = 0, -- optional; or 1, or "L"
    ["bank_index"] = "F", -- optional; using a string is preferred over a number
    ["offset"] = 0x1000 -- optional.
})
```

This approach also grants sections unique names by default.

## Working with banks

A given 20-bit linear address to memory contains the *bank type* information (highest 4 bits) and the *offset* (lowest 16 bits), but it does not contain the *bank index* itself.
For this purpose, a pseudo-symbol is defined:

```c
__attribute__((section(".rom0.gfx_font")))
const uint8_t __wf_rom gfx_font[1024] = { ... }; // array "gfx_font"

extern const void *__bank_gfx_font; // address = bank for "gfx_font"
```

To access data in a separate bank, the following method is recommended:

```c
void load_font(void) {
    // Change bank, saving the previous bank value.
    // Assumes "gfx_font" exists at ROM bank 0
    ws_bank_t previous_bank = ws_bank_rom0_save(__bank_gfx_font);

    // Load font data from ROM to IRAM.
    memcpy(MEM_TILE(0), gfx_font, sizeof(gfx_font));

    // Restore the previous bank value.
    ws_bank_rom0_restore(previous_bank);
}
```

!!! note
    `ws_bank_rom0_*` can accept both numeric values and `const void*` pointers; in the latter case, the address of the pointer will be used.

The save/restore pattern ensures that if you're using `ROM bank 0` in a parent function, or outside of an interrupt handler, the value is not changed:

```c
bool load_graphics(void) {
    ws_bank_rom0_set(__bank_gfx_title_screen);

    load_font();

    // If load_font() did not restore the previous bank value,
    // the code below would copy unexpected data.
    memcpy(screen_1, gfx_title_screen, sizeof(gfx_title_screen));

    // Likewise, if the caller of load_graphics() expects the bank value
    // to be preserved, they will be in for a moderately annoying surprise.
    return true;
}
```
