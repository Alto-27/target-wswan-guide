# File System

TODO

## .fx Files

FreyaOS uses XMODEM transfers to send and receive files through the serial port. Notably, these transfers don't include any file metadata, such as the file's name, size, permissions
or creation date. As such, `.fx` files are used - binary files with a special 128-byte header[^1]:

| Offset | Length | Type | Description |
| ------ | ------ | ---- | ----------- |
| `0x00` | 4 | `#!ws` | Magic string. | 
| `0x04` | 60 | padding | Unused; typically 0xFF. | 
| `0x40` | 16 | char[] | File name; zero-terminated Shift-JIS string. |
| `0x50` | 24 | char[] | User-friendly file name; zero-terminated Shift-JIS string.<br><br>FreyaOS displays the first 12 characters in its file selector. |
| `0x68` | 4 | uint32_t | Unknown. |
| `0x6C` | 4 | uint32_t | Total file size, in bytes, excluding the header. |
| `0x70` | 2 | uint16_t | XMODEM chunk count - above file size divided by 128, then rounded up. |
| `0x72` | 2 | uint16_t | Mode:<br><ul><li>`0x01` - e**x**ecute</li><li>`0x02` - **w**rite</li><li>`0x04` - **r**ead</li><li>`0x20` - **i**ntermediate library</li></ul> |
| `0x74` | 4 | uint32_t | Modification time - seconds since January 1st, 2000. |
| `0x78` | 4 | uint32_t | Unknown. |
| `0x7C` | 4 | int32_t | Offset to resource data, excluding the header; -1 if not present. |

The Wonderful toolchain's `wf-wwitchtool mkfent` can be used to create `.fx` files.

[^1]: XMODEM transfers are performed in 128-byte blocks; this allows fetching the file's metadata as the first block.
