---
title: Order of files inside ZIP archives
date: 2019-03-28
description: ...and other things you never thought would matter
---

One of the most annoying bugs I ever had to track down has to do with the fact that
files in the ZIP archive are **ordered**. As a ZIP file consumer, you
won't notice it: if you get two archives where only order of the files
is changed and unpack them, you won't see any difference.

However, once you embark on a journey of reproducible builds, there's
nothing as frustrating as a cache miss with unchanged sources - and the
resulting ZIPs only differ when compared by checksum.

Reproducible example:

```
>>> from zipfile import ZipFile
>>> zf1 = ZipFile('test1.zip', 'w')
>>> zf1.writestr('a', '')
>>> zf1.writestr('b', '')
>>> zf1.close()

>>> zf2 = ZipFile('test2.zip', 'w')
>>> zf2.writestr('b', '')
>>> zf2.writestr('a', '')
>>> zf2.close()
```

```
$ md5sum test1.zip
72bf1a6d0e0ad7a0c2548168b701e4ad  test1.zip
$ md5sum test2.zip
f5fc3ae82cbd94c70b27f2d479648b26  test2.zip
```

[This](https://github.com/graknlabs/bazel-distribution/pull/94/) is the fix we applied - file names are pre-sorted
before getting packaged into the archive.
