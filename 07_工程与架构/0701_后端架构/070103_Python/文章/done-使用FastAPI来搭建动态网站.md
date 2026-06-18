> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: 使用FastAPI来搭建动态网站
author: 吃着火锅K着歌
date:
url: https://mp.weixin.qq.com/s?__biz=MzIyMTcyNzQ5Mg==&mid=2247486967&idx=1&sn=3465691cbb3fb4109615d50149a64814&chksm=e9997071652bfd83997e08790c3b734de7e6e170a3826bc4ba434ffb238ce2e48549c1d9215c&mpshare=1&scene=24&srcid=1225isxsJaBlGSv5yzbx8xto&sharer_shareinfo=ddfc5ad13229e7f991c745b52238f4d0&sharer_shareinfo_first=ddfc5ad13229e7f991c745b52238f4d0#rd
---

FastAPI 不仅能提供API接口，还能像Django、Flask一样搭建网站，今天我们来使用FastAPI和Jinja2模板引擎，结合CSS 和JS来实现一个简单的网站。

虚拟环境和包安装前面的一些文章中有过说明了，这里不再啰嗦。

FastAPI默认返回JSON数据，但是它支持自定义响应类型，用HTMLResponse就能直接返回HTML内容，让浏览器解析，首先创建main.py文件，编写一个返回HTML的接口：

```
from fastapi import FastAPIfrom fastapi.responses import HTMLResponseapp = FastAPI()@app.get("/", response_class=HTMLResponse)def home():    html_content = """    <!DOCTYPE html>    <html lang="en">    <head>        <meta charset="UTF-8">        <title>Home</title>    </head>    <body>        <h1>欢迎使用FastAPI！</h1>    </body>    </html>    """    return html_content
```

运行后，访问127.0.0.1:8000 就能看到效果。接下来我们使用Jinja2模板实现动态内容。我们做一个随机颜色生成器，用Jinja2将Python生成的颜色码动态嵌入HTML：

```
import randomfrom string import hexdigitsfrom fastapi import FastAPIfrom fastapi.responses import HTMLResponsefrom jinja2 import Templateapp = FastAPI()@app.get("/", response_class=HTMLResponse)def home():    # 生成6位随机十六进制字符（颜色码）    hex_chars = "".join(random.choices(hexdigits.lower(), k=6))    hex_color = f"#{hex_chars}"  # 格式：#ffffff
    # Jinja2模板（嵌入{{ color }}变量占位符）    html_template = """    <!DOCTYPE html>    <html lang="en">    <head>        <meta charset="UTF-8">        <title>随机颜色生成器</title>        <style>            body {                height: 100vh;                display: flex;                justify-content: center;                align-items: center;                background-color: {{ color }};  <!-- 动态注入颜色 -->                color: white;                font-size: 120px;                font-family: monospace;            }        </style>    </head>    <body>        <div id="color-code">{{ color }}</div>  <!-- 显示颜色码 -->    </body>    </html>    """    # 渲染模板：将color变量注入模板    html_content = Template(html_template)    website = html_content.render(color=hex_color)    return website
```

接下来我们搭建要给完整项目结构：

```
your_project/├── main.py          # Python代码├── templates/       # Jinja2模板文件│   ├── base.html    # 基础模板（父模板）│   └── color.html   # 颜色生成页面（子模板）└── static/          # 静态文件（CSS/JS）    ├── style.css    # 样式文件    └── script.js    # 交互脚本
```

首先是基础模板templates/base.html

```
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>随机颜色生成器</title>    <link href="/static/style.css" rel="stylesheet">  <!-- 引入CSS --></head><body>    {% block content %}{% endblock content %}  <!-- 动态内容占位符 -->    <script src="/static/script.js"></script>  <!-- 引入JS --></body></html>
```

创建子模板templates/color.html

```
{% extends "base.html" %}  <!-- 继承基础模板 -->{% block content %}    <style>        body {            background-color: {{ color }};  <!-- 动态颜色 -->        }    </style>    <div id="color-code">{{ color }}</div>  <!-- 显示颜色码 -->    <button id="copy-button">复制颜色码</button>  <!-- 复制按钮 -->{% endblock %}
```

CSS样式 static/style.css

```
body {    height: 100vh;    display: flex;    justify-content: center;    align-items: center;    color: white;    font-size: 120px;    font-family: monospace;    flex-direction: column;  /* 垂直排列 */    gap: 20px;  /* 元素间距 */}#copy-button {    padding: 10px 30px;    font-size: 24px;    border: none;    border-radius: 8px;    cursor: pointer;    background-color: rgba(255,255,255,0.3);    color: white;}#copy-button:hover {    background-color: rgba(255,255,255,0.5);}
```

编写js交互 static/scripts.js

```
// 给复制按钮添加点击事件document.querySelector('#copy-button').addEventListener('click', () => {  const colorCode = document.querySelector('#color-code').textContent;  navigator.clipboard.writeText(colorCode);  // 复制到剪贴板  alert(`已复制：${colorCode}`);  // 提示复制成功});
```

改造FastAPI主程序main.py

```
import randomfrom string import hexdigitsfrom fastapi import FastAPI, Requestfrom fastapi.responses import HTMLResponsefrom fastapi.staticfiles import StaticFiles  # 用于服务静态文件from fastapi.templating import Jinja2Templates  # 用于加载模板app = FastAPI()# 挂载静态文件：访问/static/* 会指向static文件夹app.mount("/static", StaticFiles(directory="static"), name="static")# 加载模板：从templates文件夹读取模板文件templates = Jinja2Templates(directory="templates")@app.get("/", response_class=HTMLResponse)def home(request: Request):  # 必须传入Request对象    # 生成随机颜色码    hex_chars = "".join(random.choices(hexdigits.lower(), k=6))    hex_color = f"#{hex_chars}"
    # 传递给模板的参数（上下文）    context = {        "request": request,  # FastAPI要求必须传递        "color": hex_color    }
    # 渲染模板：指定模板文件、传递上下文    return templates.TemplateResponse(        name="color.html",  # 子模板文件名        context=context    )
```

到此，不妨启动一下服务器，访问127.0.0.1:8000 看看效果吧。