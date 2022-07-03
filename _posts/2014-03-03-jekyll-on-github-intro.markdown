---
layout: post
title:  用Jekyll在GitHub上建站入门
date:   2014-03-03 07:30:14
categories: Jekyll
---

Github Pages是一个运行Jekyll的静态网站Hosting服务，和GitHub无缝集成，Push到GitHub代码库的文章会自动发布。

## 1、工作原理

1. 编写Markdown格式的文件，上传到GitHub特定代码库中
2. Github Pages 使用Jekyll将这些Markdown文件转换为HTML页面，提供给用户访问。

## 2、初始化步骤

### 2.1 创建Github Pages个人站点

参照官方步骤[Github Pages](http://pages.github.com)

### 2.2 Checkout个人站点目录

```sh
git clone https://github.com/gongpengjun/gongpengjun.github.io ~/gongpengjun.github.io
```

### 2.3 本地安装Jekyll

参照官方步骤 <a href="https://jekyllrb.com/" target="_blank">Jekyll<a/>

```sh
sudo gem install jekyll
```

### 2.4 使用Jekyll创建个人站点框架文件

```sh
cd ~/gongpengjun.github.io
jekyll new .
```

### 2.5 上传Jekyll创建的个人站点文件

```sh
git push origin
```

### 2.6 打开浏览器访问个人站点

[https://gongpengjun.github.io](https://gongpengjun.github.io)

或

[https://gongpengjun.com](https://gongpengjun.com)

## 3、自定义

### 3.1 配置页面宽度

修改屏幕媒体的`.site`样式中的`width`为百分比，比如75%。[commit](https://github.com/gongpengjun/gongpengjun.github.io/commit/6d0f31c21ad087317a6151c73bce76c28feeea2b?diff=split)

```css
@media only screen and (min-width: 481px) {
  .site {
    font-size: 115%;
    margin: 1em auto;
    width: 42em;
    padding: 1em;
  }
}
```
改为：
```css
@media only screen and (min-width: 481px) {
  .site {
    font-size: 115%;
    margin: 1em auto;
    width: 75%;
    padding: 1em;
  }
}
```

## 4、本地测试

参考：[使用 Jekyll 在本地测试 GitHub Pages 站点](https://docs.github.com/cn/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)

```sh
git clone git@github.com:gongpengjun/gongpengjun.github.io.git
cd gongpengjun.github.io
bundle exec jekyll serve
```

本地访问：[http://localhost:4000](http://localhost:4000)


## 5、常见问题

### 5.1、 date导致post不显示

2022年7月03日晚上23点40分，将一篇博文时间设置下面则不显示

```
date: 2022-07-03 23:00:00
```

改为下面时间则正常显示：


```
date: 2022-07-03 09:00:00
```


