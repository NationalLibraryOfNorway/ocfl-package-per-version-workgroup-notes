# How Can Containers Limit the Number of Small Files?

1. [How Can Containers Limit the Number of Small Files?](#how-can-containers-limit-the-number-of-small-files)
   1. [What Small Files Are We Talking About?](#what-small-files-are-we-talking-about)
   1. [How Can We Limit the Number of Small Files?](#how-can-we-limit-the-number-of-small-files)
   1. [Suggested File Structures After Removal of Small Files](#suggested-file-structures-after-removal-of-small-files)
      1. [Only Removing the Inventory Files in Version Folders](#only-removing-the-inventory-files-in-version-folders)
      1. [Removing the Inventory Files in Version Folders and the Sidecar File](#removing-the-inventory-files-in-version-folders-and-the-sidecar-file)

## What Small Files Are We Talking About?
Ignoring contents of the `content` folder the basic file structure of an OCFL object is shown below:

```
[object root]
├── 0=ocfl_object_1.1
├── inventory.json
├── inventory.json.sha512
├── v1
│   ├── inventory.json
│   ├── inventory.json.sha512
│   └── content
│       └── ...
├── v2
│   ├── inventory.json
│   └── inventory.json.sha512
└── v3
    ├── inventory.json
    ├── inventory.json.sha512
    └── content
        └── ...
```

As shown there are 3 small objects in the root of the object:
- `0=ocfl_object_1.1` - a file containing the OCFL version of the object
- `inventory.json` - a file containing the inventory of the object
- `inventory.json.sha512` - a sidecar file containing the checksum of the inventory file

Both `inventory.json` and `inventory.json.sha512` are duplicated in each version folder.

## How Can We Limit the Number of Small Files?
The `inventory.json` and `inventory.json.sha512` files in root folder are required by the standard, so they cannot be removed.
It also doesn't make sense to pack them into an archive.
That's because the inventory file is needed to know what is in the object and the checksum file is needed to validate the inventory file.
Packaging them would increase latency and complexity of the software.
It's possible that if the storage system stores checksums of the files stored in it, or if the checksums are saved elsewhere (e.g. in a database), then the sidecar/checksum file could be removed.

> [!IMPORTANT]
> **Question to the OCFL community:** Does your storage system store checksums of the files stored in it? 
> If so, is it possible to use those checksums to validate the inventory file, and would you be comfortable with removing the sidecar file?
>
> Alternatively how comfortable are you with not having the checksum file at all?
> Since it only allows you to check if the inventory file was corrupted, but not fix it, it's only use is to detect if the file was corrupted.

It might be possible to move the object conformance file into the `inventory.json` file, which would remove one small file from the object.
Another option would be to remove the conformance file altogether, but that would make it more difficult to validate the object.
Lastly another option would be to rename the `inventory.json` file to include the OCFL object version, e.g.: `inventory.v1.1.json`, `inventory_1.1.json`, `0=ocfl_object_1.1_inventory.json`.
However those options are highly unlikely to work well and be implemented.

Lastly the `inventory.json` and `inventory.json.sha512` files in the version folders.
These could easily be packed into the archive files for the versions.
The `inventory.json` in root reflects the latest version and it knows what was in the previous versions.
This makes the inventory files in versions folders not needed to be immediately accessible.  
In cases of version that contain only `inventory.json` and `inventory.json.sha512` files, they could be in a zip of their own.
Or if we drop using a sidecar file then the `inventory.json` file could be left as is, or the version folder for that version could be removed altogether.

## Suggested File Structures After Removal of Small Files
### Only Removing the Inventory Files in Version Folders
This suggestion is the most lightweight, ane removes two small files per version, one if the version has inventory files only.
```
[object root]
├── 0=ocfl_object_1.1
├── inventory.json
├── inventory.json.sha512
├── v1
│   └── content.zip
├── v2
│   └── inventory.zip
└── v3
    ├── content.zip
    ├── content.z01
    └── content.z02
```

Alternatively if we don't have inventory files for version that don't have content files, then the structure would look like this:
```
[object root]
├── 0=ocfl_object_1.1
├── inventory.json
├── inventory.json.sha512
├── v1
│   └── content.zip
└── v3
    ├── content.zip
    ├── content.z01
    └── content.z02
```

### Removing the Inventory Files in Version Folders and the Sidecar File
This makes the minimal version of the object contain only two small files, and then each version loses eihter two or one small file.
```
[object root]
├── 0=ocfl_object_1.1
├── inventory.json
├── v1
│   └── content.zip
├── v2
│   └── inventory.json
└── v3
    ├── content.zip
    ├── content.z01
    └── content.z02
```