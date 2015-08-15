# Introduction #

Darkhorse is an LZSS-based algorithm, using an LZMA-like mode of the SitxRangeCoder. The algorithm has been reverse-engineered, but not written up yet. For now, there is only source code available:

http://code.google.com/p/theunarchiver/source/browse/trunk/XADMaster/XADStuffItXDarkhorseHandle.m

(The window size seems to be stored as a single byte at the start of the stream. The window size is `1<<byte`, except when the result of this is less than `0x100000` (through either a small value, or overflow), in which case the window size is `0x100000`).