File System Tagging Tools
=========================

A few command line tools to help organize files in a directory with tags that can be browsed, searched, and added through the file system itself through the use of symbolic links.

The basic idea is to have a directory full of normal files and "tag" subdirectories containing links to those files, allowing you to access all the files with a certain tag by simply accessing the files in the subdirectory.

Crosslink
---------

The `crosslink` tool allows you to access the intersection of multiple tags (i.e. files tagged with tag1 *and also* tag2) by expanding the hierarchy so that each tag subdirectory has it's own set of subdirectories of files shared with other tags.

In essense, this allows you to type any number of slash-separated tags *in any order* and find a directory containing links to all files sharing all of those tags (or not find one if no such files exist).

For example, say you have a file tree like this, with numbers being normal files and letters being directories (or links thereto):

    .
    ├── 1
    ├── 2
    ├── 3
    ├── a
    │   ├── 1 -> ../1
    │   └── 2 -> ../2
    ├── b
    │   ├── 2 -> ../2
    │   └── 3 -> ../3
    ├── c
    │   └── 2 -> ../2
    └── d
        └── 1 -> ../1

Running `crosslink` in this directory with no arguments will produce this tree:

    .
    ├── 1
    ├── 2
    ├── 3
    ├── a
    │   ├── 1 -> ../1
    │   ├── 2 -> ../2
    │   ├── b
    │   │   ├── 2 -> ../2
    │   │   └── c
    │   │       └── 2 -> ../2
    │   ├── c
    │   │   ├── 2 -> ../2
    │   │   └── b -> ../b/c
    │   └── d
    │       └── 1 -> ../1
    ├── b
    │   ├── 2 -> ../2
    │   ├── 3 -> ../3
    │   ├── a -> ../a/b
    │   └── c
    │       ├── 2 -> ../2
    │       └── a -> ../a/c
    ├── c
    │   ├── 2 -> ../2
    │   ├── a -> ../a/c
    │   └── b -> ../b/c
    └── d
        ├── 1 -> ../1
        └── a -> ../a/d

As you can see, because `2` was shared between `a`, `b`, and `c`, it can now be found in many places: `./a/b/2`, `./a/b/c/2`, `./a/c/2`, and `./b/c/2`, but through directory links also the other three permutations of two such as `./c/b/2` and the other five permutations of three such as `./c/b/a/2`. In contrast, `1` is only shared by `a` and `d`, so it only has two additional locations: `./a/c/1` and `./c/a/1`.

### Invocation

`crosslink` has three invocation methods.

* `crosslink [options]`
  * calling with no arguments will cross all directories in the current working directory
* `crosslink [options] dir`
  * calling with a single directory argument will cross all directories in the specified directory
* `crosslink [options] dir1 dir2 [dir3 ...]`
  * calling with two or more directories in the current current working directory will cross only the specified directories

#### Options

* `-d depth`
  * Sets the maximum depth to recurse through created directories. By default recursion is unlimited (but effectively limited to the number of directories minus two). Calling with `-d 0` will create only one extra level of directories.

This option can be useful when crossing a set of tags with few files sharing many tags, such that the files can be accurately identified with just a few of those tags. In all but the most extreme cases, there is little downside to simply recursing as far as possible, but if the program is being run often (e.g. by `incron` on every file addition), or if other programs need to work with the full directory structure, it can save some significant execution time. 

In extreme cases, if the depth of the tree would exceed forty, some of the links created might be unreadable as they exceed the kernel's hardcoded limit for following symlinks when resolving paths. Setting `-d 38` can prevent creating these links.

Listlinks and Makelinks
-----------------------

Two very small scripts to efficiently transport and recreate your tags, or any hierarchy of only directories and symlinks. Useful because preserving directories and link properties can be difficult over some mediums, but transporting text is easy. Just run `listlinks` in the directory you wish to recreate and send its output (however you wish) to the input of `makelinks` run in the directory you wish to recreate it in.

Relativize
----------

Simple script to convert absolute links (or any regular file really) into relative links to the same name in the parent directory. Useful if you want to tag files with a program that will only make copies or absolute links (e.g. an image viewer or graphical file manager) but still want the directory to be moveable without wasting space. Be careful about running in a directory with regular files. Though unlikely in most cases, should a different file of the same name exist in the parent directory it can result in data loss.

Future Plans
------------

Random ideas that I may or may not eventually implement: command line tagger that runs an external program on each file (e.g. image viewer, music player) while asking for tags; incron script to tag files by simply dropping them into the proper place in the crosslinked hierarchy; update relativize to link directly to original to solve (rare) symlink limit problem; triple digit lines (zero indexed).