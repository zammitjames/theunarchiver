# File formats #

  * StuffItFormat - The old StuffIt archive format
  * StuffIt5Format - The new StuffIt archive format introduced in v5

# Compression algorithms #

  * 0: No compression
  * 1: Rle90Algorithm: RLE
  * 2: CompressAlgorithm: The compress LZW algorithm with 14 bits maximum code length, and block mode
  * 3: StuffItAlgorithm3: Simple Huffman encoding of individual bytes
  * 5: StuffItAlgorithm5: "LZAH"
  * 8: StuffItAlgorithm8: Miller-Wegman
  * 13: StuffItAlgorithm13: LZSS and Huffman
  * 14: StuffItAlgorithm14: ?
  * 15: StuffItArsenicAlgorithm: BWT and arithmetic coding