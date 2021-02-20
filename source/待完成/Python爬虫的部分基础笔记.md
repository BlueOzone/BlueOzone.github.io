---
title: Python爬虫的部分基础笔记
toc: true
date: 2021-01-13 16:28:48
categories:
tags:
---


### 使用urllib
#### urlopen()
定义：可以模拟浏览器的一个请求发起过程
用法：urllib.request.urlopen(url,data=None,[timeout,]*,catfile=None,capath=None,cadefault=False,context=None)
参数说明：
url(必须)：请求的URL
data(可选)：需要传递的参数，用时需要以POST方式传输数据；data=bytes(urllib.parse.urlencode({'word':'hello'}),encoding='utf-8')
timeout(可选)：设置请求超时时间；urllib.request.urlopen('http://httpbin.org/get',timeout=0.01)
context(可选)：必须是ssl.SSLContext类型，用于指定SSL设置
cafile(可选)：指定CA证书
capath(可选)：指定CA证书路径
cadefault(可选)：已弃用
测试代码：
```
import urllib.request
import urllib.request
import urllib.error

data = bytes(urllib.parse.urlencode({'word':'hello'}),encoding='utf-8')
response = urllib.request.urlopen('http://httpbin.org/post',data=data)
print(response.read())'''
#try:
    #response = urllib.request.urlopen('http://httpbin.org/get',timeout=0.01)
#except urllib.error.URLError as e:
#    if isinstance(e.reason,socket.timeout):
#        print('time out')
```

#### Request
用法：urllib.request.Request(url,data=None,headers={},origin_req_host=None,unverifiable=False,method=None)
参数说明：
url(必须)：请求的URL
data(可选)：需要传递的参数，用时需要以POST方式传输数据；data=bytes(urllib.parse.urlencode({'word':'hello'}),encoding='utf-8')
headers(可选)：设置请求头，字典类型；headers={'Host':'https://www.baidu.com'}
origin_req_host(可选)：指定请求方的HOST名称或者IP地址；
unverifiable(可选)：表示这个请求是否是无法验证的，默认值是False;
method(可选)：用来指定请求的方法，如GET、POST、PUT等；method='POST'
测试代码：
```
from urllib import parse,request

url = 'http://httpbin.org/post'
headers = {
    'User-Agent' : 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.90 Safari/537.36',
    'Host' : 'httpbin.org'
}
dict = {
    'name' : 'Germey'
}
data = bytes(parse.urlencode(dict),encoding='utf-8')
#request = urllib.request.Request('https://python.org')
req = request.Request(url,data=data,headers=headers,method='POST')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

#### 高级用法
代理
测试代码：
```
from urllib.error import URLError
from urllib.request import ProxyHandler,build_opener

proxy_handler = ProxyHandler({
    'http' : 'http://127.0.0.1:9743',  #实际代理地址
    'https' : 'https://127.0.0.1:9743'		#实际代理地址
})

opener = build_opener(proxy_handler)
try:
    response = opener.open('https://www.baidu.com')
    print(response.read().decode('utf-8'))
except URLError as e:
    print(e.reason)
```
	
Cookies
测试代码：
```
import http.cookiejar,urllib.request

cookie = http.cookiejar.CookieJar()
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
for item in cookie:
    print(item.name+"="+item.value)
```	
	
#### 处理异常
(1)URLError
```
from urllib import request,error

try:
    response = request.urlopen('https://cuiqingcai.com/index.htm')
except error.URLError as e:
    print(e.reason)
```
	
(2)HTTPError
```
from urllib import request,error

try:
    response = request.urlopen('https://cuiqingcai.com/index.htm')
except error.HTTPError as e:
    print(e.reason,e.code,e.headers,sep='\n')
except error.URLError as e:
    print(e.reason)
else:
    print('Request Successfully!')
```

#### 解析链接
(1)urlparse()
http://www.baidu.com/index.html;user?id=5#comment
scheme://netloc/path;params?query#fragment
协议    域名 访问路径 参数 查询条件 锚点（定位页面内部下拉位置）

urllib.parse.urlparse(urlstring,scheme='',allow_fragments=True)
urlstring(必填)：待解析的URL
scheme(选填)：默认的协议，仅在URL中不含scheme信息时生效
allow_fragments(选填)：是否忽略fragment

测试代码：
```
from urllib.parse import urlparse

result = urlparse('http://www.baidu.com/index.html;user?id=5#comment')
print(type(result),'\n',result)
```

(2)urlunparse()
接收的参数个数必须为6个，否则报错

测试代码：
```
from urllib.parse import urlunparse

data = ['http','www.baidu.com','index.html','user','id=1','comment']
result = urlunparse(data)
print(result)
```

(3)urlsplit()
测试代码：
```
from urllib.parse import urlsplit

result = urlsplit('http://www.baidu.com/index.html;user?id=5#comment')
print(result)
```

(4)urlunsplit()
接收的参数个数必须为5个，否则报错

测试代码：
```
from urllib.parse import urlunsplit

data = ('http','www.baidu.com','index.html;user','id=5','comment')
result = urlunsplit(data)
print(result)
```

(5)urljoin()
urljoin(基础链接base_url，新的链接)
该方法会分析base_url中的scheme、netloc、path这三个内容并对新链接缺失的部分进行补充

(6)urlencode()
一般用于构造参数，序列化

测试代码：
```
from urllib.parse import urlencode

params = {
    'name':'tony',
    'age':'22'
}
base_url = 'http://www.baidu.com?'
result = base_url + urlencode(params)
print(result)
```

(7)parse_qs()
根据urlencode构造好的参数进行反序列化，转换成字典类型

测试代码：
```
from urllib.parse import parse_qs

params = 'name=tony&age=22'
print(parse_qs(params))
```

(8)parse_qsl()
根据urlencode构造好的参数进行反序列化，转换成元祖类型

测试代码：
```
from urllib.parse import parse_qsl

params = 'name=tony&age=22'
print(parse_qsl(params))
```

(9)quote()
使用该方法可以将内容转化为URL编码的格式

测试代码：
```
from urllib.parse import quote

data = '你好'
base_url = 'http://www.baidu.com?wd='
print(base_url+quote(data))
```

(10)unquote()
可以进行URL解码

测试代码：
```
from urllib.parse import unquote

data = 'http://www.baidu.com?wd=%E4%BD%A0%E5%A5%BD'
print(unquote(data))
```

#### 分析Robots协议
(1)Robots协议
Robots协议也称作爬虫协议、机器人协议，全名为网络爬虫排除标准，用来告诉爬虫和搜索引擎哪些页面可以抓取，哪些页面不可抓取；通常是一个叫做robots.txt的文本文件，放在网站根目录下；
当爬虫访问一个站点会先去检查并访问这个robots.txt文件，根据其中的范围进行抓取；若不存在该文件或没有找到，搜索爬虫便会访问所有可直接访问的页面；

(2)爬虫名称
Baiduspider		百度		www.baidu.com

Googlebot		谷歌		www.google.com

YoudaoBot		有道		www.youdao.com

(3)robotparser
该模块可通过robotparser类解析robots.txt文件
robotparser常用方法：
set_url():用来设置robots.txt文件的链接
read()：读取robots文件并进行分析
parse()：用来解析robots.txt文件
can_fetch()：传入user-agent以及要抓取的URL，返回的结果是搜索引擎能否抓取这个URL；true  false
mtime()：返回的是上次抓取和分析 robots.txt 的时间
modified()：它同样对长时间分析和抓取的搜索爬虫很有帮助，将当前时间设置为上次抓取和分析 robots.txt 的时间

测试代码：
```
from urllib.robotparser import RobotFileParser

rp = RobotFileParser()
rp.set_url('https://www.jianshu.com/robots.txt')
rp. read()
print(rp.can_fetch('*','https://www.jianshu.com/p/b67554025d7d'))
print(rp.can_fetch('*',"https://www.jianshu.com/search?q=python&page=1&type=collections"))
```

### 使用requests
#### 基本用法
实例引入
测试代码：
```
from urllib.robotparser import RobotFileParser

rp = RobotFileParser()
rp.set_url('https://www.jianshu.com/robots.txt')
rp. read()
print(rp.can_fetch('*','https://www.jianshu.com/p/b67554025d7d'))
print(rp.can_fetch('*',"https://www.jianshu.com/search?q=python&page=1&type=collections"))
```

#### GET请求
(1)基本实例
测试代码：
```
import requests

data = {
    'name':'tony',
    'age':'22'
}
r = requests.get('http://httpbin.org/get',params=data)
print(type(r.text))
print(r.json())
```

(2)抓取网页
测试代码：
```
import requests
import re
headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36'
                 '(KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36'
}
r =requests.get ("https://www.zhihu.com/explore", headers=headers)
pattern = re . compile ('explore-feed.*?question_link.*?>(.*?)</a>', re.S)
titles = re.findall(pattern,r.text)
print(titles)
```

(3)抓取二进制数据
测试代码：
```
import requests

r = requests.get('https://github.com/favicon.ico')
with open('favicon.ico','wb') as f:
    f.write(r.content)
```

(4)添加headers
测试代码：
```
import requests

headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36'
                 '(KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36'
}
r = requests.get('https://www.zhihu.com/explore',headers=headers)
print(r.text)
```

#### POST请求
测试代码：
```
import requests

data = {'age':22,'name':'tony'}
rr = requests.post('http://httpbin.org/post',data=data)
print(rr.text)
```

####响应
测试代码：
```
import requests

rr = requests.get('http://httpbin.org/get')
print((type(rr.cookies)),rr.cookies)
print(type(rr.status_code),rr.status_code)
print(type(rr.headers),rr.headers)
```

#### 高级用法
##### 文件上传
测试代码：
```
import requests

files = {'file':open('favicon.ico','rb')}
r = requests.post('http://httpbin.org/post',files=files)
print(r.text)
```

##### Cookies
测试代码：
```
import requests

r = requests.get('http://www.baidu.com')
print(r.cookies)
for key,value in r.cookies.items():
    print(key + '=' + value)
```
	
##### 会话维持
利用session对象可维护一个会话
测试代码：
```
import requests

s = requests.Session()
s.get('http://httpbin.org/cookies/set/number/123456789')
r = s.get('http://httpbin.org/cookies')
print(r.text)
```

##### SSL证书认证
测试代码：
```
import requests
import urllib3

urllib3.disable_warnings()
reason = requests.get('https://www.12306.cn',verify=False)
print(reason.status_code)
```

##### 代理设置
测试代码：
```
import requests

proxies = {
    "http":"http://10.10.1.10:3128",
    "https":"http://10.10.1.10:1080"
}
requests.get("http://www.taobao.com",proxies=proxies)
```

##### 超时设置
测试代码：
```
import requests

rr = requests.get("http://www.taobao.com",timeout=1)
print(rr.status_code)
```

##### 身份认证
测试代码：
```
import requests

r = requests.get('http://localhost:5000',auth = ('username','password'))
print(r.status_code)
```

### 正则表达式
match()
从字符串的起始位置匹配正则表达式，匹配成功则返回成功结果，否则返回None;

search()
在匹配时会扫描整个字符串，返回第一个成功匹配的结果

findall()
获取匹配正则表达式的所有内容

sub()
替换或者去掉某些参数

compile()
可将正则字符串编译成正则表达式对象


#### 解析库的使用
##### 使用Xpath
表达式			描述
nodename		选取此节点的所有子节点
/				从当前节点选取直接子节点
//				从当前节点选取子孙节点
.				选取当前节点
..				选取当前节点的父节点
@				选取属性

所有节点：		//*
子节点：		//li/a
子孙节点		//ul/a
父节点			//a[@href="[link4.html]"/../@class
				//a[@href="[link4.html]"/parent::*/@class
属性匹配		//li[@class="item-0"]
文本获取		//li[@class="item-0"]/a/text()
				//li[@class="item-0"]//text()
属性获取		//li/@href
属性多值匹配	//li[@class="li"]/a/text()
				//li[contains(@class,"li")]/a/text()
多属性匹配		

测试代码：

```
import re
from lxml import etree
text = '''
<d iv>
<Ul>
<li class="item-O"><a href="linkl. html">first item</a></li>
<li class=" item-1"><a href="link2.html">second item</a></li>
<li class=" item-inactive"><a href="link3.html">third item</a></h>
<li class=" item-1 "><a href="link4.html">fourth item</a></li>
<li class ＝" item-0"><a href="links html">fifth item</a>
</ul>
</div>
'''
html = etree.HTML(text)
result= etree.tostring(html)
print(result.decode('utf-8'))
```































