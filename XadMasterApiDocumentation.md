# Introduction #

XADMaster.framework is the library used for all handling of archives in The Unarchiver. It is quite a flexible library, and can be used in many other applications too.

However, it still needs some documentation. This is mostly a placeholder for now, until I have the time to actually get around to writing something about it.

# Old vs. New API #

Up until version 1.6.1 of The Unarchiver, XADMaster.framework used an API based around a class named `XADArchive`, which represents an archive and its contents. This class represents an archive as a sequential list of file and directory entries, which can be queried and extracted. It was built on top of the low-level libxad API.

  * XadArchiveDocumentation contains the documentation for this API.

In 2.0 of The Unarchiver, XADMaster.framework introduces a new low-level API, built around the `XADArchiveParser` class. This class is used to parse an archive, and calls a delegate callback to report files as it finds them while scanning the file. It can then provide the caller with generalized stream handles for reading data from each file. These are implemented as subclasses of the class `CSHandle`.

The old `XADArchive` API still exists in 2.0, but it is now implemented on top of `XADArchiveParser` instead of libxad. libxad still remains as one component of the `XADArchiveParser` API.

  * XadArchiveParserDocumentation contains the documentation for the new API.
  * CsHandleDocumentation contains the documentation for the generalized stream API.

## Differences between the APIs ##

  * The old API generally scans an entire archive, and then lets you access all the entries, while the new API gives you each entry as it is found.
  * The old API folds data and resource forks together as one single entry, while the new API gives them to you as separate entries.
  * The old API only gives you access to certain bits of file metadata, while the new API gives you all metadata the parser has managed to find.
  * The new API is meant to be cross-platform, while the old API is Mac OS X-specific.
  * Implementing support for new formats is done by extending the new API.

# Adding support for new formats and algorithms #

Parsers for new formats are implemented as subclasses of `XADArchiveParser`. They provide methods for identifying a file, for parsing it, and for providing `CSHandle`s to read the data from files contained in the archive.

New algorithms are implemented as subclasses of `CSHandle`, or more likely one of its convenience subclasses, such as `CSStream`, `CSByteStream` or `CSBlockStream`. Each `XADArchiveParser` subclass handles instantiating the correct `CSHandle` subclasses for each file.