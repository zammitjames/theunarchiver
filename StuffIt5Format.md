# Archive header #

All types larger than one byte in size are stored in big-endian order, as expected from a Mac format.

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 80       | Magic string. Has the value "StuffIt (c)1997-???? Aladdin Systems, Inc., http://www.aladdinsys.com/StuffIt/" followed by 0x0d 0x0a, where characters marked "?" can vary. |
| 80         | 4        | Unknown     |
| 84         | 4        | Total size of archive |
| 84         | 4        | Offset of first entry header |
| 86         | ?        | Unknown     |

# Entry header #

Files and folders use different entry formats.

## File header ##

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 4        | Identifier (0xa5a5a5a5) |
| 4          | 1        | Version     |
| 5          | 1        | Unknown     |
| 6          | 2        | Header size |
| 8          | 1        | Unknown     |
| 9          | 1        | Flags       |
| 10         | 4        | Creation date in classic Mac OS format (seconds since 1904) |
| 14         | 4        | Modification date in classic Mac OS format (seconds since 1904) |
| 18         | 4        | Offset of previous entry |
| 22         | 4        | Offset of next entry |
| 26         | 4        | Offset of parent folder entry |
| 30         | 2        | Filename size |
| 32         | 2        | Header CRC-16 |
| 34         | 4        | Data fork uncompressed length |
| 38         | 4        | Data fork compressed length |
| 42         | 2        | Data fork CRC-16 (Set to zero for method 15) |
| 44         | 2        | Unknown     |
| 46         | 1        | Data fork compression method (StuffIt 5 seems to only use 0, 13 and 15. See StuffItSpecs for descriptions of the algorithms.) |
| 47         | 1        | Pasword data length (meaning unclear) |
| 48         | N        | Password information (meaning unclear) |
| 48+N       | M        | File name   |
| 48+N+M     | 2        | Comment size |
| 50+N+M     | 2        | Unknown     |
| 52+N+M     | K        | Comment     |

The known bits of the "Flags" field are:

| **Bit** | **Meaning** |
|:--------|:------------|
| 5       | Encrypted (algorithm used is unknown) |
| 6       | Folder      |

## Folder header ##

If bit 6 of the "Flags" field is set, the entry is a folder, and the format of the header is slightly different.

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 4        | Identifier (0xa5a5a5a5) |
| 4          | 1        | Version     |
| 5          | 1        | Unknown     |
| 6          | 2        | Header size |
| 8          | 1        | Unknown     |
| 9          | 1        | Flags       |
| 10         | 4        | Creation date in classic Mac OS format (seconds since 1904) |
| 14         | 4        | Modification date in classic Mac OS format (seconds since 1904) |
| 18         | 4        | Offset of previous entry |
| 22         | 4        | Offset of next entry |
| 26         | 4        | Offset of parent folder entry |
| 30         | 2        | Filename size |
| 32         | 2        | Header CRC-16 |
| 34         | 4        | Data fork uncompressed length |
| 38         | 4        | Data fork compressed length |
| 42         | 2        | Data fork CRC-16 (Set to zero for method 15) |
| 44         | 2        | Unknown     |
| 46         | 2        | Number of files |
| 48         | N        | Password information (meaning unclear) |
| 48+N       | M        | File name   |
| 48+N+M     | 2        | Comment size |
| 50+N+M     | 2        | Unknown     |
| 52+N+M     | K        | Comment     |

If the "Data fork uncompressed length" field is set to "0xffffffff" for a folder entry, this entry is a special marker whose meaning is unknown. It should be ignored when reading.

## Second file header ##

The file header is followed by a second block of data of varying size.

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 2        | Flags 2. Bit 0 indicates the presence of a resource fork. |
| 2          | 2        | Unknown     |
| 4          | 4        | Mac OS file type |
| 8          | 4        | Mac OS file creator |
| 12         | 2        | Mac OS Finder flags |
| 14         | 2        | Unknown     |
| 16         | 4        | Unknown (A date value in version 3?) |
| 20         | 12       | Unknown     |
| 32         | 4        | Unknown, not included in version 3, included in version 1. |
| 32/36      | 4        | Resource fork uncompressed length. Only included if bit 0 of "Flags 2" is set. |
| 36/40      | 4        | Resource fork compressed length. Only included if bit 0 of "Flags 2" is set. |
| 40/44      | 2        | Resource fork CRC-16 (Set to zero for method 15). Only included if bit 0 of "Flags 2" is set. |
| 42/46      | 2        | Unknown     |
| 44/48      | 1        | Resource fork compression method |
| 45/49      | 1        | Unknown     |

The compressed data follows the second file header. The resource fork data is first, follwed by the data fork data. The next file header is at "Offset of next entry" bytes from the start of the file.

## CRC algorithm ##

The CRC-16 algorithm used is the CCITT one with polynomial 0x1021 and no pre- or post-conditioning.