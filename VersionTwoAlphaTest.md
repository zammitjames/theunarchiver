# The Unarchiver 2.0 #

The Unarchiver 2.0 features extensive internal rewrites. The old libxad is mostly gone except for old formats, and new Objective-C based code handles most normal formats. This makes it much easier to add support for new formats and advanced functionality, but the rewrite has likely introduced a lot of new bugs. Therefore it would be extremely helpful if users could download it and try it on as many files as possible, especially any files that have caused problems in the past, but also anything else.

If you have any problems with the alpha version, please, file bugs on the issues page, and include the files that cause problems if at all possible.

Get the latest version from the downloads page:

http://code.google.com/p/theunarchiver/downloads/list

Obviously, this version may not work for you, so be prepared to downgrade to 1.6.1 if it causes issues for you.

# New Features #

  * Partial SITX support. Most compression modes in SITX should work fine, except the JPEG compression mode which was just too huge to reverse engineer properly. If anybody wants to help out and get that working, I'd be incredible grateful.
  * RAR encryption support. Should work, maybe. Test it out, and also test if any of the other RAR problems have been fixed by the rewrite!
  * Support for WinZip-style AES encryption in Zip.
  * Much improved 7-zip support, supporting most compression algorithms (but not encryption).
  * Support for Deflate64, bzip2, PPMd and LZMA in Zip.
  * Support for old-style standalone .lzma archives (but not the new-style ones, yet).
  * Support for XAR archives (gzip and bzip2 only, not LZMA yet, needs support for new-style standalone .lzma).
  * Support for some more obscure Mac OS-specific Zip extensions.
  * Support for some more old Mac OS formats: Some more DiskDoubler modes, Compact Pro, PackIt.
  * Support for ALZip.
  * Improved Tar support thanks to halcyon (http://halcy.de/).
  * Improved RPM support.