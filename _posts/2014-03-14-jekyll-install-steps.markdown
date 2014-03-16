---
layout: post
title:  "Jekyll安装记录"
date:   2014-03-14 21:52:26
categories: study
---

## 概要 ##

去年的时候在开源中国社区看了一篇文章，介绍的就是用github来写博客。苦于当时工作的琐事繁多，一直没有好好 研究哈。最近心血来潮，下定决心来折腾折腾。

第一步当然是在github上开辟一个blog仓库啦。其实这一步也有个坑，就是在创建仓库的时候，仓库名字必须是格式 username.github.io。并且username必须和你的github username一致。
![有帮助的截图]({{ site.url }}/img/create-github.io.png)

## 安装步骤 ##

- 安装 Ruby
- 安装 DevKit
- 安装 Jekyll
- 安装 Python
- 安装 Install
- 安装 Pygments
- 开始 Jekyll

### 安装 Ruby ###

1. 到这个站点下载Ruby [http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)。
2. 在"RubyInstallers"选项里面, 选择一个和你系统相应的版本下载。 例如，我的机器装的是win7 64位系统，那么就选择Ruby 2.0.0-p451 (x64)。
3. 通过安装向导（rubyinstaller-2.0.0-p451-x64.exe）安装。注意：

	- 尽可能的保持默认路径（C:\Ruby200-x64）安装
	- 勾选上添加到PATH选项
![有帮助的截图]({{ site.url }}/img/ruby-installer.png)
4. 接下来，一路下一步，没有什么要注意的啦。安装完毕后，打开命令提示符窗口，输入
{% highlight ruby %}
ruby --version
{% endhighlight %}
测试ruby安装是否成功。如果可以看到版本号，就说明安装成功。


### 安装DevKit ###

尽管有同学提到，最好在git bash下安装，据说在DOS命令下错误百出。但是我在DOS下安装还是比较顺利的。

1. 再次到这个网站[http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)。
2. 下载DEVELOPMENT KIT安装包，一定要注意和windows操作系统的版本还有Ruby的版本匹配。比如我下载的就是DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe
3. 运行安装包，把它解压到(`C:\DevKit`)
4. 初始化并创建config.yml文件，在命令提示符窗口里输入
{% highlight ruby %}
cd C:\DevKit
ruby dk.rb init
notepad config.yml
{% endhighlight %}
5. 在打开的文件的末尾，添加一行(`- C:\Ruby200-x64`)，然后关闭并保存。
6. 回到命令提示符窗口，继续输入
{% highlight ruby %}
ruby dk.rb review
ruby dk.rb install
{% endhighlight %}
然后等待安装完成。

### 安装 Jekyll ###
1. 确认gem安装完毕
{% highlight ruby %}
gem --version
{% endhighlight %}
2. 安装Jekyll gem，这里要注意下啦，到目前为止（发布博客），最新的版本是有问题的，我们这里安装的时候，选择一下版本
{% highlight ruby %}
gem install jekyll --version "=1.4.2"
{% endhighlight %}
然后就漫长的等待。
3. 安装结束后，可以通过这个命令查看jekyll的版本(`jekyll -v`)

### 安装 Python ###
1. 到这里[www.python.org/download](http://www.python.org/download)，下载python2.7版本。
2. 下载完毕后，安装到C盘根目录。
3. 设置环境变量，把C:\Python27添加到PATH中。
4. 在命令提示符窗口输入(`python --version`)，查看python的版本，验证是否安装成功。

### 安装 Easy Install ###
1. win7的操作系统，可以直接下载[ez_setup.py](https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py)。下载保存到C盘（`C:\`），然后运行，打开命令提示符，输入(`- python C:\ez_setup.py`)。
2. 把Python Scripts的目录(C:\Python27\Scripts)添加到PATH中。

### 安装 Pygments ###
1. 检查easy_install是否安装成功：(`easy_install --version`)
2. 使用"easy_install"安装Pygments：(`easy_install Pygments`)
3. 安装完毕后，这里又有点小调整。我们需要看一下这个：(`gem uninstall pygments.rb`)；不要惊讶，我只是要确认一下，当前的pygments是否和我们的Jekyll版本一致，我在安装的时候，出错了很多次，后来 才找到原来是pygments和jekyll版本不相容。我把当前的给卸载了，安装0.5.0 (`gem install pygments.rb --version "=0.5.0"`)


### 开始 Jekyll ###
三行命令，简单搞起

	jekyll new my-awesome-site
	cd my-awesome-site
	jekyll serve

### 致谢 ###
我的安装过程，真的是坎坎坷坷，所以才决定把流程整理一遍记下来。在此过程中，参考了许多同行的经验，看了他们的blog，主要有

- http://pages.github.com/
- http://jekyllcn.com/
- http://www.cnblogs.com/BeginMan/p/3549241.html
- http://yizeng.me/2013/05/10/setup-jekyll-on-windows/#install-pygements

好吧，到此结束。