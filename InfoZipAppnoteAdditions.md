# Notes beyond the 1.93a appnote.txt: #

  1. Distance pointers never point before the beginning of the output stream.
  1. Distance pointers can point back across blocks, up to 32k away.
  1. There is an implied maximum of 7 bits for the bit length table and 15 bits for the actual data.
  1. If only one code exists, then it is encoded using one bit.  (Zero would be more efficient, but perhaps a little confusing.)  If two codes exist, they are coded using one bit each (0 and 1).
  1. There is no way of sending zero distance codes--a dummy must be sent if there are none.  (History: a pre 2.0 version of PKZIP would store blocks with no distance codes, but this was discovered to be too harsh a criterion.)  Valid only for 1.93a.  2.04c does allow zero distance codes, which is sent as one code of zero bits in length.
  1. There are up to 286 literal/length codes.  Code 256 represents the end-of-block.  Note however that the static length tree defines 288 codes just to fill out the Huffman codes.  Codes 286 and 287 cannot be used though, since there is no length base or extra bits defined for them.  Similarily, there are up to 30 distance codes. However, static trees define 32 codes (all 5 bits) to fill out the Huffman codes, but the last two had better not show up in the data.
  1. Unzip can check dynamic Huffman blocks for complete code sets. The exception is that a single code would not be complete (see #4).
  1. The five bits following the block type is really the number of literal codes sent minus 257.
  1. Length codes 8,16,16 are interpreted as 13 length codes of 8 bits (1+6+6).  Therefore, to output three times the length, you output three codes (1+1+1), whereas to output four times the same length, you only need two codes (1+3).  Hmm.
  1. In the tree reconstruction algorithm, Code = Code + Increment only if BitLength(i) is not zero.  (Pretty obvious.)
  1. Correction: 4 Bits: # of Bit Length codes - 4     (4 - 19)
  1. Note: length code 284 can represent 227-258, but length code 285 really is 258.  The last length deserves its own, short code since it gets used a lot in very redundant files.  The length 258 is special since 258 - 3 (the min match length) is 255.
  1. The literal/length and distance code bit lengths are read as a single stream of lengths.  It is possible (and advantageous) for a repeat code (16, 17, or 18) to go across the boundary between the two sets of lengths.
  1. The Deflate64 (PKZIP method 9) variant of the compression algorithm differs from "classic" deflate in the following 3 aspect:

> a) The size of the sliding history window is expanded to 64 kByte.

> b) The previously unused distance codes #30 and #31 code distances from 32769 to 49152 and 49153 to 65536.  Both codes take 14 bits of extra data to determine the exact position in their 16 kByte range.

> c) The last lit/length code #285 gets a different meaning. Instead of coding a fixed maximum match length of 258, it is used as a "generic" match length code, capable of coding any length from 3 (min match length + 0) to 65538 (min match length + 65535). This means that the length code #285 takes 16 bits (!) of uncoded extra data, added to a fixed min length of 3.

> Changes a) and b) would have been transparent for valid deflated data, but change c) requires to switch decoder configurations between Deflate and Deflate64 modes.