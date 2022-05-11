# ansible-build-data
Holds generated but persistent results from building ansible.  This information
may be referred to by other projects and scripts.

## Structure of data

::

    ansible-build-data
    └── 3
        ├── ansible-3.0.0.deps
        ├── ansible-3.1.0.deps
        ├── ansible-3.build
        └── ansible.in

* Each major release of Ansible gets a subdirectory of the repository named
  according to the X.Y version number of Ansible.  (ex: `3`)

* Within each version directory, there is an `ansible.in` file which lists the
  collections that are in this release of Ansible.  The file consists of one
  `namespace.collection` per line.  This file is constructed by the person
  building Ansible for that release.

* There will also be a file, `ansible-X.build`.  This file contains lines which
  consist of `namespace.collection` followed by a version range like::

      awx.awx: >=11.0.0,<12.0.0

  The version range specifies potential versions of the collection that are
  backwards compatible with what was available when the initial Ansible-X.Y.0
  release was frozen.  Only versions of the collections within those ranges
  will be considered for Ansible minor releases.  This file will be created by the
  `antsibull-build new-ansible` command.

* Lastly, there will be multiple, `ansible-X.Y.Z.deps` files.  Those files contain
  lines which consist of `namespace.collection` followed by a single version like::

      awx.awx: 11.2.5

  The version specifies the exact version of the collection that appeared in that
  release of Ansible.  This file will be created by the `antsibull-build single`
  command.

## Adding a new collection

### Next Ansible major release

To add a collection to the next Ansible major release that has not reached feature freeze:

* Add the collection to the `ansible.in` file in a sub-directory named with a corresponding number.
* In the same sub-directory, add the collection to the `collection-meta.yaml` file.
  - The maintainer's GitHub user names need to be listed there.
  - If the collection does not provide a changelog in `changelogs/changelog.yaml`, the URL to the actual changelog needs to be added.

### The current Ansible major release

To add a collection to the next minor release of the current Ansible major version:

* Add the collection to the `ansible.in` file in a sub-directory named with a corresponding number.
* In the same sub-directory, add the collection and its version range to the `ansible-X.build` file.
* In the same sub-directory, add the collection to the `collection-meta.yaml` file.
  - The maintainer's GitHub user names need to be listed there.
  - If the collection does not provide a changelog in `changelogs/changelog.yaml`, the URL to the actual changelog needs to be added.

## Renaming a collection

In some situations, a collection included in Ansible is renamed with its content basically unchanged (up to renaming, adjusting documentation, and potentially other very small changes). In that case, the new collection can be included and the old collection removed if the following procedure is followed.

For simplicity, assume that the next minor Ansible release is X.Y.0, and that collection `foo.bar` with latest release a.b.c has been renamed to `baz.bam` with latest release A.B.C.

1. `baz.bam` A.B.C must be compatible to `foo.bar` a.b.c up to renaming plugins. No options must be renamed or defaults changed.
2. `baz.bam` A.B.C can be added to Ansible X.Y.0.
3. A deprecation warning is added to Ansible X.Y.0's changelog (`deprecated_features`) that `foo.bar` has been renamed to `baz.bam`, that Ansible (X+1).0.0 will start having deprecated redirects from `foo.bar` to `baz.bam`, and that `foo.bar` will be removed from a later major release of Ansible.
4. A new release `foo.bar` (a+1).0.0 is made which contains no more content, but only deprecated redirects to `baz.bam`. Ideally it will have a dependency on `baz.bam` so that users that install `foo.bar` will have working deprecated redirects.
5. Ansible (X+1).0.0 contains both `foo.bar` (a+1).0.0, and either a `baz.bam` A.B'.C' release or a later major release that is still compatible with `foo.bar` a.b.c as specified in 1.
6. `foo.bar` will be dropped from Ansible (X+2).0.0 (needs to be announced in its changelog as `removed_features`).
