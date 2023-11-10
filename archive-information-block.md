# What `archiveInformation` Block Should Contain, And How To Format It?

1. [What `archiveInformation` Block Should Contain, And How To Format It?](#what-archiveinformation-block-should-contain-and-how-to-format-it)
   1. [What is The Purpose of the `archiveInformation` Block?](#what-is-the-purpose-of-the-archiveinformation-block)
   1. [How Should the `archiveInformation` Block Be Structured?](#how-should-the-archiveinformation-block-be-structured)
      1. [Field Explanation](#field-explanation)
      1. [Simplification Suggestions](#simplification-suggestions)
      1. [Additional Fields That Could Be Added](#additional-fields-that-could-be-added)
   1. [Where Should the `archiveInformation` Block Be Located?](#where-should-the-archiveinformation-block-be-located)

## What is The Purpose of the `archiveInformation` Block?
The `archiveInformation` block will contain information about the container format, version, what tool packed it, if it's compressed, etc.
The goal of this block is to provide all necessary information to be able to understand, validate, and unpack the archive files.

## How Should the `archiveInformation` Block Be Structured?
> [!NOTE]
> The names of variables, structure, etc. are subject to change and are provided here as an example.
> We are open for further discussion, open an issue if you wish to propose changes.

A proposed JSON structure of this block is shown below:

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

### Field Explanation
Proposed fields for this block are:
- `archiveFormat`: identifier for the archive format used, either `zip` or `tar`
- `version`: version of the archive format used
- `packingInformation`: information about how the archive was packed
  - `packingTool`: name of the tool that packed the archive
  - `packingToolVersion`: version of the tool that packed the archive
  - `packingCommands`: list of commands that were used to pack the archive, using a list as multiple commands might be needed to pack the archive
- `unpackingInformation`: information about how the archive can be unpacked
  - `unpackingTool`: name of the tool to unpack the archive
  - `unpackingToolVersion`: version of the tool to unpack the archive
  - `unpackingCommands`: list of commands to use to unpack the archive, using a list as multiple commands might be needed to unpack the archive
- `compression`: information about the compression used, not present if archive is uncompressed
  - `algorithm`: name of the compression algorithm used, if applicable
  - `level`: level of compression used, if applicable

### Simplification Suggestions
Depending on the use case, some of the fields might be unnecessary or can be changed:
- The JSON objects `packingInformation`, `unpackingInformation` can be made optional.
  - It might be unnecessary to include information about how to unpack the archive, as the tool used to unpack the archive might be obvious.
- The `packingCommands` and `unpackingCommands` can be removed to generalize the block.
  - If these fields are removed, then the contents of `packingInformation` and `unpackingInformation` objects can be turned into simple fields.
- If compression is not included in the standard then the `compression` object can be removed.

Thus the block could be simplified to either one of the following:  
_**No compression block:**_
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
    }
}
```

_**No compression block; `packingCommands` and `unpackingCommands` removed:**_
```json
{
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
}
```

_**No compression block; `packingCommands` and `unpackingCommands` removed; contents of `packingInformation` and `unpackingInformation` changed to simple fields:**_
```json
{
  "archiveInformation": {
    "archiveFormat": "zip",
    "version": "6.3.3",
    "packingTool": "zip",
    "packingToolVersion": "3.0",
    "unpackingTool": "unzip",
    "unpackingToolVersion": "6.0"
}
```

_**No compression block; `packingCommands`, `unpackingCommands`, and `unpackingInformation` removed; contents of `packingInformation` changed to simple fields:**_
```json
{
  "archiveInformation": {
    "archiveFormat": "zip",
    "version": "6.3.3",
    "packingTool": "zip",
    "packingToolVersion": "3.0"
  }
}
```

### Additional Fields That Could Be Added
Here is a list of other fields that this block could contain:
- `mimeType`: the MIME type of the container file, e.g. `application/zip`
- `puid`: the Pronom Unique Identifier of the container file, e.g. `x-fmt/263`

## Where Should the `archiveInformation` Block Be Located?
The block could either be located at the top level of the inventory file, or in the individual version block.

- **Top level**:
  - _Pros_:
    - Only one block to validate
    - Only one block to update when needed
    - Avoids duplication of information
    - Best option if all versions are packed with the same tool and commands
      - Could also remove the `packingCommands` and `unpackingCommands` fields which would make the block more generic and make this proposition the best option
  - _Cons_:
    - Information is not version specific
    - Requires all versions to be packed with the same tool and commands
    - Would not allow multiple archive formats in same object (if that is allowed)
    - Confusing if using mixed version (packed and unpacked versions)
- **Version level**:
  - _Pros_:
    - Information is version specific
    - Allows different versions to be packed with different tools and commands
    - More flexible, and less confusing if using mixed versions
  - _Cons_:
    - Duplicates information
    - If all versions are packed with the same tool and commands, the information is duplicated
      - If commands are removed from the block, then this option is even less attractive

**Recommendation: Top level information, especially if "command" fields are dropped.**