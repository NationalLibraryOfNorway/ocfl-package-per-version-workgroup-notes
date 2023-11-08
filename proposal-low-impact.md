# Proposal: Low Impact
Author: [Jürgen Enge](mailto:juergen@info-age.net)  
Status: Draft

"Low impact" means that the inventory.json structure is not changed.

## Decisions
* [Format](format-questions.md): ZIP
* [Single VS Multipart containers questions](single-vs-multipart-containers-questions.md): Both

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
## Option with `inventory.json` section
Based on [issue comment](https://github.com/OCFL/Use-Cases/issues/33#issuecomment-1731776524)   
There could be a section in the `inventory.json` that lists the zip files.
In case of migration, there's need for rewriting the `inventory.json` file.

Validation equivalent to the `sidecar validation` above would mean to ignore 
`manifest`, `version` and `fixity` sections and use the `zip-manifest` section instead.


### Checksum-Focus (like `manifest`)
> [!NOTE]
> This is easiest to use, since it's similar to the existing `manifest` section.

```json
...
"zip-manifest": {
    "14602dae3c3aac15a9b0eb49a26b867a6436de8b1d20073c038d7fa60dcb28b32f2c8705fea38b110826364a543cb89e3fdc484ffcb6b255ea4c357b632a3609": [ 
        "v1/content.zip"
    ],
    "1d497009f3aabd230bcd055ef09fd180e63e330c47f4c1afdfc36172e5421a220b78e3e3ce30c9ae533c516accfa8976fd3f1198bd15ff79373d5fe87fc64cd2": [
        "v1/content.z01",
    ]
}
...
```

### Version Focus
```json
...
"zip-manifest": {
    "v1": {
            "14602dae3c3aac15a9b0eb49a26b867a6436de8b1d20073c038d7fa60dcb28b32f2c8705fea38b110826364a543cb89e3fdc484ffcb6b255ea4c357b632a3609": [ 
                "content.zip"
            ],
            "1d497009f3aabd230bcd055ef09fd180e63e330c47f4c1afdfc36172e5421a220b78e3e3ce30c9ae533c516accfa8976fd3f1198bd15ff79373d5fe87fc64cd2": [
                "content.z01",
            ]
        },
    "v2": {
            "1e26aa6a40bf18236744263777db7e2da7792818063c4895c30c384ee02878e56b1cb0cf94cc6c6ccf0d08331b166967567ca039a17421f19db006fac799309d": [ 
                "content.zip"
            ]
        }
    }
}
...
```

### File-Focus
```json
...
"zip-manifest": {
    "v1/content.zip": "14602dae3c3aac15a9b0eb49a26b867a6436de8b1d20073c038d7fa60dcb28b32f2c8705fea38b110826364a543cb89e3fdc484ffcb6b255ea4c357b632a3609",
    "v1/content.z01": "1d497009f3aabd230bcd055ef09fd180e63e330c47f4c1afdfc36172e5421a220b78e3e3ce30c9ae533c516accfa8976fd3f1198bd15ff79373d5fe87fc64cd2"
}
...
```