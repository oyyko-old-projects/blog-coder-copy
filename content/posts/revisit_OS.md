---
title: "Revisit_OS"
date: 2022-08-20T16:05:28+08:00
tags: [OS]
---

For some reason, I've been reviewing OS related stuff lately.

## hardlink and softlink

![hardlink and softlink](../hardlink.jpg)

`ln` creates hark links.
`ln -s` creates soft (a.k.a symbolic) links.

`ln -l` show the number of hardlinks points to the same inode.

hardlink points to the inode.
softlink points to the path.

pros and cons:
hardlink must be on the same partition, can only link to files.
softlink can link to a file or directory, no partition limitation.

Hard links are only valid within the same File System. Symbolic links can span file systems as they are simply the name of another file.

Underneath the file system, files are represented by inodes.

A file in the file system is basically a link to an inode.
A hard link, then, just creates another file with a link to the same underlying inode.

When you delete a file, it removes one link to the underlying inode. The inode is only deleted (or deletable/over-writable) when all links to the inode have been deleted.

A symbolic link is a link to another name in the file system.

Once a hard link has been made the link is to the inode. Deleting, renaming, or moving the original file will not affect the hard link as it links to the underlying inode. Any changes to the data on the inode is reflected in all files that refer to that inode.

```bash
➜  test git:(main) ✗ touch 111 222
➜  test git:(main) ✗ echo 111 > 111
➜  test git:(main) ✗ echo 222 > 222
➜  test git:(main) ✗ ln 111 l111   
➜  test git:(main) ✗ ln -s 222 s222
➜  test git:(main) ✗ ls            
111  222  l111  s222
➜  test git:(main) ✗ ls -l
总计 12
-rw-r--r-- 2 zhangjia zhangjia 4  8月20日 16:46 111
-rw-r--r-- 1 zhangjia zhangjia 4  8月20日 16:46 222
-rw-r--r-- 2 zhangjia zhangjia 4  8月20日 16:46 l111
lrwxrwxrwx 1 zhangjia zhangjia 3  8月20日 16:46 s222 -> 222
```

Now, why the soft link's file mode is lrwxrwxrwx.

`l` means soft link. The rest is same to a normal file mode.
