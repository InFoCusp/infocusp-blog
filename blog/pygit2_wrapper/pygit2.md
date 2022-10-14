# Pygit2 

## Introduction 

Git treats each files as blob object and directory as a tree. Each and every objects are stored as pack files in compressed format in the .git directory. Every pack file will have index ('.idx') file. Index file contains metadata about object and it has the entry of object and where its stored.

pygit2 is the way to communicate with .git directory the way git command does.

[![PYPI_PACKAGE DOCUMENTATION](https://pypi.org/project/pygit2/)](https://www.pygit2.org/)

Pygit2 is python wrapper to run git functionality via python program. It is build on top of the libgit2. [libgit2](https://github.com/libgit2/libgit2) is a pure C implementation of the Git core methods that enables you to integrate Git functionality into your program.

## Features of pygit2

- Important for dataset gathering of multiple git repositories.
    - We can get any commit, file, metadata from the repository.
- Easy to code as it is in python.
- (>10x) Faster than other available git wrappers like [GitPython](https://pypi.org/project/GitPython/) or [python-git-wrapper](https://pypi.org/project/python-git-wrapper/).

## Installation

```
pip install pygit2
```


## Implementation
=====================================

### Initialize empty git repository
```
from pygit2 import init_repository
repo = init_repository('my_folder/test', bare=False)
```

### Clone the repository
```
import pygit2
repoClone = pygit2.clone_repository('https://github.com/Preet-786/test.git', 'my_folder/test')
```

### Opening existing repository
```
from pygit2 import Repository
repo = Repository('my_folder/test')
```
### Add and commit changes

After modifying the files
```
repo.index.add_all()
repo.index.write()
tree = repo.index.write_tree()
parent, ref = repo.resolve_refish(refish=repo.head.name)
repo.create_commit(
    ref.name,
    repo.default_signature,
    repo.default_signature,
    "Commit message",
    tree,
    [parent.oid],
)
```
### Get head commit
```
head = repo.head
print(f'CommitId:{head.target}, Branch:{head.name}')
msg = head.log().__next__()
print(f'Commit message: {msg.message}')
print(f'Commiter: {msg.committer}')
```
Results:
```
CommitId:08d8b7f0de4575fc9dfe9da5157e000157630ae0, Branch:refs/heads/main
Commit message: commit: Commit message
Commiter: preet <preetpatel.nu@gmail.com>
```

### Check diff of the commit

We can get patch diff of the commit
```
commit_id = repo.head.target

# we can get object from it's id
commit_obj = repo.get(commit_id)

diff_obj = repo.diff(commit_obj, commit_obj.parents[0])
print(diff_obj.patch)
```
Result:
```
diff --git a/ini.py b/ini.py
deleted file mode 100644
index 8909c17..0000000
--- a/ini.py
+++ /dev/null
@@ -1 +0,0 @@
-hii
\ No newline at end of file
```

We can see the files which are changed during commit
```
commit_id = repo.head.target

# we can get object from it's id
commit_obj = repo.get(commit_id)

# We can insert two commit ids also in diff
diff_obj = repo.diff(commit_obj.parents[0], commit_obj)

for delta in diff_obj.deltas:
    # Status: M=modified, A=added, D=Deleted
    print("Status: ",delta.status_char())
    print(f"Old File:\nPath: {delta.old_file.path}\nData: {repo.get(delta.old_file.id).data}\n")
    print(f"New File:\nPath: {delta.new_file.path}\nData: {repo.get(delta.new_file.id).data}\n")
```

Result:

```
Status:  M
Old File:
Path: ini.py
Data: b'hii'

New File:
Path: ini.py
Data: b'hii\nnewly added line'
```
### Iterate over all objects

There are three type of objects
1. Commit: Commit object contains File structure, diff, patch everything regarding commit
2. Blob: It contains objects of files.
3. Tree: It refers to directory of repo

```
for obj_id in repo:
    obj = repo.get(obj_id)
    print(f'Object id: {obj.id}, Object type: {obj.type_str}')
```
Results:

```
Object id: 2bee0c854238d42b2ad1ec27e615a0ea675771b6, Object type: commit
Object id: 4113e3ac744218e9f702ad0bb5313d4c8d79b03c, Object type: tree
Object id: 32d328df61352fe95e90dc9342d2f85d2fb07f86, Object type: blob
Object id: f54fe20e4975b4934ff5cdf75ce54d1ddb122894, Object type: tree
Object id: 04a5899b6f9c0ccd381aaa039b098eaceb5a2f01, Object type: blob
.
.
```
### Iterate over all files of commit
We can iterate over all files from the commit object. And we can recursively traverse files and it's contents via directory present in the commit.