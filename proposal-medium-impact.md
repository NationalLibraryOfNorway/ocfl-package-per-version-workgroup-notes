# Proposal: Medium Impact
Author: [Daniel Aaron Salwerowicz](mailto:daniel@salwerowi.cz), based on "[Low Impact](proposal-low-impact.md)" proposal by: [Jürgen Enge](mailto:juergen@info-age.net)  
Status: Draft

"Medium impact" means that the `inventory.json` structure is not changed.
Instead an additional `inventory-unpacked.json` is used to describe the object after it has been unpacked.
(Name is subject to change, `inventory-deflated.json` or others are also possible.)

1. [Proposal: Medium Impact](#proposal-medium-impact)
   1. [Decisions](#decisions)
   1. [Proposal](#proposal)
      1. [Changes to `inventory.json` and `inventory-unpacked.json`](#changes-to-inventoryjson-and-inventory-unpackedjson)
   1. [Migration](#migration)
   1. [Example File Structure](#example-file-structure)
      1. [Single File Container](#single-file-container)
      1. [Multi Part Containers](#multi-part-containers)
      1. [Mixed Content](#mixed-content)

## Decisions
* [Format](format-questions.md): both ZIP and TAR
* [Single VS Multipart containers questions](single-vs-multipart-containers-questions.md): both single, binary split, and independent containers

## Proposal
Replace `content` folder with `content.zip` (or `container.tar`, for simplicity only `.zip` extension is used but they are interchangeable) container.
Top level folder of the `content.zip` file should be `content` so that it can be "unpacked in situ".
Every `content.zip` container has a sidecar with the checksum of the archive file.

In this proposal the `inventory.json` describes only the `content.zip` and is unaware of its contents.
On the other hand the `inventory-unpacked.json` will exclude `content.zip` and describe OCFL objects after unpacking the archive.
It will also contain a block of information about what version of container format was used, what tool packed it, if it's compressed, etc.
The goal of this block is to provide all necessary information to be able to unpack the `content.zip` file.
A tentative json structure of this block can be as follows:

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

Note that the names of variables, structure, etc. are subject to change and are provided here as an example for further discussion.

There are two ways of validating the content of the OCFL object:
* `full validation` via validating `inventory.json` as normal, but also using `inventory-unpacked.json` to check container contents
* `sidecar validation` by validating `inventory.json` only, hoping that container was packed correctly

### Changes to `inventory.json` and `inventory-unpacked.json`
In the case of creating new OCFL object, `inventory.json` will simply reference only `content.zip` (or multiple containers) in `fixity`, `manifest`, and `state` blocks.
So, for example instead of following `inventory.json`:

```json
{
  ...
  "fixity": {
    "md5": {
      "184f84e28cbe75e050e9c25ea7f2e939": [ "v1/content/checksum.md5" ],
      "c289c8ccd4bab6e385f5afdd89b5bda2": [ "v1/content/preservation/image.tiff" ],
      "d41d8cd98f00b204e9800998ecf8427e": [ "v1/content/view/image.jpg" ],
      "66709b068a2faead97113559db78ccd4": [ "v2/content/checksum.md5" ],
      "a6357c99ecc5752931e133227581e914": [ "v2/content/preservation/new-image.tiff" ],
      "b9c7ccc6154974288132b63c15db8d27": [ "v2/content/view/new-image.jpg" ]
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
      ...
      "state": {
        "cf83e1...a3e": [ "checksum.md5" ],
        "f15428...83f": [ "preservation/image.tiff" ],
        "85f2b0...007": [ "view/image.jpg" ]
      },
      ...
    },
    "v2": {
      ...
      "state": {
        "d66d80...8bd": [ "checksum.md5" ],
        "2b0ff8...620": [ "preservation/new-image.tiff" ],
        "921d36...877": [ "view/new-image.jpg" ]
      },
      ...
    }
  }
  ...
}
```

it will only refer to containers:

```json
{
  ...
  "fixity": {
    "md5": {
      "da39a3ee5e6b4b0d3255bfef95601890": [ "v1/content.zip" ],
      "5f5afdd89b5bda2d97113559db78ccd4": [ "v2/content.zip" ],
      "ada2d97113df234b495601b053fa3e33": [ "v2/content.z01" ],
      "ea44b4a395601eb0bb4b07da2d971136": [ "v2/content.z02" ]
    }
  }
  "manifest": {
    "de823a...acc": [ "v1/content.zip" ],
    "080617...40c": [ "v2/content.zip" ],
    "adf234...53f": [ "v2/content.z01" ],
    "ea443b...76e": [ "v2/content.z02" ]
  },
  "versions": {
    "v1": {
      ...
      "state": {
        "de823a...acc": [ "content.zip" ]
      },
      ...
    },
    "v2": {
      ...
      "state": {
        "080617...40c": [ "content.zip" ],
        "adf234...53f": [ "content.z01" ],
        "ea443b...76e": [ "content.z02" ]
      },
      ...
    }
  }
  ...
}
```

While the contents of `inventory-unpacked.json` refer to unpacked files in OCFL object the same as `inventory.json` did before, with additional `archiveInformation` block.
In that case the new objects can be created with only `inventory.json`, while willfuly ignoring `inventory-unpacked.json`, if it's hard for users to create it.
That would of course lead to little information about what files exist in OCFL object, and only `sidecar validation` would be possible.
But that would allow for low effort implementation of this proposal.

Of course the better option would be to implement tool which would create `inventory-unpacked.json` for new objects.
Maybe even allow for migrating old objects to this new format, by containerizing old versions of objects and creating `inventory-unpacked.json` for them.
That would break the version immutability principle of OCFL, however users could either choose to ignore it or just create new object with same version content but packaged.

## Migration
It might be possible to automate the creation of `inventory-unpacked.json` from `inventory.json`, but it might be easier to allow users to make their own decision on it.

As we see it the most likely way how users of this feature would migrate is either:
- Create new objects following this standard, letting the old objects be.
  - Least impact to their collection, allows gradual adoption.
  - When adding new versions the object could get new version with `inventory-unpacked.json` while old versions are untouched.
  - Would result in a mixed collection of objects with and without `inventory-unpacked.json`.
- Migrate old objects when adding new version, during access, or in batches.
  - Would result in reprocessing whole object, but only when already accessing it for other reasons.
  - Less processing power needed, but more impact on users as retrieval of object would be slower (if migrating on access).
  - Reprocessing large number of files would be expensive, but could be done in small batches.

These first approach would result in a mixed collection of objects with and without `inventory-unpacked.json` where some objects are packaged while others are not.
Users could combine these approaches and the user could decide which objects to migrate and when.
This might also be done gradually, especially if no proper migration tool is developed.

While we don't think many users would reprocess their whole repository to containerize all objects at once, it is possible.
That would be a rather expensive and error prone project, however smaller repositories might want to do that.
It would be most viable option if a proper migration tool is developed.

## Example File Structure
### Single File Container
Simple case where object content is stored in a single container.

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── inventory-unpacked.json
├── inventory-unpacked.json.sha512
├── v1
│   ├── content.zip
│   ├── content.zip.sha512
│   ├── inventory.json
│   ├── inventory.json.sha512
│   ├── inventory-unpacked.json
│   └── inventory-unpacked.json.sha512
└── v2
     ├── content.zip
     ├── content.zip.sha512
     ├── inventory.json
     ├── inventory.json.sha512
     ├── inventory-unpacked.json
     └── inventory-unpacked.json.sha512
```

### Multi Part Containers
More complex case where object content is stored in multiple containers, either binary split or independently split.
Regular naming conventions of multipart containers will be used.

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── inventory-unpacked.json
├── inventory-unpacked.json.sha512
├── v1
│   ├── content.zip
│   ├── content.zip.sha512
│   ├── content.z01
│   ├── content.z01.sha512
│   ├── content.z02
│   ├── content.z02.sha512
│   ├── inventory.json
│   ├── inventory.json.sha512
│   ├── inventory-unpacked.json
│   └── inventory-unpacked.json.sha512
└── v2
     ├── content.zip
     ├── content.zip.sha512
     ├── content.z01
     ├── content.z01.sha512
     ├── inventory.json
     ├── inventory.json.sha512
     ├── inventory-unpacked.json
     └── inventory-unpacked.json.sha512
```

### Mixed Content
Object with both single and multi part containers as well as lose files.

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── inventory-unpacked.json
├── inventory-unpacked.json.sha512
├── v1
│   ├── content.zip
│   ├── content.zip.sha512
│   ├── inventory.json
│   ├── inventory.json.sha512
│   ├── inventory-unpacked.json
│   └── inventory-unpacked.json.sha512
├── v2
│   ├── content.zip
│   ├── content.zip.sha512
│   ├── content.z01
│   ├── content.z01.sha512 
│   ├── content.z02
│   ├── content.z02.sha512
│   ├── inventory.json
│   ├── inventory.json.sha512
│   ├── inventory-unpacked.json
│   └── inventory-unpacked.json.sha512
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
