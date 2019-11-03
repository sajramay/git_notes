# Git Notes

## The Blob
<insert blob picture here>
There are three types of blob.  
- Content - only content and not file names or other meta data
- Trees - which are meta data about the Content blobs such as file names and permissions.  Trees group content blobs into directories.
- Commits - commits usually point at trees

```bash
$ git init .
$ tree .git/objects
.git/objects
├── info
└── pack

2 directories, 0 files
$
```

Let's create an empty file and add it
```bash
$ touch file.txt
$ git add file.txt
$ tree .git/objects
.git/objects
├── e6
│   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
├── info
└── pack

3 directories, 1 file
$
```
The content of the file has a header added to it to indicate what kind of content it is and then the result is zlib compressed and hashed using SHA-1.  Subdirectories that start with the first two bytes of the hash are used to store the content, so the hash for the above is `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391`

The has will always be the same for the same *content*.  So an empty file will always give the above hash which is efficient for sharing.  Any empty files in git, regardless of location and filename will be represented by the above object.

Let's commit the file and see how many other objects are created.

```bash
$ tree .git/objects
.git/objects
├── b9
│   └── c6159978df9993fa252ba5bb54e989b57cd977
├── bd
│   └── d68b0120ca91384c1606468b4ca81b8f67c728
├── e6
│   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
├── info
└── pack

5 directories, 3 files
$
```
There are now three objects per the above list. The empty file, the tree and the commit object itself. 

Let's add some content to the file

```bash
$ echo "hello" >> file.txt
$ git add file.txt
$ tree .git/objects
.git/objects
├── b9
│   └── c6159978df9993fa252ba5bb54e989b57cd977
├── bd
│   └── d68b0120ca91384c1606468b4ca81b8f67c728
├── ce
│   └── 013625030ba8dba906f756967f9e9ca394464a
├── e6
│   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
├── info
└── pack

6 directories, 4 files
$
```
Git will not pick up new content unless you add it.  Note that we now have a new blob because we have new content.  

We can use `git cat-file -p` to examine the contents of each blob.
```bash
$ git cat-file -p b9c6
tree bdd68b0120ca91384c1606468b4ca81b8f67c728
author thewaterwalker <xxxxxx+github@gmail.com> 1572031531 +0000
committer thewaterwalker <xxxxxx+github@gmail.com> 1572031531 +0000

first commit
$
```
`b9c6` is a commit object.  It points to a tree and can have a parent as well.

 ```bash
$ git cat-file -p bdd6
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    file.txt
saj@querencia:~/test_git$
 ```
 `bdd6` is the tree as referenced by the commit object.  The tree object when printed shows the file permissions/type/SHA-1 hash.  Note that they type could also be a `tree` which would represent other directories.
 
 What this means is that objects are shared across different trees and different commits.

 ## Refs, Branches and Tags

 A `ref` is a pointer to an object.  A `branch` is a ref to a commit. `Tags` are refs to commits but do not move automatically unlike branches when a new commit is added.
 
 ```bash
$ git tag a-tag -m "A tag"
$ git branch a-branch
$ tree .git/refs
.git/refs
├── heads
│   ├── a-branch
│   └── master
└── tags
    └── a-tag

2 directories, 3 files
$ cat .git/refs/heads/a-branch
b9c6159978df9993fa252ba5bb54e989b57cd977
$
 ```

