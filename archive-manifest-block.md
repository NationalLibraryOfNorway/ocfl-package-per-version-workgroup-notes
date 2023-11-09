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

### How to Include Information About Which Files Are Located in Which Container?
We were thinking about implementing information about which files are located in which container in multipart archives.
We have two possible proposals for solving that issue, both of which are described below.

#### Add an `archiveContents` Block
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
    "fda345...34a": [ "content.zip" ],
    "adf234...53f": [ "content.z01" ],
    "ea443b...76e": [ "content.z02" ]
  },
  "archiveContents": {
    "fda345...34a": [ "d66d80...8bd" ],
    "adf234...53f": [ "2b0ff8...620" ],
    "ea443b...76e": [ "921d36...877" ]
  }
}
```

#### Restructure `state` and `archiveManifest` Blocks
Another proposal is to change the way `state` and `archiveManifest` block are used.
In this case the `state` block would refer only to the archives in the version folder.
The `archiveManifest` block would contain information about which files are in which archive.

Single archive:
```json
{
  "state": {
    "fda345...34a": [ "content.zip" ]
  },
  "archiveManifest": {
    "fda345...34a": {
      "cf83e1...a3e": [ "checksum.md5" ],
      "f15428...83f": [ "preservation/image.tiff" ],
      "85f2b0...007": [ "view/image.jpg" ]
    }
  }
}
```

Multiple archives:
```json
{      
  "state": {
    "fda345...34a": [ "content.zip" ],
    "adf234...53f": [ "content.z01" ],
    "ea443b...76e": [ "content.z02" ]
  },
  "archiveManifest": {
    "fda345...34a": {
      "d66d80...8bd": [ "checksum.md5" ]
    },
    "adf234...53f": {
      "2b0ff8...620": [ "preservation/large-image.tiff" ]
    },
    "ea443b...76e": {
      "921d36...877": [ "view/large-image.jpg" ]
    }
  }
}
```