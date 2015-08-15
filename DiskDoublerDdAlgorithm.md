# Introduction #

DD is a compression algorithm used by the old DiskDoubler application for Mac OS. It compresses the data stream as a series of blocks of variable size (up to a maximum of 0x1010e?).

DD is an LZSS-like algorithm that splits the literals and offsets into two separate streams, in addition to a third stream for the lengths of literal runs and matches. All three streams are (optionally, for some) Huffman encoded.

It uses a 65536-byte (probably) sliding window as its dictionary. Matches in later blocks can reference data from earlier blocks if they are within the window.

# Decoding #

## Block header ##

Each block starts with a 22-byte header. Its layout is as follows:

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 4        | Uncompressed size |
| 4          | 2        | Number of literals |
| 6          | 2        | Number of offsets |
| 8          | 2        | Compressed size of run lengths |
| 10         | 2        | Compressed size of literals |
| 12         | 2        | Compressed size of offsets |
| 14         | 1        | Flags       |
| 15         | 1        | Unused?     |
| 16         | 1        | XOR sum of compressed run lengths |
| 17         | 1        | XOR sum of compressed literals |
| 18         | 1        | XOR sum of compressed offsets |
| 19         | 1        | XOR sum of uncompressed data |
| 20         | 1        | Unused?     |
| 21         | 1        | XOR sum of preceeding 21 bytes |

Flags: Bit 6 indicates whether the block is _un\_compressed. Bit 7 indicates whether the literals block is compressed._

The following block data consists of the offset, literal and run length streams. Each one is as many bytes as the compressed size field for each indicates.

## Huffman coding ##

All streams use the same kind of Huffman coding. The Huffman parameters are transmitted at the start of each individual stream. Only the lengths of the codes are stored. The Huffman header starts with a big-endian 32-bit value that defines parameters for the code:

| **Bits** | 31-24 | 23-13 | 12-8 | 7-3 | 2 | 1 | 0 |
|:---------|:------|:------|:-----|:----|:--|:--|:--|
| **Meaning** | Number of entries minus one | Number of bytes of code length data | Max code length | Bits per code length | Uses zero coding |   |   |

This is followed by a list of code lengths for each entry. The "number of entries" field in the header indicates how many there are, up to a maximum of 256. If the "uses zero coding" bit is 0, each entry is just a bit string as long as the "bits per code length" field indicates, and gives the length of the Huffman code for that entry. If the "uses zero coding" bit is 1, each entry is preceded by a single bit. If this bit is 0, the code length for this entry is 0 (that is, it is unused), and that is all the data that is stored for that entry. If the bit is 1, it is followed by a bit string of the length given by the "bits per code length" field giving the code length.

The actual data stream starts after the number of bytes given in the "number of bytes of code length data", plus four bytes for the header fields. The bitstream starts from a byte boundary at this offset.

## Offset stream ##

Offsets are stored using a selector that selects the base offset, and a bit string that defines the lower bits of the offsets. The selectors are stored using Huffman coding while the bit strings are unencoded. To read an offset, first read a selector, look up the base offset and bit string length for it, read that many bits from the bit stream (if any), and add them to the base to get the final offset.

There are 32 selectors, and their base offsets and bit string lengths are as follows:

| **Selector** | **Base** | **Bits** |
|:-------------|:---------|:---------|
| 0            | 1        | 0        |
| 1            | 2        | 0        |
| 2            | 3        | 0        |
| 3            | 4        | 0        |
| 4            | 5        | 1        |
| 5            | 7        | 1        |
| 6            | 9        | 2        |
| 7            | 13       | 2        |
| 8            | 17       | 3        |
| 9            | 25       | 3        |
| 10           | 33       | 4        |
| 11           | 49       | 4        |
| 12           | 65       | 5        |
| 13           | 97       | 5        |
| 14           | 129      | 6        |
| 15           | 193      | 6        |
| 16           | 257      | 7        |
| 17           | 385      | 7        |
| 18           | 513      | 8        |
| 19           | 769      | 8        |
| 20           | 1025     | 9        |
| 21           | 1537     | 9        |
| 22           | 2049     | 10       |
| 23           | 3073     | 10       |
| 24           | 4097     | 11       |
| 25           | 6145     | 11       |
| 26           | 8193     | 12       |
| 27           | 12289    | 12       |
| 28           | 16385    | 13       |
| 29           | 24577    | 13       |
| 30           | 32769    | 14       |
| 31           | 49153    | 14       |

The base offset can also be calculated as `((2+(selector&1))<<bits)+1`.

## Literal stream ##

The literal stream contains of all literal bytes of the compressed stream. Depending on bit 7 of the flag field, it is either uncompressed (if the bit is 0) or compressed using Huffman coding (if the bit is 1). If it is uncompressed, then the "number of literals" field should be equal to the "compressed size of literals" field.

## Run length stream ##

The run length stream contains a series of 8 bit run length codes encoded using Huffman coding. To decode the final output stream, start a loop that does the following:

  * Read a length code from the bit stream using the Huffman coding.
  * If the code is 0, copy a single byte from the literal stream to the output.
  * If the code is less than 128, read an offset from the offset stream, then copy as many bytes as the code plus two (maybe) from the previous output to the output.
  * If the code is 128 or larger, subtract 128, raise 2 to this value, and copy that many bytes from the literal stream to the output.

Codes 0 and 128 seem to do exactly the same thing.