# Questions Regarding Single or Multipart Containers

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
* Allows users to choose the best format for their specific use case
* More possibilities to store OCFL objects in storage systems
* Easier to transfer objects over network

### Cons
* More complex to implement
* More complex to describe (naming etc.)
* More complex to handle (e.g. copy, move, validate, ...)
* Might require manual partitioning of containers (see next section)

### Types of Multipart Container
Supporting multipart containers per version can be done in different ways:

* Independently split containers
  * Every sub-containers contains a fully usable container which can be used or packaged independently
  * Can be split logically (e.g. by similar content) or physically (e.g. by size)
* Binary container split 
  * Container is split into sub-containers at binary level and must be concatenated to get a usable container
  * Can only be split by size

#### Comparison Table

| Type                               | Pros                                | Cons                                                               |
| ---------------------------------- | ----------------------------------- | ------------------------------------------------------------------ |
| __Independently Split Containers__ | Works with all formats              | Potential overhead                                                 |
|                                    | Easy to implement                   | Might need to split containers manually / with external programs   |
|                                    | Usable in any order                 | Hard to define clearly, might end up in individual implementations |
|                                    | Can retrieve a single sub-container |                                                                    |
|                                    | Clear overview over file location   |                                                                    |
|                                    |                                     |                                                                    |
| __Binary Container Split__         | Works with all formats              | Some formats require splits to be combined for extraction          |
|                                    | Native support if allowed by format | One corrupt sub-container can break the whole container            |
|                                    |                                     | Complexity depends on format                                       |
|                                    |                                     | Not all formats support this                                       |

