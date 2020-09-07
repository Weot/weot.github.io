---
title: Python爬虫实例(MOHRE)
date: 2019-01-11 12:51:03
categories:
- 编程开发
- Python
tags: [爬虫, Python, Scrapy, spider]
typora-root-url: ..
---
# Python爬虫实例(MOHRE)

[toc]

## 一. 概述

> 由于工作需要, 每过一段时间, 需要在阿联酋移民局的网站上检查某一人员是否在系统中, 而系统中人名是按字母序排列的, 找到一个人往往需要很多次翻页. 所以, 为了能够检索所有的劳工卡信息, 我们想要每次都把公司的所有雇员信息下载下来. 

MOHRE(The Ministry of Human Resources and Emiratisation), 中文阿联酋人力资源与移民局, 要获取公司的雇员信息, 首先要用公司的账户进行登录, 登录后回答相应的提示问题, 再跳转到雇员信息页面, 再解析页面上的信息就可以得到我们想要的数据了. 

登录验证 -> 问题验证 -> 数据提取

> 如何完成登录验证?

借助requests库, 模拟发送请求, 完成两次验证, 返回相应的Cookie值.

> 如何解析页面?

python有成熟的第三方数据爬取的库`scrapy`, 可以很方便的利用scrapy完成页面的解析及提取工作.

## 二. 准备工具

### 1. scrapy

> scrapy是一个为爬虫应用而生的应用框架, 能够用来解析所有结构化得页面.

详细的参考说明可以参见 [scrapy官网](https://doc.scrapy.org/en/master/)

1. 下载安装

```
pip install scrapy
```

2. 创建爬虫项目

```
scrapy startproject mohre
```

3. 编写核心程序(位于spider下面)

```
mohre/
    scrapy.cfg            # deploy configuration file

    mohre/             # project's Python module, you'll import your code from here
        __init__.py

        items.py          # project items definition file

        middlewares.py    # project middlewares file

        pipelines.py      # project pipelines file

        settings.py       # project settings file

        spiders/          # a directory where you'll later put your spiders
            __init__.py
            
            cookie.py     # the module you put the get_cookie function

            labour.py     # core spider file 
```

4. 运行爬虫程序, 输出到`csv`文件

```
scrapy crawl emp -o emplist.csv
```

### 2. requests

requests有非常好用的模拟发送请求的函数, 这里我们用来完成两次验证

安装方法:

```
pip install requests
```

requests 的使用方法易于理解, 详细的用法可以参考 [requests官网](http://www.python-requests.org/en/master/)

我们要用的主要功能有:

1. `request.session`功能
2. `session.get(url)`
3. `seesion.post(url, data, heders)`

### 3. lxml.html.etree

scrapy本身带了解析HTML页面的能力, 但是我们在获取登录cookie的时候, 我们只能先借助requests库发送请求, 再利用lxml.html.etree来解析返回的页面

详细的用法见[官网](https://lxml.de/lxmlhtml.html)

我们这里用的功能其实与scrapy中的css选择器大致相同, 函数的用法有稍许的不同.


## 三. 爬取步骤

总的步骤可以分为两部分, 一部分负责登录验证cookie的获取, 另一部分负责解析及生成所需数据.

### 1. 登录验证处理

首先, 分析登录页面的结构及登录时所需提交的数据.

登录页面效果如下:
![登录页面](/images/2019-01-11-Python爬虫实例-MOHRE/01登录页面.png)

下面是登录页面的HTML结构, 可以看到表单中有几个隐藏中的参数, 后面模拟验证登录会用到
```html
<form name="frmLogin" method="post" action="./login.aspx?lang=eng" onsubmit="javascript:return WebForm_OnSubmit();"
    id="frmLogin">
    <div>
        <input type="hidden" name="__EVENTTARGET" id="__EVENTTARGET" value="" />
        <input type="hidden" name="__EVENTARGUMENT" id="__EVENTARGUMENT" value="" />
        <input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="/wEPDwULLTE2ODcyMDQ2MDUPZBYCAgEPZBYUAhkPFgIeCWlubmVyaHRtbAVuPGEgaHJlZj0nU1NPUmVkaXJlY3QuYXNweD9BcHBJRD1OVFdTTCc+PGltZyBzcmM9J0ltYWdlcy9TbWFydFBhc3MuanBnJyBzdHlsZT0naGVpZ2h0OjQ2cHg7IHdpZHRoOjIwMHB4OycvPjwvYT5kAhsPDxYCHgRUZXh0BQgxMDgzNzMxM2RkAh0PDxYCHwEFBTExNDg2ZGQCHw8PFgIfAQUGMTI5MzI4ZGQCIQ8PFgIfAQUGNDg0NDkxZGQCLQ8PZBYCHgdPbkNsaWNrBR1KYXZhU2NyaXB0OnJldHVybiB2YWxpZGF0ZSgpO2QCMw8PFgIfAQUoV2hhdCBpcyB5b3VyIG9waW5pb24gYWJvdXQgdGhpcyB3ZWJzaXRlP2RkAjcPDxYCHwEFCUV4Y2VsbGVudGRkAjsPDxYCHwEFBEdvb2RkZAI/Dw8WAh8BBQhBY2NlcHRlZGRkGAEFHl9fQ29udHJvbHNSZXF1aXJlUG9zdEJhY2tLZXlfXxYFBQZyZEFuczEFBnJkQW5zMgUGcmRBbnMyBQZyZEFuczMFBnJkQW5zM4ZFlmWhclpng5Sf6MKt0EhdY/oPz6kkgQDpa5/C8D8N" />
    </div>
    <div>
        <input type="hidden" name="__VIEWSTATEGENERATOR" id="__VIEWSTATEGENERATOR" value="0428CBE9" />
        <input type="hidden" name="__EVENTVALIDATION" id="__EVENTVALIDATION" value="/wEdABYIEPM8/T91XneGPQ1SRtM79blI3CMu38DZhvtVg0A1jAchBDsGRG9ZfJOBdKkCF4EU9J1w2sKBHROFXnqIDBMatlx55RwyllFnTGO/io4DWWN6ZYJNGAQHn0c9zMgZU3B2NvjHOkq5wKoqN6Aim8WG8tE4qI7UAUU9EOLUxz/cCPKFQvO1JfZiM1HVv+DKjoszHc/H4FBVzxWkc9VezzQN5IN6oJBlk11msy36jadMlKM6CWabl8ZKTlSSkWx8rh5DB2YfWbjwUYntrwPx36ESX7JzMDb/wVJ9mJ810Q97izzmltaUM7aEAN+g9cP/m13hgAAu9je581zBbk7c4sh+d7RnisS1PiyBpYEMF6nUbLTE/53XZug13ygX1VEAZiC3zEBk0+CrJ9JH/kfsOtYKEEiEoCtJLWPQasqn4dMVICA5ToeEaIY8FsTKJAoH286KnJAu6CKHUdY9glvBwFvvI4R2XFuv+NyO10+mp1xBQOcgAVsosM3pxA/Mw/Sd53Q=" />
    </div>
    <div class="textbox-wap">
        <span class="username">
            <input name="txtUserName" type="text" id="txtUserName" autocomplete="off" />
        </span>
        <span class="password">
            <input name="txtPassword" type="password" id="txtPassword" />
        </span>
    </div>
    <a id="lbtnForgotPass" href="javascript:WebForm_DoPostBackWithOptions(new WebForm_PostBackOptions(&quot;lbtnForgotPass&quot;, &quot;&quot;, true, &quot;&quot;, &quot;&quot;, false, true))">Forgot
        Password /Security Question?</a>
    <div class="flexbox">
        <input type="submit" name="cmdlogin" value="Submit" onclick="javascript:WebForm_DoPostBackWithOptions(new WebForm_PostBackOptions(&quot;cmdlogin&quot;, &quot;&quot;, true, &quot;&quot;, &quot;&quot;, false, false))"
            id="cmdlogin" class="btn btn-primary" />
        <a id="lbtnNewEmployee" href="javascript:WebForm_DoPostBackWithOptions(new WebForm_PostBackOptions(&quot;lbtnNewEmployee&quot;, &quot;&quot;, true, &quot;&quot;, &quot;&quot;, false, true))"
            style="display: none">New Employees Signup here</a>
        <a id="lbtnNewUser" class="btn btn-default" href="javascript:WebForm_DoPostBackWithOptions(new WebForm_PostBackOptions(&quot;lbtnNewUser&quot;, &quot;&quot;, true, &quot;&quot;, &quot;&quot;, false, true))"
            style="display: block">New Owners Signup here</a>
    </div>
</form>
```

下面是截取提交登录的过程中传递参数的截图:
![提交登录](/images/2019-01-11-Python爬虫实例-MOHRE/02提交登录.png)

从图中可以看到, 提交了这些参数, 其中五个隐藏在页面中的参数和用户名密码是必须的, 其他的并不是很重要, 我们只要从页面中把这些提取出来, 连同用户名/密码一起提交, 就可以完成初步验证
```html
__EVENTTARGET:                          // 隐藏在页面中的参数
__EVENTARGUMENT:                        // 隐藏在页面中的参数
__VIEWSTATE:/wEPDwULLTE2ODcyMDQ2MDUPZ   // 隐藏在页面中的参数
__VIEWSTATEGENERATOR:0428CBE9           // 隐藏在页面中的参数
__EVENTVALIDATION:/wEdABYIEPM8          // 隐藏在页面中的参数
type:pages
type:news
type:events
type:faqs
type:service
txtUserName:xxxxx                       // 提交的用户名
txtPassword:xxxxx                       // 提交的密码
cmdlogin:Submit
txtName:
txtEmail:
txtComments:
hdnComno:
hdnStatus:
gPoll:rdAns1
hdnQuestion:1
```

由于需要问题验证, 此时虽然得到了一个cookie, 但不能够使用

登录成功后, 会跳转到问题验证页面:
![问题验证页面](/images/2019-01-11-Python爬虫实例-MOHRE/03问题验证页面.png)


通过查看HTML代码, 可以看到问题验证其实是一个嵌套的iframe, iframe的代码如下:
```html
<form name="Form1" method="post" action="./SecurityQuestions.aspx?type=1&amp;lang=1&amp;userID=31416076573906" id="Form1"
    dir="">
    <div>
        <input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="/wEPDwUKMTQxNDM1NTI1MA9kFgJmD2QWAgIBD2QWBAIBD2QWAgIBD2QWAgIBDxYCHgdWaXNpYmxlaBYIAgEPZBYCAgEPZBYCAgEPEGRkFgBkAgIPZBYCAgEPZBYCAgEPDxYCHgRUZXh0ZWRkAgMPZBYCAgEPZBYCAgEPEGRkFgBkAgQPZBYCAgEPZBYCAgEPDxYCHwFlZGQCAg9kFgICAQ9kFgICAQ9kFgJmD2QWAgIBD2QWAgIBDw8WAh8BBRxXaGF0IGlzIHlvdXIgZmF2b3JpdGUgY29sb3I/ZGRkuJzrJR5MIM5THyFMd50eqEc04kTwDrO3Tzcd0PFrZF8=" />
    </div>
    <div>
        <input type="hidden" name="__VIEWSTATEGENERATOR" id="__VIEWSTATEGENERATOR" value="823AD229" />
        <input type="hidden" name="__EVENTVALIDATION" id="__EVENTVALIDATION" value="/wEdAANl8JPPQXxJKQsyWwDBCkZVV8Br3qobb5+3aHdcJeGCPrDVG0sT6bnJexpzy9nEXAs8t74ulrfzb3d6yyS2rJN22uCgqMPvpf5FOWKcxsKEpQ==" />
    </div>
    <table width="100%">
        关于问题回答表单的代码
    </table>
</form>
```
同样的, 这个表单中也包含几个隐藏的参数, 将答案及隐藏的参数一块提交给验证的网址, 正常浏览器会利用返回的页面中的js代码完成进一步的验证
![问题验证](/images/2019-01-11-Python爬虫实例-MOHRE/04问题验证.png)

```html
__VIEWSTATE:/wEPDwUKMTQxN
__VIEWSTATEGENERATOR:823AD229
__EVENTVALIDATION:/wEdAANqj5rO+kwNDHGx2wX65AqaV8Br3qobb5+3aHdcJeGCPrDVG0sT6bnJexpzy9nEXAuhYPLXGl4byF9gCESzBlRvnGAMVw6dJs7rTLonqZC/Cg==
txtAnswer:GREEN
btnSubmit2:Submit
```
提交问题返回的js代码会包含安全验证的tkn码, 在利用验证网址组合tkn码进行get请求, 至此安全验证, 现在提取cookie, 就可以使用了
完整的获取cookie代码如下:

```python
import requests
from lxml.html import etree


def get_cookie():
    """
    获取登录用的Cookie
    1. 通过解析登录页面, 提交账户密码及一些其他隐藏在页面中的参数, 获取cookie;
    此时获取的Cookie不是能够直接使用, 还需要通过问题验证才可以使用
    2. 利用第一步已经登录的session, 获取隐藏在二次验证的页面中的iframe问题验证页面, 通过提交问题及其他隐藏校验信息,
    返回校验成功的页面, 此时页面中含有一个包含的tkn信息
    3. 正常的浏览器界面在验证问题后转向一个登陆成功后的公司用户信息主页面, 通过分析第二步的验证页面, 可以知道,
    页面的跳转是通过JS实现的, JS中包含的tkn信息和用户ID, 作为二次验证的参数, GET提交得到跳转的页面, 此时的cookie就可以正常使用了

    Returns:
        Dict: 完成验证的Cookie字典
    """

    # 登录用的地址
    login_url = 'https://eservices.mohre.gov.ae/enetwasal/login.aspx?lang=eng'
    # 二次验证的地址
    auth_url = 'https://eservices.mohre.gov.ae/enetwasal/SecurityAuthentication.aspx?type=1'
    # 获取问题验证的地址, 这里直接把UserID参数硬编码进去了
    question_url = 'https://eservices.mohre.gov.ae/enetwasal/SecurityQuestions.aspx?type=1&lang=1&userID=31416076573906'

    # 使用requests的session功能
    session = requests.session()

    # 获取登录页面, 并用etree.HTML进行解析
    r_login = session.get(login_url)
    page = etree.HTML(r_login.text)
    # 解析隐藏在页面中的登录需要的参数信息
    view_state = page.cssselect('input#__VIEWSTATE')[0].attrib['value']
    view_state_generator = page.cssselect('input#__VIEWSTATEGENERATOR')[
        0].attrib['value']
    event_validation = page.cssselect('input#__EVENTVALIDATION')[
        0].attrib['value']
    hdn_question = page.cssselect('input#hdnQuestion')[0].attrib['value']
    # 构造登录需要的POST参数信息
    formdata = {
        '__EVENTTARGET': '',
        '__EVENTARGUMENT': '',
        '__VIEWSTATE': view_state,
        '__VIEWSTATEGENERATOR': view_state_generator,
        '__EVENTVALIDATION': event_validation,
        'txtUserName': 'xxxxxx',
        'txtPassword': 'xxxxxx',
        'cmdlogin': 'Submit',
        'txtName': '',
        'txtEmail': '',
        'txtComments': '',
        'hdnComno': '',
        'hdnStatus': '',
        'gPoll': 'rdAns1',
        'hdnQuestion': hdn_question
    }
    # 构造USE-Agent头信息, 防止被网站屏蔽掉
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36'
    }
    # 提交登录信息, 问题验证的地址, 验证回调的二次验证也会在这里面返回
    r_after_login = session.post(login_url, data=formdata, headers=headers)

    # 使用登录后的session获取问题验证页面
    r_question = session.get(question_url)
    # 使用etree解析页面, 获取隐藏的参数信息
    page = etree.HTML(r_question.text)
    view_state = page.cssselect('input#__VIEWSTATE')[0].attrib['value']
    view_state_generator = page.cssselect('input#__VIEWSTATEGENERATOR')[
        0].attrib['value']
    event_validation = page.cssselect('input#__EVENTVALIDATION')[
        0].attrib['value']
    # 获取问题, 并判断是两个问题中的哪一个, 再给出相应的问题
    question = page.cssselect('span#lblQuestionValue')[0].text
    answer = 'GREEN ' if 'color' in question else 'Pakistan'
    # 构造问题验证提交参数
    formdata = {
        '__VIEWSTATE': view_state,
        '__VIEWSTATEGENERATOR': view_state_generator,
        '__EVENTVALIDATION': event_validation,
        'txtAnswer': answer,
        'btnSubmit2': 'Submit'
    }
    # 提交问题验证信息
    r_after_question = session.post(
        question_url, data=formdata, headers=headers)

    # 获取验证页面后的返回页面, 并解析其中的JS跳转信息中的tkn信息
    page = etree.HTML(r_after_question.text)
    js = page.cssselect('script')[0]
    tkn = js.text.split("'")[1]
    # 构造出二次验证的地址, 并直接获取页面完成验证
    new_auth_url = auth_url + tkn
    r_auth = session.get(new_auth_url)

    # 解析出字典形式的cookie信息并返回
    cookie = session.cookies.get_dict()
    return cookie
```

### 2. 页面解析

登录后的员工列表页面会有250条人员信息, 并且是分页显示的:
![员工列表页面](/images/2019-01-11-Python爬虫实例-MOHRE/05员工列表页面.png)

这个爬虫的核心部分, 就是利用上面的获取cookie函数, 作初始化, 进入员工列表页面, 解析页面结构, 并提交下一页请求.

```python
"""
一个爬取阿联酋移民与劳工局上本公司劳工卡信息的爬虫
"""
import scrapy
from .cookie import get_cookie


class EmpSpider(scrapy.Spider):
    name = 'emp'
    allowed_domains = ['eservices.mohre.gov.ae']
    # 包含雇员信息的页面, 每页可以显示250条信息
    start_urls = [
        'https://eservices.mohre.gov.ae/enetwasal/rptComEmpList.aspx?comno=587582']

    # 使用get_cookie函数获取有效cookie, 使用cookie请求雇员详细信息页面
    def start_requests(self):
        request = scrapy.Request(
            self.start_urls[0],
            callback=self.parse,
            cookies=get_cookie()
        )
        yield request

    def parse(self, response):
        """解析页面, 生成结果信息
        
        Arguments:
            response {scrapy.Response} -- 请求页面返回的页面
        """

        # 在员工信息列表页面点击下一页的过程中, 实际是触发了一个JS事件,
        # js会通过隐藏在页面中的一些参数值向服务器请求下一页
        # 通过对每一次请求及返回页面源码的查看, 可以知道每次请求, 
        # 有三个状态及验证信息的字段, 两个名字字段, 应该是标记位置用的, 
        # 另外代表下一页还上一页的的一个信息

        # 从页面提取三个验证字段
        view_state = response.css(
            'input#__VIEWSTATE::attr(value)').extract_first()
        view_state_generator = response.css(
            'input#__VIEWSTATEGENERATOR::attr(value)').extract_first()
        event_validation = response.css(
            'input#__EVENTVALIDATION::attr(value)').extract_first()
        # 从页面提取两个关于人名的字段, 标记上一页的的字段可能为空,
        # 两个名字相同时候, 说明已经是最后一页了, 本次请求得到的页面其实是重复的
        # 所以这里提前提取, 为的是首先判断页面是已经爬取过, 如果相同就可以终止了
        hdnNextCardName = response.css(
            'input#hdnNextCardName::attr(value)').extract_first() or ""
        hdnPreviousCardName = response.css(
            'input#hdnPreviousCardName::attr(value)').extract_first() or ""

        # 根据两个名字相关的字段, 判断是否已经爬完所有页面, 是否终止爬虫
        if hdnNextCardName != hdnPreviousCardName:
            # 提取页面中的数据, 主要是对页面结构的理解
            # 详细页面的结构其实是一个标题加上几条记录信息的样式
            # 详细信息所幸都包含在具有border属性的table中
            # table中的每条tr就是一行信息, 但是页面中有一个bug, 第一行居然是空的
            # 这样表头就存在每个table中第二行了, 然后我们选表头行tr的所有兄弟节点,
            # 就是每一条的人员信息了
            # 接着就是解析每一行的信息, 返回就可以了
            for emp in response.css('.reportBack table[border] tr:nth-child(2) ~ tr'):
                # 每一行信息有6列(td), 有的列包含一个字段信息, 有的包含2-3个字段
                # 前4列都是包含一个字段, 提取再做一些调整及格式化就可以了
                # 第5列包含护照号和国籍两个字段
                # 第6列不太规整, 大部分包含三个字段, 卡号, 卡类型, 过期日期, 有的状态是正在处理中
                # 没有过期日期, 所以过期日期的字段采用负数索引, 选择最后一个字段 
                yield {
                    "NO": emp.css('td:nth-child(1)::text').extract_first().strip(),
                    "PersonCode": "'" + emp.css('td:nth-child(2)::text').extract_first().strip(),
                    "Name": emp.css('td:nth-child(3)::text').extract_first().strip().title(),
                    "Job": emp.css('td:nth-child(4)::text').extract_first().strip().title(),
                    "PassportNO": "'" + emp.css('td:nth-child(5)::text').extract()[0].replace(' ', ''),
                    "Nationlity": emp.css('td:nth-child(5)::text').extract()[1].strip().title(),
                    "CardNO": emp.css('td:nth-child(6)::text').extract()[0].strip(),
                    "CardType": emp.css('td:nth-child(6)::text').extract()[1].strip(),
                    "ExpiryDate": emp.css('td:nth-child(6)::text').extract()[-1].strip()
                }

            # 构造请求下一页的信息, button值为next代表请求下一页
            formdata = {
                '__VIEWSTATE': view_state,
                '__VIEWSTATEGENERATOR': view_state_generator,
                '__EVENTVALIDATION': event_validation,
                'hdnNextCardName': hdnNextCardName,
                'hdnPreviousCardName': hdnPreviousCardName,
                'hdnButtonClicked': "next"
            }
            yield scrapy.FormRequest(self.start_urls[0], formdata=formdata, callback=self.parse)
```