# Introduction #

Blend is a compression algorithm used by the sitx format. It actually just combines SitxDarkhorseAlgorithm, SitxCyanideAlgorithm and SitxBrimstoneAlgorithm in the same stream.

# Block structure #

The data stream is made up of blocks of varying size. Each block begins with a block header, followed by the data stream from one of the supported algorithms. The header looks as follows:

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 1        | Marker byte (0x77) |
| 1          | 1        | Algorithm   |
| 2          | 4        | Uncompressed size of block |

All values are big-endian. The algorithms are as follows:

| **Number** | Algorithm |
|:-----------|:----------|
| 0          | None, uncompressed data |
| 1          | SitxDarkhorseAlgorithm |
| 2          | SitxCyanideAlgorithm |
| 3          | SitxBrimstoneAlgorithm |

## Finding blocks ##

As no compressed size is stored in the file, the only way to find the next block is to decode the entire data stream from the previous block, and then start scanning to find the next block header. As the range coder used by the algorithms can output a few more bytes than is necessary to decode, you can not just immediately start reading the next block header.

The StuffIt X code will read six bytes, apply a heuristic to see if this is a valid block header, and if not, it will discard the first byte in the buffer, and read a new byte into the other end of the buffer, and loop. The heuristic is as follows: If the first byte is 0x77, the second is 0x00, 0x01, 0x02 or 0x03, and none of the three next bytes are 0x77, this is a valid block header. If one of the three next bytes are 0x77, then if the byte following it is not 0x00, 0x01, 0x02 or 0x03, this is a valid block header. If the following byte is one of those, then this is a valid block header only if the uncompressed size value has none of the 13 lowest bits set.

Also, there is no end marker, so this has to be detected manually by checking if the input stream has ended while reading the next header.