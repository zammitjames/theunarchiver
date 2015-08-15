# Introduction #

StuffIt X uses an extended version of Deflate. See ZipSpecs for more information about Deflate itself.

# Differences #

  * The number of distance codes can vary, and the number of bits used to store the number of distance codes in the header can vary.

# Data stream #

The data stream begins with a single extra byte value. This value can range from 15 to 25 inclusive. It gives the 2-logarithm of the window size, that is, the window size is 2^byte.

If this byte is 15, the only difference between normal Deflate and this algorithm is that the number of bits used to store the number of distance codes is 6 instead of 5. All files found so far using this algorithm use exactly this setting.