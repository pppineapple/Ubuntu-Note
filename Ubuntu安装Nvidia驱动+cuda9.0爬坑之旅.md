# Ubuntu安装Nvidia驱动+cuda9.0+Pytorch爬坑之旅

这次主要的坑是在安装nvidia驱动，出现图形界面丢失，循环登录界面等情况。涉及的操作包括禁用nouveau驱动，禁用lightdm服务，更改bios中的Primary Display选项，更改启动默认内核等。折腾了快一个礼拜。。。

## 机器配置
```
#系统内核
maker@maker-System-Product-Name:~$ uname -a
Linux maker-System-Product-Name 4.15.0-38-generic #41~16.04.1-Ubuntu SMP Wed Oct 10 20:16:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
#cpu
maker@maker-System-Product-Name:~$ cat /proc/cpuinfo | grep model\ name
model name	: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
model name	: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
model name	: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
model name	: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
model name	: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
model name	: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
model name	: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
model name	: Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
#内存
maker@maker-System-Product-Name:~$ cat /proc/meminfo | grep MemTotal
MemTotal:       32502520 kB
#显卡
maker@maker-System-Product-Name:~$ lspci | grep 'VGA'
01:00.0 VGA compatible controller: NVIDIA Corporation GM204 [GeForce GTX 970] (rev a1)

```

## 安装Nvidia驱动

### 1.卸载原有的Nvidia驱动
在安装nvidia驱动之前首先要确保没有残留的nvidia驱动，可以使用下面的方法卸载

```
# 如果之前是使用apt-get方式安装的：
sudo apt-get remove --purge nvidia*
sudo apt-get autoremove
# 如果之前是使用.run文件安装的,假设是NVIDIA-Linux-x86_64-384.59.run：
sudo ./NVIDIA-Linux-x86_64-384.59.run --uninstall
```

### 2.禁用nouveau驱动
```
sudo gedit /etc/modprobe.d/blacklist-nouveau.conf
```
在新增的文件中写入以下内容并保存。

```
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```
在终端中执行。
```
echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf
sudo update-initramfs -u
```
然后重启电脑，在终端中执行。

```
lsmod | grep nouveau
```
如果没有信息输出，则说明nouveau已经屏蔽成功。
### 4.安装nvidia驱动
安装nvidia驱动的方法有好几种，可以apt-get安装，也可以用.run文件安装。这里选用了.run文件安装。去英伟达官网下载[nvidia-linux-390.48.run](https://www.nvidia.com/download/driverResults.aspx/132530/en-us)

```
#关闭图形界面lightdm服务
sudo /etc/init.d/lightdm stop
#赋予.run文件权限
sudo chmod 777 NVIDIA-Linux-x86_64-390.48.run
#安装驱动
sudo sh ./NVIDIA-Linux-x86_64-390.48.run –no-x-check -no-nouveau-check -no-opengl-files
# * –no-opengl-files：表示只安装驱动文件，不安装OpenGL文件。这个参数不可省略，否则会导致登陆界面死循环，英语一般称为”login loop”或者”stuck in login”。
# * –no-x-check：表示安装驱动时不检查X服务，非必需–
# * no-nouveau-check：表示安装驱动时不检查nouveau，非必需。
# * -Z, --disable-nouveau：禁用nouveau。此参数非必需，因为之前已经手动禁用了nouveau。
# 开启图形界面
sudo serice lightdm start
# 然后重启
```
#### <a name="fenced-code-block">注意：大坑来了</a>
* 重启之后循环登录界面，就是你输入密码登录后，它黑屏闪一下，然后又回到登录界面。这里的我试了很多办法，都没有什么效果，直到我根据[这篇文章](https://wizyoung.github.io/Ubuntu下GTX1080显卡驱动折腾小记/)的思路将bios中(Advanced -> System Agent -> Graphics Configuration)的iGPU Multi-Monitor由disabled更改为enabled，Primary Display由CPU Graphics更改为PCIE。然后重启，插上nvidia独显的HDMI接口，拔掉集成显卡的VGA接口，然后就解决的循环登录问题。
* 由于我试验的服务器中有多个Ubuntu内核，而且我之前还搞崩了一个内核4.4.0.75，所以需要注意重启之后可能会自动进入另外一个内核，所以这里需要重新设置内核默认启动项(不过，我这里很奇怪的是，设置好默认内核启动之后，开机会自动进入我设置的内核，但是重启reboot却会进入另外一个内核)。
* 如果出现图形界面丢失，可以尝试一下Ctrl+ALT+F1 进入控制台，输入用户名和密码进入系统，输入以下命令:

```
cd /etc/X11    
sudo cp xorg.conf.failsafe xorg.conf   
sudo reboot  
```


### 5.检查驱动是否安装完成
```
maker@maker-System-Product-Name:~$ nvidia-smi
Thu Nov  8 19:29:44 2018       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 390.48                 Driver Version: 390.48                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 970     Off  | 00000000:01:00.0  On |                  N/A |
| 30%   27C    P8    19W / 200W |    167MiB /  4042MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      2129      G   /usr/lib/xorg/Xorg                            55MiB |
|    0      4492      G   compiz                                        76MiB |
|    0      5670      G   /opt/teamviewer/tv_bin/TeamViewer             19MiB |
+-----------------------------------------------------------------------------+
```
在命令行中输入
```
nvidia-settings
```
弹出图形化设置窗口就说明驱动安装成功。

## 安装cuda9.0

安装好nvidia驱动之后，剩下的就很容易了。
先去[英伟达官网下载cuda9.0](https://developer.nvidia.com/cuda-90-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=runfilelocal)依次选择：**Linux**，**x86_64**，**Ubuntu**，**16.04**，**runfile(local)**。然后下载Base Installer就可以了。

然后进入下载的文件目录中，在终端输入命令：

```
sudo sh cuda_9.0.176_384.81_linux.run
# 可能文件没有.run后缀，但是没关系，输入文件名称就行
# 需要注意的是安装过程中会一些选项让你选择。
注意 Install NVIDIA Accelerated Graphics Driver for linux 找一项选择no，因为之前已经安装过驱动了。
```

然后要配置环境变量：在终端中执行命令

```
sudo vim ~/.bashrc(or .profile)
```

然后在文件编辑器中末尾添加如下代码：

```
export PATH=/usr/local/cuda-9.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64$LD_LIBRARY_PATH
```

然后执行下列代码查看cuda9.0是否安装成功

```
cd /usr/local/cuda-9.1d0/samples/1_Utilities/deviceQuery
sudo make
./deviceQuery
```
输入下列内容则表示cuda9.0安装成功，且环境变量也配置完成了

```
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GTX 970"
  CUDA Driver Version / Runtime Version          9.1 / 9.0
  CUDA Capability Major/Minor version number:    5.2
  Total amount of global memory:                 4043 MBytes (4238999552 bytes)
  (13) Multiprocessors, (128) CUDA Cores/MP:     1664 CUDA Cores
  GPU Max Clock rate:                            1317 MHz (1.32 GHz)
  Memory Clock rate:                             3505 Mhz
  Memory Bus Width:                              256-bit
  L2 Cache Size:                                 1835008 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(65536), 2D=(65536, 65536), 3D=(4096, 4096, 4096)
  Maximum Layered 1D Texture Size, (num) layers  1D=(16384), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(16384, 16384), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Supports Cooperative Kernel Launch:            No
  Supports MultiDevice Co-op Kernel Launch:      No
  Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 9.1, CUDA Runtime Version = 9.0, NumDevs = 1
Result = PASS
```

## 安装Pytorch
安装Pytorch非常简单，只需要进入[Pytorch官网](https://pytorch.org/get-started/locally/)，依次选择配置，然后在终端输入它告诉你的命令就可以了，非常方便。例如：我选择的是 Stable -> Linux -> Pip -> 9.0 ，然后它告诉我运行命令：```pip3 install torch torchvision```

#### 参考文献
* [Ubuntu16.04+Python2.7+CUDA9.0+cuDNN7.0+TensorFlow 1.6 安装随笔](https://zhuanlan.zhihu.com/p/34430408)
* [Ubuntu 16.04安装NVIDIA驱动](https://blog.csdn.net/CosmosHua/article/details/76644029)
* [Ubuntu下GTX1080显卡驱动折腾小记](https://wizyoung.github.io/Ubuntu下GTX1080显卡驱动折腾小记/)
