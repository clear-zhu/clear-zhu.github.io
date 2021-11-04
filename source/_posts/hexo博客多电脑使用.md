title: hexo博客多终端
author: carl-zhu
tags: []
categories: []
date: 2021-11-04 17:20:00
---
### 目标
有时候使用写博客记录内容的时候，自己常用的电脑并不在身边，重新在一台新的电脑上下载Node、hexo、git等等，还有各种路径，配置，还要新建仓库（当时的想法是，新建的博客都是会覆盖老的）。说实话，想想还是算了吧，干嘛瞎折腾。所以，经过不懈的努力终于配置好了多终端写Hexo（没啥大用），让自己想记录的不要因为外界环境所阻碍。

#### 新建分支
在GitHub中需要新建一个分支，新建分支方式，如下所示。

![upload successful](/images/pasted-1.png)
新建完分支后，我们需要做的是将新的分支作为默认分支（master分支是创建就有了，一般存放静态页面，配置之类的，当做所有分支的源），配置默认分支的方法如下所示。

![upload successful](/images/pasted-2.png)
到这里我们就完成了第一步。

### 克隆分支
在你想要写博客或者项目的文件夹下克隆刚刚创建好的分支。
~~~
git clone “你的仓库克隆地址”
~~~
然后将.git之外的文件全部删除。然后测试一下提交（保存到仓库）的功能。新建一个文件夹，再在文件夹下随意创建一个文件（空文件夹无法上传）。
~~~
git add new-folder/nwe-file
git commit -m "nwe file"  //""中是写到日志中的，可以更改
git push
~~~
在上面git push的时候可能会有错误。（下图中beanch是我拼写错误，应该写成分支的名称），下图的错误是http.sslverify值的错误。

![upload successful](/images/pasted-3.png)
用以下命令更改。
~~~
//将http.sslVerify改为 false
 git config --global http.sslVerify false

~~~
没有错误的情况下提交会出现如下的样子。

![upload successful](/images/pasted-4.png)

tip:有可能在push的时候会问你谁，你输入你的邮箱和姓名（GitHub）就行。

#### hexo环境
在新电脑上需要安装git、ssh-key、nodejs、hexo，这里不详细将每一个安装过程了。
安装git
~~~
sudo apt-get install git
~~~
设置git全局邮箱和用户名（第二步有可能会问你的）
~~~
git config --global user.name "yourgithubname"
git config --global user.email "yourgithubemail"
~~~
设置ssh key
~~~
sh-keygen -t rsa -C "youremail"
#验证是否成功
ssh -T git@github.com
~~~
安装nodejs
~~~
sudo apt-get install nodejs
sudo apt-get install npm
~~~
新建一个文件夹，并hexo初始化。
~~~
hexo init new folder
~~~
安装model
~~~
cd new folder 
npm install
npm install hexo-deployer-git --save
npm install --save hexo-admin   //推荐
~~~

将这个文件夹下的所有东西都复制到，第二步文件夹中（复制前应该只有.get文件）。在新的文件夹中，生成并布属。
~~~
hexo g
hexo d
hexo s
~~~
成功后，登录localhost:4000/admin

![upload successful](/images/pasted-5.png)
出现如下的界面，说明大功告成。

tip：每次写完博客不要忘记提交哦。
~~~
git add .
git commit -m "简短说明"  //""中是写到日志中的，可以更改
git push
~~~
参考博客：https://www.zhihu.com/question/21193762（直上云霄回答的）