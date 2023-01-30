---
title: 在Linux服务器上安装VSCode Server，实现免VPN访问局域网服务器
tags: TeXt
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

<img src="/assets/pic1.png" >

If you see this page, that means you have setup your site. enjoy! :ghost: :ghost: :ghost:

You may want to [config the site](https://tianqi.name/jekyll-TeXt-theme/docs/en/configuration) or [writing a post](https://tianqi.name/jekyll-TeXt-theme/docs/en/writing-posts) next. Please feel free to [create an issue](https://github.com/kitian616/jekyll-TeXt-theme/issues) or [send me email](mailto:kitian616@outlook.com) if you have any questions.

<!--more-->

---

If you like TeXt, don't forget to give me a star. :star2:

[![Star This Project](https://img.shields.io/github/stars/kitian616/jekyll-TeXt-theme.svg?label=Stars&style=social)](https://github.com/kitian616/jekyll-TeXt-theme/)
