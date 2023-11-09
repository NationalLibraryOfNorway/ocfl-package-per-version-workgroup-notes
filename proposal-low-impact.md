# Proposal: Low Impact
Author: [Jürgen Enge](mailto:juergen@info-age.net)  
Status: Draft

"Low impact" means that the inventory.json structure is not changed.

1. [Proposal: Low Impact](#proposal-low-impact)
   1. [Decisions](#decisions)
   1. [Proposal](#proposal)
      1. [Migration](#migration)
   1. [Example File Structure](#example-file-structure)
      1. [Single File Container](#single-file-container)
      1. [Multi Part Zip Container](#multi-part-zip-container)
      1. [Multi-Zip Content](#multi-zip-content)
   1. [Option With `inventory.json` Section](#option-with-inventoryjson-section)
      1. [Checksum Focus (Like `manifest`)](#checksum-focus-like-manifest)
      1. [Version Focus](#version-focus)
      1. [File Focus](#file-focus)

## Decisions
* [Container Format](container-and-format-questions.md): ZIP
* [Single VS Multipart containers questions](single-vs-multipart-containers-questions.md): both single, and multipart containers

## Proposal
Replace `content` folder with `content.zip` container.
The container contains the content of the folder __excluding__ the folder name `content`.
Every archive has a sidecar with the checksum of the file. 

There are two ways of validating the content of the ocfl object:
* `full validation` via inventory.json and zip file content check
* `sidecar validation` by checking the zip file(s) only

> [!NOTE]
> Should "mixed" versions,
> where some versions are stored in the `content` folder and some in
> the `content.zip` container be allowed?


### Migration
Since the inventory.json structure is not changed, the migration is quite easy in both directions. 
Every content folder can be replaced by the ZIP file and vice versa.

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

### Multi-Zip Content
Files of the `content` folder are distributed over separate zip files. 
The zip file numbering follows the same rules as the version folder numbering. 

```
├── 0=ocfl_object_1.1
├── extensions
├── inventory.json
├── inventory.json.sha512
├── v1
│   ├── content.1.zip
│   ├── content.1.zip.sha512
│   ├── content.2.zip
│   ├── content.2.zip.sha512
│   ├── content.3.zip
│   ├── content.3.zip.sha512
│   ├── inventory.json
│   └── inventory.json.sha512
└── v2
     ├── content.1.zip
     ├── content.1.zip.sha512
     ├── content.2.zip
     ├── content.2.zip.sha512
     ├── inventory.json
     └── inventory.json.sha512
```

## Option With `inventory.json` Section
Based on [issue comment](https://github.com/OCFL/Use-Cases/issues/33#issuecomment-1731776524)   
There could be a section in the `inventory.json` that lists the zip files.
In case of migration, there's need for rewriting the `inventory.json` file.

Validation equivalent to the `sidecar validation` above would mean to ignore 
`manifest`, `version` and `fixity` sections and use the `zip-manifest` section instead.


### Checksum Focus (Like `manifest`)
> [!NOTE]
> This is easiest to use, since it's similar to the existing `manifest` section.

```json
...
"zipManifest": {
  "14602d...609": [ "v1/content.zip" ],
  "1d4970...cd2": [ "v1/content.z01" ],
  "1e26aa...09d": [ "v2/content.zip" ]
}
...
```

### Version Focus
```json
...
"zipManifest": {
  "v1": {
    "14602d...609": [ "content.zip" ],
    "1d4970...cd2": [ "content.z01" ]
  },
  "v2": {
    "1e26aa...09d": [ "content.zip" ]
  }
}
...
```

### File Focus
```json
...
"zipManifest": {
  "v1/content.zip": "14602d...609",
  "v1/content.z01": "1d4970...cd2",
  "v2/content.zip": "1e26aa...09d"
}
...
```