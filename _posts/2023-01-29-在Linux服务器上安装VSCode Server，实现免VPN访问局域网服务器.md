---
title: 在Linux服务器上安装VSCode Server，实现免VPN访问局域网服务器
tags: Linux
pageview: true
---


## 背景
作者由于工作学习需要，需要经常在校外通过 VPN 连接到学校内网，然后再通过 SSH 或者 VSCode 连接到实验室的 Linux 服务器上写代码。代码写和跑都在 Linux 服务器上。我觉得连 VPN 有点麻烦，所以尝试用 VSCode Server 省掉这一步，虽然最后成功了，但考虑到最后的体验，还是把此方法作为了备用。 


### 优点
- 不需要 VPN。
- 可以随时随地写代码，也支持网页（iPad）。


### 缺点
- 打开代码比较慢，输入命令也比较卡。原因是 Linux 服务器和写代码的客户端都需要和 VSCode 的服务器通信，我感觉和 VPN + SSH 体验差距比较大，也是我放弃的原因。


## 步骤
下载 CLI 客户端到服务器上，解压之后就是一个文件名为 `code` 的可执行文件，然后运行 `./code tunnel`


![0](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230130/0.png)


直接回车接受，它会显示一个 GitHub 链接和一个八位的字符串，在网页中打开链接输入这个字符串，然后这个服务器便会和你的账户绑定。
![1](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230130/1.png)
![2](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230130/2.png)
![3](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230130/3.png)

然后给当前服务器起一个名字，长度不能超过20。
![4](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230130/4.png)

在自己电脑上的 VSCode 中安装这个 Remote-Tunnels 插件，便可以看到服务器了，或者打开它提供的链接直接在浏览器上写代码。。另外作者把 VSCode Server 跑在了 tmux 里，这样就可以一直开着了。
![5](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230130/5.png)
![6](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230130/6.png)
![7](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230130/7.png)



<!--more-->

---

<!-- If you like TeXt, don't forget to give me a star. :star2:

[![Star This Project](https://img.shields.io/github/stars/kitian616/jekyll-TeXt-theme.svg?label=Stars&style=social)](https://github.com/kitian616/jekyll-TeXt-theme/) -->
