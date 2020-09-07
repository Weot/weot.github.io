---
title: python开发环境配置与部署
date: 2017-09-10 03:53
updated: 2018-11-09 15:18
categories: 
- 编程开发
- Python
tags: [编程, Python, 环境, Sublime Text3, cmder]
---
## 一. 安装python

> should I use Python 2 or Python 3 for my development activity?

> Short version: Python 2.x is legacy, Python 3.x is the present and future of the language


python2还是python3, 官网上是这样[说明](https://wiki.python.org/moin/Python2orPython3)的
现在比较推崇的是python3:
    
- 首先, Python2只会停留在2.7版本, 不会再重大更新, Python3才是未来
- 其次, python3默认支持Unicode, 很多工作会变得容易
- 第三, 很多第三方库都已经支持了Python3

> windows下安装python36

直接下载python安装包，安装即可

> centos 7下安装python36

centos默认的python版本是python27, 安装python3的话需要自行下载编译安装

1. 官网下载源码文件
    ```shll
    weget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tar.xz
    ```
2. 解压缩文件
    ```shll
    xz -d Python-3.6.3.tar.xz
    tar xvf Python-3.6.3.tar
    ```
2. 编译安装
    ```shll
    sudo mkdir /usr/local/python3  # 创建安装目录
    cd Python3.6.3
    sudo ./configure --prefix=/usr/local/python3 --enable-optimizations
    sudo make
    sudo make install
    ```
4. 替换默认的python27
    ```shell
    sudo mv /usr/bin/python /usr/bin/python.bak  # 备份默认的python链接
    sudo ln -s /usr/local/python3/bin/python3.6 /usr/bin/python  # 使用python36作为默认版本
    sudo ln -s /usr/local/python3/bin/pip3 /usr/bin/pip  # 添加pip的链接
    ```
> centos 7下安装后yum命令无法使用问题

原来的脚本中yum有关的脚本中由于使用的python2, 更换python3后无法使用，需要修改脚本文件指向原来的python27
需要更改的文件：

    1./usr/bin/yum
    2./usr/libexec/urlgrabber-ext-down
遇到其他有关python出现的问题，修改相应的文件即可

> centos 7编译安装后使用pip出现错误(centos 7下先看此步骤)

错误提示:
    
```sh
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
```

解决方法:

1. 安装`openssl-devel`
    ```shell
    yum install openssl-devel -y
    ```
2. 重新编译安装python

> centos 7下ipython运行出现警告

错误提示：
```sh
/home/meo/Code/pydata/lib/python3.6/site-packages/IPython/core/history.py:226: UserWarning: IPython History requires SQLite, your history will not be saved
  warn("IPython History requires SQLite, your history will not be saved")
```
解决方法：

- 下载sqllite
```shell
wget http://www.sqlite.org/sqlite-autoconf-3071401.tar.gz
tar zxvf sqlite-autoconf-3071401.tar.gz
```
- 编译安装

```shell
cd cd sqlite-autoconf-3210000/
sudo ./configure --prefix=/usr/local/
sudo make
sudo make install
```
- 重新编译安装python

> ipython中使用matplotlib显示缺少tk

通常提示：
```python
ImportError: No module named '_tkinter'
```

tkinter和tk-devel缺失

解决方法：

- 安装缺失模块
    ```shell
    sduo yum install -y tkinter
    sudo yum install -y tk-devel
    ```
- 重新编译python

## 二. pip的配置与使用

### 1. 基本配置

pip默认会使用官方的源, 在国内下载速度不是很好, 因此可以配置国内镜像, 提高下载速度

三种顺序的配置

1. site-wide 配置文件位置: `C:\ProgramData\pip\pip.ini`or`/etc/xdg/pip/pip.conf`
2. per-user 配置文件位置: `%APPDATA%\pip\pip.ini`或者`%HOME%/pip/pip.conf`
3. virtualenv 配置文件位置: `%VIRTUAL_ENV%\pip.ini`

如果, 在以上位置没有找到`pip.ini`文件, 可以自己在任意位置创建一个, 然后设置环境变量`PIP_CONFIG_FILE`指向该文件, 文件内容为:

`pip.ini` or `pip.conf`:

```
[global]
index-url = https://pypi.douban.com/simple
```

pipy国内镜像目前有：
 
- http://pypi.douban.com/  豆瓣
- http://pypi.hustunique.com/  华中理工大学
- http://pypi.sdutlinux.org/  山东理工大学
- http://pypi.mirrors.ustc.edu.cn/  中国科学技术大学

pip.ini文件中的地址可以上面的国内镜像地中任意选择一个

需要注意的是, 在Ubuntu系统中, 配置文件名应为`pip.conf`

### 2. pip的基本操作

最常用的命令是查看安装包和安装等

#### 2.1 查看命令

```sh
pip list --format=columns  # 显示安装的包列表
pip list --format=legacy  # 传统显示
pip list --format=json  # json方式显示 
pip list --format=freeze  # freeze命令形式显示 

```

#### 2.2 安装相关命令

安装包, 有三种不同的形式供选择:
```sh
pip install SomePackage            # latest version
pip install SomePackage==1.0.4     # specific version
pip install 'SomePackage>=1.0.4'     # minimum version
```
从`wheels`文件安装包:
```sh
pip install SomePackage-1.0-py2.py3-none-any.whl
```

利用`requirement.txt`安装包:
```sh
pip install -r requirements.txt
```

`requirement.txt`是一个指定依赖包的文件, 可以很方便的下载所以的依赖包, 可以利用`pip`的命令很快的生成这个依赖文件, 在部署应用的时候应该经常用到:
```sh
pip freeze > requirement.txt
```


删除包命令:
```sh
pip uninstall SomePackage
```

更多命令, 参考pip[官方文档](https://pip.pypa.io/en/stable/reference/)

## 三. 使用虚拟环境virtualenv

> 为什么要使用虚拟环境? virtualenv可以为不同的项目生成一个个隔离运行的环境, 防止不同版本依赖问题, 便于管理

### 1. 安装virtualenv包

默认的python3环境没有安装virtualenv工具, 可以通过`pip`命令进行安装:

```sh
pip install virtualenv
```

### 2. 创建虚拟环境

在你喜欢的位置新建一个文件夹, 用来存放虚拟环境就可以了:

```
virtualenv --no-site-packages myflask
```
上述命令之后, 会在你的文件夹下产生一个`myflask`文件夹, 里面就是名为myflask的虚拟环境

### 3. 激活虚拟环境

windows环境:
```sh
./venv/myflask/Scripts/activate
```
linux环境:
```sh
source /venv/myflask/Scripts/activate
```

### 4. 使用虚拟环境过程中的问题

有些包无法用pip直接下载, 可以把下载下来的.exe安装包文件, 放在一定的位置, 然后用`easy_install`命令进行安装

### 5. 离开虚拟环境
    
```
deactivate
```

更多关于virtualenv的内容, 参考[官方文档](https://virtualenv.pypa.io/en/stable/)

## 四. 使用git

git命令:

从bae远程仓库克隆出本地仓库
```cmd
git clone https://git.duapp.com/your_bae_appid my_flask
```

本地文件复制修改完后:
```cmd
git add -A
git commit -m "initial"
git push
```

git 记住用户名密码:
```
$ git config --global user.name [name]
$ git config --global user.emial [email]
$ git config --global credential.helper store
```

更多内容, 参考[官方文档](https://git-scm.com/documentation)


## 五. 部署flask应用到远程服务器(pythonanywhere)

The process of deploying an app on PythonAnywhere involves three main steps:
    
    1. Uploading your code to PythonAnywhere
    2. Setting up a virtualenv, if you need one
    3. Configuring your application using WSGI
    
NB - only WSGI apps are supported, so that includes anything built with Django, Flask, Bottle,etc, but custom servers using, eg, Tornado, probably won't work.

#### 1. 新建一个web app, 选择手工配置

    Manual configuration involves editing your own WSGI configuration file in /var/www/. Usually this imports a WSGI-compatible application which you've stored elsewhere
    
    When you click "Next", we will create a WSGI file for you, including a simple "Hello World" app which you can use to get started, as well as some comments on how to use other frameworks.
    
    You will also be able to specify a virtualenv to use for your app.


#### 2. 从github拉取代码

#### 3. 创建virtualenv

#### 4. 在虚拟环境中下载所需的包

#### 5. 配置WSGI文件

环境变量要在这里设置

```
import os
os.environ['FLASK_CONFIG'] = 'production'
```
#### 6. flask使用flask_sqlalchemy

要下载linux下相应mysql的模块

```
pip install mysql-python
```

数据库路径也有相应的变化

```
os.environ['DATABASE_URI'] = 'mysql+mysqldb://username:password@wangmiao.mysql.pythonanywhere-services.com/databasename'
```


使用不同的mysql数据库驱动, 数据库连接路径也有不同

使用pymsql时:
```
os.environ['DATABASE_URI'] = 'mysql+pymysql://username:password@wangmiao.mysql.pythonanywhere-services.com/databasename'
```

## 六: 附加内容

### 1. 编辑器

> 编辑器当然要选择自己顺手的, 集成度较高的可以选择pycharm, 但我更喜欢用sublime text3, 选择sublime text3的原因有以下几点:

- 启动速度快, 流畅
- 界面简洁, 设置简单
- 插件比较丰富
- 快捷键比较高效

#### 1.1 下载安装

[下载地址](http://www.sublimetext.com/3)

注册码的话, 网上可以很方便找到

#### 1.2 插件安装

插件安装之前, 需要先安装`package control`插件, 用于安装其他插件

`package control`可以通过sublime text3的命令窗口进行安装,  使用`` ctrl+` ``或者 `view` > `Show Console` 菜单打开命令窗口, 输入以下代码, 回车:

```python
import urllib.request,os,hashlib;
h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76';
pf = 'Package Control.sublime-package';
ipp = sublime.installed_packages_path();
urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) );
by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read();
dh = hashlib.sha256(by).hexdigest();
print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

![image](http://ozi7kdi5x.bkt.clouddn.com/python%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE%E4%B8%8E%E9%83%A8%E7%BD%B2a388dab1dc378ae.webp.jpg)


`package control`插件安装之后就可以直接在sublime text3中通过`shift+ctrl+p`, 选择`Install Package`来安装其他插件了

![image](http://ozi7kdi5x.bkt.clouddn.com/python%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE%E4%B8%8E%E9%83%A8%E7%BD%B2489a01f707ab906.webp.jpg)

python环境比较有用的插件

- Anaconda: 代码规范检查, 代码提示
- Git: git版本管理插件
- AutoDocstring: 文档注释插件
- DocBlockr_Python: 文档注释

### 2. win下的增强命令行工具

`cmder`, [下载地址](http://cmder.net/), 可以下载mini版, 启动速度快, 主要是几款比较好看的配色方案, 字体可以自定义

#### 2.1 1703win10下中文重叠问题解决方法

![image](http://ozi7kdi5x.bkt.clouddn.com/python%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE%E4%B8%8E%E9%83%A8%E7%BD%B237bebc5761110a0.jpg)

设置使用旧版控制台

#### 2.2  中文乱码问题

在cmder中设置environment变量

![image](http://ozi7kdi5x.bkt.clouddn.com/python%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE%E4%B8%8E%E9%83%A8%E7%BD%B2423a747e6d0c352.jpg)

```cmd
set LANG=zh_CN.UTF8
```

#### 2.3 设置cmder中python虚拟环境提示符

在新版cmder中, 默认情况, `(pydata)` 类似虚拟环境提示符是显示在第二行命令行提示符前面的, 而不是之前在路径前面那样, 要变成之前那样, 需要修改cmder文件夹下的`clink.lua`文件:

```lua
function set_prompt_filter()
    -- get_cwd() is differently encoded than the clink.prompt.value, so everything other than
    -- pure ASCII will get garbled. So try to parse the current directory from the original prompt
    -- and only if that doesn't work, use get_cwd() directly.
    -- The matching relies on the default prompt which ends in X:\PATH\PATH>
    -- (no network path possible here!)
    local old_prompt = clink.prompt.value
    local cwd = old_prompt:match('.*(.:[^>]*)>')
    if cwd == nil then cwd = clink.get_cwd() end
    
    -- environment systems like pythons virtualenv change the PROMPT and usually
    -- set some variable. But the variables are differently named and we would never
    -- get them all, so try to parse the env name out of the PROMPT.
    -- envs are usually put in round or square parentheses and before the old prompt
    local env = old_prompt:match('.*%(([^%)]+)%).+:')
    -- also check for square brackets
    if env == nil then env = old_prompt:match('.*%[([^%]]+)%].+:') end
    
    -- build our own prompt
    -- orig: $E[1;32;40m$P$S{git}{hg}$S$_$E[1;30;40m{lamb}$S$E[0m
    -- color codes: "\x1b[1;37;40m"
    local cmder_prompt = "\x1b[1;32;40m{cwd} {git}{hg} \n\x1b[1;30;40m{lamb} \x1b[0m"
    cmder_prompt = string.gsub(cmder_prompt, "{cwd}", cwd)
    if env == nil then
        lambda = "λ"
    else
        lambda = "("..env..") λ"
    end
    clink.prompt.value = string.gsub(cmder_prompt, "{lamb}", lambda)
end
```

默认情况下, 其中的`set_prompt_filter`函数原版是这样的, 修改这个函数就可以达到目的, 不喜欢`λ`符号的, 也可以在这里修改, 下面是修改虚拟环境提示符的代码:

```
function set_prompt_filter()
    -- get_cwd() is differently encoded than the clink.prompt.value, so everything other than
    -- pure ASCII will get garbled. So try to parse the current directory from the original prompt
    -- and only if that doesn't work, use get_cwd() directly.
    -- The matching relies on the default prompt which ends in X:\PATH\PATH>
    -- (no network path possible here!)
    local old_prompt = clink.prompt.value
    local cwd = old_prompt:match('.*(.:[^>]*)>')
    if cwd == nil then cwd = clink.get_cwd() end
    
    -- environment systems like pythons virtualenv change the PROMPT and usually
    -- set some variable. But the variables are differently named and we would never
    -- get them all, so try to parse the env name out of the PROMPT.
    -- envs are usually put in round or square parentheses and before the old prompt
    local env = old_prompt:match('.*%(([^%)]+)%).+:')
    -- also check for square brackets
    if env == nil then env = old_prompt:match('.*%[([^%]]+)%].+:') end
    
    -- build our own prompt
    -- orig: $E[1;32;40m$P$S{git}{hg}$S$_$E[1;30;40m{lamb}$S$E[0m
    -- color codes: "\x1b[1;37;40m"
    local cmder_prompt = "\x1b[1;37;40m{env}\x1b[1;32;40m{cwd} {git}{hg} \n\x1b[1;30;40m{lamb} \x1b[0m"
    if env == nil then
        env = ''
    else
        env = "("..env..") "
    end
    cmder_prompt = string.gsub(cmder_prompt, "{env}", env)
    cmder_prompt = string.gsub(cmder_prompt, "{cwd}", cwd)
    lambda = "λ"
    clink.prompt.value = string.gsub(cmder_prompt, "{lamb}", lambda)
end

```

### 3. 不容易安装的python包whl包下载地址

在windows下, 有些包不能直接通过pip下载官网的包, 比如`scipy`,  可以在这个[地址](http://www.lfd.uci.edu/~gohlke/pythonlibs/), 找到相应的包, 下载安装就可以了