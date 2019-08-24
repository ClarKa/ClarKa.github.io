---
layout: post
title: "一亩三分地自动签到python脚本"
date: 2019-08-21
tags: [爬虫]
comments: true

---

最近跳槽在一亩三地上看面经，发现很多都需要较高的积分才能看。深深后悔之前没有好好的每天签到涨积分。正好最近在学python就顺手抓包分析了一下一亩三分地的打卡机制，然后写了一个简单的python脚本部署到群晖上每天凌晨自动打卡。

目前只支持每日签到功能，后续会慢慢支持每日答题，现在正在收集题库。

用的是python3，代码如下：

```python
import requests
import re
import json

class AutoPunch:
    def __init__(self, username, pwd):
        self.username = username
        self.pwd = pwd
        self.login_url = 'https://www.1point3acres.com/bbs/member.php?mod=logging&action=login&loginsubmit=yes&infloat=yes&lssubmit=yes&inajax=1'
        self.punch_url = 'https://www.1point3acres.com/bbs/plugin.php?id=dsu_paulsign:sign&operation=qiandao&infloat=1&sign_as=1&inajax=1'
        self.session = requests.Session()
        self.headers = {
                'origin': 'https://www.1point3acres.com',
                'referer': 'https://www.1point3acres.com/bbs/',
                'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36',
            }

    def login(self):
        res = self.session.post(self.login_url,
            headers=self.headers,
            data={
                'username': self.username,
                'password': self.pwd,
                'quickforward': 'yes',
                'handlekey': 'ls'
            })

        return res

    def get_formhash(self):
        res = self.session.get('https://www.1point3acres.com/bbs/', headers=self.headers)
        result = re.search('dsu_paulsign:sign&(.{8})', res.text)

        if result:
            return result.group(1)

    # post body里的qdxq=签到心情，todaysay=今日想说的话。
    def punch(self, formhash):
        res = self.session.post(self.punch_url,
            headers=self.headers,
            data={
                'formhash': formhash,
                'qdxq': 'kx',
                'qdmode': 2,
                'todaysay': 'check in',
                'fastreply': 0
            })

        print(account['id'] + ' status: ' + str(res.status_code))


if __name__ == '__main__':
    with open('accounts.json') as json_file:
        data = json.load(json_file)

        for account in data:
            ap = AutoPunch(account['id'], account['password'])
            ap.login()
            formhash = ap.get_formhash()

            if formhash:
                ap.punch(formhash)
            else:
                print(account['id'] + ' 今日已签到')

```

脚本会从当前目录下的accounts.json来读取账号密码。

需要注意的是，一亩三分地会在前端直接吧密码加密，然后把加密后的密码发送到后端。我比较懒暂时没有去研究这个前端加密的算法，所以直接用chrome抓了登录时候的post request，然后从里面提取了加密之后密码复制粘贴到了accounts.json里面。

accounts.json的格式如下：

```json
[
    {
        "id": "用户名1",
        "password": "加密密码1"
    },
    {
        "id": "用户名2",
        "password": "加密密码2"
    }
]
```