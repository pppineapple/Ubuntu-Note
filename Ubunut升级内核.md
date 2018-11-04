# Ubunut升级内核

## 1.查看系统及内核版本
```
# 查看发布版本号
cat /etc/issue
lsb_release -a

# 查看内核版本号
uname -sr
uname -a
```

## 2.升级内核
* 到Ubuntu官网：<http://kernel.ubuntu.com/~kernel-ppa/mainline/>

* 进入所需要的的ubuntu内核版本目录。例如：**v4.19**(发布于2018年10月22日)

* 根据硬件架构选择合适的内核版本。例如：**X86**硬件架构**64位操作系统**应选择**AMD64**

* 下载3个内核**.deb**安装文件

```
* linux-headers-4.19.0-041900_4.19.0-041900.201810221809_all.deb
* linux-headers-4.19.0-041900-generic_4.19.0-041900.201810221809_amd64.deb
* linux-image-unsigned-4.19.0-041900-generic_4.19.0-041900.201810221809_amd64.deb
```

* 安装内核文件:
```
sudo dpkg -i *.deb
```

* 安装完成之后，重启系统，验证内核版本:
```
uname -sr
```


