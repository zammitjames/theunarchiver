# File format #

## Archive header ##

The only magic number for a Compact Pro file is that the first byte is "0x01". All types larger than one byte in size are stored in big-endian order, as expected from a Mac format.

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 1        | File identifier, 0x01 |
| 1          | 1        | Volume number (meaning not entirely clear, is 0x01 for single-volume archives) |
| 2          | 2        | Cross-volume magic number (meaning not entirely clear) |
| 4          | 4        | Offset to file headers from beginning of file |

The offset points to a second part of the archive header, followed by file and directory headers. The second part is:

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 4        | CRC-32 of the header |
| 4          | 2        | Total number of files and directories |
| 6          | 1        | Comment length |
| 7          | N        | Comment with the length indicated above |

Next come as many file or directory headers as indicated by the header.

## File header ##

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 1        | Name length and type flag. The highest bit of this field is zero for a file entry. |
| 1          | N        | File name with the length indicated above |
| N+1        | 1        | Volume number (meaning not entirely clear) |
| N+2        | 4        | Offset to file data from beginning of file |
| N+2        | 4        | Offset to file data |
| N+6        | 4        | Mac OS file type |
| N+10       | 4        | Mac OS file creator |
| N+14       | 4        | Creation date in classic Mac OS format (seconds since 1904) |
| N+18       | 4        | Modification date in classic Mac OS format (seconds since 1904) |
| N+22       | 2        | Mac OS Finder flags |
| N+24       | 4        | Uncompressed file data CRC. This is calculated for the concaternation of the resource and data forks. |
| N+28       | 2        | File flags (see below) |
| N+30       | 4        | Resource fork uncompressed length |
| N+34       | 4        | Data fork uncompressed length |
| N+38       | 4        | Resource fork compressed length |
| N+42       | 4        | Data fork compressed length |

The flags field contains at least the following bits:

| **Bit** | **Meaning** |
|:--------|:------------|
| 0       | File is encrypted (the algorithm for this is unknown) |
| 1       | Resource fork uses LZH compression |
| 2       | Data fork uses LZH compression |

## Directory header ##

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 1        | Name length and type flag. The highest bit of this field is one for a directory entry. |
| 1          | N        | Directory name with the length indicated above |
| N+1        | 2        | Total number of files and directory in this directory, counting files in sub-directories |

The directory structure can be entirely inferred from the entry count field.

# Algorithms #

Files are compressed using either just an RLE algorithm, or RLE followed by an LZSS and Huffman based algorithm, depending on bits 1 and 2 of the flags as described above. The resource and data forks are compressed separately even though the CRC is calculated for their combination. The resource fork appears first in the file, followed by the data fork. Thus the offset to the resource fork is the same as the "Offset to file data" field, while the offset to the data fork is "Offset to file data" plus "Resource fork compressed length".

The algorithms are described on these pages:

  * Rle8182Algorithm - An RLE encoding
  * CompactProLzhAlgorithm - An LZSS+Huffman encoding