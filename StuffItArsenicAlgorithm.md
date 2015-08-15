# Introduction #

This text was written by Matthew Russotto. It was copied from http://www.russotto.net/arseniccomp.html at November 24, 2008 for archival purposes. As long as the page remains at that URL, reference that instead for possible updates.

An errata section was added at the end with some updates to the information contained in this document.

# Stuffit Method 15 compression format #

Stuffit method 15 (internally called "Arsenic", possibly standing for some combination of Arithmetic, RLE and block\_s\_orting) is the method used for "best compression" by current (as of 2 July 2002) versions of the Stuffit engine. It uses an virtual queue arithmetic coder similar to that used by Mahesh Naik in his Multiprecision Arithmetic Coder Module (MACM). Several different models are used, and the input to the coder is the output of a Burrows-Wheeler transform (blocksort). The parameters of the block sort are themselves compressed with the arithmetic coder, though the compression gain of doing so is trivial at best.

## The Coder ##

The coder is mathematically equivalent to the MACM with a variable number of bits of precision, but the order of operations (and hence the rounding) is different, and there is a small change (for the worse) in the DecodeRange function.

### Decoder pseudocode ###

```
Parameters:
nbits = 26;
One = 1<<(nbits - 1);
Half = 1<<(nbits - 2);

Initialization:
Range = One
Code = 0;
for i = 1 to nbits
  Code = (Code << 1) | getbit()
endfor

Decode a frequency:
frequency = Code / (Range / symtot);
// MACM uses ((Code + 1) * symtot - 1) / Range
// and thus avoids the necessity of ever having a code of One.

Remove symbol (Renormalize):
lowincr = (Range / symtot) * symlow; 
  // The lowincr value is mathematically the same as 
  // (Range * symlow)/symtot as in MACM, but different due to rounding 
  // and overflow
Code -= lowincr;
if (symhigh == symtot)
  Range -= lowincr;
else
  Range = (symhigh - symlow) * (Range/symtot)
// The "if" and "else" formulations are the same, excepting roundoff

while ( Range <= Half) {
  Range = Range << 1;
  Code = (Code << 1) | getbit();
}
```

## The Models ##

Each model used by the Arsenic coder has the following parameters

  * symtot: Total frequency of all symbols in model
  * increment: Amount to increase a frequency when a symbol is found
  * freqlimit: Maximum frequency 'symtot' can reach before reducing the model. (that is no reduction until symtot > freqlimit)
  * symbols: Values of the symbols to be output
  * frequencies: Frequencies for each symbol. The initial frequency is the same for all symbols, and is equal to the increment
> (alternatively and as in MACM, cumulative frequencies for each symbol can be kept, with a 0 for the first symbol, to give the symlow values. The symhigh values would be symlow for the next symbol up, with symtot being the last symhigh)

Reduction of the model is accomplished by dividing all the frequencies by 2 (with rounding) and recalculating 'symtot'.

The initial model is a binary one:

  * symtot: 2
  * increment: 1
  * Symbols: (0,1)
  * freqlimit: 256

Other models are

  * The selector model
    * symtot: 88
    * increment: 8
    * Symbols: (0..10)
    * freqlimit: 1024
  * Model 3
    * symtot: 16
    * increment: 8
    * Symbols: (2,3)
    * freqlimit: 1024
  * Model 4
    * symtot: 16
    * increment: 4
    * Symbols: (4..7)
    * freqlimit: 1024
  * Model 5
    * symtot: 32
    * increment: 4
    * Symbols: (8..15)
    * freqlimit: 1024
  * Model 6
    * symtot: 64
    * increment: 4
    * Symbols: (16..31)
    * freqlimit: 1024
  * Model 7
    * symtot: 64
    * increment: 2
    * Symbols: (32..63)
    * freqlimit: 1024
  * Model 8
    * symtot: 128
    * increment: 2
    * Symbols: (64..127)
    * freqlimit: 1024
  * Model 9
    * symtot: 128
    * increment: 1
    * Symbols: (128..255)
    * freqlimit: 1024

## Decoding ##

All values should be read least significant bit first. Start by reading 8 bits from the arithcoder using the initial model. The result should be 0x41. Then read another 8 bits; this should be 0x73. (thus "As", the chemical symbol for Arsenic) Then read 4 more bits; this is a code for the block size for the blocksorter — specifically, it is (log2 block\_size) - 9. The block size must be between 29 and 224 inclusive.

Following the block size code are the blocks. Decoding proceeds by processing each block in turn until the end-of-file block is reached. For each block, the selector model and models 3-9 are re-initialized, as is the MTF decoder.

Each block has a block header encoded with the initial model:

```
First bit: 1 for end-of-file (no more bits), 0 normally.
Second bit: 1 for randomization, 0 normally.
Blockbits bits: Index of last character for block sort
```

Directly following this is the block data, encoded with a variety of models as well as move-to-front and zero suppression. To decode this, read a symbol with the selector model. A selector of 10 means you are at the end of the block. A selector of 2 means to use a literal 1 as the input to your move-to-front decoder. A selector between 3 and 9 means to read another symbol using the model number corresponding to the selector, and use that as input to your move-to-front decoder. A selector of 0 or 1 means to start counting zeros.

### Zero Counting ###

You'll need two variables — your zero\_count and your zero\_state. Both start out set to 0. When you get a selector 0, set your zero\_state to 1 and your zero\_count to 1. When you get a selector 1, set your zero\_state to 1 and your zero\_count to 2. Continue reading using the selector model. Each time you get a 0 or a 1, double the zero\_state. Each time you get a 0, add the new zero\_state to the zero\_count. Each time you get a 1, add twice the new zero state to the zero\_count. When your zero\_state is nonzero and you get a selector other than 0 or 1 (including 10, end of block), immediately (before processing that selector) send a number of zeros equal to your zero\_count to your MTF decoder, and reset the zero\_state to 0.

### Transforming ###

Once you've read the EOF selector, you have the actual length of the block (if you were counting as you fed them to your MTF decoder), the index of the last character, and the BWT last column output. This is sufficient to do an inverse BWT, and recover the input to the transform. The BWT is the standard BWT which wraps at the end of a string, not the Dr. Dobbs version with an implicit high-value character at the end of the string.
(needless to say, if the actual length of the block is greater than the block size indicated in the header, you have corrupt data)

### Randomization ###

Like BZIP2, in order to avoid repetitive sequences, Arsenic has the option of applying a "randomization" (something of a misnomer) to the data. If the randomization bit is set, then after the inverse BWT on the data, do the following:

```
rndindex = 0;
blockptr = block;
blockend = block + blocklength;

blockptr += rndtable[rndindex];
while (blockptr < blockend) {
  *blockptr ^= 1; /* not *blockptr++ */
  rndindex++;
  if (rndindex == sizeof(rndtable)/sizeof(rndtable[0]))
    rndindex = 0;
  blockptr += rndtable[rndindex];
}
```

### The final RLE ###

That's not quite the end, though, because another RLE process was applied in compression before the BWT and randomization. This is a "byte stuffing" RLE. Runs of 3 identical symbols or less are unchanged. Runs of 4-255 symbols are replaced with a run of 4 symbols followed by a byte indicating the length of the remaining run. A run of N symbols, N > 255 is encoded just like a run of 255 symbols followed by a run of N-255 symbols. (While the scheme should allow for runs of 259 symbols to be encoded as one run, these have not been observed and are assumed to be illegal). It is unknown if this RLE is applied on a per-block basis or if it is applied to the whole file, though per-block seems most likely. Decoding is simple; any time you see a run of 4 symbols, take the next symbol as a repeat count and repeat the last symbol that number of times. Finally, you have your uncompressed output. (note that this final RLE decoding is easily combined with the derandomization)

## CRC ##

Following the last block are 32 bits compressed with the initial model. This is the standard (used by PKZIP, Ethernet, etc) 32 bit CRC of the uncompressed data, stored least-significant-bit first. (Polynomial 0x04c11db7, reflection: yes, initial value = 0xFFFFFFFF, final XOR = 0xFFFFFFFF)

## Appendix A: Randomization table ##

```
static unsigned short rndtable[] = 
{
 0xee,  0x56,  0xf8,  0xc3,  0x9d,  0x9f,  0xae,  0x2c,
 0xad,  0xcd,  0x24,  0x9d,  0xa6, 0x101,  0x18,  0xb9,
 0xa1,  0x82,  0x75,  0xe9,  0x9f,  0x55,  0x66,  0x6a,
 0x86,  0x71,  0xdc,  0x84,  0x56,  0x96,  0x56,  0xa1,
 0x84,  0x78,  0xb7,  0x32,  0x6a,   0x3,  0xe3,   0x2,
 0x11, 0x101,   0x8,  0x44,  0x83, 0x100,  0x43,  0xe3,
 0x1c,  0xf0,  0x86,  0x6a,  0x6b,   0xf,   0x3,  0x2d,
 0x86,  0x17,  0x7b,  0x10,  0xf6,  0x80,  0x78,  0x7a,
 0xa1,  0xe1,  0xef,  0x8c,  0xf6,  0x87,  0x4b,  0xa7,
 0xe2,  0x77,  0xfa,  0xb8,  0x81,  0xee,  0x77,  0xc0,
 0x9d,  0x29,  0x20,  0x27,  0x71,  0x12,  0xe0,  0x6b,
 0xd1,  0x7c,   0xa,  0x89,  0x7d,  0x87,  0xc4, 0x101,
 0xc1,  0x31,  0xaf,  0x38,   0x3,  0x68,  0x1b,  0x76,
 0x79,  0x3f,  0xdb,  0xc7,  0x1b,  0x36,  0x7b,  0xe2,
 0x63,  0x81,  0xee,   0xc,  0x63,  0x8b,  0x78,  0x38,
 0x97,  0x9b,  0xd7,  0x8f,  0xdd,  0xf2,  0xa3,  0x77,
 0x8c,  0xc3,  0x39,  0x20,  0xb3,  0x12,  0x11,   0xe,
 0x17,  0x42,  0x80,  0x2c,  0xc4,  0x92,  0x59,  0xc8,
 0xdb,  0x40,  0x76,  0x64,  0xb4,  0x55,  0x1a,  0x9e,
 0xfe,  0x5f,   0x6,  0x3c,  0x41,  0xef,  0xd4,  0xaa,
 0x98,  0x29,  0xcd,  0x1f,   0x2,  0xa8,  0x87,  0xd2,
 0xa0,  0x93,  0x98,  0xef,   0xc,  0x43,  0xed,  0x9d,
 0xc2,  0xeb,  0x81,  0xe9,  0x64,  0x23,  0x68,  0x1e,
 0x25,  0x57,  0xde,  0x9a,  0xcf,  0x7f,  0xe5,  0xba,
 0x41,  0xea,  0xea,  0x36,  0x1a,  0x28,  0x79,  0x20,
 0x5e,  0x18,  0x4e,  0x7c,  0x8e,  0x58,  0x7a,  0xef,
 0x91,   0x2,  0x93,  0xbb,  0x56,  0xa1,  0x49,  0x1b,
 0x79,  0x92,  0xf3,  0x58,  0x4f,  0x52,  0x9c,   0x2,
 0x77,  0xaf,  0x2a,  0x8f,  0x49,  0xd0,  0x99,  0x4d,
 0x98, 0x101,  0x60,  0x93, 0x100,  0x75,  0x31,  0xce,
 0x49,  0x20,  0x56,  0x57,  0xe2,  0xf5,  0x26,  0x2b,
 0x8a,  0xbf,  0xde,  0xd0,  0x83,  0x34,  0xf4,  0x17
};
```

## References ##

http://www.dogma.net/markn/articles/bwt/bwt.htm, M. Nelson. Data compression with the Burrows-Wheeler transform. Dr. Dobb's Journal of Software Tools, 21(9):46--50, 1996.
A very useful article on blocksorting. Contains source, but the method in the source is incompatible with Arsenic.

P. Fenwick. "Block Sorting Text Compression -- Final Report", The University of Auckland, Department of Computer Science Report No 130, Apr. 1996.
Among other things, this contains a description of the zero suppression method used in Arsenic, credited to Wheeler. Combine that, the structured coding model also described here in section 8, and RLE preprocessing, and you almost have Arsenic.

http://www.cbloom.com/news/macm.html, FastArith Encoding using the generalized BitQueue (MACM-96)
An arithmetic coder close to the one used in Arsenic.

## Errata ##

Addition by Dag Ågren: The RLE encoding mentioned in "The final RLE" is actually applied to the whole file before block splitting. Thus the final output of decoding one block can be larger than the block size.