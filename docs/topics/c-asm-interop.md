# Using ASM code in a C project

While the gcc-ia16 compiler provides decent levels of optimization, specialized and/or heavily used code can often benefit from a hand-written rewrite in assembly, or the use of some assembly snippets.

!!! note
    This guide does not cover the subject of writing assembly for the NEC V30MZ by itself.

    The NEC V30MZ is, from a typical code writing perspective, indistinguishable from an Intel 80186; as such, any 8086/80186 assembly tutorials and reference guides are applicable.

## Inline assembly

The gcc-ia16 compiler supports inline assembly written with the AT&T syntax using the `__asm` keyword.

!!! warning
    Most standalone source code in the Wonderful toolchain uses the Intel syntax instead.

For example, to enable CPU interrupts, we could write the following function:

```c
void enable_cpu_interrupts(void) {
    __asm ("sti");
}
```

However, this wouldn't be ideal. The optimization passes of the compiler may freely re-order instruction blocks, including the interrupt enabler we just wrote;
this could lead the CPU interrupts to be enabled at a different time in the function's execution than we expect it! To prevent this, we can add the `volatile` keyword, like so:

```c hl_lines="2"
void enable_cpu_interrupts(void) {
    __asm volatile ("sti");
}
```

### Passing values between C and assembly

A more advanced form of using inline assembly is specifying constraints. The `__asm` keyword supports three types of constraints:

```c
    __asm ("code",
        : outputs // (1)!
        : inputs // (2)!
        : clobbers // (3)!
    );
```

1. A list of variables to assign to register values at the end of the assembly block.
2. A list of registers to assign the values of specified variables at the beginning of the assembly block.
3. A list of registers which may be arbitrarily modified (or *clobbered*) by the assembly block.

Each constraint consists of a type and expression - for example, `"a" (value)` connects the constraint `"a"` - the register AX - to the variable `value`.
If we wish for a constraint to be an assignment, we should prefix the type with the character `=`.

For example, we could model an I/O port output call: one which has inputs - the port and the value to write - but no outputs:

```c
void outportw(uint8_t port, uint16_t value) {
    __asm volatile (
        "outw %0, %1"
        :
        : "a" (value), "Nd" ((uint16_t) port)
    );
}
```

Every constraint maps to a `%`-prefixed value in order - so `%0` is mapped to `value`, and `%1` is mapped to `port`.

Another example could involve querying the current value of the code segment:

```c
uint16_t get_cs_value(void) {
    uint16_t result;
    __asm (
        "mov %%cs, %0" // (1)!
        : "=r" (result)
    );
    return result;
}
```

1. Note that as percentages are used for constraint mapping, specifying an actual *code* percentage requires writing `%` twice.

Finally, we could look at a more complex example: performing a FreyaOS interrupt call.

```c
uint16_t sys_get_version(void) {
        uint16_t result;
        __asm volatile (
                "int $0x17"
                : "=a" (result)
                : "Rah" ((uint8_t) 0x12)
                : "cc", "memory"
        );
        return result;
}
```

This ASM block will set `AH` to the constant `0x12`, then call the interrupt, then set `result` to the value of `AX`. As this is an interrupt call, we may not be certain what it will modify - so we assume it modifies both flags and memory.

### Some constraints available in gcc-ia16

| Type | Description |
| ---- | ----------- |
| `X` | Any operand. |
| `m` | Memory address. |
| `g` | Any non-segment register, memory address or immediate integer. |
| `r` | Any register. |
| `a` | The register `AX`; for an 8-bit operand, the register `AL` or `AH` as chosen by the compiler. |
| `b` | The register `BX`. |
| `c` | The register `CX`. |
| `d` | The register `DX`. |
| `S` | The register `SI`. |
| `D` | The register `DI`. |
| `Ral` | The register `AL`. |
| `Rah` | The register `AH`. |
| `Rcl` | The register `CL`. |
| `Rbp` | The register `BP`. |
| `Rds` | The register `DS`. |
| `e` | The register `ES`. |
| `q` | Any 8-bit register - `AL`, `AH`, `BL`, `BH`, `CL`, `CH`, `DL`, `DH`. |
| `T` | Any 16-bit register, including segment registers. |
| `A` | The 32-bit register pair `DX:AX`. |
| `j` | The 32-bit register pair `BX:DX`. |
| `l` | Lower 8-bit registers - `AL`, `BL`, `CL`, `DL`. |
| `u` | Upper 8-bit registers - `AH`, `BH`, `CH`, `DH`. |
| `k` | Any 32-bit register pair, where the lower word is one of `AX`, `BX`, `CX`, `DX`. |
| `x` | The register `SI` or `DI`. |
| `w` | The register `BX` or `BP`. |
| `B` | The register `SI`, `DI`, `BX` or `BP`. |
| `Q` | A free segment register - `DS` or `ES`. |

#### Constants

| Type | Description |
| ---- | ----------- |
| `i` | Any numeric constant. |
| `Z` | The constant `0`. |
| `P1` | The constant `1`. |
| `M1` | The constant `-1`. |
| `Um` | The constant `-256`. |
| `Lbm` | The constant `255`. |
| `Lor` | Constants 128 through 254. |
| `Lom` | Constants 1 through 254. |
| `Lar` | Constants -255 through 129. |
| `Lam` | Constants -255 through -2. |
| `N` | Constants 0 through 255. |

#### Clobbers

| Type | Description |
| ---- | ----------- |
| `cc` | Processor flags. |
| `memory` | Memory. |

## Writing assembly files

For writing entire functions in ASM, as well as more complex projects, it can be preferable to write external assembly files.

In the Wonderful toolchain, these files use the `.s` extension and are compiled using the GNU assembler.

It is recommended to start with a preamble similar to the following:

``` c
#include <wonderful.h> // (1)!
#include <ws.h> // (2)!

    .code16 // (3)!
    .arch i186 // (4)!
    .intel_syntax noprefix // (5)!
```

1. Include the Wonderful toolchain's basic definitions. This provides some useful assembly defines and macros.
2. Include the libws library's definitions. This library supports being included in an assembler, and provides hardware-related defines. You may wish to skip this line if targetting the WonderWitch.
3. Tell the assembler to emit 16-bit code; this is useful as the GNU assembler supports all kinds of x86 code, including 32-bit and 64-bit.
4. Tell the assembler to emit 80186 code; this unlocks some additional 80186-exclusive opcodes supported on the NEC V30MZ
5. Tell the assembler to emit Intel-syntax assembly; this is entirely optional, but the examples for external assembly files in this guide all make use of the Intel syntax, as opposed to the AT&T syntax used in inline assembly.

### C calling convention

A C compiler typically defines a *calling convention* - a set of rules used for mapping C function arguments and return values to the CPU's registers and stack, allowing
the *caller* (the function making the call) and the *callee* (the function being called) to put and retrieve data from the same locations.

If you're writing both the callee and caller in assembly, you are free to define your own calling convention. However, if you're interfacing with C code, you will need
to follow the C compiler's expectations for where this data is to be placed.

!!! note
    Wonderful currently uses the `20180813` version of the `regparmcall` calling convention as defined by the gcc-ia16 compiler.

#### Function arguments

For typical functions, the first three arguments or words, whichever comes first, are passed via the registers
`AX`, `DX` and `CX`, in this order. Bytes are passed via `AL`, `DL` and `CL`. The remaining arguments are pushed
onto the stack.

Arguments are not split between registers and stack. A far pointer, or 32-bit integer, will be passed via `DX:AX`, `CX:DX`, or entirely on the stack.

For functions which contain *variable arguments*, all arguments are pushed onto the stack. It is the callee's responsibility to remove arguments off the stack.

For example, the following function signature:

```c
    void outportw(uint8_t port, uint16_t value);
```

results in the argument `port` being passed in the register `AL`, and `value` - in the register `DX`.

The following function signature:

```c
    void __far* memcpy(void __far* s1, const void __far* s2, size_t n);
```

results in the following calling convention:

 * `DX:AX` = `s1`,
 * stack (4 bytes allocated) = `s2`,
 * stack (2 bytes allocated) = `n`,
 * the value is returned in `DX:AX`.

#### Caller/callee responsibilities

The registers `AX`, `BX`, `CX`, `DX` can be modified by the callee freely. This means that the caller *cannot* expect their value to stay the same before and after calling the function.

Conversely, the remaining registers - `SI`, `DI`, `BP`, `DS`, `ES` and `SS` must be *saved* by the callee. This allows the caller to expect their value to stay unchanged at the expense of the callee, who
is now the party needing to ensure that this is the case - that is, that their values are the same at the beginning and end of the function.

