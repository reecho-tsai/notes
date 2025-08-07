# 在 CentOS 7 Server 上通过 KVM + UEFI 安装 Ubuntu 24.04 虚拟机

众所周知，CentOS 7 是一款 **“久负盛名”** 的服务器操作系统——以至于已经2025年了，我司还在使用这种内核版本 **3.10.x** 的构式玩意。但是没办法，咱就一实习生，咱能怎么着？git、cmake、python 自己个编译了一个遍，调 qemu 得从编译开始调。一开始想装个 Docker 凑活用，但是发现这机器上 Docker 都是 2017 年我上初中那会的老版本，一堆东西跑不起来；LXC / LXD 更是别提。痛定思痛，看见这机器内核上至少开了 KVM，那干脆跑个虚拟机算了。

本人有洁癖，所以倾向于用 UEFI 启动虚拟机 Guest；如果你不 care 用 BIOS boot，那么本文对你价值不大，直接按照默认方式安装即可；但是第一节的管理方式可以参考。

## 安装 GUI 管理工具 Virtual Machine Manager

我本地的办公电脑是 Windows 11，已安装 WSL；利用 WSLg 特性可以直接运行 Linux 下的 GUI 程序。

```bash
sudo apt install virt-manager
```

之后添加你的服务器即可，默认设置、默认端口。

## 升级 qemu-kvm

qemu 中，要求使用 Q35 机器才可以提供 UEFI 支持。CentOS 7 yum 源的 qemu 并不支持这一功能，所以需要先卸载自带的 qemu-kvm 之后编译安装较为新版本的 qemu。我安装的是 qemu 7.2，这个版本中已经把 qemu-kvm 合入 qemu-system-x86。编译安装的部分从略，缺啥配啥吧，没办法。
