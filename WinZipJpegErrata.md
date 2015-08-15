# Introduction #

WinZip has, in their "zipx" format, added a JPEG compression mode to the Zip format. There is a published spec for the format available at:

http://www.winzip.com/wz_jpg_comp.pdf

However, this spec has a number of omissions and errors that make it impossible to write a functioning implementation of the format. For my implementation in The Unarchiver, I did some reverse engineering on the WinZip implementation to find out how the compression is actually implemented. This page contains some corrections and additions to the spec, which will be required for anyone wanting to implement this compression themselves.

I can't guarantee that everything on this page is correct either, nor that there aren't further errors or omissions in the spec itself that I do not cover here. However, I have managed to implement a decompressor that correctly unpacks all my test cases. If anything here is unclear, you might want to look at the source code for my implementation:

http://code.google.com/p/theunarchiver/source/browse/#hg%2FXADMaster%2FWinZipJPEG

If you find anything obviously wrong or missing on this page, please contact me!

# Errata #

In the following, things marked "Error" are outright wrong in the spec, "Omission" is for things required for correct implementation but not mentioned, "Clarification" is for things that might be guessed but are not clearly explained, and "Unclear" for things I have not yet managed to confirm.

## Section 4.1.1 ##

**Omission:** If the "compressed size" field is set to 0, the metadata is stored uncompressed. The "uncompressed size" contains the number of bytes stored.

## Section 5.4.1 ##

**Omission:** At the start of decoding each slice, the arithmetic decoder is re-initialized by calling the initialization routine. The bins for that component remain unchanged from last slice.

**Unclear:** It is not clear how to handle the part about "the encoder's FLUSH routine" at the end of each slice in the decoder. It seems from a reading of the code in the patent that this involves calling the RENORM function, and then discarding one byte if the current and last bytes read were 0xff. This may not be entirely correct.

## Section 5.5 ##

**Clarification:** The blocks are stored in each slice in linear order, not in the order they appear in MCUs. That is, if a a component has 2x2 blocks in each MCU, those blocks will appear in the compressed stream in image order, not in groups of four as in the MCU.

## Section 5.6.1 ##

**Clarification:** The north block is maintained across slice boundaries, so the final row of the previous slice is required when reading the next slice.

## Section 5.6.2.2 ##

**Error:** The expression

```
sum += (abs(Bn[x]) + abs(Bw[k])) * S[x] / S[k]
```

should actually read

```
sum += (abs(Bn[x]) + abs(Bw[x])) * S[x] / S[k]
```

**Omission:** The DC component never participates in the calculation. Also, the value of "count" is 4 for the given example. "count" can take all values between 1 and 4, depending on position. The value of "count" for different coefficients is:

| - | 1 | 2 | 2 | 2 | 2 | 2 | 2 |
|:--|:--|:--|:--|:--|:--|:--|:--|
| 1 | 3 | 4 | 4 | 4 | 4 | 4 | 4 |
| 2 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |
| 2 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |
| 2 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |
| 2 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |
| 2 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |
| 2 | 4 | 4 | 4 | 4 | 4 | 4 | 4 |

## Section 5.6.4 ##

**Omission:** This section is very vague on how the various bits read are mapped into BAC bins. The unary header is mapped into one set of bins, with the first bit always being mapped to one bin, the second to another, and so on. However, there is an upper limit to the number of bins used, referred to as a "cap" in sections 5.6.6.3 and 5.6.7.3.1, which means that any bits higher than the "cap" are mapped to the same bin, the final one in the set.

The bits of the remainder are sent MSB first, and they are mapped into bins so that the LSB is always mapped into the same bin, the next least significant bit into another, and so on. Thus, which bin to map the first bit of the remainder to will depend on the magnitude.

**Unclear:** If the maximum bit length is 14, there is no need to send a final zero bit in the unary header if the number uses all 14 bits. However, the spec does not specify whether the zero is sent or not, and I have been unable to test this.

## Section 5.6.7.1 ##

**Error:** The lines

```
p0 = round(Bn[0] – 2.2076 * S[2] / (2 * S[0]) * (Bn[2] – Bc[2])) 
...
p1 = round(Bw[0] – 2.2076 * S[1] / (2 * S[0]) * (Bw[1] – Bc[1])) 
...
t0 = Bn[0] * 10000 – 11038 * S[2] * (Bn[2] – Bc[2]) / S[0] 
...
t1 = Bw[0] * 10000 – 11038 * S[1] * (Bw[1] – Bc[1]) / S[0]
```

should actually read

```
p0 = round(Bn[0] – 2.2076 * S[2] / (2 * S[0]) * (Bn[2] + Bc[2])) 
...
p1 = round(Bw[0] – 2.2076 * S[1] / (2 * S[0]) * (Bw[1] + Bc[1])) 
...
t0 = Bn[0] * 10000 – 11038 * S[2] * (Bn[2] + Bc[2]) / S[0] 
...
t1 = Bw[0] * 10000 – 11038 * S[1] * (Bw[1] + Bc[1]) / S[0]
```

## Section 5.6.7.2 ##

**Error:** The lines

```
d0 += abs(abs(Bn[x]) – abs(Bc[x]))
...
d1 += abs(abs(Bw[x]) – abs(Bc[x])
```

should actually read

```
d0 += abs(Bn[x] – Bc[x])
...
d1 += abs(Bw[x] – Bc[x])
```

**Clarification:** The calculation of DCp has to be done with 64-bit maths, as it would overflow if done in 32 bits.

## Section 5.6.7.3.2 ##

**Error:** The lines

```
sgnn = Bn[0] < 0
sgnw = Bw[0] < 0
```

should actually read

```
sgnn = Bn[0] < DCp
sgnw = Bw[0] < DCp
```

## Section 6 ##

**Clarification:** The referenced patent can only be obtained as scanned PDF pages:

http://www.google.com/patents?vid=USPAT4791403

As it contains some very large tables, it is extremely time-consuming to actually implement and be sure that you have gotten all details correct. The tables used in my implementation have been confirmed to match those found in the WinZip implementation. They can be copied from the source code found here:

http://code.google.com/p/theunarchiver/source/browse/XADMaster/WinZipJPEG/ArithmeticDecoder.c

**Clarification:** Fig. 18 is just an addition at the top of the diagram, while fig. 13 actually involves one addition at the top of the diagram, and removing one element (LOG(I)=0).

**Omission:** It is unclear what bit precision the implementation of the binary coder in the patent uses, as the language used is mostly entirely unknown these days. However, even though it looks like some values might use 16-bit maths, the implementation in WinZip seems to use 32-bit fields for all internal state. This is important, as some operations might overflow a signed 16-bit value, causing decoding failure.