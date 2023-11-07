# Basic Questions regarding single or multipart containers

## Single File
### Pros
* Easy to implement
* Easy to describe (naming etc.)

### Cons
* Impossible to span very large containers over multiple files
* Some containers can only get so big (e.g. ZIP 4GB)

## Multipart Files
### Pros
* Possible to span very large containers over multiple files
* More possibilities to store OCFL objects in storage systems
* Easier to transfer objects over network

### Cons
* More complex to implement
* More complex to describe (naming etc.)
* More complex to handle (e.g. copy, move, validate, ...)

### Types of multipart container files
Supporting multipart containers per version can be done in different ways:
* Full container per file (every file contains a fully usable container)
* Binary file split (all files must be concatenated to get a usable container)
* Mulipart split by container format (every file contains a part of the container)

| Type                                    | Pros                   | Cons                                |
| --------------------------------------- | ---------------------- | ----------------------------------- |
| __Full Container__                      | works with all formats | potential overhead                  |
|                                         | easy to implement      |                                     |
|                                         | usable in any order    |                                     |
| __Binary File Split__                   | works with all formats | needs to be combined for extraction |
| __Multipart split by Container Format__ | native support         | complexity depends on format        |

