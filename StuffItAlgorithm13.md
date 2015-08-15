# Introduction #

This algorithm is yet another combination of LZSS and Huffman coding. It uses a sliding window of 64kb, and two different prefix codes for a combination of literals and lengths, and a third prefix code for the dictionary offset. The codes are either stored in the file, or are one of five predefined codes.

All values are read least significant bit first, starting from the least significant bit of each consecutive byte.

# Header #

The header starts with a single byte that selects the prefix codes to use for decoding the data. The four high bits of this value specify whether the codes are stored in the data stream (for value 0) or whether to use predefined tables (values 1-5).

## Codes in data stream ##

The codes are stored as a list of code lengths. This list is itself compressed using a prefix code. This meta-code has 37 entries, and is listed in the Tables section at the end of this text.

The header contains one or two combined literal-length codes with 321 entires, followed by one code for the dictionary offset value bit lengths (not the actual offset themselves). This is of variable length, and this length is equal to the three low bits of the header byte, plus 10.

If bit 3 of the header byte is set, the second code is not included, and is assumed to be equal to the first.

### Reading a code ###

To read the code lengths for one of the codes, keep a length variable that starts out at 0, and start a loop that reads code lengths sequentially starting from 0:

First, read a value from the input stream using the meta-code.

  * If this value is less than 31, set the length variable to the value plus one.
  * If the value is 31, set the length variable to an "invalid code" marker (-1 is used in the current implementation, it is unknown if this is significant).
  * If the value is 32, increment the length variable.
  * If the value is 33, decrement the length variable.
  * If the value is 34, read 1 bit from the input stream, and set the length of that many code entries to be equal to the length variable. (This is a no-op if the bit is 0, which is somewhat strange.)
  * If the value is 35, read 3 bits from the input stream, and set the length of that many plus 2 code entries to be equal to the length variable.
  * If the value is 36, read 6 bits from the input stream, and set the length of that many plus 10 code entries to be equal to the length variable.

Finally, set the length of the next code entry to be equal to the length variable, in addition to any code entries you may have set while processing the input value. The loop until all code lengths for the current tree have been read.

### Building the codes ###

When all the code lengths for a given set of codes have been read, the bit sequences themselves can be built using a variation on the algorithm given in APPNOTE.TXT for the Zip implode algorithm (see ZipSpecs). The variation is to process the entries in ascending order of length and value instead of descending, and to use 32 bits instead of 16.

A naïve example implementation follows (this avoids sorting the codes at the expense of looking through the list multiple times):

```
	unsigned int code=0;

	for(int length=1;length<=32;length++)
	for(int i=0;i<numcodes;i++)
	{
		if(lengths[i]!=length) continue;
		AddCode(Reverse32(code),length,i);
		code+=1<<32-length;
	}
```

`numcodes` is the number of code entries in the code, and `lengths[]` is an array of code lengths. `Reverse32()` is a function that reverses the bits in a 32-bit value, and `AddCode(code,length,value)` is a function that adds the generated bit string `code` of length `length` to the code tree with a value of `value`.

## Pre-defined codes ##

There are five pre-defined sets of codes. These can be built using the same algorithm as described in the previous section, using pre-defined tables of code lengths. As above, the two literal codes both have 321 entries. The offset cod, for code sets 1 through 5, has 11, 13, 14, 11, or 11 entires. The actual lengths are listed at the end of this text, in the "Tables" section.

# Decoding #

Start by setting the current literal code to the first one. Then start the main read loop:

Read one value using the current literal code.

  * If the value is less than 256, output it as a literal, set the current code to the first one, and loop.
  * If the value is between 256 and 317 inclusive, the size of the window match is this value minus 253.
  * If the value is 318, read 10 bits from the input stream. The match size is this value plus 65.
  * If the value is 319, read 15 bits from the input stream. The match size is this value plus 65.
  * If the value is 320, the end of the data has been reached.

Next, read another value using the offset code. This is the length in bits of the offset value. The highest bit is always 1, except when the value is 0. The lower `n-1` bits of the offset value are then read from the input stream. The window match starts at the position of the last byte written minus the offset value. Copy the number of bytes determined by the match size calculated above to the output stream, set the current tree to the second one, and loop.

# Tables #

## Meta-code bit sequences and lengths ##

```
static const int MetaCodes[37]=
{
	0x5d8,0x058,0x040,0x0c0,0x000,0x078,0x02b,0x014,
	0x00c,0x01c,0x01b,0x00b,0x010,0x020,0x038,0x018,
	0x0d8,0xbd8,0x180,0x680,0x380,0xf80,0x780,0x480,
	0x080,0x280,0x3d8,0xfd8,0x7d8,0x9d8,0x1d8,0x004,
	0x001,0x002,0x007,0x003,0x008
};


static const int MetaCodeLengths[37]=
{
	11,8,8,8,8,7,6,5,5,5,5,6,5,6,7,7,9,12,10,11,11,12,
	12,11,11,11,12,12,12,12,12,5,2,2,3,4,5
};
```

## Pre-defined code lengths ##

```
static const int FirstTreeLengths_1[321]=
{
	 4, 5, 7, 8, 8, 9, 9, 9, 9, 7, 9, 9, 9, 8, 9, 9,
	 9, 9, 9, 9, 9, 9, 9,10, 9, 9,10,10, 9,10, 9, 9,
	 5, 9, 9, 9, 9,10, 9, 9, 9, 9, 9, 9, 9, 9, 7, 9,
	 9, 8, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9,
	 9, 8, 9, 9, 8, 8, 9, 9, 9, 9, 9, 9, 9, 7, 8, 9,
	 7, 9, 9, 7, 7, 9, 9, 9, 9,10, 9,10,10,10, 9, 9,
	 9, 5, 9, 8, 7, 5, 9, 8, 8, 7, 9, 9, 8, 8, 5, 5,
	 7,10, 5, 8, 5, 8, 9, 9, 9, 9, 9,10, 9, 9,10, 9,
	 9,10,10,10,10,10,10,10, 9,10,10,10,10,10,10,10,
	 9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,
	 9,10,10,10,10,10,10,10, 9, 9,10,10,10,10,10,10,
	10,10,10,10,10,10,10,10,10,10, 9,10,10,10,10,10,
	 9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,
	10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,
	 9,10,10,10,10,10,10,10,10,10,10,10, 9, 9,10,10,
	 9,10,10,10,10,10,10,10, 9,10,10,10, 9,10, 9, 5,
	 6, 5, 5, 8, 9, 9, 9, 9, 9, 9,10,10,10, 9,10,10,
	10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,
	10,10,10, 9,10, 9, 9, 9,10, 9,10, 9,10, 9,10, 9,
	10,10,10, 9,10, 9,10,10, 9, 9, 9, 6, 9, 9,10, 9,
	 5,
};

static const int SecondTreeLengths_1[321]=
{
	 4, 5, 6, 6, 7, 7, 6, 7, 7, 7, 6, 8, 7, 8, 8, 8,
	 8, 9, 6, 9, 8, 9, 8, 9, 9, 9, 8,10, 5, 9, 7, 9,
	 6, 9, 8,10, 9,10, 8, 8, 9, 9, 7, 9, 8, 9, 8, 9,
	 8, 8, 6, 9, 9, 8, 8, 9, 9,10, 8, 9, 9,10, 8,10,
	 8, 8, 8, 8, 8, 9, 7,10, 6, 9, 9,11, 7, 8, 8, 9,
	 8,10, 7, 8, 6, 9,10, 9, 9,10, 8,11, 9,11, 9,10,
	 9, 8, 9, 8, 8, 8, 8,10, 9, 9,10,10, 8, 9, 8, 8,
	 8,11, 9, 8, 8, 9, 9,10, 8,11,10,10, 8,10, 9,10,
	 8, 9, 9,11, 9,11, 9,10,10,11,10,12, 9,12,10,11,
	10,11, 9,10,10,11,10,11,10,11,10,11,10,10,10, 9,
	 9, 9, 8, 7, 6, 8,11,11, 9,12,10,12, 9,11,11,11,
	10,12,11,11,10,12,10,11,10,10,10,11,10,11,11,11,
	 9,12,10,12,11,12,10,11,10,12,11,12,11,12,11,12,
	10,12,11,12,11,11,10,12,10,11,10,12,10,12,10,12,
	10,11,11,11,10,11,11,11,10,12,11,12,10,10,11,11,
	 9,12,11,12,10,11,10,12,10,11,10,12,10,11,10, 7,
	 5, 4, 6, 6, 7, 7, 7, 8, 8, 7, 7, 6, 8, 6, 7, 7,
	 9, 8, 9, 9,10,11,11,11,12,11,10,11,12,11,12,11,
	12,12,12,12,11,12,12,11,12,11,12,11,13,11,12,10,
	13,10,14,14,13,14,15,14,16,15,15,18,18,18, 9,18,
	 8,
};

static const int OffsetTreeLengths_1[11]=
{
	 5, 6, 3, 3, 3, 3, 3, 3, 3, 4, 6,
};

static const int FirstTreeLengths_2[321]=
{
	 4, 7, 7, 8, 7, 8, 8, 8, 8, 7, 8, 7, 8, 7, 9, 8,
	 8, 8, 9, 9, 9, 9,10,10, 9,10,10,10,10,10, 9, 9,
	 5, 9, 8, 9, 9,11,10, 9, 8, 9, 9, 9, 8, 9, 7, 8,
	 8, 8, 9, 9, 9, 9, 9,10, 9, 9, 9,10, 9, 9,10, 9,
	 8, 8, 7, 7, 7, 8, 8, 9, 8, 8, 9, 9, 8, 8, 7, 8,
	 7,10, 8, 7, 7, 9, 9, 9, 9,10,10,11,11,11,10, 9,
	 8, 6, 8, 7, 7, 5, 7, 7, 7, 6, 9, 8, 6, 7, 6, 6,
	 7, 9, 6, 6, 6, 7, 8, 8, 8, 8, 9,10, 9,10, 9, 9,
	 8, 9,10,10, 9,10,10, 9, 9,10,10,10,10,10,10,10,
	 9,10,10,11,10,10,10,10,10,10,10,11,10,11,10,10,
	 9,11,10,10,10,10,10,10, 9, 9,10,11,10,11,10,11,
	10,12,10,11,10,12,11,12,10,12,10,11,10,11,11,11,
	 9,10,11,11,11,12,12,10,10,10,11,11,10,11,10,10,
	 9,11,10,11,10,11,11,11,10,11,11,12,11,11,10,10,
	10,11,10,10,11,11,12,10,10,11,11,12,11,11,10,11,
	 9,12,10,11,11,11,10,11,10,11,10,11, 9,10, 9, 7,
	 3, 5, 6, 6, 7, 7, 8, 8, 8, 9, 9, 9,11,10,10,10,
	12,13,11,12,12,11,13,12,12,11,12,12,13,12,14,13,
	14,13,15,13,14,15,15,14,13,15,15,14,15,14,15,15,
	14,15,13,13,14,15,15,14,14,16,16,15,15,15,12,15,
	10,
};

static const int SecondTreeLengths_2[321]=
{
	 5, 6, 6, 6, 6, 7, 7, 7, 7, 7, 7, 8, 7, 8, 7, 7,
	 7, 8, 8, 8, 8, 9, 8, 9, 8, 9, 9, 9, 7, 9, 8, 8,
	 6, 9, 8, 9, 8, 9, 8, 9, 8, 9, 8, 9, 8, 9, 8, 8,
	 8, 8, 8, 9, 8, 9, 8, 9, 9,10, 8,10, 8, 9, 9, 8,
	 8, 8, 7, 8, 8, 9, 8, 9, 7, 9, 8,10, 8, 9, 8, 9,
	 8, 9, 8, 8, 8, 9, 9, 9, 9,10, 9,11, 9,10, 9,10,
	 8, 8, 8, 9, 8, 8, 8, 9, 9, 8, 9,10, 8, 9, 8, 8,
	 8,11, 8, 7, 8, 9, 9, 9, 9,10, 9,10, 9,10, 9, 8,
	 8, 9, 9,10, 9,10, 9,10, 8,10, 9,10, 9,11,10,11,
	 9,11,10,10,10,11, 9,11, 9,10, 9,11, 9,11,10,10,
	 9,10, 9, 9, 8,10, 9,11, 9, 9, 9,11,10,11, 9,11,
	 9,11, 9,11,10,11,10,11,10,11, 9,10,10,11,10,10,
	 8,10, 9,10,10,11, 9,11, 9,10,10,11, 9,10,10, 9,
	 9,10, 9,10, 9,10, 9,10, 9,11, 9,11,10,10, 9,10,
	 9,11, 9,11, 9,11, 9,10, 9,11, 9,11, 9,11, 9,10,
	 8,11, 9,10, 9,10, 9,10, 8,10, 8, 9, 8, 9, 8, 7,
	 4, 4, 5, 6, 6, 6, 7, 7, 7, 7, 8, 8, 8, 7, 8, 8,
	 9, 9,10,10,10,10,10,10,11,11,10,10,12,11,11,12,
	12,11,12,12,11,12,12,12,12,12,12,11,12,11,13,12,
	13,12,13,14,14,14,15,13,14,13,14,18,18,17, 7,16,
	 9,
};

static const int OffsetTreeLengths_2[13]=
{
	 5, 6, 4, 4, 3, 3, 3, 3, 3, 4, 4, 4, 6,
};

static const int FirstTreeLengths_3[321]=
{
	 6, 6, 6, 6, 6, 9, 8, 8, 4, 9, 8, 9, 8, 9, 9, 9,
	 8, 9, 9,10, 8,10,10,10, 9,10,10,10, 9,10,10, 9,
	 9, 9, 8,10, 9,10, 9,10, 9,10, 9,10, 9, 9, 8, 9,
	 8, 9, 9, 9,10,10,10,10, 9, 9, 9,10, 9,10, 9, 9,
	 7, 8, 8, 9, 8, 9, 9, 9, 8, 9, 9,10, 9, 9, 8, 9,
	 8, 9, 8, 8, 8, 9, 9, 9, 9, 9,10,10,10,10,10, 9,
	 8, 8, 9, 8, 9, 7, 8, 8, 9, 8,10,10, 8, 9, 8, 8,
	 8,10, 8, 8, 8, 8, 9, 9, 9, 9,10,10,10,10,10, 9,
	 7, 9, 9,10,10,10,10,10, 9,10,10,10,10,10,10, 9,
	 9,10,10,10,10,10,10,10,10, 9,10,10,10,10,10,10,
	 9,10,10,10,10,10,10,10, 9, 9, 9,10,10,10,10,10,
	10,10,10,10,10,10,10,10,10,10, 9,10,10,10,10, 9,
	 8, 9,10,10,10,10,10,10,10,10,10,10, 9,10,10,10,
	 9,10,10,10,10,10,10,10,10,10,10,10,10,10,10, 9,
	 9,10,10,10,10,10,10, 9,10,10,10,10,10,10, 9, 9,
	 9,10,10,10,10,10,10, 9, 9,10, 9, 9, 8, 9, 8, 9,
	 4, 6, 6, 6, 7, 8, 8, 9, 9,10,10,10, 9,10,10,10,
	10,10,10,10,10,10,10,10,10,10,10,10,10,10, 7,10,
	10,10, 7,10,10, 7, 7, 7, 7, 7, 6, 7,10, 7, 7,10,
	 7, 7, 7, 6, 7, 6, 6, 7, 7, 6, 6, 9, 6, 9,10, 6,
	10,
};

static const int SecondTreeLengths_3[321]=
{
	 5, 6, 6, 6, 6, 7, 7, 7, 6, 8, 7, 8, 7, 9, 8, 8,
	 7, 7, 8, 9, 9, 9, 9,10, 8, 9, 9,10, 8,10, 9, 8,
	 6,10, 8,10, 8,10, 9, 9, 9, 9, 9,10, 9, 9, 8, 9,
	 8, 9, 8, 9, 9,10, 9,10, 9, 9, 8,10, 9,11,10, 8,
	 8, 8, 8, 9, 7, 9, 9,10, 8, 9, 8,11, 9,10, 9,10,
	 8, 9, 9, 9, 9, 8, 9, 9,10,10,10,12,10,11,10,10,
	 8, 9, 9, 9, 8, 9, 8, 8,10, 9,10,11, 8,10, 9, 9,
	 8,12, 8, 9, 9, 9, 9, 8, 9,10, 9,12,10,10,10, 8,
	 7,11,10, 9,10,11, 9,11, 7,11,10,12,10,12,10,11,
	 9,11, 9,12,10,12,10,12,10, 9,11,12,10,12,10,11,
	 9,10, 9,10, 9,11,11,12, 9,10, 8,12,11,12, 9,12,
	10,12,10,13,10,12,10,12,10,12,10, 9,10,12,10, 9,
	 8,11,10,12,10,12,10,12,10,11,10,12, 8,12,10,11,
	10,10,10,12, 9,11,10,12,10,12,11,12,10, 9,10,12,
	 9,10,10,12,10,11,10,11,10,12, 8,12, 9,12, 8,12,
	 8,11,10,11,10,11, 9,10, 8,10, 9, 9, 8, 9, 8, 7,
	 4, 3, 5, 5, 6, 5, 6, 6, 7, 7, 8, 8, 8, 7, 7, 7,
	 9, 8, 9, 9,11, 9,11, 9, 8, 9, 9,11,12,11,12,12,
	13,13,12,13,14,13,14,13,14,13,13,13,12,13,13,12,
	13,13,14,14,13,13,14,14,14,14,15,18,17,18, 8,16,
	10,
};

static const int OffsetTreeLengths_3[14]=
{
	 6, 7, 4, 4, 3, 3, 3, 3, 3, 4, 4, 4, 5, 7,
};

static const int FirstTreeLengths_4[321]=
{
	 2, 6, 6, 7, 7, 8, 7, 8, 7, 8, 8, 9, 8, 9, 9, 9,
	 8, 8, 9, 9, 9,10,10, 9, 8,10, 9,10, 9,10, 9, 9,
	 6, 9, 8, 9, 9,10, 9, 9, 9,10, 9, 9, 9, 9, 8, 8,
	 8, 8, 8, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9,10,10, 9,
	 7, 7, 8, 8, 8, 8, 9, 9, 7, 8, 9,10, 8, 8, 7, 8,
	 8,10, 8, 8, 8, 9, 8, 9, 9,10, 9,11,10,11, 9, 9,
	 8, 7, 9, 8, 8, 6, 8, 8, 8, 7,10, 9, 7, 8, 7, 7,
	 8,10, 7, 7, 7, 8, 9, 9, 9, 9,10,11, 9,11,10, 9,
	 7, 9,10,10,10,11,11,10,10,11,10,10,10,11,11,10,
	 9,10,10,11,10,11,10,11,10,10,10,11,10,11,10,10,
	 9,10,10,11,10,10,10,10, 9,10,10,10,10,11,10,11,
	10,11,10,11,11,11,10,12,10,11,10,11,10,11,11,10,
	 8,10,10,11,10,11,11,11,10,11,10,11,10,11,11,11,
	 9,10,11,11,10,11,11,11,10,11,11,11,10,10,10,10,
	10,11,10,10,11,11,10,10, 9,11,10,10,11,11,10,10,
	10,11,10,10,10,10,10,10, 9,11,10,10, 8,10, 8, 6,
	 5, 6, 6, 7, 7, 8, 8, 8, 9,10,11,10,10,11,11,12,
	12,10,11,12,12,12,12,13,13,13,13,13,12,13,13,15,
	14,12,14,15,16,12,12,13,15,14,16,15,17,18,15,17,
	16,15,15,15,15,13,13,10,14,12,13,17,17,18,10,17,
	 4,
};

static const int SecondTreeLengths_4[321]=
{
	 4, 5, 6, 6, 6, 6, 7, 7, 6, 7, 7, 9, 6, 8, 8, 7,
	 7, 8, 8, 8, 6, 9, 8, 8, 7, 9, 8, 9, 8, 9, 8, 9,
	 6, 9, 8, 9, 8,10, 9, 9, 8,10, 8,10, 8, 9, 8, 9,
	 8, 8, 7, 9, 9, 9, 9, 9, 8,10, 9,10, 9,10, 9, 8,
	 7, 8, 9, 9, 8, 9, 9, 9, 7,10, 9,10, 9, 9, 8, 9,
	 8, 9, 8, 8, 8, 9, 9,10, 9, 9, 8,11, 9,11,10,10,
	 8, 8,10, 8, 8, 9, 9, 9,10, 9,10,11, 9, 9, 9, 9,
	 8, 9, 8, 8, 8,10,10, 9, 9, 8,10,11,10,11,11, 9,
	 8, 9,10,11, 9,10,11,11, 9,12,10,10,10,12,11,11,
	 9,11,11,12, 9,11, 9,10,10,10,10,12, 9,11,10,11,
	 9,11,11,11,10,11,11,12, 9,10,10,12,11,11,10,11,
	 9,11,10,11,10,11, 9,11,11, 9, 8,11,10,11,11,10,
	 7,12,11,11,11,11,11,12,10,12,11,13,11,10,12,11,
	10,11,10,11,10,11,11,11,10,12,11,11,10,11,10,10,
	10,11,10,12,11,12,10,11, 9,11,10,11,10,11,10,12,
	 9,11,11,11, 9,11,10,10, 9,11,10,10, 9,10, 9, 7,
	 4, 5, 5, 5, 6, 6, 7, 6, 8, 7, 8, 9, 9, 7, 8, 8,
	10, 9,10,10,12,10,11,11,11,11,10,11,12,11,11,11,
	11,11,13,12,11,12,13,12,12,12,13,11, 9,12,13, 7,
	13,11,13,11,10,11,13,15,15,12,14,15,15,15, 6,15,
	 5,
};

static const int OffsetTreeLengths_4[11]=
{
	 3, 6, 5, 4, 2, 3, 3, 3, 4, 4, 6,
};

static const int FirstTreeLengths_5[321]=
{
	 7, 9, 9, 9, 9, 9, 9, 9, 9, 8, 9, 9, 9, 7, 9, 9,
	 9, 9, 9, 9, 9, 9, 9,10, 9,10, 9,10, 9,10, 9, 9,
	 5, 9, 7, 9, 9, 9, 9, 9, 7, 7, 7, 9, 7, 7, 8, 7,
	 8, 8, 7, 7, 9, 9, 9, 9, 7, 7, 7, 9, 9, 9, 9, 9,
	 9, 7, 9, 7, 7, 7, 7, 9, 9, 7, 9, 9, 7, 7, 7, 7,
	 7, 9, 7, 8, 7, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9,
	 9, 7, 8, 7, 7, 7, 8, 8, 6, 7, 9, 7, 7, 8, 7, 5,
	 6, 9, 5, 7, 5, 6, 7, 7, 9, 8, 9, 9, 9, 9, 9, 9,
	 9, 9,10, 9,10,10,10, 9, 9,10,10,10,10,10,10,10,
	 9,10,10,10,10,10,10,10,10,10,10,10, 9,10,10,10,
	 9,10,10,10, 9, 9,10, 9, 9, 9, 9,10,10,10,10,10,
	10,10,10,10,10,10, 9,10,10,10,10,10,10,10,10,10,
	 9,10,10,10, 9,10,10,10, 9, 9, 9,10,10,10,10,10,
	 9,10, 9,10,10, 9,10,10, 9,10,10,10,10,10,10,10,
	 9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,
	 9,10,10,10,10,10,10,10, 9,10, 9,10, 9,10,10, 9,
	 5, 6, 8, 8, 7, 7, 7, 9, 9, 9, 9, 9, 9, 9, 9, 9,
	 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9,
	 9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,
	10,10,10,10,10,10,10,10, 9,10,10, 5,10, 8, 9, 8,
	 9,
};

static const int SecondTreeLengths_5[321]=
{
	 8,10,11,11,11,12,11,11,12, 6,11,12,10, 5,12,12,
	12,12,12,12,12,13,13,14,13,13,12,13,12,13,12,15,
	 4,10, 7, 9,11,11,10, 9, 6, 7, 8, 9, 6, 7, 6, 7,
	 8, 7, 7, 8, 8, 8, 8, 8, 8, 9, 8, 7,10, 9,10,10,
	11, 7, 8, 6, 7, 8, 8, 9, 8, 7,10,10, 8, 7, 8, 8,
	 7,10, 7, 6, 7, 9, 9, 8,11,11,11,10,11,11,11, 8,
	11, 6, 7, 6, 6, 6, 6, 8, 7, 6,10, 9, 6, 7, 6, 6,
	 7,10, 6, 5, 6, 7, 7, 7,10, 8,11, 9,13, 7,14,16,
	12,14,14,15,15,16,16,14,15,15,15,15,15,15,15,15,
	14,15,13,14,14,16,15,17,14,17,15,17,12,14,13,16,
	12,17,13,17,14,13,13,14,14,12,13,15,15,14,15,17,
	14,17,15,14,15,16,12,16,15,14,15,16,15,16,17,17,
	15,15,17,17,13,14,15,15,13,12,16,16,17,14,15,16,
	15,15,13,13,15,13,16,17,15,17,17,17,16,17,14,17,
	14,16,15,17,15,15,14,17,15,17,15,16,15,15,16,16,
	14,17,17,15,15,16,15,17,15,14,16,16,16,16,16,12,
	 4, 4, 5, 5, 6, 6, 6, 7, 7, 7, 8, 8, 8, 8, 9, 9,
	 9, 9, 9,10,10,10,11,10,11,11,11,11,11,12,12,12,
	13,13,12,13,12,14,14,12,13,13,13,13,14,12,13,13,
	14,14,14,13,14,14,15,15,13,15,13,17,17,17, 9,17,
	 7,
};

static const int OffsetTreeLengths_5[11]=
{
	 6, 7, 7, 6, 4, 3, 2, 2, 3, 3, 6,
};
```