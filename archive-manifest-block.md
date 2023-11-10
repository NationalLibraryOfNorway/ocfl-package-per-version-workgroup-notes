# How To Implement Archive Manifest Information?

## What is The Purpose of the `archiveManifest` Block?
The `archiveManifest` block will contain information about the archives that contain the files of the version.
It will also indicate that the version has been packed into an archive

This block will be the manifest for the actual files in the version folder, while the existing `state` block describes the state of the object after unpacking the archive.

## How Should the `archiveManifest` Block Be Structured
> [!NOTE]
> The names of variables, structure, etc. are subject to change and are provided here as an example.
> We are open for further discussion, open an issue if you wish to propose changes.

The example of an `archiveManifest` with a single archive is shown below:
```json
{
  ...
  "state": {
    "cf83e1...a3e": [ "checksum.md5" ],
    "f15428...83f": [ "preservation/image.tiff" ],
    "85f2b0...007": [ "view/image.jpg" ]
  },
  "archiveManifest": {
    "fda345...34a": [ "content.zip" ]
  }
  ...
}
```     

The example of an `archiveManifest` with a several archives is shown below:
```json
{
  ...
  "state": {
    "d66d80...8bd": [ "checksum.md5" ],
    "2b0ff8...620": [ "preservation/large-image.tiff" ],
    "921d36...877": [ "view/large-image.jpg" ]
  },
  "archiveManifest": {
    "fda345...34a": [ "content.zip" ],
    "adf234...53f": [ "content.z01" ],
    "ea443b...76e": [ "content.z02" ]
  }
  ...
}
```

> [!IMPORTANT]
> As an alternative the contents of `state` and `archiveManifest` blocks could be swapped, if people think that would be more intuitive.
> But based on comment on [Use-Case 33](https://github.com/OCFL/Use-Cases/issues/33#issuecomment-1731776524) we went with the structure shown above.

## How to Include Information About Which Files Are Located in Which Container?
We were thinking about implementing information about which files are located in which container in multipart archives.
We have two possible proposals for solving that issue, both of which are described below.

### Add an `archiveContents` Block (Proposition 1)
Add an additional block that would contain information about which files are in which archive.
This would be conveyed by using the same identifiers (SHA-512 hashes) as in the `state` and `archiveManifest` blocks.

Single archive:
```json
{
  "state": {
    "cf83e1...a3e": [ "checksum.md5" ],
    "f15428...83f": [ "preservation/image.tiff" ],
    "85f2b0...007": [ "view/image.jpg" ]
  },
  "archiveManifest": {
    "fda345...34a": [ "content.zip" ]
  },
  "archiveContents": {
    "fda345...34a": [ "cf83e1...a3e", "f15428...83f", "85f2b0...007" ]
  }
}
```

Multiple archives:
```json
{      
  "state": {
    "d66d80...8bd": [ "checksum.md5" ],
    "2b0ff8...620": [ "preservation/large-image.tiff" ],
    "921d36...877": [ "view/large-image.jpg" ]
  },
  "archiveManifest": {
    "fda345...34a": [ "checksums.zip" ],
    "adf234...53f": [ "preservation.zip" ],
    "ea443b...76e": [ "view.zips" ]
  },
  "archiveContents": {
    "fda345...34a": [ "d66d80...8bd" ],
    "adf234...53f": [ "2b0ff8...620" ],
    "ea443b...76e": [ "921d36...877" ]
  }
}
```

### Restructure `state` and `archiveManifest` Blocks (Proposition 2)
Another proposal is to change the way `state` and `archiveManifest` block are structured and used.
In this case the `state` block would contain sub-objects, one for each archive file, and inside them there would be a listing of files in container.
The `archiveManifest` block would list archives with their identifiers, and those would be used as identifiers of sub-objects in `state` block.

Single archive:
```json
{
  "state": {
    "fda345...34a": {
      "cf83e1...a3e": [ "checksum.md5" ],
      "f15428...83f": [ "preservation/image.tiff" ],
      "85f2b0...007": [ "view/image.jpg" ]
    }
  },
  "archiveManifest": {
    "fda345...34a": [ "content.zip" ]
  }
}
```

Multiple archives:
```json
{      
  "state": {
    "fda345...34a": {
      "d66d80...8bd": [ "checksum.md5" ]
    },
    "adf234...53f": {
      "2b0ff8...620": [ "preservation/large-image.tiff" ]
    },
    "ea443b...76e": {
      "921d36...877": [ "view/large-image.jpg" ]
    }
  },
  "archiveManifest": {
    "fda345...34a": [ "checksums.zip" ],
    "adf234...53f": [ "preservation.zip" ],
    "ea443b...76e": [ "view.zips" ]
  }
}
```

**Recommendation: Proposal 2 seems like the better option to us, as it avoids duplication, and seems to be most consistent with current inventory.json structure.**

## What About Cases Where File Locations Are Not Known?
In some cases it might not be possible to know which files are located in which archive.
That's usually in case of binary split archives, where the archive is split into several parts.
In that case the `archiveManifest` block would list all files as belonging to the main archive, for example:

**Proposition 1:**
```json
{      
  "state": {
    "d66d80...8bd": [ "checksum.md5" ],
    "2b0ff8...620": [ "preservation/large-image.tiff" ],
    "921d36...877": [ "view/large-image.jpg" ]
  },
  "archiveManifest": {
    "fda345...34a": [ "content.zip" ],
    "adf234...53f": [ "content.z01" ],
    "ea443b...76e": [ "content.z02" ]
  },
  "archiveContents": {
    "fda345...34a": [ "d66d80...8bd", "2b0ff8...620", "921d36...877" ]
  }
}
```

**Proposition 2:**

```json
{      
  "state": {
    "fda345...34a": {
      "cf83e1...a3e": [ "checksum.md5" ],
      "f15428...83f": [ "preservation/image.tiff" ],
      "85f2b0...007": [ "view/image.jpg" ]
    }
  },
  "archiveManifest": {
    "fda345...34a": [ "content.zip" ],
    "adf234...53f": [ "content.z01" ],
    "ea443b...76e": [ "content.z02" ]
  }
}
```

This proposition would pretty much treat the binary split archives the same was as single archive files.

**Recommendation: We think that this is the best way to handle this case.**

## What About Cases Where One File Is Located In Several Archives?
We don't think this would happen at all, or at least very often.
In proposal 1 users could simply add the file identifier in several lists in `archiveContents` block.
In proposal 2 users could list the file in several sub-objects in `state` block.

Alternatively it can be decided that file is listed in only one archive.

**Recommendation: We think that we need more information to make a definitive decision on that problem.**
**However we think that the best solution would be to allow listing files in several archives as discussed above.**