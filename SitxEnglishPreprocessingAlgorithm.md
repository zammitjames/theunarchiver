# Introduction #

This algorithm is used to pre-process data before compression in StuffIt X.

# Data stream #

The data stream begins with four byte values. These specify four different codes used later in the data stream.

  * Escape code
  * Word code
  * Capitalized word code
  * Upper-case word code

The algorithm also maintains a case switching flag, which is set at the beginning of the stream.

To decode the stream, start a loop that works as follows:

  * Read a byte.
  * If the byte is the escape code, read another byte and output it, then loop.
  * If the byte is one of the three word codes,
    * Read bytes until a byte is encountered that is not either an upper or lower case letter between a and z.
    * Calculate a dictionary index by treating the bytes read as a number in base 52, with the most significant digit being the first read, but also adding 1 to each digit. In other words, the letters a-z represent the numbers 1-26 and A-Z the numbers 27-52. (This is more properly known as a 52-adic number).
    * Look up this word in the dictionary.
    * If the code used was the upper case word code, change all letters to upper case.
    * Otherwise, if the code used was the capitalized word code, change the first letter of the word to upper case.
    * If the case switch flag is set,
      * If the first character is an upper case character between A and Z, make it lower case.
      * Otherwise, if the first character is a lower case character between a and z, make upper case.
    * If the last byte read was the escape code, read another byte.
    * If the last byte read wasn't end-of-file, add it to the output.
    * If the last byte read was '.', '!' or '?', set the case switching flag, otherwise clear it.
    * Output the word and possible last byte, and loop.
  * If the byte is anything else,
    * If the case switch flag is set,
      * If the byte is an upper case character between A and Z, make it lower case, and clear the flag.
      * Otherwise, if the byte is a lower case character between a and z, make upper case, and clear the flag.
    * If the byte is '.', '!' or '?', set the case switch flag.
    * Otherwise, unless the byte is space, newline, carriage return or tab, clear the case switch flag.
    * Output the byte, and loop.

# Dictionary #

The dictionary is a known list of English words sorted by frequency, with all words less than three letters removed, and with the rest slightly permuted. The original seems to be available in various places, such as [here](http://google.com/codesearch/p?hl=en#NsbnComaqOY/ocaml/misc/freqs.txt).

The StuffIt X version has 100366 entries.