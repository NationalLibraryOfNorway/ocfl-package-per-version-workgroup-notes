# Proposal: High Impact
Author: [Daniel Aaron Salwerowicz](mailto:daniel@salwerowi.cz), based on "[Medium Impact](proposal-medium-impact.md)" proposal and discussion in [Use-Case 33](https://github.com/OCFL/Use-Cases/issues/33) 
Status: Draft

"High impact" means that the `inventory.json` structure is changed, so the existing tools would break if they were not updated.

## Decisions
* [Format](format-questions.md): both ZIP and TAR
* [Single VS Multipart containers questions](single-vs-multipart-containers-questions.md): Both


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
      "184f84e28cbe75e050e9c25ea7f2e939": [ "v1/content.zip" ],
      "c289c8ccd4bab6e385f5afdd89b5bda2": [ "v2/content.zip" ],
      "d41d8cd98f00b204e9800998ecf8427e": [ "v2/content.z01" ],
      "66709b068a2faead97113559db78ccd4": [ "v2/content.z02" ],
      "a6357c99ecc5752931e133227581e914": [ "v3/content/checksum.md5" ],
      "a6357c99ecc5752931e133227581e914": [ "v3/content/preservation/unpacked-image.tiff" ],
      "b9c7ccc6154974288132b63c15db8d27": [ "v3/content/view/unpacked-image.jpg" ]
    }
  }
  "manifest": {
    "cf83e1...a3e": [ "v1/content/checksum.md5" ],
    "f15428...83f": [ "v1/content/preservation/image.tiff" ],
    "85f2b0...007": [ "v1/content/view/image.jpg" ],
    "d66d80...8bd": [ "v2/content/checksum.md5" ],
    "2b0ff8...620": [ "v2/content/preservation/new-image.tiff" ],
    "921d36...877": [ "v2/content/view/new-image.jpg" ]
  },
  "versions": {
    "v1": {
      "state": {
        "cf83e1...a3e": [ "checksum.md5" ],
        "f15428...83f": [ "preservation/image.tiff" ],
        "85f2b0...007": [ "view/image.jpg" ]
      },
      "archiveManifest": {
        "fda345...34a": [ "content.zip" ]
      },
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
      ...
    },
    "v2": {
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
      "archiveInformation": {
        "archiveFormat": "zip",
        "version": "6.3.3",
        "packingInformation": {
          "packingTool": "zip",
          "packingToolVersion": "3.0",
          "packingCommands" : [ "zip -r -s 10 -q -o content.zip content" ]
        },
        "unpackingInformation": {
          "unpackingTool": "unzip",
          "unpackingToolVersion": "6.0",
          "unpackingCommands" : [ "zip -FF content.zip --out merged.zip", "unzip -q merged.zip" ]
        }
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

### The `archiveManifest` Block
The name of the block is subject to change, could be `containerManifest`, `packageManifest`, etc.

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

#### Proposal Comparison
First proposal has the benefit of making the `archiveContents` block optional.
So in cases where the user does not care about the contents of the archives, they can omit the block.
Alternatively it might be impossible to determine the contents of the archives, for example in case of binary splits.
It can also handle cases where a single file is spread over multiple archives.
Negative is the duplication of data, which can be a problem in case of objects with many files.

The second proposal tries to make the `state` block more consistent with the way it has been used so far.
Additionally it avoids duplicating information.
However it would require the `archiveManifest` block to be present in all cases.
This can be a problem if contents of the archives are not known, or when same file is spread over several archives.

### The `archiveInformation` Block
The name of the block is subject to change, could be `containerInformation`, `packageInformation`, etc.
This block contains information about the archive file.

```json
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
```

Proposed contents:
- `archiveFormat`: either `zip` or `tar`
- `version`: version of the archive format
- `packingInformation`: information about the tool that packed the archive
  - `packingTool`: name of the tool that packed the archive
  - `packingToolVersion`: version of the tool that packed the archive
  - `packingCommands`: list of commands that were used to pack the archive, using a list here as you can use multiple commands to pack the archive
- `unpackingInformation`: information about the known tool that can be used to unpack the archive
  - `unpackingTool`: name of the tool to unpack the archive
  - `unpackingToolVersion`: version of the tool to unpack the archive
  - `unpackingCommands`: list of commands to use to unpack the archive, using a list here as you can use multiple commands to unpack the archive
- `compression`: information talking about the compression, not present if compression was not used
  - `algorithm`: name of the compression algorithm used, if applicable
  - `level`: level of compression used, if applicable

All of those elements are subject to change as the proposal is discussed.

### Validation
There are two ways of validating the content of the OCFL object:
* `full validation` validation with both `state` and `archiveManifest` blocks, would require either a tool that can validate contents of the archive or unpacking the archive and validating the contents
* `simple valication` by validating only archive or unpacked files, while not checking the contents of the archives

### Problems With The Proposal
We have identified following problems with the proposal:
* There is no easy way to validate the contents of the archive. 
  * The `inventory.json` can be validated as normal, but the container contents cannot be validated without unpacking the archive.
* There is no information about which files are in which container in multipart archives.
* The `state` block has to pretend that the files are unpacked, even though they are not.
* The `archiveInformation` block repeats a lot of data. 
  The standard could restrict OCFL objects to only one archive format per object.
  Then this block can be moved out of the `versions` block and into the `inventory.json` itself.
  That would mean that the `packingInformation` and `unpackingInformation` blocks would removed and users would have to unpack archives on their own.

## Migration
This proposition has very similar migration suggestion as for [Medium Impact](proposal-medium-impact.md) proposal.

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
