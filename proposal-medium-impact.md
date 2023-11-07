# Proposal: Medium Impact
Author: [Daniel Aaron Salwerowicz](mailto:daniel@salwerowi.cz), based on "[Low Impact](proposal-low-impact.md)" proposal by: [Jürgen Enge](mailto:juergen@info-age.net)  
Status: Draft

"Medium impact" means that the inventory.json structure is not changed.
Instead an additional inventory-unpacked.json is used to describe the object after it has been unpacked.
(Name is subject to change, inventory-deflated.json or others are also possible.)

## Decisions
* [Format](format-questions.md): ZIP (although TAR could be used as well since it also supports multi-volume archives)
* [Single VS Multipart containers questions](single-vs-multipart-containers-questions.md): Both

## Proposal
Replace `content` folder with `content.zip` file.
Top level folder of the `content.zip` file should be `content` so that it can be "unpacked in situ".
Every `content.zip` file has a sidecar with the checksum of the file.

In this proposal the inventory.json described only the `content.zip` and is unaware of its contents.
On the other hand the inventory-unpacked.json will exclude `content.zip` and describe its unpacked content.
It will also contain a block of information about what version of zip file format was used, what tool packed it, if it's compressed, etc.
The goal of this block is to provide all necessary information to be able to unpack the `content.zip` file.
A tentative json structure of this block can be as follows:

```json
{
  "archiveInformation": {
    "zip": {
      "version": "6.3.3",
      "packingTool": "zip",
      "packingToolVersion": "3.0",
      "packingCommand" : "zip -r -6 -q -o content.zip content",
      "unpackingTool": "unzip",
      "unpackingToolVersion": "6.0",
      "unpackingCommand" : "unzip -q content.zip",
      "compression" : {
        "algorithm": "deflate",
        "level": 6
      }
    }
  }
}
```

There are two ways of validating the content of the ocfl object:
* `full validation` via validating inventory.json as normal, but also using inventory-unpacked.json to check zip file content
* `sidecar validation` by checking the zip file(s) only

### Migration
In the simplest version of migration the inventory.json changes the files references to only `content.zip` file, eg:

```json
{
  ...
  "fixity": {
    "md5": {
      "184f84e28cbe75e050e9c25ea7f2e939": [ "v1/content/checksums.md5" ],
      "c289c8ccd4bab6e385f5afdd89b5bda2": [ "v1/content/preservation/image.png" ],
      "d41d8cd98f00b204e9800998ecf8427e": [ "v1/content/view/image.jpg" ],
      "66709b068a2faead97113559db78ccd4": [ "v2/content/checksums.md5" ],
      "a6357c99ecc5752931e133227581e914": [ "v2/content/preservation/new-image.png" ],
      "b9c7ccc6154974288132b63c15db8d27": [ "v2/content/view/new-image.jpg" ]
    }
  }
  "manifest": {
    "cf83e1...a3e": [ "v1/content/checksums.md5" ],
    "f15428...83f": [ "v1/content/preservation/image.png" ],
    "85f2b0...007": [ "v1/content/view/image.jpg" ],
    "d66d80...8bd": [ "v2/content/checksums.md5" ],
    "2b0ff8...620": [ "v2/content/preservation/new-image.png" ],
    "921d36...877": [ "v2/content/view/new-image.jpg" ]
  },
  "versions": {
    "v1": {
      ...
      "state": {
        "cf83e1...a3e": [ "checksums.md5" ],
        "f15428...83f": [ "preservation/image.png" ],
        "85f2b0...007": [ "view/image.jpg" ]
      },
      ...
    },
    "v2": {
      ...
      "state": {
        "d66d80...8bd": [ "checksums.md5" ],
        "f15428...83f": [ "preservation/image.png" ],
        "2b0ff8...620": [ "preservation/new-image.png" ],
        "85f2b0...007": [ "view/image.jpg" ],
        "921d36...877": [ "view/new-image.jpg" ]
      },
      ...
    }
  }
  ...
}
```

will be changed to:

```json
{
  ...
  "fixity": {
    "md5": {
      "da39a3ee5e6b4b0d3255bfef95601890": [ "v1/content.zip" ],
      "5f5afdd89b5bda2d97113559db78ccd4": [ "v2/content.zip" ]
    }
  }
  "manifest": {
    "de823a...acc": [ "v1/content.zip" ],
    "080617...40c": [ "v2/content.zip" ]
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
        "080617...40c": [ "content.zip" ]
      },
      ...
    }
  }
  ...
}
```

While the contents of inventory-unpacked.json refer to files in OCFL object the same as inventory.json did before, with additional `archiveInformation` block.
It might be able to automate the creation of inventory-unpacked.json from inventory.json, but it might be easier to allow users to make their own decision on it.
In my mind the most likely way this feature would be implemented is where user creates new objects following this standard, or only migrate when adding new version to existing objects.
While migrating they could either package old versions (I don't think this is recommended as it would end up changing old versions) or just add new version as a package.
When adding new version as a package they would have to create inventory-unpacked.json for new version only and in inventory json just add reference to new content.zip file.
This could probably be automated in the tool to allow that.

### Single File Container
Simple case where object content is stored in a single zip file.

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── inventory-unpacked.json
├── inventory-unpacked.json.sha512
├── v1
│  ├── content.zip
│  ├── content.zip.sha512
│  ├── inventory.json
│  ├── inventory.json.sha512
│  ├── inventory-unpacked.json
│  └── inventory-unpacked.json.sha512
└── v2
    ├── content.zip
    ├── content.zip.sha512
    ├── inventory.json
    ├── inventory.json.sha512
    ├── inventory-unpacked.json
    └── inventory-unpacked.json.sha512
```

### Multi Part Zip Container
More complex case where object content is stored in multiple zip files.
Naming conventions of multipart zip files are used.

> [!NOTE]
> Since this feature is not supported by all zip libraries and tools, it is __not__ recommended to make it part of the standard.  
> Alternatively if the standard goes for TAR instead of ZIP it might be possible to use TAR multi-volume archives.

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── inventory-unpacked.json
├── inventory-unpacked.json.sha512
├── v1
│  ├── content.zip
│  ├── content.zip.sha512
│  ├── content.z01
│  ├── content.z01.sha512
│  ├── content.z02
│  ├── content.z02.sha512
│  ├── inventory.json
│  ├── inventory.json.sha512
│  ├── inventory-unpacked.json
│  └── inventory-unpacked.json.sha512
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
 
### Multi-Zip Content
Files of the `content` folder are distributed over separate zip files. 
The zip file numbering follows the same rules as the version folder numbering. 

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── inventory-unpacked.json
├── inventory-unpacked.json.sha512
├── v1
│  ├── content.1.zip
│  ├── content.1.zip.sha512
│  ├── content.2.zip
│  ├── content.2.zip.sha512
│  ├── content.3.zip
│  ├── content.3.zip.sha512
│  ├── inventory.json
│  ├── inventory.json.sha512
│  ├── inventory-unpacked.json
│  └── inventory-unpacked.json.sha512
└── v2
    ├── content.1.zip
    ├── content.1.zip.sha512
    ├── content.2.zip
    ├── content.2.zip.sha512
    ├── inventory.json
    ├── inventory.json.sha512
    ├── inventory-unpacked.json
    └── inventory-unpacked.json.sha512
```
