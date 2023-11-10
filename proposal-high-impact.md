# Proposal: High Impact
Author: [Daniel Aaron Salwerowicz](mailto:daniel@salwerowi.cz), based on "[Medium Impact](proposal-medium-impact.md)" proposal and discussion in [Use-Case 33](https://github.com/OCFL/Use-Cases/issues/33) 
Status: Draft

"High impact" means that the `inventory.json` structure is changed, so the existing tools would break if they were not updated.

1. [Proposal: High Impact](#proposal-high-impact)
   1. [Decisions](#decisions)
   1. [Proposal](#proposal)
      1. [The `archiveInformation` Block](#the-archiveinformation-block)
      1. [The `archiveManifest` Block](#the-archivemanifest-block)
      1. [Questions About `fixity` and `manifest` Blocks](#questions-about-fixity-and-manifest-blocks)
      1. [Validation](#validation)
      1. [Problems With The Proposal](#problems-with-the-proposal)
   1. [Migration](#migration)
   1. [Example File Structure](#example-file-structure)
      1. [Single File Container](#single-file-container)
      1. [Multi Part Containers](#multi-part-containers)
      1. [Mixed Content](#mixed-content)

## Decisions
* [Container Format](container-and-format-questions.md): both ZIP and TAR
* [Single VS Multipart containers questions](single-vs-multipart-containers-questions.md): both single, binary split, and independent containers

## Proposal
Instead of an additional `inventory-unpacked.json` file, the `inventory.json` is extended with a new `archiveManifest` block in version block.
This block works like a manifest for the version, while the existing `state` block describes the state of the object after unpacking the archive.
It will also contain a block of information about what kind of container is is, its format version, what tool packed it, if it's compressed, etc.
The goal of this block is to provide all necessary information to be able to unpack the container file.
Container compression is strictly optional, and by default not used.

Using both of those blocks would theoretically allow users to create mixed versions and even use various archive formats in the same object.

```json
{
  ...  
  "fixity": {
    "md5": {
      "8e34ba6c538f91951f1ca51b43d04274": [ "v1/content/checksum.md5" ],
      "cc37f0b10ae1387c4d7a8fe497440704": [ "v1/content/preservation/image.tiff" ],
      "d8bbe7df10e99deec69bfac0c2c6604d": [ "v1/content/view/image.jpg" ],
      "934ac782f62006d67d365e5842e069bf": [ "v2/content/checksum.md5" ],
      "5eda828cd9dde1d4c2ea5af43862b92b": [ "v2/content/preservation/new-image.tiff" ],
      "1264930d711a18138a670225a8875d1e": [ "v2/content/view/new-image.jpg" ],
      "a6357c99ecc5752931e133227581e914": [ "v3/content/checksum.md5" ],
      "a6357c99ecc5752931e133227581e914": [ "v3/content/preservation/unpacked-image.tiff" ],
      "b9c7ccc6154974288132b63c15db8d27": [ "v3/content/view/unpacked-image.jpg" ]
    }
  },
  "manifest": {
    "cf83e1...a3e": [ "v1/content/checksum.md5" ],
    "f15428...83f": [ "v1/content/preservation/image.tiff" ],
    "85f2b0...007": [ "v1/content/view/image.jpg" ],
    "d66d80...8bd": [ "v2/content/checksum.md5" ],
    "2b0ff8...620": [ "v2/content/preservation/new-image.tiff" ],
    "921d36...877": [ "v2/content/view/new-image.jpg" ],
    "d66d80...8bd": [ "v3/content/checksum.md5" ],
    "2b0ff8...620": [ "v3/content/preservation/unpacked-image.tiff" ],
    "921d36...877": [ "v3/content/view/unpacked-image.jpg" ]
  },
  "archiveInformation": {
    "archiveFormat": "zip",
    "version": "6.3.3",
    "packingInformation": {
      "packingTool": "zip",
      "packingToolVersion": "3.0"
    },
    "unpackingInformation": {
      "unpackingTool": "unzip",
      "unpackingToolVersion": "6.0"
    }
  },
  "versions": {
    "v1": {
      "state": {
        "2462fa...12b": {
          "cf83e1...a3e": [ "checksum.md5" ],
          "f15428...83f": [ "preservation/image.tiff" ],
          "85f2b0...007": [ "view/image.jpg" ]
        }
      },
      "archiveManifest": {
        "2462fa...12b": [ "content.zip" ]
      }
      ...
    },
    "v2": {
      "state": {
        "fda345...34a": {
          "d66d80...8bd": [ "checksum.md5" ],
          "2b0ff8...620": [ "preservation/large-image.tiff" ],
          "921d36...877": [ "view/large-image.jpg" ]
        }
      },
      "archiveManifest": {
        "fda345...34a": [ "content.zip" ],
        "adf234...53f": [ "content.z01" ],
        "ea443b...76e": [ "content.z02" ]
      }
      ...
    },
    "v3": {
      "state": {
        "d66d80...8bd": [ "checksum.md5" ],
        "2b0ff8...620": [ "preservation/unpacked-image.tiff" ],
        "921d36...877": [ "view/unpacked-image.jpg" ]
      }
      ...
    }
  }
  ...
}
```

### The `archiveInformation` Block
See the [document about `archiveInformation` block](archive-information-block.md) for more details.

In summary we propose this structure for the `archiveInformation` block:

_**Most complex solution:**_
```json
{
  "archiveInformation": {
    "archiveFormat": "zip",
    "version": "6.3.3",
    "packingInformation": {
      "packingTool": "zip",
      "packingToolVersion": "3.0",
      "packingCommands" : [ "zip -r -6 -q -o content.zip content" ]
    },
    "unpackingInformation": {
      "unpackingTool": "unzip",
      "unpackingToolVersion": "6.0",
      "unpackingCommands" : [ "unzip -q content.zip" ]
    },
    "compression" : {
      "algorithm": "deflate",
      "level": 6
    }
  }
}
```

_**Most bare bones solution:**_
```json
{
  "archiveInformation": {
    "archiveFormat": "zip",
    "version": "6.3.3",
    "packingTool": "zip",
    "packingToolVersion": "3.0"
}
```

### The `archiveManifest` Block
See the [document about `archiveManifest` block](archive-manifest-block.md) for more details.

In summary we propose this structure for the `archiveManifest` block:

_**Single archive:**_
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

_**Multiple archives:**_
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

_**Binary split archives:**_
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

### Questions About `fixity` and `manifest` Blocks
The big question is what to do with the `fixity` and `manifest` blocks.
For now in our proposal both blocks contain only information about unpacked files.
It might not be the best idea as that does not allow for storing alternative fixities for archives.
The other option is to either include information about both archives and unpacked files in both blocks or only in fixity block.

### Validation
There are two ways of validating the content of the OCFL object:
* `full validation` validation with both `state` and `archiveManifest` blocks, would require either a tool that can validate contents of the archive or unpacking the archive and validating the contents
* `simple valication` by validating only archive or unpacked files, while not checking the contents of the archives

### Problems With The Proposal
We have identified following problems with the proposal:
* There is no easy way to validate the contents of the archive
  * The `inventory.json` can be validated as normal, but the container contents cannot be validated without unpacking the archive
* There is no information about which files are in which container in multipart archives
* The `state` block has to pretend that the files are unpacked, even though they are not
* The `archiveInformation` block repeats a lot of data.
  The standard could restrict OCFL objects to only one archive format per object.
  Then this block can be moved out of the `versions` block and into the `inventory.json` itself.
  That would mean that the `packingInformation` and `unpackingInformation` blocks would removed and users would have to unpack archives on their own.

## Migration
This proposition has very similar migration suggestions as for [Medium Impact](proposal-medium-impact.md) proposal.
Here is the migration proposal:

Only create new objects following the new standard, while old are left as is.

The object can be fully migrated either on access or in batches.
That would involve packaging the files in `content` folder as desired and then creating a new object with new `inventory.json` file.

The object can also be allowed to be partially migrated while adding new version to an object.
That would involve packaging new version files and putting the required information into new blocks in `inventory.json` file.
The old versions would be left as is.

These approaches would result in a mixed collection of objects with and without new blocks in `inventory.json` with and without packaged versions.
Users could combine these approaches and the user could decide which objects to migrate and when.
This might also be done in batches, especially if a migration tool is developed.

I don't think many users would reprocess their whole repository to pack all objects at once.
That would be a rather expensive and error prone project.
However smaller repositories might want to do that.

## Example File Structure
### Single File Container
Simple case where object content is stored in a single zip file.

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── v1
│   ├── content.zip
│   ├── content.zip.sha512
│   ├── inventory.json
│   └── inventory.json.sha512
└── v2
     ├── content.zip
     ├── content.zip.sha512
     ├── inventory.json
     └── inventory.json.sha512
```

### Multi Part Containers
More complex case where object content is stored in multiple archives.
Naming conventions of multipart archive files are used.

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── v1
│   ├── content.zip
│   ├── content.zip.sha512
│   ├── content.z01
│   ├── content.z01.sha512
│   ├── content.z02
│   ├── content.z02.sha512
│   ├── inventory.json
│   └── inventory.json.sha512
└── v2
     ├── content.zip
     ├── content.zip.sha512
     ├── content.z01
     ├── content.z01.sha512
     ├── inventory.json
     └── inventory.json.sha512
```

### Mixed Content
Object with both single and multi part containers as well as lose files.

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── v1
│   ├── content.zip
│   ├── content.zip.sha512
│   ├── inventory.json
│   └── inventory.json.sha512
├── v2
│   ├── content.zip
│   ├── content.zip.sha512
│   ├── content.z01
│   ├── content.z01.sha512 
│   ├── content.z02
│   ├── content.z02.sha512
│   ├── inventory.json
│   └── inventory.json.sha512
└── v3
     ├── inventory.json
     ├── inventory.json.sha512
     └── content.z01
          ├── checksum.md5
          ├── preservation
          │   └── unpacked-image.tiff
          └── view
               └── unpacked-image.jpg
```
