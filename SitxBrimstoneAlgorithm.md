# Introduction #

Brimstone is based on [PPMd variant G](http://compression.ru/ds/), with some changes.

# Differences #

  * The first 128 states of MaxContext in the initialization are given the frequency 2. The last 128 are given the frequency 1 as in the original.
  * All code related to the LastBreath variable in the suballocator is removed.
  * The range coder uses 65536 as the "bottom" value, instead of 32768.

# Data stream #

The data stream begins with two bytes giving the PPMd settings. The first is the suballocator size, given as the two-logarithm of the number of bytes allocated. In other words, the size will be 2^byte1. Allowed values are between 16 and 30, inclusive. The second is the max order value. Allowed values are between 2 and 16, inclusive.

This is followed by the byte stream from the range coder.