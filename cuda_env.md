## 如何安装CUDA via conda？

有时候想配置一个环境隔离的cuda toolkit，用来编译对应的程序，可以通过下述方法：

[官方教程](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#conda-overview)，[Conda详情页](https://anaconda.org/nvidia/cuda)

这样安装的cuda是完整的，带nvcc的。

```
conda install -y cuda -c nvidia/label/cuda-11.3.0
conda install -y cuda -c nvidia/label/cuda-11.3.1
conda install -y cuda -c nvidia/label/cuda-11.4.0
conda install -y cuda -c nvidia/label/cuda-11.4.1
conda install -y cuda -c nvidia/label/cuda-11.4.2
conda install -y cuda -c nvidia/label/cuda-11.4.3
conda install -y cuda -c nvidia/label/cuda-11.4.4
conda install -y cuda -c nvidia/label/cuda-11.5.0
conda install -y cuda -c nvidia/label/cuda-11.5.1
conda install -y cuda -c nvidia/label/cuda-11.5.2
conda install -y cuda -c nvidia/label/cuda-11.6.0
conda install -y cuda -c nvidia/label/cuda-11.6.1
conda install -y cuda -c nvidia/label/cuda-11.6.2
conda install -y cuda -c nvidia/label/cuda-11.7.0
conda install -y cuda -c nvidia/label/cuda-11.7.1
conda install -y cuda -c nvidia/label/cuda-11.8.0
conda install -y cuda -c nvidia/label/cuda-12.0.0
conda install -y cuda -c nvidia/label/cuda-12.0.1
conda install -y cuda -c nvidia/label/cuda-12.1.0
conda install -y cuda -c nvidia/label/cuda-12.1.1
conda install -y cuda -c nvidia/label/cuda-12.2.0
conda install -y cuda -c nvidia/label/cuda-12.2.1
conda install -y cuda -c nvidia/label/cuda-12.2.2
conda install -y cuda -c nvidia/label/cuda-12.3.0
conda install -y cuda -c nvidia/label/cuda-12.3.1
conda install -y cuda -c nvidia/label/cuda-12.3.2
conda install -y cuda -c nvidia/label/cuda-12.4.0
conda install -y cuda -c nvidia/label/cuda-12.4.1
conda install -y cuda -c nvidia/label/cuda-12.5.0
conda install -y cuda -c nvidia/label/cuda-12.5.1
conda install -y cuda -c nvidia/label/cuda-12.6.0
conda install -y cuda -c nvidia/label/cuda-12.6.1
conda install -y cuda -c nvidia/label/cuda-12.6.2
conda install -y cuda -c nvidia/label/cuda-12.6.3
conda install -y cuda -c nvidia/label/cuda-12.8.0
conda install -y cuda -c nvidia/label/cuda-12.8.1
```

## 如何安装GCC和G++？


```
conda install gcc=13 gxx=13 -c conda-forge
```

## 如何安装FlashAttention？

通过编译安装（编译可能需要数个小时）：

```
pip install packaging
pip install ninja
pip install flash-attn --no-build-isolation
```

还有一种方式是通过prebuilt二进制包安装：

在 https://github.com/Dao-AILab/flash-attention/releases 找自己合适的版本：

确认自己的torch版本：

```
python -c "import torch; print(torch.__version__)"
```

确认自己的torch的CXX11 ABI：

```
python -c "import torch; print(torch._C._GLIBCXX_USE_CXX11_ABI)"
```

然后确定自己的python版本和cuda版本，根据这些信息在release中挑选合适的whl包，复制链接然后安装，比如：

```
pip install https://github.com/Dao-AILab/flash-attention/releases/download/v2.7.4.post1/flash_attn-2.7.4.post1+cu12torch2.2cxx11abiFALSE-cp310-cp310-linux_x86_64.whl
```


