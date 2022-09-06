# Linux 链接概念

​	Linux 链接分两种，一种被称为硬链接（Hard Link），另一种被称为符号链接（Symbolic Link）。默认情况下，**ln** 命令产生硬链接。

## 硬连接

​	硬连接指通过索引节点来进行连接。在 Linux 的文件系统中，保存在磁盘分区中的文件不管是什么类型都给它分配一个编号，称为索引节点号(Inode Index)。在 Linux 中，多个文件名指向同一索引节点是存在的。比如：A 是 B 的硬链接（A 和 B 都是文件名），则 A 的目录项中的 inode 节点号与 B 的目录项中的 inode 节点号相同，即一个 inode 节点对应两个不同的文件名，两个文件名指向同一个文件，A 和 B 对文件系统来说是完全平等的。删除其中任何一个都不会影响另外一个的访问。

​	硬连接的作用是允许一个文件拥有多个有效路径名，这样用户就可以建立硬连接到重要文件，以防止“误删”的功能。其原因如上所述，因为对应该目录的索引节点有一个以上的连接。只删除一个连接并不影响索引节点本身和其它的连接，只有当最后一个连接被删除后，文件的数据块及目录的连接才会被释放。也就是说，文件真正删除的条件是与之相关的所有硬连接文件均被删除。

> 硬连接相当于是引用计数。

​	硬链接存在 2 个限制：

1. 不允许用户给目录创建硬链接，即：用户不可以，操作系统可以(想一下每个目录下的 . 和 ..);
2. 只有在同一个文件系统中的文件，才能创建硬链接，也就是说：不能跨文件系统;

## 软连接

​	另外一种连接称之为符号连接（Symbolic Link），也叫软连接。软链接文件有类似于 Windows 的快捷方式。它实际上是一个特殊的文件。在符号连接中，文件实际上是一个文本文件，其中包含的有另一文件的位置信息。比如：A 是 B 的软链接（A 和 B 都是文件名），A 的目录项中的 inode 节点号与 B 的目录项中的 inode 节点号不相同，A 和 B 指向的是两个不同的 inode，继而指向两块不同的数据块。但是 A 的数据块中存放的只是 B 的路径名（可以根据这个找到 B 的目录项）。A 和 B 之间是“主从”关系，如果 B 被删除了，A 仍然存在（因为两个是不同的文件），但指向的是一个无效的链接。

​	软链接应用之：

1. 灵活切换不同版本的目标程序

​	在开发的过程中，对于同一个工具软件，可能要安装多个不同的版本，例如：`Python2` 和 `Python3`， `JDK8` 和 `JDK9` 等等。

​	此时就可以通过软链接来指定当前使用哪个版本。例如在我的电脑中：

```shell
$ ll -l /usr/bin/python*


lrwxrwxrwx 1 root root       9 12月 31 08:19 /usr/bin/python -> python2.7*
lrwxrwxrwx 1 root root       9 12月 31 08:19 /usr/bin/python2 -> python2.7*
-rwxr-xr-x 1 root root 3492624 3月   2 04:47 /usr/bin/python2.7*
lrwxrwxrwx 1 root root       9 12月 31 08:19 /usr/bin/python3 -> python3.5*
-rwxr-xr-x 2 root root 4456208 1月  27 02:48 /usr/bin/python3.5*
```

​	当在终端窗口中输入：`python` 时，启动的是 `python2.7` 版本。

​	如果有一天我需要使用 `python3.5` 版本，只需要把软链接 `python` 指向 `python3.5` 即可。



2. 动态库版本管理

​	在 `Linux` 系统的动态库版本管理中，有一个 `SONAME` 的概念。

​	我们在编译一个动态链接库时，一般使用如下编译命令：

```shell
$ gcc -fPIC -shared -o libhello.so hello.c
```

​	在使用这个动态库时，需要链接这个库：`-llibhello`。

​	简单的 `demo` 可以这么来写，但是如果遇到一些比较大的项目，需要执行严格的版本管理，那应该怎么来操作呢？

​	`Linux` 系统已经为我们想到了问题的解决方案，利用 `SO-NAME`。

​	首先，在编译动态链接库文件时，就指定产生 `SO-NAME`，它会被存储在动态链接库 `ELF` 文件中。

​	我们来直接看一个优秀的开源工具 `libevent` 的例子：

```go
$ ll /usr/lib/libevent-2.1.so*

lrwxrwxrwx 1 root root     17 Jul 27  2020 /usr/lib/libevent-2.1.so -> libevent-2.1.so.7
lrwxrwxrwx 1 root root     21 Jul 27  2020 /usr/lib/libevent-2.1.so.7 -> libevent-2.1.so.7.0.1
-rw-r--r-- 1 root root 412016 Jul 27  2020 /usr/lib/libevent-2.1.so.7.0.1
```

​	此时使用 `readelf` 命令来查看生成的动态库文件 `libevent-2.1.so.7.0.1`：

```go
$ readelf -a libevent-2.1.so.7.0.1 | grep SONAME
 0x000000000000000e (SONAME)             Library soname: [libevent-2.1.so.7]
```

​	它这么做有什么好处呢？

​	`Linux` 系统在查找动态链接库文件时，会到下面这 `3` 个默认目录下查找(当然然还有其他目录，比如：当前目录，`LD_LIBRARY_PATH` 指定的目录)

- /lib: 存放操作系统最关键和基础的库文件;
- /usr/lib: 存放一些非系统运行时所需要的关键库文件;
- /usr/local/lib: 存放用户自己安装的一些第三方库文件;

​	系统中安装的所有动态链接库，借助 `ldconfig` 这个程序，会自动的创建、更新或者删除对应的 `SONAME`(它是一个软链接，链接到 实际的库文件)，并把这些 `SONAME` 汇总到一个文件 `/etc/ld.so.cache` 中缓存起来。

​	这样，当动态库加载器查找动态库文件时，就可以直接在这个缓存文件中进行查找，加快了动态库的查找速度。



3. 快捷方式

​	利用软链接的快捷方式功能就比较好理解了，想一想：我们为什么在 `Windows` 的桌面上创建很多软件的快捷方式啊？

​	在 `Linux` 中同样如此！比如：最近一段时间的工作，每次都要打开一个路径很深的文件。如果在资源管理器中，一层一层的点击鼠标，是不是比较浪费时间。此时，就可以在桌面上创建一个软链接，每次直接双击就打开所链接的目标文件了。

## 通过实验加深理解

```shell
[oracle@Linux]$ touch f1          #创建一个测试文件f1
[oracle@Linux]$ ln f1 f2          #创建f1的一个硬连接文件f2
[oracle@Linux]$ ln -s f1 f3       #创建f1的一个符号连接文件f3
[oracle@Linux]$ ls -li            # -i参数显示文件的inode节点信息
total 0
9797648 -rw-r--r--  2 oracle oinstall 0 Apr 21 08:11 f1
9797648 -rw-r--r--  2 oracle oinstall 0 Apr 21 08:11 f2
9797649 lrwxrwxrwx  1 oracle oinstall 2 Apr 21 08:11 f3 -> f1
```

​	从上面的结果中可以看出，硬连接文件 f2 与原文件 f1 的 inode 节点相同，均为 9797648，然而符号连接文件的 inode 节点不同。

```shell
[oracle@Linux]$ echo "I am f1 file" >>f1
[oracle@Linux]$ cat f1
I am f1 file
[oracle@Linux]$ cat f2
I am f1 file
[oracle@Linux]$ cat f3
I am f1 file
[oracle@Linux]$ rm -f f1
[oracle@Linux]$ cat f2
I am f1 file
[oracle@Linux]$ cat f3
cat: f3: No such file or directory
```

​	通过上面的测试可以看出：当删除原始文件 f1 后，硬连接 f2 不受影响，但是符号连接 f3 文件无效

## 总结

​	依此您可以做一些相关的测试，可以得到以下全部结论：

- 1).删除符号连接f3,对f1,f2无影响；
- 2).删除硬连接f2，对f1,f3也无影响；
- 3).删除原文件f1，对硬连接f2没有影响，导致符号连接f3失效；
- 4).同时删除原文件f1,硬连接f2，整个文件会真正的被删除。