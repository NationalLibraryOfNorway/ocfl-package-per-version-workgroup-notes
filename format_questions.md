# Basic questions regarding containerized OCFL versions

## Which format should the standard allow?

### One format
Software must only know about one format to deal with OCFL objects.

### Multiple formats
More work to implement software. But software can choose the best format for the use case.

### Any format
Impossible to implement software, which can deal with all formats.

## Should the standard allow multiple files per version?

For very large objects, it might be useful to split the container into multiple files.

## Should the standard allow encrypted containers?

 
## [File formats](https://en.wikipedia.org/wiki/List_of_archive_formats)

### [ZIP](https://en.wikipedia.org/wiki/ZIP_(file_format)) 
* [ZIP_PK](https://www.loc.gov/preservation/digital/formats/fdd/fdd000354.shtml), ZIP file format (PKWARE)
* [ZIP_6_2_0](https://www.loc.gov/preservation/digital/formats/fdd/fdd000355.shtml), ZIP file format, Version 6.2.0 (PKWARE)
* [ZIP_6_3_3](https://www.loc.gov/preservation/digital/formats/fdd/fdd000362.shtml), ZIP file format, Version 6.3.3 (PKWARE)
* [ZIP_21320_1](https://www.loc.gov/preservation/digital/formats/fdd/fdd000361.shtml), Document Container File: Core (based on ZIP 6.3.3)

To use filesizes greater than 4GB, ZIP64 extension has to be used

#### Pro
* Well known
* Proven to work in formats like jar, odt, docx, ...
* fast due to directory entries
* streamable
* compression ([deflate](https://de.wikipedia.org/wiki/Deflate)) optional per file
* good support for development

#### Contra
* some overhead due to directory
* appending new files normally not supported by common libraries

### [TAR](https://en.wikipedia.org/wiki/Tar_%28computing%29)

* [Tar](https://www.loc.gov/preservation/digital/formats/fdd/fdd000531.shtml), Tape Archive (tar) File Format Family

Tar can be used in conjunction with compression formats
* gz
* bz
* br
* z
* ...

#### Pro
* streamable
* very simple
* any compression can be used
* easy to append new files (uncompressed only)
* good support for development

#### Contra
* no direct file access
* compression not on a per file basis 

### [7z](https://de.wikipedia.org/wiki/7z)

* [7z](https://www.loc.gov/preservation/digital/formats/fdd/fdd000539.shtml), 7z File Format
 
