# 在 CentOS 7 Server 上通过 KVM + UEFI 安装 Ubuntu 24.04 虚拟机

众所周知，CentOS 7 是一款 **“久负盛名”** 的服务器操作系统——以至于已经2025年了，我司还在使用这种内核版本 **3.10.x** 的构式玩意。但是没办法，咱就一实习生，咱能怎么着？git、cmake、python 自己个编译了一个遍，调 qemu 得从编译开始调。一开始想装个 Docker 凑活用，但是发现这机器上 Docker 都是 2017 年我上初中那会的老版本，一堆东西跑不起来；LXC / LXD 更是别提。痛定思痛，看见这机器内核上至少开了 KVM，那干脆跑个虚拟机算了。公司机器不好外传东西，此文只记录大概思路，不保证 100% 复现成功。

本人有洁癖，所以倾向于用 UEFI 启动虚拟机 Guest；如果你不 care 用 BIOS boot，那么本文对你价值不大，直接按照默认方式安装即可；但是第一节的管理方式可以参考。

## 安装 GUI 管理工具 Virtual Machine Manager

我本地的办公电脑是 Windows 11，已安装 WSL；利用 WSLg 特性可以直接运行 Linux 下的 GUI 程序。

```bash
sudo apt install virt-manager
```

之后添加你的服务器即可，默认设置、默认端口。要求服务器上安装好了 ```libvirt```。

## 升级 qemu-kvm

qemu 中，要求使用 Q35 机器才可以提供 UEFI 支持。CentOS 7 yum 源的 qemu 并不支持这一功能，所以需要先卸载自带的 qemu-kvm 之后编译安装较为新版本的 qemu。我安装的是 qemu 7.2，这个版本中已经把 qemu-kvm 合入 qemu-system-x86。编译安装的部分从略，缺啥配啥吧，没办法。

这里先看一下系统的 ```qemu-kvm``` 配置在什么位置；之后```sudo semanage fcontext -l```并记录下现在的 SELinux 上下文配置、卸载自带的 qemu，```sudo make install```、并把新的 qemu 做软链接到原位置。

## 安装 UEFI 固件

对于 UEFI 这种西洋景、摩登物，CentOS 的源百分之一万是没有的。实在是不想编译了，在互联网的角落找到了一个可以安装的 RPM：https://oraclelinux.pkgs.org/8/ol8-appstream-x86_64/edk2-ovmf-20220126gitbb1bba3d77-13.el8_10.noarch.rpm.html 。下载安装之。

## 配置 SELinux

到这里，你可能天真的以为这就算大功告成了；但是一旦在 virt-manager 里尝试运行虚拟机，便是各种 ```No such file or directory``` 的报错。询问 GPT，得知是 SELinux 在作祟——```sudo setenforce 0```之后，这些情况都不再发生。按照之前 SELinux 中对 qemu Binary、UEFI 固件等的设置配置好上下文即可，参考：

```bash
sudo semanage fcontext -a -t qemu_exec_t "/usr/local/qemu/bin/qemu-system-x86_64" # 链接最好也设置一下
sudo restorecon -Fv /usr/local/qemu/bin/qemu-system-x86_64
sudo semanage fcontext -a -t svirt_image_t "/usr/share/seabios/kvmvapic.bin"
sudo restorecon -v /usr/share/seabios/kvmvapic.bin
sudo systemctl restart libvirtd
```
Done.
