# Introduction #

This algorithm is a combination of LZSS and Huffman coding. It uses a sliding window of 8kb, and three different prefix codes for literals, lengths, and offsets. Data is encoded in blocks of fixed size, measured in symbols read from the input bit stream.

All values are read most significant bit first, starting from the most significant bit of each consecutive byte.

# Block header #

Each block header contains the three prefix codes used to read the data in the block. The codes are of fixed size: 256 entries in the literal code, 64 entries in the length code, and 128 entries in the offset code. The codes are stored as simple lists of code lengths for each code, encoded two to a byte as nibbles. The high nibble is the earlier code length, and the low nibble the later.

To build the bit sequences used in the codes, use a variation on the algorithm given in APPNOTE.TXT for the Zip implode algorithm (see ZipSpecs). The variation is to process the entries in ascending order of length and value instead of descending, and to use a maximum of 15 bits instead of 16.

A naïve example implementation follows (this avoids sorting the codes at the expense of looking through the list multiple times):

```
	unsigned int code=0;

	for(int length=1;length<=15;length++)
	for(int i=0;i<numcodes;i++)
	{
		if(lengths[i]!=length) continue;
		AddCode(Reverse16(code),length,i);
		code+=1<<16-length;
	}
```

`numcodes` is the number of code entries in the code, and `lengths[]` is an array of code lengths. `Reverse16()` is a function that reverses the bits in a 16-bit value, and `AddCode(code,length,value)` is a function that adds the generated bit string `code` of length `length` to the code tree with a value of `value`.

# Block data #

Reading block data follows the usual LZSS pattern.

  * Read a single bit from the input stream.
  * If this bit is 1, read a byte value using the literal code and output it.
  * If it is 0,
    * Read a match length using the length code.
    * Read the upper 7 bits of the match offset using the offset code.
    * Read the lower 6 bits of the match offset directly.
    * Copy the match from the window starting at one byte less than the offset value before the previously output byte (that is, the offset is measured from the byte about to be output).

## Block size ##

Blocks are of fixed size, and there is no end-of-block marker. However, blocks are not measured in output bytes, but rather in the number and type of symbols read from the input stream. Literals increase the block position counter by 2, while window matches increase the block position counter by 3. The block ends once the block position counter reaches _or exceeds_ the block size (that is, the last symbol of a block may overflow the block size).

  * Compact Pro uses a block size of 0x1fff0.

## Input stream flush ##

At the end of each block, the input stream is flushed. This seems to be a direct consequence of the inner workings of the original implementation, which pre-buffers bits in blocks of 16. The correct way to get the input stream back in sync at the end of a block seems to be:

  * Seek to the next byte boundary.
  * If the number of bytes read since the start of the block data (not counting the block header) is odd, skip three bytes.
  * if it is even, skip two bytes.