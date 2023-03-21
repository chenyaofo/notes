
## 如何压缩和解压TAR压缩文件？

linux 下解压任何 tar 压缩包，都是这样就行：
```
tar xf xxx.tar.bz2
```

xf 不用写成 -xf ，更不用加更多字母

压缩命令则是：

```
tar caf xxx.tar.xz xxx/
```

a 表示根据文件名，自动选择对应压缩算法

参考：[Linux 下用 tar 和 zip 压缩文件夹有啥区别](https://www.v2ex.com/t/909851)

---

## Find命令使用

将当前目录及其子目录下所有文件后缀为 .c 的文件列出来:

```
find . -name "*.c"
```

将当前目录及其子目录中的所有文件列出：

```
find . -type f
```

将当前目录及其子目录下所有最近 20 天内更新过的文件列出:

```
find . -ctime  20
```

查找 /var/log 目录中更改时间在 7 日以前的普通文件，并在删除之前询问它们：

```
find /var/log -type f -mtime +7 -ok rm {} \;
```

查找当前目录中文件属主具有读、写权限，并且文件所属组的用户和其他用户具有读权限的文件：

```
find . -type f -perm 644 -exec ls -l {} \;
```

查找系统中所有文件长度为 0 的普通文件，并列出它们的完整路径：

```
find / -type f -size 0 -exec ls -l {} \;
```

再看看xargs命令是如何同find命令一起使用的，并给出一些例子。

查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件：

```
find . -type f -print | xargs file 
```

在整个系统中查找内存信息转储文件(core dump) ，然后把结果保存到/tmp/core.log 文件中：

```
find / -name "core" -print | xargs echo "" >/tmp/core.log
```

用grep命令在所有的普通文件中搜索hostname这个词：

```
find . -type f -print | xargs grep "hostname" 
```

删除3天以前的所有东西：

```
find ./ -mtime +3 -print|xargs rm -f –r 
```

删除文件大小为零的文件：

```
find ./ -size 0 | xargs rm -f & 
```

find命令配合使用exec和xargs可以使用户对所匹配到的文件执行几乎所有的命令。

参考：[【日常小记】linux中强大且常用命令：find](https://www.cnblogs.com/skynet/archive/2010/12/25/1916873.html)