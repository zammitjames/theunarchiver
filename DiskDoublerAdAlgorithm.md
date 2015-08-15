# Introduction #

AD is a compression algorithm used by the old DiskDoubler application for Mac OS. It compresses the data stream as a series of blocks of fixed size (except for final blocks). The block size is either 4096 or 8192 bytes, depending on which variant is used.

It is a simple LZSS algorithm, with no Huffman coding, just a number of special cases for encoding matches more efficiently. It also doesn't use a sliding window, it uses the entire current block as its dictionary.

# Decoding #

## Block header ##

Each block starts with a 12-byte header. Its layout is as follows:

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 2        | Compressed size |
| 2          | 2        | Uncompressed size |
| 4          | 4        | Unused?     |
| 8          | 1        | XOR sum of compressed data |
| 9          | 1        | Flags       |
| 10         | 1        | Unused?     |
| 11         | 1        | XOR sum of preceeding 11 bytes |

This is followed by a data stream of the size given by the "compressed size" field of the header. The simple bytewise XOR sum of this compressed data is also stored in the header.

Bit 0 of the "flags" field indicates whether the block is compressed or not. An uncompressed block contains "uncompressed size" bytes of data. The block can apparently be padded, so "compressed size" might be larger than "uncompressed size" for an uncompressed block. Always use the "compressed size" field for seeking to the next block.

Bit 1 and 2 seem to contain the number of bytes used to pad a block. Bit 3 might indicate that the larger block size is used. Other bits are also used but their meaning is unknown. None of the bits except 0 seem to be important for unpacking.

## Compressed bit stream ##

If the block is compressed, it contains a bit stream where each byte contains bits in most-significant-bit-first order. Bit strings longer than one bit are also read most significant bit first. To decompress the stream, start a loop that does the following, and which terminates when "uncompressed size" bytes have been unpacked from the stream:

  * Read one bit.
  * If this bit is zero, read eight more bits and output them as a literal byte.
  * Otherwise if the bit is one, read one more bit.
  * If this bit is zero, read an eight bit offset.
  * Otherwise if this bit is one, read a twelve bit offset.
  * Read one more bit.
  * If this bit is zero, copy two bytes at the given offset to the output buffer, and loop.
  * Otherwise if this bit is one,  read another bit.
  * If this bit is zero, do the following:
    * Read one more bit.
    * If this bit is zero, copy three bytes at the given offset to the output buffer, and loop.
    * Otherwise if this bit is one, copy four bytes at the given offset to the output buffer, and loop.
  * If the previously mentioned bit is instead one, do the following:
    * Read four bits of length.
    * Copy that many plus five bytes at the given offset to the output buffer, and loop.

All offsets counted backwards from the position in the output buffer where the next byte is about to be written. Also, the last match can apparently stretch outside the block size. In this case, the match should be truncated to fit.

### Length as a prefix code ###

Alternately, you could treat the match lengths as a static prefix code. That code would be as follows:

| **Code** | **Match length** |
|:---------|:-----------------|
| 0        | 2                |
| 100      | 3                |
| 101      | 4                |
| 110000   | 5                |
| 110001   | 6                |
| 110010   | 7                |
| 110011   | 8                |
| 110100   | 9                |
| 110101   | 10               |
| 110110   | 11               |
| 110111   | 12               |
| 111000   | 13               |
| 111001   | 14               |
| 111010   | 15               |
| 111011   | 16               |
| 111100   | 17               |
| 111101   | 18               |
| 111110   | 19               |
| 111111   | 20               |

## Steam footer ##

The block stream is followed by a footer that contains information about the stream, and a list of offsets for all the blocks. The exact format is not yet determined.