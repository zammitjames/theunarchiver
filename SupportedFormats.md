# Supported popular formats #

| Format | Support level | Notes |
|:-------|:--------------|:------|
| Zip    | Full          | Full support for the normal zip format, with additional support for AES encryption, Zip64 extensions for large files, Mac OS extensions of many different kinds, and several unusual compression methods (Deflate64, Bzip, LZMA, PPMd). Can also extract .EXE self-extracting files using Zip. |
| RAR    | Full          | Including encryption and multiple volumes. Can also extract .EXE self-extracting files using RAR. |
| 7z     | No encryption | Most compression methods are supported, but no encryption. Also supports Unix extensions. |
| Tar    | Full          |       |
| Gzip   | Full          |       |
| Bzip2  | Full          |       |
| LZMA, XZ | Full          | Both the old "LZMA-alone" format, usually named .lzma, and the new .xz format. |
| CAB    | Full          |       |
| MSI    | Full          | This format is also used by many other Microsoft formats, meanings that you can use The Unarchiver to extract internal data from DOC and PPT files, and others. There is probably no reason to do this, but you can. |
| NSIS   | Extensive     | Supports many different versions, starting from version 1.1o |
| EXE    | Some          | Many kinds of .exe self-extracting formats are supported. However, if you find one that is not, please post an issue on the bug tracker. |
| ISO    | Full          | ISO 9660 CDROM filesystem images, and also .bin raw images. |
| Split files | Basic         | Can join files named .001, .002 that do not use any extra wrapper format. |

# Supported old formats #

| Format | Support level | Was popular on | Notes |
|:-------|:--------------|:---------------|:------|
| StuffIt | No encryption | Mac OS         | Can unpack all files I've been able to locate. |
| StuffIt X | Partial       | Mac OS         | Can unpack many files, some more obscure features are still unsupported. JPEG compression is also unsupported. |
| DiskDoubler | Almost full   | Mac OS         | All compression algorithms are implemented, but a few rare ones are untested. If you have some files that do not work, please post them in [issue 149](http://code.google.com/p/theunarchiver/issues/detail?id=149) on the bug tracker. |
| Compact Pro | No encryption | Mac OS         |       |
| PackIt | Full          | Mac OS         |       |
| Cpio   | Full          | Unix           |       |
| Compress (.Z) | Full          | Unix           |       |
| ARJ    | No multi-part | DOS            |       |
| ARC, PAK | Full          | DOS            | Full support for all algorithms, including proprietary ones from PAK. Encryption only works in command-line utilities. |
| Ace    | Only old files | DOS            | No support for Ace 2.0 and up (WinAce). |
| Zoo    | Full          | DOS            |       |
| LZH    | Full          | DOS, Amiga (as LhA) |       |
| ADF    | FFS           | Amiga (emulated) | Can extract files from Amiga disk images using the regular FFS file system. |
| DMS    | FFS           | Amiga          | Can extract files from compressed Amiga disk images using the regular FFS file system. |
| LZX    | Full          | Amiga          |       |
| PowerPacker | Full          | Amiga          |       |
| LBR    | Full          | CP/M, DOS      |       |
| Squeeze | Full          | CP/M, DOS      |       |
| Crunch | Full          | CP/M           |       |
| ...    |               |                | Many other old formats, especially Amiga-specific ones, are also supported through libxad, but I have not made a full survey of which ones. |

# Supported unusual formats #

| Format | Support level | Notes |
|:-------|:--------------|:------|
| XAR    | Full          | Suggested replacement for Tar on Unix. Used in some newer .pkg files on Mac OS X. |
| RPM    | Full          | Linux package format. |
| ALZip  | No encryption | Archive format which is mainly popular in South Korea. Support for all known compression methods, including Bzip2, Deflate and obfuscated Deflate. |
| NSA, SAR | Partial       | Game data file. Can unpack all files I've found. If you have ones that do not unpack, please post an issue. |
| NDS    | Full          | Nintendo DS ROM image, which can contain a file system. |
| Zipx   | Full          | WinZip extended Zip files |

# Unsupported formats, for which support would be good to have #

| Format | Notes | Issue number |
|:-------|:------|:-------------|
| Ace 2.0 | Proprietary, requires reverse engineering. | [58](http://code.google.com/p/theunarchiver/issues/detail?id=58) |
| StuffIt X JPEG | Propietary, hugely complicated algorithm. Would require lots of reverse engineering. Supposedly patented, but I have been unable to locate the patent (which would contain useful information about the format). | [146](http://code.google.com/p/theunarchiver/issues/detail?id=146) |
| DMG    | Mac OS X disk images with a HFS+ filesystem. |              |
| PAR    | Not an archive format, but recovery records for missing archive parts. | [39](http://code.google.com/p/theunarchiver/issues/detail?id=39) |
| SHK    | Apple II archive format. | [119](http://code.google.com/p/theunarchiver/issues/detail?id=119) |
| Amiga compressors | The Amiga had a lot of different single-file compressors. Support for more of these would be useful. libxfd supports many of them, but is mostly 68k only. |              |
| Deb    | Linux package format. |              |