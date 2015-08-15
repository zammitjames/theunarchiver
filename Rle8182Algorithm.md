# Description #

This is a simple RLE algorithm used by some old Mac archivers such as Compact Pro. It uses a double-byte escape code to encode repeated runs of characters.

  * The two bytes 0x81 0x82 followed by a a single non-zero byte N start a run of N-1 repetitions of the last byte previously output.
  * 0x81 0x82 0x00 outputs the byte sequence "0x81 0x82".
  * 0x81 followed by any other byte is treated as a literal byte 0x81 (and whatever follows it).
  * Any other byte sequence is a literal string.

As a consequence,

  * 0x81 0x81 0x82 N encodes a run of 0x81 bytes.
  * 0x81 0x82 0x00 0x81 0x82 N encodes the bytes "0x81 0x82" followed by a run of 0x82 bytes.
  * 0x81 0x82 0x01 appears to be a no-op or illegal.