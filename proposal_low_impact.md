# Proposal: Low Impact
Author: [Jürgen Enge](mailto:juergen@info-age.net)  
Status: Draft

"Low impact" means that the inventory.json structure is not changed.

## Decisions
* [Format](format_questions.md): ZIP
* [Single or Multifile Containers](single_multifile_questions.md): Both

## Proposal
Replace `content` folder with `content.zip` file.
The file contains the content of the folder __excluding__ the folder name `content`.
Every file has a sidecar with the checksum of the file. 

There are two ways of validating the content of the ocfl object:
* `full validation` via inventory.json and zip file content check
* `sidecar validation` by checking the zip file(s) only

### Migration
Since the inventory.json structure is not changed, the migration is quite easy in both directions. 
Every content folder can be replaced by the ZIP file and vice versa.

### Single File Container
Simple case where object content is stored in a single zip file.

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── v1
│  ├── content.zip
│  ├── content.zip.sha512
│  ├── inventory.json
│  └── inventory.json.sha512
└── v2
    ├── content.zip
    ├── content.zip.sha512
    ├── inventory.json
    └── inventory.json.sha512
```

### Multi Part Zip Container
More complex case where object content is stored in multiple zip files.
Naming conventions of multipart zip files are used.

> [!NOTE]
> Since this feature is not supported by all zip libraries and tools, it is __not__ recommended to make it part of the standard. 

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── v1
│  ├── content.zip
│  ├── content.zip.sha512
│  ├── content.z01
│  ├── content.z01.sha512
│  ├── content.z02
│  ├── content.z02.sha512
│  ├── inventory.json
│  └── inventory.json.sha512
└── v2
    ├── content.zip
    ├── content.zip.sha512
    ├── content.z01
    ├── content.z01.sha512
    ├── inventory.json
    └── inventory.json.sha512
```
 
### Multi-Zip Content
Files of the `content` folder are distributed over separate zip files. 
The zip file numbering follows the same rules as the version folder numbering. 

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── v1
│  ├── content.1.zip
│  ├── content.1.zip.sha512
│  ├── content.2.zip
│  ├── content.2.zip.sha512
│  ├── content.3.zip
│  ├── content.3.zip.sha512
│  ├── inventory.json
│  └── inventory.json.sha512
└── v2
    ├── content.1.zip
    ├── content.1.zip.sha512
    ├── content.2.zip
    ├── content.2.zip.sha512
    ├── inventory.json
    └── inventory.json.sha512
```

