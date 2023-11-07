# Questions regarding single or multipart containers

## Single File Containers
### Pros
* Easy to implement
* Easy to describe (naming etc.)

### Cons
* Impossible to span very large files over multiple containers
* Some containers can only get so big (e.g. ZIP 4GB)

## Multipart Containers
### Pros
* Possible to span very large files over multiple containers
* More possibilities to store OCFL objects in storage systems
* Easier to transfer objects over network

### Cons
* More complex to implement
* More complex to describe (naming etc.)
* More complex to handle (e.g. copy, move, validate, ...)
* Might require manual partitioning of files (see next section)

### Types of multipart container files
Supporting multipart containers per version can be done in different ways:

* Independently split containers
  * Every sub-containers contains a fully usable container which can be used or packaged independently
* Binary container split 
  * Container is split into sub-containers at binary level and must be concatenated to get a usable container
* Multipart split by container format
  * Every container file contains a part of the complete container

| Type                                    | Pros                                | Cons                                                      |
| --------------------------------------- | ----------------------------------- | --------------------------------------------------------- |
| __Independently Split Containers__      | Works with all formats              | Potential overhead                                        |
|                                         | Easy to implement                   | Need to split containers manually/ with external programs |
|                                         | Usable in any order                 |                                                           |
|                                         | Can retrieve a single sub-container |                                                           |
|                                         |                                     |                                                           |
| __Binary Container Split__              | Works with all formats              | Needs to be combined for extraction                       |
|                                         |                                     | One corrupt sub-container breaks whole container          |
|                                         |                                     |                                                           |
| __Multipart split by Container Format__ | Native support if allowed by format | Complexity depends on format                              |
|                                         |                                     | Not all formats support this                              |

