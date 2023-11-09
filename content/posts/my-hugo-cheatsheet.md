---
title: "My Hugo Cheatsheet"
date: 2023-11-05T16:09:58+08:00
draft: false
---

安装 Hugo：

[Hugo - Install](https://gohugo.io/installation/)

```bash
go install -tags extended github.com/gohugoio/hugo@latest
```

Mac 也可以使用Brew，省去配置
```bash
brew install hugo
```

使用 Hugo
```bash
hugo new posts/your-post.md

hugo serve
```

Hugo 图片

存放在 content/images 下。markdown中引用时：```![](../../images/xxx.jpg)```