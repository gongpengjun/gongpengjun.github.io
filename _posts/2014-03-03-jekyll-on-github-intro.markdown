---
layout: post
title:  用Jekyll在GitHub上建站入门
date:   2014-03-03 07:30:14
categories: jekyll
---

Github Pages是一个运行Jekyll的静态网站Hosting服务，和GitHub无缝集成，Push到GitHub代码库的文章会自动发布。

# 工作流程

1. 编写Markdown格式的文件，上传到GitHub特定代码库中
2. Github Pages 使用Jekyll将这些Markdown文件转换为HTML页面，提供给用户访问。

<br />

# 安装步骤

## 1. 创建Github Pages个人站点：参照官方步骤 <a href="http://pages.github.com" target="_blank">Github Pages</a>
<br />

## 2. Checkout个人站点目录
{% highlight sh %}
$ git clone https://github.com/gongpengjun/gongpengjun.github.io ~/gongpengjun.github.io
{% endhighlight %}
<br />

## 3. 本地安装Jekyll：参照官方步骤 <a href="http://jekyllrb.com" target="_blank">Jekyll<a/>
{% highlight sh %}
$ sudo gem install jekyll
{% endhighlight %}
<br />

## 4. 使用Jekyll创建个人站点框架文件
{% highlight sh %}
$ cd ~/gongpengjun.github.io
$ jekyll new .
{% endhighlight %}
<br />

## 5. 上传Jekyll创建的个人站点文件
{% highlight sh %}
$ git push origin
{% endhighlight %}
<br />

## 6. 打开浏览器访问个人站点
{% highlight sh %}
$ open http://gongpengjun.github.io
{% endhighlight %}
<br />
