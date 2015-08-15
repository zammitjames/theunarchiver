# File Formats #

  * DiskDoublerSingleFormat
  * DiskDoublerArchiveFormat1
  * DiskDoublerArchiveFormat2

(See http://code.google.com/p/theunarchiver/source/browse/trunk/XADMaster/XADDiskDoublerParser.m for now.)

# Algorithms #

  * 0: No compression.
  * 1: Unix Compress, with optional XOR garbling, named "DD A".
  * 2: Unknown algorithm, with optional XOR garbling.
  * 3: Not used.
  * 4: Not used.
  * 5: Same algorithm as 2 with different parameter
  * 6: DiskDoublerAdAlgorithm with block size 8192, named "ADS" or "AD2".
  * 7: Not used.
  * 8: Rle8182Algorithm and optionally CompactProLzhAlgorithm (with blocksize 0xfff0 instead of 0x1fff0), named "DD B".
  * 9: DiskDoublerAdAlgorithm with block size 4096, named "AD" or "AD1".
  * 10: DiskDoublerDdAlgorithm, named "DD1" and "DD2".