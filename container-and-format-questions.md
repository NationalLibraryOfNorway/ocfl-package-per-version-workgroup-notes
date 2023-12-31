# Basic Questions Regarding Containerized OCFL Versions

> [!NOTE]
> In this document container, package, and archive are used interchangeably.
> All of them refer to either packages of zip, tar, 7zip or other similar formats.
> By default the term container is used, and it refers to an uncompressed container.

1. [Basic Questions Regarding Containerized OCFL Versions](#basic-questions-regarding-containerized-ocfl-versions)
   1. [How Many Formats Should the Standard Allow?](#how-many-formats-should-the-standard-allow)
      1. [One Format](#one-format)
      1. [Multiple Formats](#multiple-formats)
      1. [Any Format](#any-format)
   1. [Should The Standard Allow Multipart Containers Per Version?](#should-the-standard-allow-multipart-containers-per-version)
   1. [Should The Standard Allow Encrypted Containers?](#should-the-standard-allow-encrypted-containers)
   1. [Should The Standard Allow Compression?](#should-the-standard-allow-compression)
   1. [Should The Standard Allow For Mixed Versions?](#should-the-standard-allow-for-mixed-versions)
   1. [What Container File Formats Should The Standard Allow?](#what-container-file-formats-should-the-standard-allow)
      1. [ZIP](#zip)
         1. [Pros](#pros)
         1. [Cons](#cons)
      1. [TAR](#tar)
         1. [Pros](#pros-1)
         1. [Cons](#cons-1)
      1. [7z](#7z)
         1. [Pros](#pros-2)
         1. [Cons](#cons-2)

## How Many Formats Should the Standard Allow?
It's a good idea to allow multiple formats.
But this will make it more difficult to implement software, which can deal with all formats.
Therefore in case where we have to choose one format over the other we should consider the following criteria:
- How well supported is the format?
- How likely is it that the format will be used and supported in the future?
- How many current users of OCFL use the format, either on it's own or in combination with OCFL?
  - If there are not enough OCFL users who employ file packaging, what format is most used in digital preservation in general?

### One Format
Implementing software must only know how to deal with one container format in OCFL objects, which makes implementation easier.
Standard might be too restrictive to users for whom given format is not suitable, so they might not use OCFL or make their own extensions.
Possible solution is to allow one package format, but allow users to add their own extensions to the standard, which they then have to implement themselves.

If the standard allows only one format, what format should it be?
Should the new format be chosen based on the criteria above, or strictly based on popularity or practical considerations?

**Recommendation: too restrictive**

### Multiple Formats
Would lead to more work to implement an OCFL compliant software. 
But software users can choose the best format for their specific use case.

If the standard allows multiple archive formats, how to handle them in the repository or object?
Should it be possible to use multiple container formats in one OCFL object?
How about several formats in the repository?

**Recommendation: in-scope for V2. Allowed formats: ZIP and TAR.**

### Any Format
It is practically impossible to implement software, which can deal with all container formats.
That would most likely result in low adoption rate and a lot of work for implementers.
Which could also result in either buggy/incomplete implementations or very specific implementations, which are not useful for others.
However that would provide most freedom to the users.

**Recommendation: out of scope for V2.**

## Should The Standard Allow Multipart Containers Per Version?
For very large objects, it might be useful to split the container into multiple files.
In addition it could help with transfer of large objects as it might be easier to transfer multiple smaller files than one large file.
For some institutions, it may be desirable to package files according to type of content - for example metadata, preservation files, access files, etc. 
This makes it possible to extract a container with only metadata, while avoiding touching a large container.

Since there are quite a bit of questions surrounding that topic, see [Single VS Multipart containers questions](single-vs-multipart-containers-questions.md) for more details.

**Recommendation: in-scope for V2.**

## Should The Standard Allow Encrypted Containers?
Encryption while useful will make it more difficult to implement OCFL software capable of handling that.
If you need to store sensitive data/files in the container, you should encrypt the files before packaging them.
Alternatively only allows access to the container/OCFL repository to authorized users.
In other words, encryption is not necessary for the container itself, there are other ways of handling that.

**Recommendation: out of scope for V2.**

## Should The Standard Allow Compression?
Compression could be useful to reduce the size of the container.
Some organizations might have limited funds and saving storage might be more important than saving processing power or risking quality loss.
In addition it might be useful to compress the container to reduce the time needed to transfer it.
Information about compression should be preserved in some way, so that the container can be extracted more easily in the future.
It's also important to consider how that would impact implementing software.

**Recommendation: in-scope for V2, but strictly optional, uncompressed by default.**

## Should The Standard Allow For Mixed Versions?
By mixed version we mean that some versions are in a container, while others are not (e.g. version 1 is in a container, while version 2 contains lose files).
Would there be any issues with such a solution, either in terms of implementation or preservation, maybe some other practical limitations?

**Recommendation: in-scope for V2.**

Another form of mixed versions would allow some versions to use one container format, while others use a different format (eg. mixing zip and tar).
Such a solution would be more complex to implement, and we don't see any benefit or use-case for it.

**Recommendation: out of scope for V2.**

## What Container [File Formats](https://en.wikipedia.org/wiki/List_of_archive_formats) Should The Standard Allow?

### [ZIP](https://en.wikipedia.org/wiki/ZIP_(file_format)) 
* [ZIP_PK](https://www.loc.gov/preservation/digital/formats/fdd/fdd000354.shtml), ZIP file format (PKWARE)
* [ZIP_6_2_0](https://www.loc.gov/preservation/digital/formats/fdd/fdd000355.shtml), ZIP file format, Version 6.2.0 (PKWARE)
* [ZIP_6_3_3](https://www.loc.gov/preservation/digital/formats/fdd/fdd000362.shtml), ZIP file format, Version 6.3.3 (PKWARE)
* [ZIP_21320_1](https://www.loc.gov/preservation/digital/formats/fdd/fdd000361.shtml), Document Container File: Core (based on ZIP 6.3.3)

To use filesizes greater than 4GB, ZIP64 extension has to be used

#### Pros
* Well known
* Proven to work in formats like jar, odt, docx, etc...
* Fast due to directory entries
* Streamable
* Compression ([deflate](https://en.wikipedia.org/wiki/Deflate)) optional per file
* Good development support

#### Cons
* Some overhead due to directory entries
* Appending new files normally not supported by common libraries
  * Not a problem since versions are immutable by standard

### [TAR](https://en.wikipedia.org/wiki/Tar_%28computing%29)
* [Tar](https://www.loc.gov/preservation/digital/formats/fdd/fdd000531.shtml), Tape Archive (tar) File Format Family

Tar can be used in conjunction with compression formats:
* gz
* bz
* br
* z
* ...

#### Pros
* Streamable
* Very simple
* Any compression can be used
* Easy to append new files (uncompressed only)
* Good development support

#### Cons
* No direct file access
* Cannot compress on a per file basis 

### [7z](https://en.wikipedia.org/wiki/7z)
* [7z](https://www.loc.gov/preservation/digital/formats/fdd/fdd000539.shtml), 7z File Format

#### Pros
* Open architecture (perhaps too open?)
* Supports multiple compression algorithms
* Large file size support
* Supports unicode filenames

#### Cons
* Lack of official format specification
  * The [LMZA compression specification](https://github.com/jljusten/LZMA-SDK/blob/master/DOC/7zFormat.txt) serves as a de facto specification, but there are other specifications as well
* Does not store filesystem permissions
* Lacks recovery information in case of corruption
