---
title: "My Hugo Cheatsheet"
date: 2023-11-05T16:09:58+08:00
draft: false
---

## 安装 Hugo：

[Hugo - Install](https://gohugo.io/installation/)

```bash
go install -tags extended github.com/gohugoio/hugo@latest
```

Mac 也可以使用Brew，省去配置
```bash
brew install hugo
```

## 使用 Hugo
```bash
hugo new posts/your-post.md

hugo serve
```

## Hugo 图片

存放在 content/images 下。markdown中引用时：```![](../../images/xxx.jpg)```

## Hugo 内链

```markdown
[Hugo cheatsheet](/posts/my-hugo-cheatsheet/)
```

效果：[Hugo cheatsheet](/posts/my-hugo-cheatsheet/)


## wsl无法预览问题解决

来源：[Toko's blog](https://bestoko.cc/p/hugo_wsl2/)

```bash
hugo server --bind 172.20.3.63 --baseURL=http://172.20.3.63
```

ip地址可以通过 [golang-wsl](/posts/golang-wsl2-centos8-setup/#配置clash或其他代理软件) 中的 ```proxy ip``` 命令获取