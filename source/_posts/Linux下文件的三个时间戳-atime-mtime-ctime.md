---
title: 'Linux下文件的三个时间戳:atime,mtime,ctime'
date: 2018-02-18 21:25:44
categories: Linux
tags: Linux
---

在linux系统下每个文件都有三个时间戳，分别为atime，mtime，ctime，具体解释如下：
- atime(access time):最近访问内容的时间
- mtime(modify time):最近修改内容的时间
- ctime(change time):最近更改文件的时间，包括文件名、大小、内容、权限、属主、属组等。

查看一个文件的这三个时间戳可以用stat命令：
```bash
[haohao@localhost test]$ stat file_timestamp 
  File: 'file_timestamp'
  Size: 12        	Blocks: 8          IO Block: 4096   regular file
Device: 100f5c8h/16840136d	Inode: 97910802    Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/  haohao)   Gid: ( 1000/  haohao)
Access: 2017-05-21 11:11:48.882598473 +0800
Modify: 2017-05-21 11:11:48.882598473 +0800
Change: 2017-05-21 11:11:48.882598473 +0800
 Birth: -

```
**查看文件可以更改文件的atime**，比如cat，more，less一个文件后，文件的atime会更新，还是上面示例文件：
```bash
[haohao@localhost test]$ cat file_timestamp 
hello,world
[haohao@localhost test]$ stat file_timestamp 
  File: 'file_timestamp'
  Size: 12        	Blocks: 8          IO Block: 4096   regular file
Device: 100f5c8h/16840136d	Inode: 97910802    Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/  haohao)   Gid: ( 1000/  haohao)
Access: 2017-05-21 11:14:09.815598462 +0800
Modify: 2017-05-21 11:11:48.882598473 +0800
Change: 2017-05-21 11:11:48.882598473 +0800
 Birth: -
```
可以看出文件的atime已经更新成最新的了。
**更改文件的内容会同时更新文件的mtime和ctime**，还是上面的示例文件，改变文件内容，然后stat查看：
```bash
[haohao@localhost test]$ echo "RNG" >> file_timestamp 
[haohao@localhost test]$ stat file_timestamp 
  File: 'file_timestamp'
  Size: 16        	Blocks: 8          IO Block: 4096   regular file
Device: 100f5c8h/16840136d	Inode: 97910802    Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/  haohao)   Gid: ( 1000/  haohao)
Access: 2017-05-21 11:14:09.815598462 +0800
Modify: 2017-05-21 11:17:23.968598448 +0800
Change: 2017-05-21 11:17:23.968598448 +0800
 Birth: -

```
可以看出文件内容更改后文件的mtime和ctime都更新了，atime保持不变。
**更改文件的名称，大小，权限等只会更新文件的ctime**，还是上面示例文件，更改下文件的文件名，然后stat查看：
```bash
[haohao@localhost test]$ mv file_timestamp file_timestamp.txt  
[haohao@localhost test]$ stat file_timestamp.txt 
  File: 'file_timestamp.txt'
  Size: 16        	Blocks: 8          IO Block: 4096   regular file
Device: 100f5c8h/16840136d	Inode: 97910802    Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/  haohao)   Gid: ( 1000/  haohao)
Access: 2017-05-21 11:14:09.815598462 +0800
Modify: 2017-05-21 11:17:23.968598448 +0800
Change: 2017-05-21 11:22:32.097598424 +0800
 Birth: -

```
可以看出只有ctime发生了变化。

