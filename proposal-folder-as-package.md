# Proposal: Folder as package
Author: [Jürgen Enge](mailto:juergen@info-age.net)  
Status: Draft

This proposal has no impact on `inventory.json` and defines, that every package (i.e. zip) is treated as a folder. This applies to any known folder of the OCFL Object standard. 

### Known Folders
* __\<version>__  (v1, v2, ...)
   * __\<version>/content__ (v1/content, v2/content, ...)
* __logs__
* __extensions__
   * __extensions/\<name>__ (extensions/0001-digest-algorithms, ...)

### Rules
* Known object folders may be replaced by a package with the foldername and the extension (i.e. `extensions.zip`) of the package format
* Any package has a sidecar file with the package name and the checksum algorithm as extension (i.e. `extensions.zip.sha512`) containing the checksum.
* A package MUST NOT contain another package

### Example
This means, that a file structure like

```
[object root]
    ├── 0=ocfl_object_1.1
    ├── extensions
    │   └── 0001-digest-algorithms
    │       └── config.json
    ├── inventory.json
    ├── inventory.json.sha512
    └── v1
        ├── inventory.json
        ├── inventory.json.sha512
        └── content
            └── file.txt
```

could look like

```
[object root]
    ├── 0=ocfl_object_1.1
    ├── extensions.zip
    ├── extensions.zip.sha512
    ├── inventory.json
    ├── inventory.json.sha512
    ├── v1.zip
    └── v1.zip.sha512
```

or

```
[object root]
    ├── 0=ocfl_object_1.1
    ├── extensions
    │   ├── 0001-digest-algorithms.zip
    │   └── 0001-digest-algorithms.zip.sha512
    ├── inventory.json
    ├── inventory.json.sha512
    └── v1
        ├── inventory.json
        ├── inventory.json.sha512
        ├── content.zip
        └── content.zip.sha512
```
