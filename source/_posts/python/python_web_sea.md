---
title: 在sae中运行web.py应用
date: 2019-02-24 23:24:30
tags: cloud
categories: python
---

sae 是新浪推出的PaaS业务，可以提供免运维的容器服务，官方网站（ https://www.sinacloud.com/ ）
假设您已经在本地开发好了web.py 应用，您可以通过github客户端上传代码到sae中新建的python应用中。
只要web.py应用包含文件index.wsgi，新浪云就会加载我们的应用。

<!--more-->
```index.wsgi
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# file code.py
import sys
import os
import web

urls = ("/", "Index",
        )
app = web.application(urls, globals())
application = app.wsgifunc()
web.config.debug = True

class Index:
    def __init__(self):
        pass

    def GET( self ):
        if session.login == 1:
            data = index1(session.user)
            return render_template('index.html', name=data )
        else:
            raise web.seeother('/login')

if __name__ == "__main__":
    app.run()
```

其他内容，可以自行参考sae官方文档。https://www.sinacloud.com/doc/sae/python.html