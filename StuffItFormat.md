# Archive header #

All types larger than one byte in size are stored in big-endian order, as expected from a Mac format.

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 4        | Magic number 1 (Can have various values, see table below) |
| 4          | 2        | Number of files (Meaning not quite clear, perhaps top-level files only) |
| 6          | 4        | Total size of archive |
| 10         | 4        | Magic number 2 (0x724c6175) |
| 14         | 1        | Version (Meaning not quite clear) |
| 15         | 7        | Unknown     |

Known values for "Magic number 1":

| `SIT!` |
|:-------|
| `ST46` |
| `ST50` |
| `ST60` |
| `ST65` |
| `STin` |
| `STi2` |
| `STi3` |
| `STi4` |

The archive header is followed by the first file header.

# File headers #

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 1        | Resource fork compression method |
| 1          | 1        | Data fork compression method |
| 2          | 1        | File name length |
| 3          | 63       | File name   |
| 66         | 4        | Mac OS file type |
| 70         | 4        | Mac OS file creator |
| 74         | 2        | Mac OS Finder flags |
| 76         | 4        | Creation date in classic Mac OS format (seconds since 1904) |
| 80         | 4        | Modification date in classic Mac OS format (seconds since 1904) |
| 84         | 4        | Resource fork uncompressed length |
| 88         | 4        | Data fork uncompressed length |
| 92         | 4        | Resource fork compressed length |
| 96         | 4        | Data fork compressed length |
| 100        | 2        | Resource fork CRC-16 |
| 102        | 2        | Data fork CRC-16 |
| 104        | 6        | Unknown     |
| 110        | 2        | Header CRC-16 |

The "compression method" fields use the lowest four bits to encode the actual compression method, from 0 to 15 (0 means no compression, and see StuffItSpecs for a list of the other algorithms used). Bit 4 indicates whether the data is encrypted (the algorithm used is unknown).

If the compression method field is 32, this indicates that the entry marks the start of a folder. The folder name is taken from the file name field as usual.

If the compression field is 33, this indicates the end of a folder. The size of the header is 112 bytes in all cases.

The compressed data follows each file header. The resource fork data is first, follwed by the data fork data. The next file header is at "Resource fork compressed length" plus "Data fork compressed length" bytes from the end of the last header.

## CRC algorithm ##

The CRC-16 algorithm used is the CCITT one with polynomial 0x1021 and no pre- or post-conditioning.