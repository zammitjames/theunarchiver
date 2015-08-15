# Description #

The compress algorithm uses LZW encoding with an increasing code length and a variable maximum code length, and an optional block mode. All values are read least significant bit first, starting from the least significant bit of each consecutive byte.

See LzwAlgorithm for more information about the basics of the LZW algorithm. Only the specifics of the compress version are described on this page.

# Initialization #

The dictionary is pre-populated with byte values 0-255 as usual. If block mode is used, value 256 is reserved for end-of-block. The code length starts at 9.

# Decoding #

Read and decode dictionary codes as per the usual LZW algorithm. The code length increases  by one each time the dictionary size reaches a power of two, as long as the code length is less than the maximum. When the code length reaches the maximum, stop recording new words in the dictionary, but keep decoding.

In block mode, if a code of 256 is encountered, the dictionary is re-initialized and the bit size is reset to 9. The bit stream may also need to be re-synchronized, as described below.

# Bit stream synchronization #

The original compress implementation reads codes in batches. The batch size is 8 codes. This means that the batch read always starts and stops at a byte boundary, and that the batch size in bytes is the same as the code length. In normal operation, each block of data using one specific code length can always be read as a whole number of such batches, so this it is not necessary to take this into account.

However, when encountering a clear code in block mode, the rest of the batch is discarded. Thus the bit reader needs to skip codes of the current size (before resetting it to 9) until it reaches a multiple of 8 codes read.

# Compress file format #

The compress (.Z) file format is just a compress bit stream, with a three-byte header. Two bytes of magic number (0x1f 0x9d) and one byte that specifies the maximum bit length (the five lowest bits, 0-31) and whether to use block mode (bit 7 set to 1).