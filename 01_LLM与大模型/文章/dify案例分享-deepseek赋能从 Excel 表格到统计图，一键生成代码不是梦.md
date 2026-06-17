---
title: dify案例分享-deepseek赋能从 Excel 表格到统计图，一键生成代码不是梦
author: 海老豹666
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486139&idx=1&sn=9942b990665c5af507f37ac32dff7d93&chksm=ced7ec8a19fa38aefb4e989b91cb0376fb1c25a06ee11578fa38268ac01b09be2f416520a243&mpshare=1&scene=24&srcid=0321ex6ezXyURgRALw8QFFdB&sharer_shareinfo=35df2ac21d9757f2a357e3c5700f3844&sharer_shareinfo_first=35df2ac21d9757f2a357e3c5700f3844#rd
---

# 

# 1.前言

基于大模型自动生成代码并实现在线运行是一种利用人工智能技术，通过自然语言描述或部分代码片段，自动生成满足需求的代码，并在特定平台上直接运行的技术。这种技术的核心在于大模型（如GPT-3、Codex等）的强大语言理解和生成能力，能够将用户的自然语言需求转化为具体的代码实现。

之前有给大家介绍过[dify案例分享-ECharts打造上证综指5日K线可视化，超详细教程来袭](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247485766&idx=1&sn=ee4fb241a8b33b28b2564d2c0ff13d1b&scene=21#wechat_redirect)

该案例展示了如何通过代码，利用用户上传 Excel 表格的方式来实现 ECharts 图表功能。不过，当时的 ECharts 代码是基于固定模板的 Excel 表格，一旦更换表格，程序就无法呈现相同效果，可扩展性欠佳。实际上，那时仅仅是为了介绍 ECharts 功能，并未实现根据任意 Excel 表格内容动态生成代码的方法。而今天，我们将带领大家达成这一功能，即基于用户表格生成动态代码，进而生成图表。

下面我们看一下工作流整体情况：

我们看下生成的效果：

点击查看可以看生成的html代码效果

大家有没有发现这个是不是有一点像DeepSeek官方网站提供的在线代码运行的效果。

下面就介绍一下这个工作流是如何制作和实现的。

# 2.工作流制作

## 开始

  开始节点这个地方我们有2个参数。其中第一个参数是用户上传的excel表格。第二个参数使用需要生成的图（柱状图、线性图表、饼图、雷达图、气泡图、双Y轴图、散点图）

第一个参数是个文件类型。

第二参数我们设定一个下拉选项。

## 文档提取器

 这个地方很好理解就是用户上传的excel表格，需要通过这个文档提取器提取表格里面的内容信息。

## 大语言模型生成html代码

这个地方比较关键，就是利用大语言模型的编程能力将表格里面的数据通过代码形式生成html对应的代码。为什么生成html因为生成的其他编程语言都需要代码运行环境，而HTML可以在浏览器打开并运行。模型的系统提示词如下：

```
# Role: Excel表格数据可视化代码生成专家  
  
## Profile  
- 专长: 将Excel表格数据转换为HTML和Chart.js可视化代码  
- 目标: 生成准确反映原始数据的交互式图表  
  
## Background  
您是一位精通数据可视化的编程专家，能够将Excel表格数据精确转换为使用HTML和Chart.js的交互式图表。  
  
## Skills  
- 精通HTML、JavaScript和Chart.js库  
- 能够处理多列Excel数据，保留原始列名和数据项名称  
- 可以生成多种类型的图表，如柱状图、线性图表、饼图、面积图、雷达图、气泡图、双Y轴图、散点图等  
  
## Responsibilities  
1. 解析用户提供的Excel表格数据，包括列名和具体数据项  
2. 根据用户指定的图表类型，生成相应的HTML和Chart.js代码  
3. 确保生成的代码完整展示所有原始数据，包括准确的列名和数据项名称  
4. 优化代码以实现最佳的可视化效果，同时保持数据的原始结构和含义  
  
## Workflow  
1. 接收用户上传的Excel表格数据和指定的图表类型  
2. 分析Excel数据结构（列数、行数、列标题、具体数据项）  
3. 选择适合的Chart.js图表类型和配置  
4. 生成完整的HTML文档，包含必要的Chart.js库引用  
5. 创建canvas元素和对应的JavaScript代码  
6. 使用原始Excel数据填充图表配置  
7. 在HTML中添加一个表格元素，完整展示原始Excel数据内容  
8. 确保所有列名和数据项名称在图表和表格中得到准确反映  
  
## Guidelines  
1. 只输出可直接运行的HTML代码，不包括其他说明或注释  
2. 使用最新版本的Chart.js库（CDN链接）  
3. 将Excel的列名用作图表的标签或图例  
4. 在数据集中使用Excel的原始数据项名称  
5. 根据数据特性自动选择合适的颜色方案  
6. 为图表添加适当的标题，反映数据内容  
7. 确保生成的图表具有响应式设计，适应不同屏幕大小  
  
## Output Format  
```html  
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>Excel数据可视化</title>  
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>  
    <style>  
        table {  
            width: 100%;  
            border-collapse: collapse;  
            margin-bottom: 20px;  
        }  
        table, th, td {  
            border: 1px solid #ddd;  
        }  
        th, td {  
            padding: 8px;  
            text-align: left;  
        }  
        th {  
            background-color: #f2f2f2;  
        }  
        .chart-container {  
            max-width: 800px;  
            margin: auto;  
        }  
    </style>  
</head>  
<body>  
    <h1 style="text-align: center;">Excel数据可视化</h1>  
  
    <!-- 原始表格 -->  
    <h2 style="text-align: center;">原始数据表</h2>  
    <table>  
        <thead>  
            <tr>  
                <!-- Excel列名将插入此处 -->  
            </tr>  
        </thead>  
        <tbody>  
            <!-- Excel数据行将插入此处 -->  
        </tbody>  
    </table>  
  
    <!-- 图表容器 -->  
    <div class="chart-container">  
        <canvas id="myChart"></canvas>  
    </div>  
  
    <script>  
        // 这里是Chart.js配置和数据，使用Excel的原始列名和数据项名称  
    </script>  
</body>  
</html>
```

用户提示词这块我们也需要对大模型有一定约束，提示词内容如下

```
{{#sys.query#}}请根据用户上传的excle表格{{#1741751683774.text#}}将表格以html和 chart.js生成{{#1741751677458.tpye#}}要求只生成html和chart.js 等在浏览器直接运行的代码，其他解释内容不要输出。
```

   通过上面的额方式让大模型根据表格数据动态生成对应的HTML代码，这样利用模型的代码生成能力就完成了各种表格代码自动编写工作了。

## 代码处理生成html调用

上面的流程中大模型生成了HTML 代码它需要把它转化成文件输出，这里我们利用了后端服务代码的能力。这里我们有2个地方需要讲解。第一个地方是我们编写了服务端代码，这个是为了在服务端生成HTML代码，并上传到一个第三方公网访问的地址信息（我这里用了腾讯云COS存储）。第二个地方是获取这个生成的html代码链接地址返回给dify以方便后续流程使用。

### 1.服务端代码

这个服务端代码主要作用就是使用fastapi提供一个http请求接口，后端通过python代码生成html并上传腾讯COS

makehtmlapi.py

```
from fastapi import FastAPI, HTTPException,Depends, Header  
from pydantic import BaseModel  
import logging  
import time  
import uvicorn  
import configparser  
import os  
import json  
import datetime  
import random  
from qcloud_cos import CosConfig  
from qcloud_cos import CosS3Client  
  
app = FastAPI()  
  
# 读取配置文件中的API密钥  
config = configparser.ConfigParser()  
config.read('config.ini', encoding='utf-8')  
  
# Tencent Cloud COS configuration  
region = config.get('common', 'region')  
secret_id = config.get('common', 'secret_id')  
secret_key = config.get('common', 'secret_key')  
bucket = config.get('common', 'bucket')  
  
# 设置输出路径  
output_path = config.get('html', 'output_path', fallback='html_output')  
  
# 确保输出目录存在  
os.makedirs(output_path, exist_ok=True)  
  
# 设置日志  
logging.basicConfig(level=logging.INFO)  
logger = logging.getLogger(__name__)  
  
class HTMLRequest(BaseModel):  
    html_content: str  
    filename: str = None  # 可选参数，如果不提供则自动生成  
  
def verify_auth_token(authorization: str = Header(None)):  
    """验证 Authorization Header 中的 Bearer Token"""  
    if not authorization:  
        raise HTTPException(status_code=401, detail="Missing Authorization Header")  
      
    scheme, _, token = authorization.partition(" ")  
    if scheme.lower() != "bearer":  
        raise HTTPException(status_code=401, detail="Invalid Authorization Scheme")  
      
    # 从配置文件读取有效token列表  
    valid_tokens = json.loads(config.get('auth', 'valid_tokens'))  
    if token not in valid_tokens:  
        raise HTTPException(status_code=403, detail="Invalid or Expired Token")  
      
    return token  
def generate_timestamp_filename(extension='html'):  
    timestamp = datetime.datetime.now().strftime("%Y%m%d%H%M%S")  
    random_number = random.randint(1000, 9999)  
    filename = f"{timestamp}_{random_number}.{extension}"  
    return filename  
  
def save_html_file(html_content, filename=None, output_dir=None):  
    # 如果没有提供文件名，则生成一个  
    if not filename:  
        filename = generate_timestamp_filename()  
      
    # 如果没有提供输出目录，则使用默认目录  
    if not output_dir:  
        output_dir = output_path  
      
    # 确保输出目录存在  
    os.makedirs(output_dir, exist_ok=True)  
      
    # 组合完整的输出路径  
    file_path = os.path.join(output_dir, filename)  
      
    # 写入HTML内容  
    with open(file_path, 'w', encoding='utf-8') as file:  
        file.write(html_content)  
      
    # 返回文件名和输出路径  
    return filename, file_path  
  
def upload_cos(region, secret_id, secret_key, bucket, file_name, base_path):  
    config = CosConfig(  
        Region=region,  
        SecretId=secret_id,  
        SecretKey=secret_key  
    )  
    client = CosS3Client(config)  
    file_path = os.path.join(base_path, file_name)  
    response = client.upload_file(  
        Bucket=bucket,  
        LocalFilePath=file_path,  
        Key=file_name,  
        PartSize=10,  
        MAXThread=10,  
        EnableMD5=False  
    )  
    if response['ETag']:  
        url = f"https://{bucket}.cos.{region}.myqcloud.com/{file_name}"  
        return url  
    else:  
        return None  
  
@app.post("/generate-html/")  
async def generate_html(request: HTMLRequest,auth_token: str = Depends(verify_auth_token)):  
    try:  
        logger.info("开始处理HTML生成请求")  
        start_time = time.time()  
          
        # 保存HTML文件  
        filename, file_path = save_html_file(request.html_content, request.filename)  
          
        # 上传到腾讯云COS  
        html_url = upload_cos(region, secret_id, secret_key, bucket, filename, output_path)  
          
        elapsed_time = time.time() - start_time  
        logger.info(f"HTML生成和上传完成，耗时 {elapsed_time:.2f} 秒，返回 URL: {html_url}")  
          
        if html_url:  
            return {  
                "success": True,  
                "html_url": html_url,  
                "filename": filename  
            }  
        else:  
            raise HTTPException(status_code=500, detail="上传HTML文件到COS失败")  
    except Exception as e:  
        logger.error(f"处理HTML生成请求时发生错误: {str(e)}")  
        raise HTTPException(status_code=500, detail=str(e))  
  
if __name__ == "__main__":  
    uvicorn.run(app, host="0.0.0.0", port=8088)
```

代码里面用到config.ini配置文件

```
[html]  
output_path = E:\\work\\code\\2024pythontest\\makehtml\\html_output  
  
[common]  
region = xxx         腾讯云OSS存储Region  
secret_id = xxx      腾讯云OSS存储SecretId  
secret_key = xxx     腾讯云OSS存储SecretKey  
bucket = xxx         腾讯云OSS存储bucket  
  
[auth]  
valid_tokens = ["sk-zhouhui1xxx", "zhouhui112xxx"]
```

上面代码需要再服务器或者本地运行起来。对外提供 8088端口（端口你也可以自己改）

上面步骤服务端就启动好了。

### 1.客户端代码

接下来我们在dify 代码执行中添加。

这里我们有4个参考。分别是1.json\_html   大语言模型生成的html代码。2 apiurl 就是上面服务端代码的请求地址。3 apikey 服务端代码请求APIkey. 4 strtype 就是开始节点中的第二个参数

其中 apiurl 和apikey 我们这里用环境变量方式来实现。

如果你本地电脑 URL 可以是 192.168.XX.XX 或则127.0.0.1 如果是服务器 可以是局域网IP 也是可以公网IP 

客户端APIKEY  和服务端APIKEY 保持一致。服务端APIKEY  就是config.ini 对应的

我上面服务端数组定义2个值，客户端有一个值在这个数组就可以了。

客户端代码如下：

```
import json  
import re  
import time  
import requests  
  
def main(json_html: str, apikey: str,apiurl: str,strtype: str) -> dict:  
    try:  
        # 去除输入字符串中的 ```html 和 ``` 标记  
        html_content = re.sub(r'^```html\s*|\s*```$', '', json_html, flags=re.DOTALL).strip()  
          
        # 生成时间戳，确保文件名唯一  
        timestamp = int(time.time())  
        filename = f"{strtype}_{timestamp}.html"  
          
        # API端点（假设本地运行）  
        url = f"{apiurl}"  
          
        # 请求数据  
        payload = {  
            "html_content": html_content,  
            "filename": filename  # 使用传入的文件名  
        }  
          
        # 设置请求头（包含认证token）  
        headers = {  
            "Authorization": f"Bearer {apikey}",  # 替换为实际的认证token  
            "Content-Type": "application/json"  
        }  
          
        try:  
            # 发送POST请求  
            response = requests.post(url, json=payload, headers=headers)  
              
            # 检查响应状态  
            if response.status_code == 200:  
                result = response.json()  
                html_url = result.get("html_url", "")  
                generated_filename = result.get("filename", "")  
                  
                # 返回结果  
                return {  
                    "html_url": html_url,  
                    "filename": generated_filename,  
                    "markdown_result":  f"[点击查看]({html_url})"  
                }  
            else:  
                raise Exception(f"HTTP Error: {response.status_code}, Message: {response.text}")  
          
        except requests.exceptions.RequestException as e:  
            raise Exception(f"Request failed: {str(e)}")  
      
    except Exception as e:  
        return {  
            "error": f"Error: {str(e)}"  
        }
```

返回3个值html\_url、filename、markdown\_result

以上就完成了服务端代码生成和客户端代码调用的功能了。

## 直接回复

这里我们定义2个输出。第一个是LLM大语言模式生成的html代码，这样我们很方便看到大模型生成的代码，也可以帮我把生成代码保存 html帮助我们测试和验证。第二个参数就是上面代码返回的URL链接（这里相当于模拟deepseek 官方网站提供的在线运行）。

以上我们就完成了工作流的配置。

# 3.验证及测试

下面我们对该工作流进行测试一下。我们的测试数据如下：

| 年份 | 总费用 | AA部门 | BB部门 | CC部门 | DD部门 |
| --- | --- | --- | --- | --- | --- |
| 2025 | 59.69 | 41.28 | 9.78 | 2.17 | 6.48 |
| 2026 | 56.07 | 38.78 | 9.18 | 2.04 | 6.1 |
| 2027 | 52.46 | 36.28 | 8.59 | 1.91 | 5.71 |
| 2028 | 48.84 | 33.78 | 8 | 1.78 | 5.32 |
| 2029 | 45.22 | 31.27 | 7.41 | 1.65 | 4.93 |
| 2030 | 41.6 | 28.76 | 6.81 | 1.51 | 4.53 |

把上面表格的数据上传工作流。

测试效果如下。

接下来我们可以把工作流分享给小伙伴了， 分享工作流地址：

http://dify.duckcloud.fun/chat/xmgyFaapZCHc5LbQ

以上我们就完成了工作流的测试。

相关资料和文档可以看我开源的项目 https://github.com/wwwzhouhui/dify-for-dsl

# 4.总结

今天主要带大家基于大模型实现了一个基于用户上传 Excel 表格动态生成代码统计图的工作流。详细介绍了整个工作流的实现步骤，工作流包含开始节点、文档提取器、大语言模型生成 html 代码、代码处理生成 html 调用等部分。本次工作流涉及到 Dify、FastAPI 等工具的使用，代码中运用了 Python 的多个库，如`fastapi`、`qcloud_cos`等，还涉及到对大模型提示词的配置。虽然内容有一定难度，但实现了根据任意 Excel 表格内容动态生成代码和图表的功能，具有较好的可扩展性。感兴趣的小伙伴可以动手尝试，今天的分享就到这里结束了，我们下一篇文章见。

[dify案例分享-股票分析系统A股、港股、美股、ETF、LOF](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486113&idx=1&sn=c463a790dd688a684b50b0cb0001e917&scene=21#wechat_redirect)

[dify案例分享-API文档生成接口代码](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486087&idx=1&sn=016dc5756d3a1c86653c5334a09f5d2b&scene=21#wechat_redirect)

[全网独家！即梦AI绘画 + 飞书 + 企业微信整合工作流大揭秘](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486050&idx=1&sn=bb27fe1b76e7bfab6eb79c14b869301d&scene=21#wechat_redirect)

[dify案例分享-知识库检索整合Ragflow](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486016&idx=1&sn=47ca4c1b15524eba72829fd88660d598&scene=21#wechat_redirect)

[cursor案例分享-前后端分离任务清单制作](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486015&idx=1&sn=ff1bde8a67aa62cc0f6152ff68e4c5f5&scene=21#wechat_redirect)