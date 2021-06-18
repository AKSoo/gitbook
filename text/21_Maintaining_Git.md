# Maintaining Git

## Ensuring good performance

On large repositories, git depends on compression to keep the history
information from taking up too much space on disk or in memory.

This compression is not performed automatically.  Therefore you
should occasionally run [git gc](https://git-scm.com/docs/git-gc):

    git gc

to recompress the archive.  This can be very time-consuming, so
you may prefer to run git-gc when you are not doing other work.

## Ensuring reliability

The [git fsck](https://git-scm.com/docs/git-fsck) command runs a number of
self-consistency checks on the repository, and reports on any problems. This
may take some time. The most common warning by far is about "dangling" objects:

    $ git fsck
    dangling commit 7281251ddd2a61e38657c827739c57015671a6b3
    dangling commit 2706a059f258c6b245f298dc4ff2ccd30ec21a63
    dangling commit 13472b7c4b80851a1bc551779171dcb03655e9b5
    dangling blob 218761f9d90712d37a9c5e36f406f92202db07eb
    dangling commit bf093535a34a4d35731aa2bd90fe6b176302f14f
    dangling commit 8e4bec7f2ddaa268bef999853c25755452100f8e
    dangling tree d50bb86186bf27b681d25af89d3b5b68382e4085
    dangling tree b24c2473f1fd3d91352a624795be026d64c8841f
    ...

Dangling objects are not a problem.  At worst they may take up a little
extra disk space.  They can sometimes provide a last-resort method for
recovering lost work.

## Removing large files

Git can become slow if very large files or a large number of smaller files are
stored. (Git is meant for code, not data.) Due to the structure of the Git
repositories, and the amount of information stored after each commit, deleting
a file does not necessarily delete it from your repository since the file may
still exist in the history. If you accidentally commit a large file, you should
try to fix it as soon as possible.

To delete any file (large, sensitive, etc.) completely from the repository, you
may need to modify the history by deleting the file from every commit that
contains it:

1. Since small mistakes rewriting Git history can corrupt your version control,
   you should first clone and back up your repository.
2. [git filter-branch](https://git-scm.com/docs/git-filter-branch) command with
   `--tree-filter` option can be used to delete the file from all commits.
3. After deleting the file completely, [git gc](https://git-scm.com/docs/git-gc)
   may be used to remove data without reference and recompress the repository.
