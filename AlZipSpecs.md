# Description #

ALZip is a Korean archive format, closely based on Zip, and using bzip2 and deflate compression.

## References ##

  * http://en.wikipedia.org/wiki/ALZip
  * http://www.altools.com/ALTools/ALZip.aspx
  * http://www.kipple.pe.kr/win/unalz/ - Reverse engineered unarchiver in Korean.

# File format #

ALZip files contain an archive header, followed by file headers interleaved with compressed data, and finally an end-of-archive marker. All values larger than one byte are little-endian.

In multi-volume files, each volume has an archive header and an end-of-archive marker. There is however only one file header even if a file stretches across multiple volumes.

## Naming ##

ALZip files are named `.alz`. Further volumes in multi-volume archives are named `.a00`, `.a01`, `.a02` and so on.

## Archive header ##

Files begin with an eight-byte header.

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 4        | Magic number (`0x41 0x4c 0x5a 0x01`, "ALZ\001") |
| 4          | 3        | Unknown     |
| 7          | 1        | Volume number |

## File header ##

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 4        | Magic number (`0x42 0x4c 0x5a 0x01`, "BLZ\001") |
| 4          | 2        | Filename length |
| 6          | 1        | File attributes |
| 7          | 4        | File time stamp in DOS format |
| 11         | 1        | Flags       |
| 12         | 1        | Unknown     |
| 13         | 1        | Compresison method (optional) |
| 14         | 1        | Unknown (optional) |
| 15         | 4        | CRC-32 (optional) |
| 19         | N        | File compressed size (optional) |
| 19+N       | N        | File uncompressed size (optional) |
| 19+2N / 13 | M        | File name   |

The known bits in the "File attributes" field are as follows:

| **Bit** | **Meaning** |
|:--------|:------------|
| 0       | File is read-only |
| 1       | File is hidden |
| 4       | Entry is a directory |
| 5       | Entry is a file |

The known bits in the "Flags" field are as follows:

| **Bit** | **Meaning** |
|:--------|:------------|
| 0       | File is encrypted |
| 4-7     | Size field byte count |

Bits 4-7 of the "Flags" field determine the existence of the optional fields, and the size ("N") of the "File compressed size" and "File uncompressed size" fields. The value encoded is the number of bits used to represent the size fields. If this value is zero, none of the optional fields are included.

The compressed data follows the file header. The "File compressed size" can be used to seek to the next header.

### Encryption ###

The encryption algorithm used is as yet unknown.

### Compression methods ###

The compression methods are:
  * 0: No compression
  * 1: Bzip2
  * 2: Deflate
  * 3: Deflate with obfuscated initialization. See below for more information.

## End of archive ##

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 4        | Magic number (`0x43 0x4c 0x5a 0x01`, "CLZ\001") |
| 4          | 4        | Unknown     |
| 8          | 4        | Unknown, possible CRC32, sometimes 0 |
| 12         | 4        | Magic number 2 |

The "Magic number 2" value is `0x43 0x4c 0x5a 0x02`, "CLZ\002" in single-volume archives and in the final volume in a multi-volume archive, and `0x43 0x4c 0x5a 0x03`, "CLZ\003" in non-final volumes of multi-volume archives.

# Obfuscated deflate #

Compression method 3 is a trivial obfuscation of the PKZIP deflate algorithm. The obfuscation consists of using a randomized order for the initial 3-bit bit lengths in the Huffman header.

APPNOTE.TXT, in the "Expanding Huffman codes" section, gives the correct ordering as `   16, 17, 18, 0, 8, 7, 9, 6, 10, 5, 11, 4, 12, 3, 13, 2, 14, 1, 15`. ALZip method 3 instead calculates a randomized bit order using the following algorithm:

  * Initialize the order list with the values 0..18.
  * For each element in the list, starting from 0 and going up to 18,
    * Calculate a random index as the current element's index modulo 6, times 3, plus a value given by the four least significant bits of the file's size.
    * If this value is larger than 18, use the value modulo 18 instead. Note: 18, and not 19 as expected.
    * Swap the current element with the element at this index.

Or in C99 code:

```
	int param=size%16;

	for(int i=0;i<19;i++) order[i]=i;

	for(int i=0;i<19;i++)
	{
		int swapindex=(i%6)*3+param;
		if(swapindex>18) swapindex%=18;
		if(swapindex!=i)
		{
			int tmp=order[i];
			order[i]=order[swapindex];
			order[swapindex]=tmp;
		}
	}
```

Where `order` is the order table being generated, and `size` is simply the uncompressed size of the file being decompressed.