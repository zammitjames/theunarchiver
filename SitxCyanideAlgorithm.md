# Introduction #

Cyanide is a compression algorithm used by the sitx format. It is block-based, using BWT combined with M1FF2, ternary coding and range coding. It is likely based on the paper
["One attempt of a compression algorithm using the BWT"](http://www.mathematik.uni-bielefeld.de/sfb343/preprints/pr99133.ps.gz) by Bernhard Balkenhol and Yuri M. Shtarkov.

# Decoding #

## Block structure ##

The data stream is made up of blocks of varying size. Each block begins with a block header, followed by the data stream from the range coder. The header looks as follows:

| **Offset** | **Size** | **Meaning** |
|:-----------|:---------|:------------|
| 0          | 1        | Marker byte. |
| 1          | 4        | Block size  |
| 5          | 4        | Index of first byte |
| 9          | 1        | Number of symbols |

All values are big-endian. The marker byte is 0x77 for each block containing data, and 0xff at the end of the stream (in this case, the rest of the fields are not included).

## Ternary coder ##

The data is encoded as a "ternary sequence" as described in the paper by Balkenhol and Shtarkov. In practice this means that the data stream is split into two interleaved streams: In the first one, all values larger than 1 are replaced by 2, and the second one contains the replaced values. All values are encoded using Dmitry Subbotin's carryless rangecoder (see SitxRangeCoder).

### Reading the ternary sequence ###

The ternary sequence is encoded using a Markov chain of order 3. This means that the three ternary symbols preceding the current one determine the frequencies used for the range coder. This would normally mean that there are 27 different sets of frequencies, but certain Markov contexts are grouped together, giving 14 different sets of frequencies. The grouping is different from the one persented in the paper, and is as follows:

| **Context** | **Group**| **Context** | **Group**| **Context** | **Group** |
|:------------|:---------|:------------|:---------|:------------|:----------|
| 000         | 0        | 001         | 1        | 002         | 2         |
| 010         | 3        | 011         | 4        | 012         | 5         |
| 020         | 6        | 021         | 7        | 022         | 8         |
| 100         | 3        | 101         | 9        | 102         | 10        |
| 110         | 3        | 111         | 4        | 112         | 5         |
| 120         | 11       | 121         | 11       | 122         | 8         |
| 200         | 6        | 201         | 2        | 202         | 5         |
| 210         | 6        | 211         | 7        | 212         | 8         |
| 220         | 12       | 221         | 12       | 222         | 13        |

The contexts list the most recent symbol on the right.

Next, the frequencies are sorted in ascending order before reading each symbol. The sorting is entirely static, and the previous order never affects the new ordering. If `a`, `b` and `c` are the frequencies for 0, 1 and 2 respectively, the orderings will be:

| **Condition 1** | **Condition 2** | **Condition 3** | Ordering |
|:----------------|:----------------|:----------------|:---------|
| a<b             | a<c             | b<c             | 012      |
| a<b             | a<c             | b>=c            | 021      |
| a<b             | a>=c            |                 | 201      |
| a>=b            | b<c             | c<a             | 120      |
| a>=b            | b<c             | c>=a            | 102      |
| a>=b            | b>= c           |                 | 210      |

Finally, each frequency is temporarily increased by one. These frequencies in this order are then used to read a ternary symbol from the range coder. For example, if the frequencies for 0, 1 and 2 are 10, 20 and 15, the ordering will be 021, and thus the input frequencies to the coder will be 11, 16 and 21, and if the result is the third symbol, the output ternary value will be 1.

All frequencies are intialized to 0 at the beginning of decoding.

### Adjusting and renormalizing the ternary frequenceies ###

After reading a ternary symbol, the frequencies in the set used are adjusted. There are two different ways this can happen. This behaviour is controlled in part by a flag, which is set at intialization.

If the symbol read was a 0, and if is the context was 000, and the flag is cleared, a special case is triggered. In this case, the three frequencies are divided by 2, rounding down, and the frequency for the current symbol (which in this case is always 0) is increased by three. The flag is then set.

In the usual case, which happens if the conditions above are not met, the following happens:

  * If the symbol read was not 0, the flag is cleared.
  * The frequencies (including the added ones) are summed.
  * A limit is selected. If the flag is cleared, the limit is 128. If it is set, the limit is 4096.
  * If the sum of the frequencies exceeds the limit, the frequencies (not including the added ones) are halved, rounding down.
  * The frequency for the symbol read is increased by two.

### Reading the larger values ###

If the ternary symbol was a 2, we need to read the actual value from the input stream. The value is encoded as two parts: The number of bits it contains, and the bits themselves.

This is done by partitioning the possible values into several groups. The partitioning is dependent on the "number of symbols" value in the block header. The first parition contains one symbol, 2. The second contains two symbols, 3 and 4. The third contains four, and so on. However, if the last partition does not have enough symbols (as determined by the upper limit on the total number of symbols given by the "number of symbols" field) to fill it up to a full power of two, all the symbols from that parition will be added to the previous partition instead. So if the number of symbols is 9, the partitions will be (2), (3,4), (5,6,7,8,9,10).

We initialize several sets of frequencies: One containing a number of symbols equal to the number of paritions, and several sets corresponding to each partition, except for the first one. For each set of frequencies, the symbols are ordered in descending order, and their frequencies are set to 0.

#### Bumping and renormalizing the frequencies ####

Whenever a symbol has been read using one set of frequencies, the frequencies are re-normalized, the frequency for the symbol that was read is bumped, and the symbols are re-ordered to keep the list sorted in ascending order of frequency. The particulars vary for the different parts of the algorithm, but the common steps are as follows:

  * The frequencies are summed.
  * If the sum of the frequencies exceed or equal a given limit, all frequencies are divided by 2, rounding up.
  * The frequency of the symbol that was last read is increased by one.
  * If this frequency is now larger than the frequency of the symbol following it, this symbol is then moved upwards until it is placed just below the first frequency equal to or larger than its new value.

#### Reading the high bit ####

After encountering a ternary symbol of value 2, we first read the value of the high bit of the next symbol. More accurately, we read the index of the parition to use to read the full value, as described earlier.

To read this bit, we use the set of frequencies with the same number of entries as the number of partitions. After reading the index using these frequencies, we then bump the frequency of the symbol found using a limit of 256. We then bump it a second time, in its new position, with no upper limit on the total frequency (or simply a limit larger than 256).

#### Reading the lower bits ####

If the index read in the previous step was 0, we can immediately output a 2, as that is the only symbol in the first parition. If it was larger than 0, we pick the frequency set corresponding to that parition, and read the actual symbol using those frequencies. We then bump the frequency of the symbol read using a limit equal to the number of symbols in the partition times 128, or 0x4000, whichever is smaller. This frequency is only bumped once.

## Move-to-front coder ##

After reading all the symbols in a block, the data is then untransformed using a reverse move-to-front coding. A variant of move-to-front usually called M1FF2 is used, where a symbol is moved to the second position when accessed instead of the first, except when the second symbol itself is accessed and the first symbol was not accessed last time, in which case it is moved to the first position. At the beginning of decoding, the decoder acts as if the first symbol had not been accessed previously.

## BWT sorting ##

Finally, a reverse Burrows-Wheeler transform is applied. This seems to be a regular BWT transform.