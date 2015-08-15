# Marker #

StuffIt X files begin with `StuffIt!` or `StuffIt?`.

# File structure #

## P2 Universal Code ##

Many values in StuffIt X files are encoded using a universal code apparently called "P2". These values can be decoded as follows:

  * Read and count bits, stopping when you encounter a 0 bit.
  * The number of bits read (including the final 0) is the number of 1 bits in the value.
  * Read bits until you have read as many 1 bits as the previously determined count.
  * The final value is the value encoded by this bit string, interpreted as starting with the least significant bit, minus 1.

Bits are read starting from the least significant bit of every successive byte, similarly to the Zip format.

### Examples ###

  * 0: 0 1
  * 1: 0 01
  * 2: 10 11
  * 3: 0 001
  * 4: 110 101
  * 5: 110 011
  * 6: 1110 111
  * 14: 11110 1111
  * 15: 0 00001
  * 16: 10 10001

## Header ##

Unknown. At least two bytes after the `StuffIt!` or `StuffIt?` marker exist before the start of the elements that make up the main file structure.

## Elements ##

The main data in the header is encoded as a series of "elements". These are identified by a numerical ID, and contain a series of attributes followed by an optional data section. An element header is parsed as follows:

  * Synchronize to a byte boundary.
  * Read a single bit flag (meaning unknown).
  * Read the element ID as a P2 value.
  * Loop through the attributes:
    * Read an attribute key as a P2 value.
    * If the key is 0, stop the loop.
    * Otherwise, read the attribute value as a P2 value.
  * Loop through the algorithm list:
    * Read an algorithm key as a P2 value.
    * If the key is 0, stop the loop.
    * Otherwise, read the algorithm value as a P2 value.
      * If the key is 4, read a second P2 value.

This may be followed by further data, and a data stream.

### Meaning of the attributes ###

  * Attribute 1 seems to be some kind of ID number.
  * Attribute 2 seems to be a parent reference, giving the ID number of the parent element.
  * Attribute 3 seems to also be some kind of reference.
  * Attribute 5 seems to be a length attribute. It can appear both on elements with and without a data payload. On elements with data payloads it seems to give the uncompressed size of the payload.

### Meaning of the algorithm list ###

  * Key 1 is the algorithm used for the data stream.
    * 0 is SitxBrimstoneAlgorithm.
    * 1 is SitxCyanideAlgorithm.
    * 2 is SitxDarkhorseAlgorithm.
    * 3 is SitxDeflateAlgorithm.
    * 4 is SitxBlendAlgorithm, which mixes SitxBrimstoneAlgorithm, SitxCyanideAlgorithm and SitxDarkhorseAlgorithm per-block.
    * 5 is no compression. However, the data is encrypted! The encryption is RC4, and the key is a single byte, which is the third byte in the data stream. The first two seem unimportant. The encrypted data begins with the fourth byte.
    * 6 is SitxIronAlgorithm.
    * 7 is SitxJpegAlgorithm.
  * Key 2 seems to be related to the data stream checksum.
    * 0 means CRC32?
  * Key 3 is the pre-processing algorithm used on the data before compressing.
    * 0 is SitxEnglishPreprocessingAlgorithm.
    * 1 is SitxBiffPreprocessingAlgorithm,
    * 2 is SitxX86PreprocessingAlgorithm,
    * 3 is SitxPeffPreprocessingAlgorithm,
    * 4 is SitxM68kPreprocessingAlgorithm,
    * 5 is SitxSparcPreprocessingAlgorithm,
    * 6 is SitxTiffPreprocessingAlgorithm,
    * 7 is SitxWavPreprocessingAlgorithm,
    * 8 is SitxWrtPreprocessingAlgorithm,

### Element types ###

| **Element ID** | **Name** | **Contents** | **Data payload**| **Meaning** |
|:---------------|:---------|:-------------|:----------------|:------------|
| 0              | End      |              | No              | End of archive |
| 1              | Data     |              | Yes             | A data stream |
| 2              | File     |              | No              | Defines a file in the directory tree structure. Parent flag gives the parent directory. |
| 3              | Fork     | 1\*P2        |No               | Ties a data stream to a file. Parent attribute gives the file, attribute 3 gives the data stream, attribute 5 the length of the data (might not be same as the length of the data stream - solid archives?). The P2 value defines whether this is a data (0) or resource (1) fork. |
| 4              | Directory |              | No              | Defines a directory in the directory tree structure. Parent flag gives the parent directory. |
| 5              | Catalog  |              | Yes             | Defines all metadata for the directory tree. Usually compressed with method 0. |
| 6              | Clue     | N bytes      | No              | Contains N bytes, starting at the next byte boundary, where N is given by the length attribute, 5. |
| 7              | Root     | 1\*P2        | No              |             |
| 8              | Boundary |              | No              |             |
| 9              | ?        |              | ?               |             |
| 10             | Receipt  |              | ?               |             |
| 11             | Index    |              | Yes             |             |
| 12             | Locator  |              | Yes             |             |
| 13             | ID       |              | Yes             |             |
| 14             | Link     |              | Yes             |             |
| 15             | Segment index |              | Yes             |             |
| >10            |          |              | Yes             | All elements larger than 10 seem to have a data payload |

## Data block streams ##

Some elements contain a data stream. This is encoded as series of blocks. Each block is preceeded by the block size encoded as a P2 value. This value is always placed on a byte boundary, and the following data also starts at a byte boundary. If the length is 0, this marks the end of the stream.

The data stream is followed by a checksum stream in the same format as the data stream. If no checksum is present, the checksum stream still exists but is of length 0.

## Catalog ##

The catalog element contains the metadata for all files and directories in the directory tree. It is made up of a list of keys stored as P2 and values in various formats, terminated by a 0 key for every file.

| **Key** | **Data type** | **Meaning** |
|:--------|:--------------|:------------|
| 0       |               | End of metadata for the current file. Bit stream is moved forward to the next byte boundary. |
| 1       | String        | File name.  |
| 2       | 64 bits       | File modification date given as ten millionths of a second since Jan 1, 1601. |
| 3       | 32 bits       | ?           |
| 4       | 32\*8 bits    | Finder info? |
| 5       | 32\*8 bits    | ?           |
| 6       | 8+32+(32+32) bits | Unix file attributes. First 8 bits give the fields present. If 0, only a 32-bit permissions value is present. If 1, 32-bit UID and GID values are also present. |
| 7       | P2            | ?           |
| 8       | 64 bits       | File creation date given as ten millionths of a second since Jan 1, 1601. |
| 9       | String        | Comment?    |
| 10      | P2, N strings | Some kind of path. First P2 value is a component count, then each component follows as a separate string. |
| 11      | String        | ?           |
| 12      | String        | ?           |
| >12     | P2+?          | All larger values are assumed to start with a P2 length. |

All P2 values and bit strings are packed together and not stored on byte boundaries. Strings are stored as a P2 value giving the length, followed by that many bytes which are stored at byte boundaries.