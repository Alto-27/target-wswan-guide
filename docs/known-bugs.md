# Known bugs

Due to the niche nature of the platform and a small resulting availability of development time, the toolchain has some
known bugs. These are listed here, complete with workarounds.

## Bugs

### Far function pointers

In some cases, when calling pointers from arrays of far function pointers in optimization modes >= `-O1`,
the code will be miscompiled. This is [a known issue](https://github.com/tkchia/gcc-ia16/issues/120), with no ETA for a fix.

One can work around this by annotating the affected function to be compiled without optimizations:

```c
__attribute__((optimize("-O0"))) // https://github.com/tkchia/gcc-ia16/issues/120
void call_to_my_function_table(uint8_t index) {
        my_function_table[index]();
}
```
