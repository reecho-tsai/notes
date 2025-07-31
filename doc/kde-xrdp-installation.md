# 在 KDE & X11 环境下配置 XRDP 远程桌面

## 为什么选择 xrdp

首先我的最主要诉求是「用的舒服」：画面要清晰流畅、适配 Retina Mac，能自动调节分辨率，最起码的剪贴板互动要有。搜了一圈发现 VNC 这类没有明确的说法表明是否支持，也不是很想用商业的远控软件；咨询 Copilot，给我推荐了 ```xrdp```。

一开始没想着能多么惊艳，毕竟复用的是微软的东西（微软和开源的关系懂得都懂）；但是一圈装下来发现效果非凡的好，优化一下可以媲美原生了。遂撰文以记之。

顺便说一下，很长时间没去写技术类教程 / 文章了；最近参加了 AOSCC，想把做开源当个人生爱好培养一下，于是折腾着装了 安同 OS，于是有了本文。b站、小红书上不少人说人是要有个输出型爱好的，那我就姑且在这输出输出吧，正好也锻炼一下我技术向的表达能力，省的导师再觉得我一天天啥也不干😄。

这次拿了个空 repo 就开始写了，想着是写东西重要、环境啥的都是其次。不能算是写文章，就算是随笔写个 worklog 吧；哪天有心情就美美化，等有空了搞个 GitHub Pages publish 一下。（上次想做博客还是个初中生，七八年都过去了…

## Steps

```bash
sudo apt install xrdp
sudo systemctl enable xrdp
sudo systemctl enable xrdp-sesman5
sudo echo "exec startplasma-x11" >> /etc/xrdp/startwm.sh # 最好是 vi 添加到第一行
sudo systemctl restart xrdp
sudo systemctl restart xrdp-sesman
```

之后注销登录、连接即可。注意：xrdp 不允许本地和远程同时登录同一个用户的 GUI。

由于我使用 Hyper-V + IPv6 外网访问，需要用如下的配置进行端口转发：

``` powershell
netsh interface portproxy add v6tov4 listenport=33389 listenaddress=2001:*******:2333 connectport=3389 connectaddress=172.*.*.170    # Run as Administrator
```

```listenaddr``` 是宿主机外网地址，```connectaddr``` 是虚拟机内网地址。如果是 v4 场景，参数改为 ```v4tov4``` 即可。

如果依然不通，请在 Windows 防火墙中放行 ```svchost.exe```。

## 调优

### HiDPI 支持

在 Mac RDP Client 中选择会话，进行设置：

* Color 设置为 High
* HiDPI 优化打开

之后在 KDE 中设置缩放为 200%，重启后生效。