# Introduction #

This text was written by Roger Seeck. It was copied from http://www.binaryessence.com/dct/imp/en000225.htm on November 31, 2008 for archival purposes.

See also InfoZipAppnoteAdditions.

# Deflate64™ (Enhanced Deflate) #

Deflate64™ is a slightly modified variant of the Deflate procedure. The fundamental mechanisms remained completely unchanged, only the following features were improved:

## Expansion of the dictionary on 64 Kbytes. ##

The addressable size of the sliding dictionary is extended from 32 kbytes to 64 kbytes.

## Expansion of the distance code ##

The distance codes (30 and 31) not used until now are extended to address a range of 64 kbytes. According to the conventional Deflate definition these codes were not used. 14 extra bits are assigned to each of them.

## Expansion of the length codes ##

In contrast to Deflate, the last length code (285) will be extended by 16 extra bits. The code defines lengths in a range between 3 and 65.538 byte. The original limitation to 258 byte sequences is dropped with that.

Deflate64™ achieves a slightly improved compression rate in comparison to Deflate at a lower processing speed. The format was specified by PKWARE, inc. and is part of the ZIP specification (compression method 9).

Unlike all other compression methods which are specified in ZIP, Deflate64™ is not described in detail. Additionally PKWARE claims to use Deflate64™ as a trademark. The information for this article is derived from the source code of inflate.c (Info-ZIP, UnZip 5.51).

Besides PKWARE and a large number of commercial application some freeware and open source projects support Deflate64™:
  * 7-Zip
  * Info-ZIP (Zip, UnZip)

The zlib project does not support Deflate64™. According to the zlib FAQ, this is due to the low improvements and the decision of PKWARE to handle Deflate64™ as a proprietary format.