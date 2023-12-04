
## Python 库打包分发指南

 - `setuptools` vs. `distutils` vs. `distribute` vs. `distutils2`

> `setuptools`是目前python打包分发的事实标准了，另外几个库：`distutils`功能太少，`distribute`已经合并回`setuptools`了，`distutils2`已废弃。

 - 包格式 `Wheel` vs. `Egg`

> Egg 和 Wheel 本质上都是一个 zip 格式包，Egg 文件使用 .egg 扩展名，Wheel 使用 .whl 扩展名。Wheel 的出现是为了替代 Egg，其现在被认为是 Python 的二进制包的标准格式。


 - `setup.py` 文件有很多内置命令可供使用，查看所有支持的命令`python setup.py –help-commands`，此处列举一些常用命令：

   - `build`: 构建安装时所需的所有内容

   - `build_ext`:构建扩展，如用 C/C++, Cython 等编写的扩展，在调试时通常加 --inplace 参数，表示原地编译，即生成的扩展与源文件在同样的位置。

   - `sdist`:构建源码分发包，在 Windows 下为 zip 格式，Linux 下为 tag.gz 格式 。

   - `bdist`:构建一个二进制的分发包。

   - `bdist_egg`:构建一个 egg 分发包，经常用来替代基于 bdist 生成的模式

   - `bdist_wheel`:构建一个 wheel 分发包，egg 包是过时的，whl 包是新的标准

   - `install`:安装包到系统环境中。

   - `develop`:以开发方式安装包，该命名不会真正的安装包，而是在系统环境中创建一个软链接指向包实际所在目录。这边在修改包之后不用再安装就能生效，便于调试。

   - `register、upload`:用于包的上传发布，不过用 upload 命令上传包已经过时，可以使用`twine`工具上传至pypi `python setup.py sdist bdist_wheel && twine upload dist/*`

从源码安装或者打包的新姿势：
```
python setup.py bdist_wheel -> pip wheel .
python setup.py install -> pip install .
python setup.py develop -> pip install -e .
```

在上述新姿势中，`pip`将作为前端与用户交互，隐藏了后端的细节。

参考：[Python 库打包分发(setup.py 编写)简易指南](http://kuanghy.github.io/2018/04/29/setup-dot-py); [Python打包指南2021](https://frostming.com/2020/12-25/python-packaging/); [Python 打包的新动态2023](https://frostming.com/2023/python-packaging-status/); [PyCon China 2021 演讲——Python 打包 101](https://frostming.com/2021/10-20/pycon-china-2021/)

### 安装包`natten-0.14.4+torch1130cpu-cp37-cp37m-linux_x86_64.whl`中的各个部分意义是指什么？

每个部分的意义如下`{distribution}-{version}(-{build tag})?-{python tag}-{abitag}-{platform tag}.whl`

- `natten`表示包的名称
- `0.14.4+torch1130cpu`表示包的版本号
- `cp37`是`{python tag}`，表示python的实现和该包适配的python版本，主流的实现如下：`py`: `Generic Python` (does not require implementation-specific features)，`cp`: `CPython`，`ip`: `IronPython`，`pp`: `PyPy`，`jy`: `Jython`，可使用代码`sys.implementation.name`查看python的实现类型
- `cp37m`是`{abitag}`，其中的`m`表示额外的标记，详见[PEP 3149 – ABI version tagged .so files](https://peps.python.org/pep-3149/)，其中`--with-pydebug`（flag：`d`), `--with-pymalloc`（flag：`m`), `--with-wide-unicode`（flag：`u`)
- `linux_x86_64`表示支持的平台，可用代码`distutils.util.get_platform()`查看所在平台

参考：1) [PEP 425 – Compatibility Tags for Built Distributions](https://peps.python.org/pep-0425/); 2) [Platform Compatibility Tags](https://packaging.python.org/en/latest/specifications/platform-compatibility-tags/#platform-compatibility-tags); 3) [PyPA Specifications](https://packaging.python.org/en/latest/specifications/)

---

### 安装包中的`manylinux`是什么意思？

Python的预编译二进制包致力于能够一次编译，尽可能运行在更多的Linux发行版上。但是不同Linux发行版的库和版本有着较大的区别，为此，`manylinux`项目约束了预编译二进制包必须满足两个条件：1）仅能依赖一些十分底层的共享库，详情见PEP 513（manylinux1），PEP 571（manylinux2010），PEP 599（manylinux2014）和PEP 600（manylinux_x_y）；2）仅能构建于某个版本的GLIBC库，因为GLIBC库向后兼容性，这样在更高版本的GLIBC库上也能运行。

最新的标记如`manylinux_2_17_x86_64`直接将GLIBC的版本写上了，其表示`manylinux_${GLIBCMAJOR}_${GLIBCMINOR}_${ARCH}`，这个标记的二进制包在GLIBC的2.17及以上版本应该可以运行。

参考：[GitHub - pypa/manylinux](https://github.com/pypa/manylinux)

---

### Python预编译二进制包如何支持`AVX`指令？

以`numpy`库为例，其在安装是指定新的编译参数可以执行并行化CPU指定`python setup.py build --cpu-baseline="avx2 fma3" install`，在numpy的默认打包过程中，`--cpu-baseline="min"`是默认值，其仅仅考虑了非常基础的CPU并行指令集。此外，可以通过`--cpu-dispatch`选项使得其可以在运行时根据CPU指令集的可用性，动态选择不同指令集的函数实现，详见[CPU build options in numpy](https://numpy.org/devdocs/reference/simd/build-options.html)。这样就可以在一个二进制库编译时支持各种指令集命令，在运行时根据CPU能力运行最快的指令集函数。这里[Issue](https://github.com/tensorflow/tensorflow/issues/12541)中提到OpenCV库也支持这一特性。

---

### 安装包时如何指定多个`index-url`？

可以通过选项`--extra-index-url`指定额外的index-url，见下述帮助文档：

```
   -i,--index-url <url>
          Base URL of Python Package Index (default https://pypi.python.org/simple/).

   --extra-index-url <url>
          Extra URLs of package indexes to use in addition to --index-url.
```

也可以修改`pip.conf`做到这一点：

```
[global]
index-url = http://download.zope.org/simple
trusted-host = download.zope.org
               pypi.org
               secondary.extra.host
extra-index-url= https://pypi.org/simple
                 http://secondary.extra.host/simple
```

参考：[Can pip.conf specify two index-url at the same time?](https://stackoverflow.com/questions/30889494/can-pip-conf-specify-two-index-url-at-the-same-time)

---

### `--find-links`和`--index-url`的区别

先看官方文档：

> `-i, --index-url <url>`

> Base URL of Python Package Index (default https://pypi.python.org/simple). This should point to a repository compliant with PEP 503 (the simple repository API) or a local directory laid out in the same format.

> `-f, --find-links <url>`

> If a url or path to an html file, then parse for links to archives. If a local path or file:// url that's a directory, then look for archives in the directory listing.

这个`--find-links`的html文件每行包含一个超链接（样例如下），记录了包的名称和位置，可以用[脚本](https://github.com/SHI-Labs/NATTEN/blob/ad39be7a8ef00ed2c4c225fd30dedd1fe4279420/dev/packaging/gen_wheel_index.sh)轻松生成。

```

<a href="natten-0.14.4%2Btorch1130cpu-cp39-cp39-linux_x86_64.whl">natten-0.14.4+torch1130cpu-cp39-cp39-linux_x86_64.whl</a><br>
<a href="natten-0.14.4%2Btorch1130cpu-cp38-cp38-linux_x86_64.whl">natten-0.14.4+torch1130cpu-cp38-cp38-linux_x86_64.whl</a><br>
<a href="natten-0.14.4%2Btorch1130cpu-cp37-cp37m-linux_x86_64.whl">natten-0.14.4+torch1130cpu-cp37-cp37m-linux_x86_64.whl</a><br>
<a href="natten-0.14.4%2Btorch1130cpu-cp311-cp311-linux_x86_64.whl">natten-0.14.4+torch1130cpu-cp311-cp311-linux_x86_64.whl</a><br>
<a href="natten-0.14.4%2Btorch1130cpu-cp310-cp310-linux_x86_64.whl">natten-0.14.4+torch1130cpu-cp310-cp310-linux_x86_64.whl</a><br>
```

参考：[What's the difference between --find-links and --index-url pip flags?](https://stackoverflow.com/questions/46651454/whats-the-difference-between-find-links-and-index-url-pip-flags)

---

### pip安装包的4种来源

 - Installing from PyPI

```
python3 -m pip install "SomeProject"

python3 -m pip install "SomeProject==1.4"
```

 - Installing from VCS

```
python3 -m pip install -e SomeProject @ git+https://git.repo/some_pkg.git          # from git
python3 -m pip install -e SomeProject @ hg+https://hg.repo/some_pkg                # from mercurial
python3 -m pip install -e SomeProject @ svn+svn://svn.repo/some_pkg/trunk/         # from svn
python3 -m pip install -e SomeProject @ git+https://git.repo/some_pkg.git@feature  # from a branch
python3 -m pip install -e "git+https://git.repo/some_pkg.git@feature#subdirectory=src" # from a branch and subfolder
```

- Installing from a local src tree

```
python3 -m pip install -e <path>
```

- Installing from remote/local archives

```
python3 -m pip install ./downloads/SomeProject-1.0.4.tar.gz
python3 -m pip install https://repo.com/SomeProject-1.0.4.tar.gz
```

参考：[Pip install examples](https://pip.pypa.io/en/latest/cli/pip_install/#pip-install-examples); [Install packages](https://packaging.python.org/en/latest/tutorials/installing-packages/#installing-packages)

---

### `requirements.txt`文件格式规范

```
# This is a comment, to show how #-prefixed lines are ignored.
# It is possible to specify requirements as plain names.
pytest
pytest-cov
beautifulsoup4

# The syntax supported here is the same as that of requirement specifiers.
docopt == 0.6.1
requests [security] >= 2.8.1, == 2.8.* ; python_version < "2.7"
urllib3 @ https://github.com/urllib3/urllib3/archive/refs/tags/1.26.8.zip

# It is possible to refer to other requirement files or constraints files.
-r other-requirements.txt
-c constraints.txt

# It is possible to refer to specific local distribution paths.
./downloads/numpy-1.9.2-cp34-none-win32.whl

# It is possible to refer to URLs.
http://wxpython.org/Phoenix/snapshot-builds/wxPython_Phoenix-3.0.3.dev1820+49a8884-cp34-none-win_amd64.whl

# use --find-links
-f https://download.pytorch.org/whl/cu117
torch==1.13.0
-f https://download.pytorch.org/whl/cu117
torchvision==0.14.0
```

参考：[requirements-file-format](https://pip.pypa.io/en/latest/reference/requirements-file-format/#requirements-file-format)

---

### 如何构建manylinux的python包？

先正常构建wheel，然后使用`auditwheel`工具将二进制包转换为manylinux形式。`auditwheel`会将外部共享库复制到wheel里面， 并自动修改共享库的相关内容。构建例子可以参考[这里](https://github.com/pypa/python-manylinux-demo/blob/master/travis/build-wheels.sh)。

参考：[Github - auditwheel](https://github.com/pypa/auditwheel)

### 如何使用conda/mamba安装CUDA和GCC？

 - 安装CUDA

检查可用版本，请到[官方页面](https://anaconda.org/nvidia/cuda)检查所有可用的版本，[nvidia帮助文档](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#conda-installation)提供了更多的信息

```
mamba install -c "nvidia/label/cuda-11.8.0" cuda
```

这样安装得到的是完整的CUDA，和从官网下载得到的是一样的，安装好激活环境之后会自动设置`CUDA_HOME`和`CUDA_PATH`

 - 安装GCC

检查可用版本：

```
mamba search gcc
```

```
# Name                       Version           Build  Channel
gcc                            8.5.0      h143be6b_1  conda-forge
gcc                            8.5.0     h143be6b_10  conda-forge
gcc                            8.5.0      h143be6b_2  conda-forge
gcc                            8.5.0      h143be6b_3  conda-forge
gcc                            8.5.0      h143be6b_4  conda-forge
gcc                            8.5.0      h143be6b_5  conda-forge
gcc                            8.5.0      h143be6b_6  conda-forge
gcc                            8.5.0      h143be6b_7  conda-forge
gcc                            8.5.0      h143be6b_8  conda-forge
gcc                            8.5.0      h143be6b_9  conda-forge
gcc                            9.4.0      h192d537_1  conda-forge
gcc                            9.4.0     h192d537_10  conda-forge
gcc                            9.4.0      h192d537_2  conda-forge
gcc                            9.4.0      h192d537_3  conda-forge
gcc                            9.4.0      h192d537_4  conda-forge
gcc                            9.4.0      h192d537_5  conda-forge
gcc                            9.4.0      h192d537_6  conda-forge
gcc                            9.4.0      h192d537_7  conda-forge
gcc                            9.4.0      h192d537_8  conda-forge
gcc                            9.4.0      h192d537_9  conda-forge
gcc                            9.5.0     h1fea6ba_10  conda-forge
gcc                            9.5.0     h1fea6ba_11  conda-forge
gcc                            9.5.0     h1fea6ba_12  conda-forge
gcc                            9.5.0     h1fea6ba_13  conda-forge
gcc                           10.3.0      he2824d0_1  conda-forge
gcc                           10.3.0     he2824d0_10  conda-forge
gcc                           10.3.0      he2824d0_2  conda-forge
gcc                           10.3.0      he2824d0_3  conda-forge
gcc                           10.3.0      he2824d0_4  conda-forge
gcc                           10.3.0      he2824d0_5  conda-forge
gcc                           10.3.0      he2824d0_6  conda-forge
gcc                           10.3.0      he2824d0_7  conda-forge
gcc                           10.3.0      he2824d0_8  conda-forge
gcc                           10.3.0      he2824d0_9  conda-forge
gcc                           10.4.0     hb92f740_10  conda-forge
gcc                           10.4.0     hb92f740_11  conda-forge
gcc                           10.4.0     hb92f740_12  conda-forge
gcc                           10.4.0     hb92f740_13  conda-forge
gcc                           11.1.0      hee54495_1  conda-forge
gcc                           11.2.0      h702ea55_1  conda-forge
gcc                           11.2.0     h702ea55_10  conda-forge
gcc                           11.2.0      h702ea55_2  conda-forge
gcc                           11.2.0      h702ea55_3  conda-forge
gcc                           11.2.0      h702ea55_4  conda-forge
gcc                           11.2.0      h702ea55_5  conda-forge
gcc                           11.2.0      h702ea55_6  conda-forge
gcc                           11.2.0      h702ea55_7  conda-forge
gcc                           11.2.0      h702ea55_8  conda-forge
gcc                           11.2.0      h702ea55_9  conda-forge
gcc                           11.3.0     h02d0930_10  conda-forge
gcc                           11.3.0     h02d0930_11  conda-forge
gcc                           11.3.0     h02d0930_12  conda-forge
gcc                           11.3.0     h02d0930_13  conda-forge
gcc                           12.1.0     h9ea6d83_10  conda-forge
gcc                           12.2.0     h26027b1_11  conda-forge
gcc                           12.2.0     h26027b1_12  conda-forge
gcc                           12.2.0     h26027b1_13  conda-forge
```

选择一个合适的版本进行安装：

```
mamba install gcc=11.2.0 gxx=11.2.0 make cmake
```

安装完成之后激活环境，相关的环境变量也会设置好
