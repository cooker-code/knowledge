---
title: LiteLLM：一个统一管理大模型并发调用、负载均衡、资源监控的开源神器附教程
author: 忧郁的茄子
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg2MzgyMTIxOQ==&mid=2247483968&idx=1&sn=4a6a0517a966e863e1d887334d1db5ae&chksm=cf88bda0ebc85517502b0b1c997f454a80d18d8d9de958172a8ff6861dfc94145085b621a3fe&mpshare=1&scene=24&srcid=0420yZEz8xbEpk0znEHdvS6G&sharer_shareinfo=e6ff81f738a47605c2df42cb9eabac93&sharer_shareinfo_first=e6ff81f738a47605c2df42cb9eabac93#rd
---

我们在日常开发中经常会遇到这样的场景：

* 项目A用Claude，项目B用Qwen，测试环境还跑着本地Ollama
* 电脑里配置着十几个API Key，切换模型时总要改代码
* 费用开始失控，不知道钱花在哪
* 受大模型服务供应商调用量峰值影响，模型响应时延高

当大模型用得越来越多，这种混乱几乎不可避免。那么，有没有一个工具能统一管理所有大模型？

答案是：**LiteLLM**。

---

## 一、LiteLLM是什么？

LiteLLM是一个开源的大模型网关和SDK，它允许我们通过单一、兼容OpenAI格式的API接口，调用100+个大语言模型。

**核心理念**：统一调用所有接入的大模型。

简单来说，LiteLLM本身不是模型，而是一个**统一的入口+管理中枢**。它对外暴露OpenAI兼容的API，对内可以接入各种不同来源的大模型。

同时，LiteLLM可以通过对应参数的改动来设置模型的并发调用数量（小于等于对应模型供应商允许的RPM(每分钟最大调用次数)、TPM（每分钟最大消耗Token数）），负载均衡，故障转移（某一个模型调用错误，转而调用指定的备用模型来完成相应的调用任务）

附上传送门：

* GitHub仓库：github.com/BerriAI/litellm
* PyPI仓库：pypi.org/project/litellm
* 官方文档：docs.litellm.ai

---

## 二、LiteLLM解决了什么问题？

### 2.1 API差异问题

不同大模型提供商有各自的API格式、认证方式和响应结构。过去，如果你想同时使用OpenAI和Anthropic的模型，需要分别写两套集成代码。

LiteLLM将这些差异统一为OpenAI格式，你只需要学会一种调用方式。

### 2.2 切换成本高

业务代码与特定模型强绑定，换模型意味着改代码、重新测试、重新部署。

使用LiteLLM后，切换模型只需改一个模型参数名称。

### 2.3 费用失控

多模型、多项目、多团队成员同时使用，费用难以追踪和分配。

LiteLLM提供集中式成本追踪和预算控制。

### 2.4 可靠性问题

单个模型服务可能出问题，缺乏自动故障转移机制。

LiteLLM支持重试和故障转移，当一个提供商出问题时，自动切换到备选模型。

### 2.5 可用性问题

LiteLLM支持模型并发调用以及负载均衡，可以很好地集成不同的大模型供应商的API资源来支持项目。

---

## 三、快速部署

### 3.1 下载项目

```
git clone https://github.com/BerriAI/litellm.git && cd litellm
```

### 3.2 Docker部署

step 1. 打开`docker-compose.yml`文件，并替换为一下内容

```
services:  
  litellm:  
    # build:  
    #   context: .  
    #   args:  
    #     target: runtime  
    # image: docker.litellm.ai/berriai/litellm:main-stable  
    image:swr.cn-north-4.myhuaweicloud.com/ddn-k8s/ghcr.io/berriai/litellm:v1.82.3-stable  
    #########################################  
    ## Uncomment these lines to start proxy with a config.yaml file ##  
    volumes:  
     -./config.yaml:/app/config.yaml  
    command:  
     -"--config=/app/config.yaml"  
    ##############################################  
    ports:  
      -"4000:4000"# Map the container port to the host, change the host port if necessary  
    environment:  
      DATABASE_URL:"postgresql://llmproxy:dbpassword9090@db:5432/litellm"  
      STORE_MODEL_IN_DB:"True"# allows adding models to proxy via UI  
    env_file:  
      -.env# Load local .env file  
    depends_on:  
      -db# Indicates that this service depends on the 'db' service, ensuring 'db' starts first  
    healthcheck:# Defines the health check configuration for the container  
      test:  
        -CMD-SHELL  
        -python3-c"import urllib.request; urllib.request.urlopen('http://localhost:4000/health/liveliness')"# Command to execute for health check  
      interval:30s# Perform health check every 30 seconds  
      timeout:10s   # Health check command times out after 10 seconds  
      retries:3     # Retry up to 3 times if health check fails  
      start_period:40s# Wait 40 seconds after container start before beginning health checks  
  
db:  
    # image: postgres:16  
    image:swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/postgres:16.9-alpine  
    restart:always  
    container_name:litellm_db  
    environment:  
      POSTGRES_DB:litellm  
      POSTGRES_USER:llmproxy  
      POSTGRES_PASSWORD:dbpassword9090  
    ports:  
      -"5432:5432"  
    volumes:  
      -postgres_data:/var/lib/postgresql/data# Persists Postgres data across container restarts  
    healthcheck:  
      test:["CMD-SHELL","pg_isready -d litellm -U llmproxy"]  
      interval:1s  
      timeout:5s  
      retries:10  
  
prometheus:  
    # image: prom/prometheus  
    image:swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/prom/prometheus:v3.9.1  
    volumes:  
      -prometheus_data:/prometheus  
      -./prometheus.yml:/etc/prometheus/prometheus.yml  
    ports:  
      -"9090:9090"  
    command:  
      -"--config.file=/etc/prometheus/prometheus.yml"  
      -"--storage.tsdb.path=/prometheus"  
      -"--storage.tsdb.retention.time=15d"  
    restart:always  
  
volumes:  
prometheus_data:  
    driver:local  
postgres_data:  
    name:litellm_postgres_data# Named volume for Postgres data persistence
```

step 2. 复制`.env.example`文件为`.env`文件，然后将自己拥有的API\_KEY，填到该文件中。

step 3. 新建一个config.yaml文件，填入配置信息，以下是模板：

```
general_settings:  
    store_model_in_db:true  
    store_prompts_in_spend_logs:true  
    master_key:sk-1234  
      
  
model_list:  
-model_name:glm-4.7  
    litellm_params:  
        model:openai/glm-4.7  
        api_key:"os.environ/ZHIPUAI_API_KEY"  
        api_base:https://open.bigmodel.cn/api/paas/v4/  
  
-model_name:deepseek-v3  
    litellm_params:  
        model:deepseek/deepseek-chat  
        api_key:"os.environ/DEEPSEEK_API_KEY"  
        api_base:https://api.deepseek.com  
        rpm:5      # [OPTIONAL] Rate limit for this deployment: in requests per minute (rpm)  
  
-model_name:deepseek-v3-1  
    litellm_params:  
        model:openai/deepseek-v3-2-251201  
        api_key:"os.environ/ARK_API_KEY"  
        api_base:https://ark.cn-beijing.volces.com/api/v3  
        rpm:100      # [OPTIONAL] Rate limit for this deployment: in requests per minute (rpm)  
  
-model_name:qwen3-max  
    litellm_params:  
        model:openai/qwen3-max  
        api_key:"os.environ/DASHSCOPE_API_KEY"  
        api_base:https://dashscope.aliyuncs.com/compatible-mode/v1  
        rpm:100      # [OPTIONAL] Rate limit for this deployment: in requests per minute (rpm)  
  
  
litellm_settings:  
    drop_params:True  
    telemetry:False  
    num_retries:3# retry call 3 times on each model_name (e.g. deepseek-v3)  
    request_timeout:1200# raise Timeout error if call takes longer than 10s. Sets litellm.request_timeout   
    fallbacks:[  
        {"deepseek-v3":["deepseek-v3-1"]},  
        {"deepseek-v3-1":["qwen3-max"]},  
        {"glm-4.7":["qwen3-max"]}  
    ]# fallback to glm-4.7 if call fails num_retries   
    context_window_fallbacks:[{"deepseek-v3":["qwen3-max"]}]# fallback to glm-4.7 if context window error  
    allowed_fails:3# cooldown model if it fails > 1 call in a minute.   
  
  
router_settings:  
routing_strategy:simple-shuffle
```

step 4. 启动

```
docker-compose up -d
```

## 四、调用代码示例

### 4.1 安装sdk

```
pip install openai
```

### 4.2 调用代码

```
import openai  
  
llm_client = openai.OpenAI(  
    api_key="sk-1234",  
    base_url="http://serverhost:4000"  
)  
  
def call_litellm_api(prompt, model="qwen3-max"):  
    response = llm_client.chat.completions.create(  
        model=model,  
        messages=[  
            {"role": "system", "content": "你是一个乐于解答各种问题的助手，你的任务是为用户提供专业、准确、有见地的建议。"},  
            {"role": "user", "content": prompt}  
        ],  
        max_tokens=8192,  
        extra_body={  
            "top_k": 20,  
            "enable_thinking": False  
        }  
    )  
    return response.choices[0].message.content, response.usage.total_tokens  
  
if __name__ == "__main__":  
    prompt = "帮我生成一篇800字的作文，主题随机"  
    resp, _ = call_litellm_api(prompt)  
    print(resp)
```

## 总结

LiteLLM 不会让模型本身更智能，却能帮我们把大模型调用这件事做得更规范、更省心：

* **用得稳**：内置自动重试、故障转移，大幅降低调用失败率
* **换得快**：业务代码零改造，一键切换各家厂商模型
* **看得透**：统一归集成本数据、全链路详细日志，消耗一目了然

除此之外，LiteLLM 在**团队与权限管理**上同样具备显著优势：支持统一 API 入口、Virtual Key 虚拟密钥管控、精细化团队权限分配，让多模型、多成员、多环境的调用管理更安全、更有序，真正实现从 “能用” 到 “好用、好管” 的升级。

---

看到这里，觉得有用的朋友，点个赞吧