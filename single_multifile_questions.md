# Basic Questions regarding single or multifile containers

## Single File
### Pro
* Easy to implement
* Easy to describe (naming etc.)
### Contra
* impossible to span very large containers over multiple files

## Multiple Files
### Pro
* possible to span very large containers over multiple files
* more possibilities to store OCFL objects in storage systems
### Contra
* more complex to implement
* more complex to describe (naming etc.)
* more complex to handle (e.g. copy, move, validate, ...)

### Types of multiple files
Supporting multiple containers per version can be done in different ways:
* full container per file (every file contains a fully usable container)
* binary file split (all files must be concatenated to get a usable container)
* split by container format (every file contains a part of the container)

| Type                          | Pro                    | Contra                              |
|-------------------------------|------------------------|-------------------------------------|
| __Full Container__            | works with all formats | potential overhead                  |
|                               | easy to implement      |                                     |
|                               | usable in any order    |                                     |
| __Binary File Split__         | works with all formats | needs to be combined for extraction |
| __Split by Container Format__ | native support         | complexity depends on format        |

