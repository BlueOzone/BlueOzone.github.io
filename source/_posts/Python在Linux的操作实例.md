---
title: Python实例
date: 2021-01-13 15:51:27
categories: Python
tags: Python
---


### Python在Linux上的操作实例
<!--more-->

```
import logging
import re
import subprocess

# 日志级别默认是WARNING
#logging.basicConfig(level=logging.INFO,filename='run.log',
#                    format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')
# 开始使用log功能
logger = logging.getLogger(__name__)
logger.setLevel(level=logging.INFO)
handler = logging.FileHandler("chartslog.txt")
handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

console = logging.StreamHandler()
console.setLevel(logging.INFO)

# 日志是否保存在日志文件里
logger.addHandler(handler)
#日志是否在前台显示
logger.addHandler(console)


#根据监听端口获取进程号,执行命令：netstat -lptn |grep port，获取成功返回进程号，获取失败返回0
def getPid(port):
    result = subprocess.check_output("netstat -lptn |grep " + port, shell=True).decode('utf-8')
    if len(result) != 0:
        processid = re.match('.*?((\d+)(\D+)?)$', result).group(2)
        logger.info("pid为：" + processid)
        return processid
    else:
        logger.error("获取进程失败，请检查该端口：" + port + "是否被使用！netstat -lptn |grep命令结果为：" + result)
        return 0

#根据进程号获取进程所在路径,执行命令：pwdx pid，获取成功返回路径，获取失败返回0
def getPath(pid):
    result = subprocess.check_output("pwdx " + pid, shell=True).decode('utf-8')
    if len(result) != 0:
        processpath = re.match('\d+: (.*)', result).group(1)
        logger.info("进程所在路径为：" + processpath)
        return processpath
    else:
        logger.error("获取路径失败，请检查进程号：" + pid + "是否存在！pwdx命令结果为：" + result)
        return 0

#根据进程所在路径进入到进程所在位置,执行命令：cd path、pwd，获取成功无返回，获取失败返回0
def goPath(path):
    result = subprocess.check_output("cd " + path, shell=True).decode('utf-8')
    if len(result) == 0:
        # processpath = subprocess.check_output("pwd", shell=True).decode('utf-8')
        # logger.info("当前所在路径为：" + processpath)
        pass
    else:
        logger.error("进入路径失败，请检查路径：" + path + "是否存在！cd命令结果为：" + result)
        return 0

#查询kafka积压
def kafkaCheck(port):
    kfkpid = getPid(str(port))
    kfkpath = getPath(kfkpid)

    kfkoverstock = "/bin/kafka-consumer-offset-checker.sh --topic antispider --zookeeper localhost:2181 --group groupid"

    if kfkpid != 0 and kfkpath != 0:
        result = subprocess.Popen(kfkpath + kfkoverstock, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        logger.info("kafka结果为:" + '\n' + str(result.communicate()))


#压缩生成的日志文件
def setZip():
    result = subprocess.check_output("pwd", shell=True).decode('utf-8')
    if len(result) != 0:

        logger.info("当前路径为：" + result)
        try:
            zipcmd = subprocess.check_output("zip -r checkResult.zip " + result, shell=True).decode('utf-8')
        except Exception as e:
            logger.error(e)
        finally:
            pass
    else:
        logger.error("获取路径失败!pwd命令结果为：" + result)


#该函数可以执行一些状态检测的命令，并输出结果
def systemstatus():
    # 参考文章：https://blog.csdn.net/baidu_36943075/article/details/105681683
    # subprocess.Popen虽然既可以判断执行是否成功，还可以获取执行结果，但是该方式返回的结果不够直观，不能记录在终端下执行命令的效果
    # subprocess.call方式仅返回执行命令成功或失败的结果，执行失败不需要特殊处理，命令执行失败会直接报错；结果为0则表示执行成功，为其他值则表示执行不成功
    # subprocess.check_output方式执行失败不需要特殊处理，命令执行失败会直接报错，该方式返回执行结果，但是结果返回的是一个str字符串（不论有多少行），并且返回的结果需要转换编码

    # p = subprocess.Popen('df -h',shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
    # result = subprocess.call("df -h",shell=True,stdout=subprocess.PIPE)

    command = {"磁盘空间":"df -h","CPU与内存使用情况":"vmstat","端口开放情况":"netstat -lptn"}

    command2 = {"防火墙是否开放":"firewall-cmd --state","防火墙开放列表":"firewall-cmd --list-all"}

    for key,value in command.items():
        result = subprocess.check_output(value, shell=True).decode('utf-8')
        logger.info(str(key) + '\n' + result)

    for key,value in command2.items():
        result = subprocess.Popen(value, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        logger.info(str(key) + '\n' + str(result.communicate()))


#该函数会通过ip a命令输出ip地址；且可以通过ping的方式去检查跟某些域名的连通性
def networkstatus():
    #配置ping检测的域名
    cloudurl = ["www.xx.com","www.xx.com"]
    command = {"ip地址": "ip a"}

    for key,value in command.items():
        result = subprocess.check_output(value, shell=True).decode('utf-8')
        logger.info(str(key) + '\n' + result)

    try:
        for url in cloudurl:
            result = subprocess.check_output("ping " + url + " -c 5", shell=True).decode('utf-8')
            logger.info("ping " + url + '\n' + result)

    except Exception as e:
        logger.error("连接域名异常！")
        logger.error(e)
    finally:
        pass


if __name__ == '__main__':
    systemstatus()
    networkstatus()
    kafkaCheck("port")

    #生成zip文件
    setZip()

```



### Python京东爬虫实例

爬取京东指定页面的电脑价格、名称好评率等。

```
import requests
import csv
from requests.exceptions import RequestException
from bs4 import BeautifulSoup


def download(url, headers, num_retries=3):
    print("download", url)
    try:
        response = requests.get(url, headers=headers)
        print(response.status_code)
        if response.status_code == 200:
            return response.content
        return None

    except RequestException as e:
        print(e.response)
        html = ""
        if hasattr(e.response, 'status_code'):
            code = e.response.status_code
            print('error code', code)
            if num_retries > 0 and 500 <= code < 600:
                html = download(url, headers, num_retries - 1)
        else:
            code = None
    return html


def find_Computer(url, headers):
    r = download(url, headers=headers)
    page = BeautifulSoup(r, "lxml")
    all_items = page.find_all('li', attrs={'class' : 'gl-item'})

    with open("computer.csv", 'w', newline='') as f:
        writer = csv.writer(f)
        fields = ('ID', '名称', '价格', '评论数', '好评率')
        writer.writerow(fields)

        for all in all_items:
            # 取每台电脑的ID
            Computer_id = all["data-sku"]
            print(f"电脑ID为：{Computer_id}")

            # 取每台电脑的名称
            Computer_name = all.find('div', attrs={'class':'p-name p-name-type-2'}).find('em').text
            print(f"电脑的名称为：{Computer_name}")

            # 取每台电脑的价格
            Computer_price = all.find('div', attrs={'class':'p-price'}).find('i').text
            print(f"电脑的价格为：{Computer_price}元")

            # 取每台电脑的Json数据(包含 评价等等信息)
            Comment = f"https://club.jd.com/comment/productCommentSummaries.action?referenceIds={Computer_id}"
            comment_count, good_rate = get_json(Comment)
            print('评价人数：', comment_count)
            print('好评率：', good_rate)

            row = []
            row.append(Computer_id)
            row.append(Computer_name)
            row.append(str(Computer_price) + "元")
            row.append(comment_count)
            row.append(str(good_rate) + "%")
            writer.writerow(row)


def get_json(url):
    data = requests.get(url).json()
    result = data['CommentsCount']
    for i in result:
        return i["CommentCountStr"], i["GoodRateShow"]


def main():
    headers = {
        'User-agent': "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36",
        "referer": "https://passport.jd.com"
    }
    URL = "https://search.jd.com/Search?keyword=%E7%94%B5%E8%84%91&enc=utf-8&suggest=1.def.0.V01--12s0,20s0,38s0&wq=diann&pvid=efce27b0971d40aa8ec7c6ef04c4f8e9"

    find_Computer(URL, headers=headers)

if __name__ == '__main__':
    main()

```

### Python爬取猫眼top100实例

结果会生成result.txt文件。

```
import requests
import re
import json
import time

def get_one_page(url):
    headers = {
        'User - Agent': 'Mozilla / 5.0(Windows NT 10.0; WOW64) AppleWebKit / 537.36(KHTML, like Gecko) Chrome / 75.0.3770.100 Safari / 537.36'
    }

    result = requests.get(url,headers)
    if result.status_code == 200:
        return result.text
    else:
        return None

def parse_one_page(html):
    patt =re.compile('<dd>.*?board-index.*?>(\d+).*?data-src="(.*?)".*?'
                     'name.*?a.*?>(.*?)</a>.*?star">(.*?)</p>.*?'
                     'releasetime">(.*?)</p>.*?integer">(.*?)</i>.*?'
                     'fraction">(.*?)</i>.*?</dd>',re.S)
    hh = re.findall(patt,html)
    for item in hh:
        yield {
            'index':item[0],
            'image':item[1],
            'title':item[2].strip(),
            'actor':item[3].strip()[3:] if len(item[3]) > 3 else '',
            'time':item[4].strip()[5:] if len(item[4]) > 5 else '',
            'score':item[5].strip()+item[6].strip()
        }

def write_to_file(content):
    with open('result.txt','a',encoding='utf-8') as f:
        print(type(json.dumps(content)))
        f.write(json.dumps(content,ensure_ascii=False)+'\n')

def main(offset):
    url = 'https://maoyan.com/board/4?offset='+str(offset)
    html = get_one_page(url)
    for item in parse_one_page(html):
        write_to_file(item)

if __name__ == '__main__':
    for i in range(10):
        main(offset=i*10)
        time.sleep(1)
```


### Python爬虫绕过selenium检测实例

```
from selenium import webdriver

#绕过检测selenium机制
options = webdriver.ChromeOptions()
options.add_experimental_option("excludeSwitches", ["enable-automation"])
options.add_experimental_option('useAutomationExtension', False)
driver = webdriver.Chrome(options=options)
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
  "source": """
    Object.defineProperty(navigator, 'webdriver', {
      get: () => undefined
    })
  """
})


#打开搜狗微信搜索页面，模拟点击
driver.get('https://weixin.sogou.com/')

driver.find_element_by_xpath("//*[@id=\"query\"]").send_keys("公路养护")

driver.find_element_by_xpath("//*[@id=\"searchForm\"]/div/input[3]").click()

print(driver.find_element_by_xpath("//*[@id=\"sogou_vr_11002601_title_0\"]").get_attribute("href"))

driver.find_element_by_xpath("//*[@id=\"sogou_vr_11002601_title_0\"]").click()

# time.sleep(15)
# print(driver.current_url)

print(driver.page_source)

# print(driver.find_element_by_xpath("//*[@id=\"js_content\"]/section[1]/section").text)
# print(driver.find_element_by_xpath("//*[@id=\"js_content\"]/section[1]/section").text)

```

