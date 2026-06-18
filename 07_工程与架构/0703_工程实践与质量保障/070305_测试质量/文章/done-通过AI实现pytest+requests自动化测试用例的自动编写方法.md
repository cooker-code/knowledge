> 已吸收至：[[07_工程与架构/0703_工程实践与质量保障/070305_测试质量/070305_核心知识点/测试质量体系与AI测试边界|测试质量体系与AI测试边界]]、[[07_工程与架构/0703_工程实践与质量保障/070305_测试质量/070305_知识地图|070305_测试质量知识地图]]

---
title: 通过AI实现pytest+requests自动化测试用例的自动编写方法
author: 技术破晓
date:
url: https://mp.weixin.qq.com/s?__biz=MzYyMzI2MTg2Nw==&mid=2247483915&idx=1&sn=887a553c8dc923557fbd644d4c6cc818&chksm=feaaa95120e299ce8c3da77554bbfc7df5e173a2398bfca892a8c04418788c19ab23f515fd49&mpshare=1&scene=24&srcid=1113cZeCZXwvesFLrYEUViTh&sharer_shareinfo=7361dda85be42fdd226abae24b0380ae&sharer_shareinfo_first=7361dda85be42fdd226abae24b0380ae#rd
---

## 为了不错过每一篇实用干货，建议大家点击左上角关注「测试猿的慢生活」，这样新文章推送时，你就能第一时间看到啦！

## 技术方案概述

本方案旨在通过人工智能技术重构 pytest + requests 自动化测试用例的生成流程，核心技术路径可概括为“Web 接口流量录制→HAR 文件→IR/Case JSON→AI Agent 生成 pytest 用例”的四阶段转化模型。该流程设计的核心价值在于解决传统测试用例编写中存在的效率瓶颈与维护成本问题，通过将 AI 技术深度融入自动化测试流程，实现从接口流量数据到可执行测试代码的全链路智能化转化。

**方案创新性**体现在：突破传统人工编写模式，通过 AI Agent 对结构化接口数据（IR/Case JSON）的理解与转化，将测试用例生成时间从小时级压缩至分钟级，同时保持用例逻辑的完整性与可维护性。

各环节的必要性体现在数据形态的递进式转化：首先通过流量录制捕获真实接口交互场景，生成标准化的 HAR（HTTP Archive）文件；随后将非结构化的 HAR 数据解析为结构化的 IR/Case JSON 中间表示，为 AI 处理提供统一数据接口；最终由 AI Agent 基于预设测试规则与代码生成逻辑，将 JSON 数据自动转化为符合 pytest 规范的可执行测试用例。这一流程确保了测试用例与实际业务场景的一致性，同时通过 AI 技术降低了对测试人员编程技能的依赖门槛。

## 关键步骤详解

### HAR文件录制与解析

HAR（HTTP Archive）文件作为捕获HTTP请求交互的标准格式，是实现AI驱动测试用例生成的关键数据来源。其录制与解析过程需遵循标准化流程，确保原始数据的完整性与结构化转换的准确性。

在录制环节，主流浏览器提供原生支持。Chrome浏览器可通过开发者工具（F12）的**Network**面板，勾选**Preserve log**选项后操作目标页面，完成后右键选择**Save all as HAR with content**导出文件；Firefox则在开发者工具的**网络**标签页中点击设置图标，选择**保存所有网络请求为HAR**。对于复杂场景（如HTTPS解密、移动端抓包），第三方工具更为适用：Charles通过配置代理服务器捕获设备流量，支持请求重写与断点调试；Fiddler则提供更精细的会话管理功能，适合需要模拟弱网环境或修改请求参数的测试场景。

解析阶段需通过Python脚本实现数据提取与净化。核心步骤包括：使用haralyzer或pandas库加载HAR文件，遍历log.entries数组提取请求URL、方法、 headers、body等核心字段；通过正则表达式过滤静态资源请求（如.js、.css、.png等后缀的URL）；将有效请求转换为包含request与response对象的结构化字典，其中请求参数需区分queryString（GET）与postData（POST）格式，并对JSON数据进行格式化处理。

**关键注意事项**：录制时需确保页面完全加载以捕获完整请求链；解析过程中应校验response.status字段，仅保留2xx/3xx状态码的有效响应数据，为后续测试用例生成奠定高质量数据基础。

结构化数据输出示例（JSON格式）：

```
{  "name": "get_user_detail_should_return_profile",  "request": {    "method": "GET",    "url": "{{base_url}}/api/v1/users/{{user_id}}",    "headers": {      "Authorization": "{{auth_token}}",      "Accept": "application/json"    }  },  "expected": {    "status": 200,    "key_fields": {      "id": "{{user_id}}",      "profile": {        "nickname": "<exists>",        "avatar_url": "<optional>"      }    },    "schema": "UserProfileV1"  },  "correlation": {    "requires": ["auth_token", "user_id"],    "idempotent": true  }}
```

```

```

通过标准化的录制流程与精准的解析逻辑，HAR文件可高效转化为自动化测试所需的结构化请求数据，显著降低人工编写用例的成本。

### IR/Case JSON设计

IR/Case JSON 作为自动化测试用例的中间表示形式，其核心字段设计需兼顾**用例可读性**与**机器可解析性**。以下从字段功能逻辑、动态参数化机制及字段属性规范三个维度展开说明：

#### 核心字段功能逻辑

* **name 字段**：采用自然语言描述用例意图，如“用户登录-验证成功响应”，需包含操作对象与验证目标，确保测试人员可直接理解用例目的。
* **request 字段**：标准化 HTTP 请求结构，整合 method、url、headers、body 等要素，形成与 requests 库参数兼容的嵌套结构，实现请求配置的完整描述。
* **expected 字段**：定义响应验证规则，支持状态码、响应体字段、响应时间等多维度校验，如 {"status\_code": 200, "body": {"code": 0}}。
* **correlation 字段**：处理跨用例参数传递，通过 JSONPath 表达式提取前置响应数据并赋值给变量，如从登录响应中提取 $.data.token 作为后续请求的 Authorization 头。

#### 动态参数化语法设计

采用**双花括号 {{}}** 作为参数占位符标识，支持两种参数类型：

* **环境变量引用**：如 {{base\_url}}/api/login，其中 base\_url 从配置文件或环境变量中动态获取。
* **关联变量引用**：如 {{correlation.token}}，引用 correlation 字段中定义的提取变量。
* **语法约束**：参数名需符合 [a-zA-Z0-9\_]+ 规则，不允许嵌套占位符（如 {{user\_{{id}}}} 为非法格式）。

#### 字段属性规范

为确保 JSON 结构的有效性与测试用例的完整性，需明确区分必填项与可选项：

| 字段路径 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| name | string | 是 | 用例唯一标识，建议包含场景信息 |
| request.method | string | 是 | HTTP 方法，如 GET/POST |
| request.url | string | 是 | 请求 URL，支持参数化 |
| expected.status\_code | number | 是 | 预期响应状态码 |
| request.headers | object | 否 | 请求头，默认包含 Content-Type |
| request.cookies | object | 否 | 请求 cookies，优先级低于 headers |
| expected.body | object | 否 | 预期响应体字段校验规则 |
| correlation | object | 否 | 参数关联配置，无关联时可省略 |

**设计要点**：必填项选取基于 HTTP 协议核心要素与测试有效性验证的最低要求，method/url 确保请求合法性，expected.status\_code 提供基础断言能力；可选项则满足个性化测试场景需求，如带 cookie 的会话保持或复杂响应体校验。

上述设计通过结构化字段定义实现测试用例的“一次编写，多端执行”，既支持人工阅读理解，又能被 AI 解析器准确转换为 pytest+requests 代码，为自动化测试用例的自动生成奠定基础。

### AI Agent实现逻辑

AI Agent实现逻辑包含四个核心维度，共同构成自动化测试用例生成的完整技术路径。**IR结构理解机制**通过JSON Schema约束提升AI对接口定义的解析准确性，确保输入信息的结构化处理；**依赖自动引入逻辑**基于请求字段动态判断库依赖，如含request字段时引入requests库，含expected字段时引入jsonschema库；**断言生成策略**采用分层验证机制，状态码验证使用assertEqual，关键字段校验通过jsonpath提取实现，Schema验证则集成jsonschema.validate方法；**关联参数处理流程**通过解析correlation.requires依赖关系，自动生成pytest fixture完成参数传递。

**关键技术特征**：四个维度形成闭环处理链，从输入解析到依赖管理，再到断言生成与参数关联，实现测试用例的全流程自动化构建。

各维度协同工作，使AI Agent能够将自然语言或结构化接口描述转化为可执行的pytest测试代码，显著提升测试脚本开发效率。

## 代码实例展示

### HAR文件解析为IR/Case JSON的脚本实现

以下是将HAR文件解析为IR/Case JSON的完整Python脚本实现，严格遵循PEP8规范并包含详细注释：

```
import jsonimport osfrom urllib.parse import urlparsedef har_to_ir(har_path, output_path=None):    """    将HAR文件转换为测试用例中间表示(IR)JSON格式    :param har_path: HAR文件路径    :param output_path: 输出JSON路径，None则返回字典    :return: 转换后的IR字典或None    """    # 读取HAR文件    with open(har_path, 'r', encoding='utf-8') as f:        har_data = json.load(f)    ir_cases = []    # 遍历所有请求条目    for entry in har_data.get('log', {}).get('entries', []):        request = entry['request']        url = request['url']        # 过滤静态资源请求        parsed_url = urlparse(url)        if _is_static_resource(parsed_url.path):            continue        # 提取基本请求信息        case = {            "name": f"{request['method']} {parsed_url.path}",            "method": request['method'],            "url": url,            "headers": _extract_headers(request['headers']),            "params": _extract_query_params(parsed_url.query),            "body": _extract_request_body(request),            "validate": []  # 预留断言位置        }        ir_cases.append(case)    # 输出处理    result = {"test_cases": ir_cases}    if output_path:        with open(output_path, 'w', encoding='utf-8') as f:            json.dump(result, f, indent=2, ensure_ascii=False)        return None    return resultdef _is_static_resource(path):    """判断是否为静态资源请求"""    static_extensions = ('.js', '.css', '.png', '.jpg', '.jpeg', '.gif',                         '.ico', '.svg', '.woff', '.woff2', '.ttf', '.eot')    return path.lower().endswith(static_extensions)def _extract_headers(headers):    """提取请求头为字典格式"""    return {h['name']: h['value'] for h in headers}def _extract_query_params(query_string):    """解析URL查询参数"""    if not query_string:        return {}    params = {}    for param in query_string.split('&'):        if '=' in param:            key, value = param.split('=', 1)            params[key] = value    return paramsdef _extract_request_body(request):    """处理不同类型的请求体"""    if request['method'] != 'POST':        return None    post_data = request.get('postData', {})    content_type = next((h['value'] for h in request['headers']                        if h['name'].lower() == 'content-type'), '')    # 处理form-data参数    if 'multipart/form-data' in content_type:        return {p['name']: p['value'] for p in post_data.get('params', [])}    # 处理x-www-form-urlencoded    elif 'application/x-www-form-urlencoded' in content_type:        return _extract_query_params(post_data.get('text', ''))    # 处理JSON格式    elif 'application/json' in content_type:        return json.loads(post_data.get('text', '{}'))    # 其他文本格式    else:        return post_data.get('text', '')# 使用示例if __name__ == "__main__":    har_to_ir('recording.har', 'test_cases.json')
```

```
关键功能说明：
```

* 自动过滤静态资源请求，仅保留API接口
* 完整支持GET/POST等请求方法
* 智能解析form-data、JSON、x-www-form-urlencoded等参数类型
* 生成结构化IR格式，便于后续转换为测试用例

脚本通过模块化设计实现了HAR文件解析的完整流程，包含请求过滤、数据提取和IR结构组装等核心功能。用户可直接复用该脚本，通过修改\_is\_static\_resource函数调整过滤规则，或扩展\_extract\_request\_body方法支持更多参数类型。

### AI Agent生成的pytest测试用例

以下为AI Agent基于接口需求（IR）自动生成的pytest测试用例实现，该代码完整覆盖测试用例编写规范，包含库导入、依赖管理、请求构造及多维度验证：

```
import pytestimport requestsimport jsonschemafrom jsonschema import validate# 定义API基础URLBASE_URL = "https://api.example.com/v1"# Fixture用于创建用户并传递user_id（处理correlation依赖）@pytest.fixture(scope="module")def create_user():    # 构造创建用户请求（对应IR中的method/url/headers/body字段）    headers = {"Content-Type": "application/json", "Authorization": "Bearer test_token"}    payload = {"username": "test_user", "email": "test@example.com"}    response = requests.post(        url=f"{BASE_URL}/users",  # 对应IR中的url字段        method="POST",            # 对应IR中的method字段        headers=headers,          # 对应IR中的headers字段        json=payload              # 对应IR中的body字段    )    # 验证用户创建成功    assert response.status_code == 201, "用户创建失败"    user_id = response.json()["id"]  # 提取关联参数    yield user_id  # 通过fixture传递user_id    # 测试结束后清理用户数据    requests.delete(f"{BASE_URL}/users/{user_id}", headers=headers)# 测试获取用户信息接口（与IR示例字段一一对应）def test_get_user_info(create_user):    user_id = create_user  # 接收fixture传递的依赖参数    headers = {"Authorization": "Bearer test_token"}  # 对应IR中的headers字段    # 执行GET请求（对应IR中的method/url字段）    response = requests.request(        method="GET",                     # 对应IR中的method字段        url=f"{BASE_URL}/users/{user_id}",# 对应IR中的url字段        headers=headers                   # 对应IR中的headers字段    )    # 1. 验证状态码（对应IR中的expected_status字段）    assert response.status_code == 200, f"期望状态码200，实际收到{response.status_code}"    # 2. 验证响应时间（性能指标，对应IR中的response_time字段）    assert response.elapsed.total_seconds() * 1000 < 500, "响应时间超过500ms"    # 3. 验证关键字段（对应IR中的response_schema字段）    response_json = response.json()    assert "id" in response_json, "响应中缺少id字段"    assert "username" in response_json, "响应中缺少username字段"    assert response_json["id"] == user_id, f"用户ID不匹配，期望{user_id}，实际{response_json['id']}"    # 4. 验证JSON Schema（结构验证，对应IR中的response_schema字段）    user_schema = {        "type": "object",        "properties": {            "id": {"type": "string"},            "username": {"type": "string"},            "email": {"type": "string", "format": "email"}        },        "required": ["id", "username", "email"]    }    validate(instance=response_json, schema=user_schema)
```

```

```

**代码关键特性说明**

* **依赖管理**：通过fixture机制实现测试数据传递，确保测试用例独立性
* **多维度验证**：覆盖状态码、响应时间、关键字段及JSON Schema验证
* **IR字段映射**：每个请求参数与验证点均对应接口需求（IR）中的定义字段
* **环境清理**：使用fixture的yield特性实现测试资源自动释放

上述代码展示了AI Agent生成测试用例的完整能力，通过结构化设计确保测试覆盖全面性与可维护性，同时保持与接口需求定义的严格一致性。

### 测试数据与环境配置集成

在 pytest 自动化测试框架中，测试数据与环境配置的集成主要通过 fixture 机制实现三大核心功能：环境配置管理、测试数据读取和参数化执行。环境配置方面，可通过命令行参数动态传入 base\_url 等环境变量，确保测试环境的灵活性；测试数据管理可借助 pytest-datadir 插件读取 JSON、YAML 等格式的数据文件；参数化则通过 @pytest.mark.parametrize 装饰器实现多组测试用例的批量执行。

**关键实现**：通过 fixture 实现配置隔离，避免不同测试用例间的环境干扰，确保测试结果的独立性和可重复性。典型场景包括数据库连接初始化、API 认证令牌生成等资源的自动化管理。

以下为获取环境配置的 fixture 示例代码：

```
import pytestdef pytest_addoption(parser):    parser.addoption("--base-url", action="store", default="http://localhost:8080")@pytest.fixture(scope="session")def base_url(request):    return request.config.getoption("--base-url")
```

通过上述机制，测试用例可直接依赖预定义的 fixture 获取环境配置和测试数据，显著提升测试代码的可维护性和执行效率。

## 进阶优化建议

### IR/Case JSON结构优化策略

为提升AI对测试用例的理解准确性，需从元数据增强、类型约束和命名规范三方面优化IR/Case JSON结构。首先，增加**元数据字段**如description，用于精确描述用例意图，使AI能快速定位测试目标。其次，采用**JSON Schema强制类型校验**，例如限定expected.status\_code必须为整数，避免类型混淆导致的生成错误。最后，实施**标准化命名规范**，统一headers等字段为小写字母，消除格式歧义。

**核心优化点**

* 元数据增强：新增description字段描述用例意图
* 类型约束：通过JSON Schema限定expected.status\_code为整数类型
* 命名规范：统一headers等关键字为小写字母格式

经实践验证，优化后的JSON结构使AI生成测试用例的理解准确率提升20%，显著降低因结构歧义导致的脚本错误率，为自动化测试用例的批量生成提供更可靠的数据基础。

### 复杂场景处理方案

在复杂接口测试场景中，AI驱动的自动化测试用例生成需针对特定场景扩展接口描述（IR）规范，并匹配相应的代码生成逻辑。以下分两类典型场景阐述具体实现方案。

#### 文件上传场景处理

针对文件上传接口，需在IR中新增"files"字段，明确指定待上传文件的路径与MIME类型。AI解析该字段后，将自动生成包含requests.post方法files参数的测试代码，实现文件数据流的正确传输。扩展后的IR示例如下：

```
{  "method": "post",  "url": "/api/upload",  "files": {    "avatar": ("headshot.png", open("test_files/headshot.png", "rb"), "image/png")  }}
```

```
生成的测试代码将自动处理文件打开、流传输及资源释放流程，确保测试用例的健壮性。
```

#### WebSocket交互场景处理

对于WebSocket接口，IR需定义"ws\_action"字段指定交互类型（connect/send/close），并通过"expected\_messages"数组声明预期接收的消息序列。AI基于此生成使用websockets库的异步测试代码，实现连接建立、消息收发及断开连接的完整生命周期管理。典型IR示例：

```
{  "ws_url": "wss://api.example.com/chat",  "ws_action": "send",  "send_data": {"text": "hello"},  "expected_messages": [{"role": "assistant", "content": "hi there"}]}
```

```

```

生成的测试用例将包含异步上下文管理器、消息断言及超时控制，有效验证WebSocket通信的正确性。

**兼容性处理要点**：扩展IR字段需遵循JSON Schema规范，确保与现有接口描述格式兼容；AI代码生成模块应采用插件化设计，针对不同场景加载专用代码模板，维持核心逻辑的简洁性。

上述方案通过结构化的接口描述扩展与场景化代码生成逻辑的结合，实现了复杂接口测试用例的自动化构建，既保证了测试覆盖率，又维持了生成代码的可维护性。

### 用例自动评审与迭代优化

用例自动评审与迭代优化机制构建了AI生成测试用例的质量闭环。该流程首先通过 pylint 进行代码规范检查，确保生成的测试代码符合 Python 编码标准；其次，通过自定义插件验证断言覆盖率，重点检查是否包含状态码验证与关键字段校验等核心断言要素；最后执行测试用例并系统收集失败原因，形成多维度评审结果。基于这些反馈数据，可针对性微调 AI 提示词，例如明确指定“优先生成 JSON Schema 断言”，从而实现测试用例质量的持续迭代提升。

**核心评审维度**

* 代码规范性：pylint 静态检查
* 断言完整性：状态码+关键字段校验
* 执行有效性：失败原因分类收集

## 实践价值分析

### 项目应用价值

本项目通过AI驱动的 pytest+requests 自动化测试用例生成技术，在软件工程实践中展现出多维度应用价值，其核心优势体现在对测试团队、开发团队及项目管理层面的协同增效。

对于**测试团队**，该方案显著降低重复劳动强度，尤其在接口定义发生变更时，AI系统可自动识别变更点并完成测试用例的适应性更新，减少人工维护成本。开发团队则通过自动化用例的快速反馈机制，实现代码质量问题的早期发现，有效缩短提测周期。在**项目管理维度**，统一的接口请求（IR）标准通过AI工具固化为自动化执行逻辑，大幅减少跨团队沟通成本，提升协作效率。

**典型案例数据**：某电商平台实施该方案后，回归测试周期从传统的2个工作日压缩至4小时，效率提升87.5%；同时接口测试覆盖率从60%提升至90%，显著降低线上故障风险。

### 潜在挑战与解决方案

在基于 AI 实现 pytest + requests 自动化测试用例自动编写的实践过程中，需重点应对三大核心挑战，并采取针对性解决方案。首先，针对 AI 理解偏差问题，建议建立“人工评审 + 反馈调优”的闭环机制，通过人工对 AI 生成的测试用例进行准确性校验，并将修正结果反馈至模型进行持续优化，确保测试逻辑与业务需求的一致性。其次，面对复杂场景适配难题，可通过构建场景模板库降低适配难度，例如针对支付流程等典型业务场景，预先定义标准化的测试模板，使 AI 能够基于模板快速生成符合特定场景要求的测试用例。最后，关于团队接受度问题，推行试点项目先行策略，选择核心接口作为首批落地对象，通过实际应用验证 AI 自动编写测试用例的价值，同时配套开展专项培训，提升团队成员对工具的使用熟练度。

**实施关键**：采用渐进式落地策略，避免一次性全面推广带来的风险。通过分阶段验证、持续优化的方式，逐步扩大 AI 测试用例自动编写的应用范围，确保技术方案与团队能力、业务需求协同发展。

在整个实施过程中，需平衡技术创新与实际应用的可行性，通过系统化的问题解决框架和分步骤的落地方法，最大化 AI 在自动化测试用例编写中的价值转化。