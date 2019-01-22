---
title: CentOS 7 安装使用与美化
date: 2017-10-28 09:50
updated: 2018-11-09 15:11
categories:
- 编程开发
- Linux
tags: [编程, Linux, 系统安装, 源, 主题, 输入法]
---

## 一. 系统安装

在下次安装时补上教程吧，双系统的安装基本上都是一样的


## 二. 关于源的更换

1. 源的分类

    - 官方源
    - 第三方源

与源相关的文件夹中的文件：

    yum.repos.d/
    ├── adobe-linux-x86_64.repo  
    ├── CentOS-Base.repo         # 官方源
    ├── CentOS-Base.repo.backup
    ├── CentOS-CR.repo
    ├── CentOS-Debuginfo.repo
    ├── CentOS-fasttrack.repo
    ├── CentOS-Media.repo
    ├── CentOS-Sources.repo
    ├── CentOS-Vault.repo
    ├── epel.repo                
    ├── epel.repo.rpmnew
    ├── epel-testing.repo
    ├── google-chrome.repo
    └── sublime-text.repo

更换国内源, 其实是更换了其中的`CentOS-Base.repo`文件

2. 更换阿里源：

>更换base源
    
```sh
sudo mv /etc/yum.repos.d/CentOS-Base.repo CentOS-Base.repo.backup  # 备份原来的源文件
sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo  # 使用阿里的源
```

>更换epel源

```sh
sudo wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```
>清理缓存，重建缓存

```sh
sudo yum clean all
sudo yum makecache
```

## 三. 输入法安装

1. 搜狗输入法第三方rpm包安装
    - 用网上的rpm包， root用户下安装成功，但是普通用户搜狗输入法的面板无法加载，暂时无法解决
    - sublime text3无法输入中文(显示是正常的), 网上解决方法比较麻烦， 暂时不配置
    
2. yum第三方源安装

    估计不会成功，未尝试(当时的第三方源加入社区源, 里面包含的私人软件去掉了, 应该是不能用了)

## 四. Gnome桌面美化

> Gnome作为CentOS的默认桌面，乍一看感觉很丑，其实有很多可以配置的地方，这一点与windows截然不同，只要你有心，从字体，图标，再到窗口都是可自己更改的，并且网上有很多现成的主题可以使用

![image](http://img3.doubanio.com/view/status/raw/public/175873a6d2b9550.jpg)

这是一个简单配置后的效果图

Gnome自定义配置的内容：

- 主题
- 图标
- Shell主题
- 扩展工具

Gnome美化的一个关键工具是`gnome-tweak-tool`，CentOS 7安装的时候应该是默认有的(中文名字应该叫`优化工具`)：

![image](http://img1.doubanio.com/view/status/raw/public/7516b9cf68665ba.jpg)

从这个工具中可以看到，主题中有GTK+主题，图标，光标，shell主题内容等几项，另外，还有扩展选项也是更改窗口一些行为的；

- GTK+主题：更改整体窗口的形式
- 图标：更改应用程序的图标等
- Gnome Shell主题：更改一些细节上的显示

1. 更改主题
    - 下载主题，[gnome-look地址](https://www.gnome-look.org/)，当然也可以从其他地方下载
    - 解压下载后的主题，放到`/usr/share/themes/`下
    - 在`gnome-tweak-tool`中选用对应的主题
    
    ![image](http://img3.doubanio.com/view/status/raw/public/4285944da29cbcf.jpg)
    
    例如，我下载了一个`OSX-Arc-White`主题，下载后我放在`/usr/share/themes/`下，之后重启`gnome-twea-too`工具，在gtk主题中选择对应的主题，就可以看到主题效果了
2. 更改图标
    - 下载图标主题，上面地址中也有关于图标的主题，自行下载
    - 解压缩，放到`/usr/share/icons/`目录下
    - 在`gnome-tweak-tool`中选用对应的图标
3. 使用扩展
    `gnome-tweak-tool`外观选项中有一个使用shell主题的的选项，但是默认无法选择，是由于缺少扩展插件的原因：
    
    ![image](http://img1.doubanio.com/view/status/raw/public/f825d998ee6ae67.jpg)

    点击获取跟多扩展，会跳转到[gnome-shell](https://extensions.gnome.org/)插件地址，你可以把对应的插件下载下来，也可以通过一个浏览器插件实时安装，我选择第一种方式：
    
    - 下载扩展包
    - 从安装`Shell扩展`处选用对用的扩展包
    
    下载插件：
    ![下载插件](https://img1.doubanio.com/view/status/raw/public/4b5f96d6725b429.jpg)
    
    选用插件：
    ![image](http://img3.doubanio.com/view/status/raw/public/b3509882aea32d5.jpg)
    
我安装的插件：
- User themes：用来启用shell主题的插件
- Dash to dock: 用来控制默认在左侧的快捷栏(我用来将它放在底部，透明化)
- Not title bar: 用来将窗口最大化后的标题，防止在顶部的状态栏(默认情况下最大化后标题会占用一行，感觉难受)

4. Shell主题

    有些内容完善的Gtk主题包中也会含有gnome-shell主题，把主题放置到`themes`文件夹后，在`gnome-tweak-tool`工具中就可以自行选择了，对于自己下载的gnome-shell主题，也可以自行放置后选择
    
    一个主题的文件夹的结构一般是`/usr/share/themes/主题名字/gnome-shell/`，
    下面还会有一些其他的东西
    
    使用`gnome-tweak-tool`安装shell主题：
    ![gnome-shell主题安装](http://img3.doubanio.com/view/status/raw/public/3465a7b411489e5.jpg)
    
5. 移除底部任务栏
    
    先备份原来的配置文件，文件路径`/usr/share/gnome-shell/extensions/window-list@gnome-shell-extensions.gcampax.github.com`

    ```
    cp window-list@gnome-shell-extensions.gcampax.github.com window-list@gnome-shell-extensions.gcampax.github.com.backupr
    ```
    
    然后删除配置文件，就可以达到目的了
    ```
    rm -rf window-list@gnome-shell-extensions.gcampax.github.com
    ```

