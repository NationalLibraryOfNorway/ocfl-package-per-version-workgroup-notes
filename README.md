# OCFL Package Per Version Workgroup Notes
Repository for notes, ideas, etc, from the the workgroup dedicated to introducing package per version model into OCFL specification.

## General Questions
* [Container And Format Questions](container-and-format-questions.md)
* [Single VS Multipart Containers questions](single-vs-multipart-containers-questions.md)
* [What `archiveInformation` Should Contain?](archive-information-block.md)
* [How To Structure Archive Manfiest?](archive-manifest-block.md)
* [How Can Containers Limit the Number of Small Files?](limit-number-of-small-files.md)

## Proposal
* [Low Impact Proposal](proposal-low-impact.md): a sidecar file for each container
* [Medium Impact Proposal](proposal-medium-impact.md): add an `inventory-unpacked.json` file to each container
* [High Impact Proposal](proposal-high-impact.md): add new blocks to `inventory.json` file

## Motivation
[The Oxford Common File Layout (OCFL)](https://github.com/OCFL/spec "OCFL spec repository on GitHub") offers a good way of organizing files with focus on versioning.
However some people are interested in the use case where they can package contents of the version folders.
This has been discussed in [Use-Case 33](https://github.com/OCFL/Use-Cases/issues/33 "Discussion surrounding 'Package per version storage' use-case").
To further discuss and develop this idea this working group has been formed as we attempt to create a suggestion for how to define this use-case in the specification.

## Workgroup Members
Currently the workgroup consists of:

- [Jürgen Enge, University of Bazel](https://www.linkedin.com/in/j%C3%BCrgen-enge-287873)
- [Thomas Edvardsen, National Library of Norway](https://www.linkedin.com/in/thomasedvardsen)
- [Daniel Aaron Salwerowicz, National Library of Norway](https://www.linkedin.com/in/salwerowicz)
