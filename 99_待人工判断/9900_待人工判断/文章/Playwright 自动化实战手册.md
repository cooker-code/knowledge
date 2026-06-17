---
title: Playwright 自动化实战手册
author: ALL程序猿
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzOTIyMjA0Mw==&mid=2247490123&idx=1&sn=6454a2f50d77467d9f6d5b5f95c354f8&chksm=c3c0869d042dce4a3ba8737bafd26bf1578580558deb1f8acce121c74e1e3c447e5064a8419d&mpshare=1&scene=24&srcid=0418QAEzdbSNTBqT7fq8rPVS&sharer_shareinfo=d76b8b93241056a2398dd9d8935aa491&sharer_shareinfo_first=d76b8b93241056a2398dd9d8935aa491#rd
---

> 面向自动化场景，快速查阅、开箱即用。

---

## 目录

1. **核心概念：三层架构**
2. **启动与配置**
3. **Context 与 Page**
4. **登录状态复用**
5. **元素定位**
6. **常用交互操作**
7. **等待机制**
8. **iframe 处理**
9. **遍历动态列表与表格**
10. **反爬与稳定性技巧**
11. **异常处理**
12. **完整流程模板**

---

## 1. 核心概念：三层架构

```
Playwright Engine  
    └── Browser（浏览器实例）  
            └── Context（独立会话，相当于隔离的用户环境）  
                    └── Page（标签页）
```

**记住这条原则：**

| 场景 | 做法 |
| --- | --- |
| 不同网站 | 不同 Context（避免 Cookie/Storage 污染） |
| 同一网站的多个页面 | 同一 Context 内多个 Page（共享登录态） |
| 并发采集多个账号 | 多个 Context，每个 Context 独立登录 |

---

## 2. 启动与配置

### 同步模式（脚本/简单任务推荐）

```
from playwright.sync_api import sync_playwright  
  
p = sync_playwright().start()  
browser = p.chromium.launch(  
    headless=True,          # 生产环境用 True；调试时改 False  
    slow_mo=0,              # 调试时设 300~500，每步之间加延迟  
    args=[  
        '--no-sandbox',  
        '--disable-blink-features=AutomationControlled',  # 隐藏自动化特征  
    ]  
)
```

### 异步模式（高并发/大批量任务推荐）

```
import asyncio  
from playwright.async_api import async_playwright  
  
async def main():  
    async with async_playwright() as p:  
        browser = await p.chromium.launch(headless=True)  
        # ... 你的逻辑  
        await browser.close()  
  
asyncio.run(main())
```

> **同步 vs 异步选哪个？**单任务脚本用同步，代码更简洁；需要并发处理多个账号/多个页面时用异步，性能差距明显。

---

## 3. Context 与 Page

### 创建 Context

```
context = browser.new_context(  
    no_viewport=True,              # 自适应窗口（与 viewport 二选一）  
    viewport={'width': 1920, 'height': 1080},  
    user_agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64)...',  
    locale='zh-CN',  
    timezone_id='Asia/Shanghai',  
    bypass_csp=True,               # 绕过 CSP，调试时有用  
    ignore_https_errors=True,      # 忽略证书错误（内网场景常用）  
    storage_state='session.json',  # 加载已保存的登录态  
)
```

### 创建 Page

```
page = context.new_page()  
  
# 导航  
page.goto('https://example.com', wait_until='domcontentloaded')  
# wait_until 选项：'load' | 'domcontentloaded' | 'networkidle' | 'commit'  
# 大多数场景用 'domcontentloaded'，速度更快；需要等 API 数据用 'networkidle'  
  
# 当前 URL  
print(page.url)  
  
# 页面标题  
print(page.title())
```

### 多标签页处理

```
# 监听新标签页打开事件  
with context.expect_page() as new_page_info:  
    page.get_by_role("link", name="在新窗口打开").click()  
new_page = new_page_info.value  
new_page.wait_for_load_state('domcontentloaded')  
# 操作 new_page...
```

### 清理资源

```
# 用完记得关，防止资源泄漏  
context.close()  
browser.close()  
p.stop()
```

---

## 4. 登录状态复用

**一次登录，多次复用**，避免重复走验证码/短信验证流程。

### 第一步：登录并保存状态

```
context = browser.new_context()  
page = context.new_page()  
  
page.goto('https://example.com/login')  
page.get_by_label("用户名").fill('myuser')  
page.get_by_label("密码").fill('mypassword')  
page.get_by_role("button", name="登录").click()  
page.wait_for_load_state('networkidle')  # 等登录接口完成  
  
# 保存 cookies + localStorage  
context.storage_state(path='session.json')  
context.close()
```

### 第二步：后续直接加载

```
context = browser.new_context(storage_state='session.json')  
page = context.new_page()  
page.goto('https://example.com/dashboard')  # 已是登录态
```

> **注意：**`session.json` 包含敏感信息，加入 `.gitignore`，不要上传到代码仓库。

---

## 5. 元素定位

### 定位优先级（从稳到脆）

```
data-testid  >  role  >  label  >  placeholder  >  text  >  CSS/XPath
```

### 决策树

```
看到一个元素  
   │  
   ├─ 有 data-testid？ → get_by_test_id("xxx")  
   │  
   ├─ 是按钮/链接/输入框？ → get_by_role("button/link/textbox", name="...")  
   │  
   ├─ 是表单输入框且有 <label>？ → get_by_label("标签文字")  
   │  
   ├─ 输入框有 placeholder？ → get_by_placeholder("提示文字")  
   │  
   ├─ 有明显可见文本？ → get_by_text("文字")  
   │  
   └─ 以上都不行 → locator("#id") / locator("xpath")
```

### 各方法示例

```
# 1. 按测试 ID（最稳定，需要开发配合）  
page.get_by_test_id("submit-btn")  
  
# 2. 按角色（官方最推荐）  
page.get_by_role("button", name="提交")  
page.get_by_role("link", name="首页")  
page.get_by_role("textbox", name="搜索")  
page.get_by_role("checkbox", name="记住我")  
page.get_by_role("combobox", name="城市")    # <select>  
  
# 3. 按 label / placeholder  
page.get_by_label("用户名")  
page.get_by_placeholder("请输入邮箱")  
  
# 4. 按文本  
page.get_by_text("欢迎回来")  
page.get_by_text("提交", exact=True)  # 精确匹配，避免误命中  
  
# 5. CSS / XPath（兜底）  
page.locator("#submit-btn")  
page.locator(".form > input[name='username']")  
page.locator("//button[text()='提交']")
```

### 常用 Role 速查表

| HTML 标签 | role 值 |
| --- | --- |
| `<button>` | `"button"` |
| `<a>` | `"link"` |
| `<input type="text">` | `"textbox"` |
| `<input type="checkbox">` | `"checkbox"` |
| `<input type="radio">` | `"radio"` |
| `<select>` | `"combobox"` |
| `<li>` | `"listitem"` |
| `<h1>` ~`<h6>` | `"heading"` |
| `<img>` | `"img"` |

### 链式定位与过滤

```
# 在某个容器内查找（缩小范围）  
page.get_by_role("form").get_by_label("用户名").fill("张三")  
  
# 过滤包含特定文本的元素  
page.get_by_role("listitem").filter(has_text="已完成").click()  
  
# 过滤包含特定子元素的元素  
page.get_by_role("listitem").filter(has=page.get_by_role("checkbox", checked=True))  
  
# nth：获取第 N 个匹配项（从 0 开始）  
page.get_by_role("listitem").nth(0)   # 第一个  
page.get_by_role("listitem").last     # 最后一个
```

### 定位器是"惰性"的

```
# 这行不会立即查找元素，只是定义了查找策略  
button = page.get_by_role("button", name="提交")  
  
# 直到这里才真正查找 + 等待 + 操作  
button.click()
```

好处：页面刷新或元素重新渲染后，定位器依然有效，会自动重试。

## 6. 常用交互操作

### 点击

```
page.get_by_role("button", name="提交").click()  
page.get_by_role("button", name="编辑").dblclick()           # 双击  
page.get_by_role("button", name="更多").click(button="right") # 右键  
page.get_by_role("button", name="提交").click(delay=100)     # 带延迟，防风控
```

### 文本输入

```
page.get_by_label("用户名").fill("zhangsan")       # 清空后输入（推荐）  
page.get_by_label("备注").type("内容", delay=50)   # 模拟真人打字速度  
page.get_by_label("搜索").clear()                  # 清空输入框  
page.keyboard.press("Enter")                       # 按回车  
page.keyboard.press("Control+A")                   # 快捷键
```

### 下拉框

```
page.locator("select#city").select_option("beijing")          # 按 value  
page.locator("select#city").select_option(label="北京市")     # 按显示文本  
page.locator("select#tags").select_option(["tag1", "tag2"])   # 多选
```

### 复选框 / 单选框

```
page.get_by_label("同意协议").check()  
page.get_by_label("订阅邮件").uncheck()  
is_checked = page.get_by_label("记住我").is_checked()
```

### 文件上传

```
page.get_by_label("上传文件").set_input_files("./file.pdf")  
page.get_by_label("上传文件").set_input_files(["./a.pdf", "./b.pdf"])  # 多文件  
page.get_by_label("上传文件").set_input_files([])  # 清除已选文件
```

### 获取元素信息

```
text = page.get_by_role("heading").inner_text()  
html = page.locator(".content").inner_html()  
value = page.get_by_label("用户名").input_value()  
attr = page.locator("img").get_attribute("src")  
is_visible = page.get_by_role("button").is_visible()  
is_enabled = page.get_by_role("button").is_enabled()
```

### 鼠标 & 滚动

```
# 悬停（触发 hover 菜单）  
page.get_by_role("button", name="更多").hover()  
  
# 滚动到元素  
page.get_by_text("底部内容").scroll_into_view_if_needed()  
  
# 滚动页面  
page.mouse.wheel(0, 3000)  # 向下滚动 3000px  
  
# 拖拽  
page.drag_and_drop("#source", "#target")
```

---

## 7. 等待机制

**Playwright 大多数操作自带重试等待（默认超时 30 秒），但复杂场景需要显式等待。**

### 等待页面状态

```
page.wait_for_load_state("domcontentloaded")  # DOM 解析完成（快）  
page.wait_for_load_state("load")              # 所有资源加载完成  
page.wait_for_load_state("networkidle")       # 网络请求全部完成（慢但稳）
```

### 等待元素状态

```
# 等待元素可见  
page.locator(".result-list").wait_for(state="visible")  
  
# 等待元素消失（如 loading 遮罩）  
page.locator(".loading-mask").wait_for(state="hidden")  
  
# 等待元素从 DOM 中移除  
page.locator(".toast").wait_for(state="detached")  
  
# 等待元素出现在 DOM（不一定可见）  
page.locator(".data").wait_for(state="attached")
```

### 等待网络请求

```
# 等待特定接口响应（用于数据加载完成的判断）  
with page.expect_response("**/api/list**") as resp_info:  
    page.get_by_role("button", name="查询").click()  
response = resp_info.value  
data = response.json()  # 直接拿接口返回数据！
```

> **直接拦截接口数据**是自动化采集的一个强力技巧，比解析 DOM 更可靠。

### 固定等待（尽量少用）

```
page.wait_for_timeout(2000)  # 等待 2 秒，最后的手段
```

### 自定义超时

```
# 单次操作超时  
page.get_by_role("button").click(timeout=5000)  # 超时 5 秒  
  
# 全局默认超时  
page.set_default_timeout(60000)  # 改为 60 秒
```

### 等待策略组合（推荐模板）

```
# 点击触发数据加载后的最佳等待姿势  
page.get_by_role("button", name="查询").click()  
page.wait_for_load_state("networkidle")          # 等网络空闲  
page.locator(".data-table").wait_for(state="visible")  # 再确认关键元素
```

---

## 8. iframe 处理

```
# 通过 CSS 选择器进入 iframe  
frame = page.frame_locator("iframe#editor")  
  
# 在 iframe 内操作，和普通 page 完全一样  
frame.get_by_role("textbox").fill("Hello World")  
frame.get_by_role("button", name="保存").click()  
  
# 操作完直接用 page 即可回到主页面，无需显式切换  
page.get_by_role("button", name="关闭").click()
```

### 嵌套 iframe

```
frame = page.frame_locator("iframe#outer").frame_locator("iframe#inner")  
frame.get_by_role("button", name="提交").click()
```

---

## 9. 遍历动态列表与表格

### 正确姿势：先 count，再 nth

```
rows = page.locator("table tbody tr")  
count = rows.count()  
  
for i in range(count):  
    row = rows.nth(i)  
    name = row.locator("td:nth-child(1)").inner_text()  
    status = row.locator("td:nth-child(2)").inner_text()  
    print(f"{name}: {status}")
```

> 为什么不用 `rows.all()` 直接遍历？`all()` 返回的是快照，如果操作过程中页面内容变化（如分页、弹窗导致刷新），会出现 stale element。用 `nth(i)` 每次都重新查找，更健壮。

### 操作每行并处理弹窗

```
rows = page.locator("table tbody tr")  
count = rows.count()  
  
for i in range(count):  
    row = rows.nth(i)  
    row.get_by_role("button", name="编辑").click()  
  
    # 操作弹窗  
    dialog = page.locator(".modal")  
    dialog.wait_for(state="visible")  
    dialog.get_by_label("备注").fill("已处理")  
    dialog.get_by_role("button", name="保存").click()  
    dialog.wait_for(state="hidden")  # 等弹窗关闭  
  
    # 下一行会自动重新查找，不受影响
```

### 点击"加载更多"后继续采集

```
while True:  
    # 采集当前页数据  
    items = page.get_by_role("listitem").all()  
    for item in items:  
        print(item.inner_text())  
  
    # 检查是否还有下一页  
    load_more = page.get_by_role("button", name="加载更多")  
    ifnot load_more.is_visible():  
        break  
  
    load_more.click()  
    # 等新内容加载完（用旧内容数量作为基准）  
    page.wait_for_function(f"document.querySelectorAll('li').length > {len(items)}")
```

## 10. 反爬与稳定性技巧

### 隐藏自动化特征

```
browser = p.chromium.launch(  
    args=['--disable-blink-features=AutomationControlled']  
)  
context = browser.new_context(  
    user_agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36...'  
)  
  
# 注入脚本，覆盖 webdriver 特征  
page.add_init_script("Object.defineProperty(navigator, 'webdriver', {get: () => undefined})")
```

### 模拟真人行为

```
import random, time  
  
# 随机延迟  
time.sleep(random.uniform(1.0, 3.0))  
  
# 打字带延迟  
page.get_by_label("搜索").type("关键词", delay=random.randint(50, 150))  
  
# 点击加延迟  
page.get_by_role("button").click(delay=random.randint(50, 200))
```

### 截图与调试

```
# 出错时截图留证  
try:  
    page.get_by_role("button", name="提交").click()  
except Exception as e:  
    page.screenshot(path=f"error_{int(time.time())}.png")  
    raise  
  
# 全页截图（包括不可见区域）  
page.screenshot(path="full.png", full_page=True)  
  
# 对特定元素截图  
page.locator(".chart").screenshot(path="chart.png")
```

### 网络拦截（加速/Mock）

```
# 屏蔽图片和媒体资源，加速加载  
def block_resources(route):  
    if route.request.resource_type in ["image", "media", "font"]:  
        route.abort()  
    else:  
        route.continue_()  
  
page.route("**/*", block_resources)
```

---

## 11. 异常处理

### 常见错误类型

| 错误 | 原因 | 解决方案 |
| --- | --- | --- |
| `TimeoutError` | 元素未在超时时间内出现 | 增加 timeout，或检查等待条件 |
| `ElementNotFound` | 选择器无法匹配元素 | 检查选择器，或加等待 |
| `StaleElementReference` | 元素已被 DOM 替换 | 改用 `nth(i)` 替代 `all()` 后的元素 |
| `TargetClosedError` | Page/Context 被关闭 | 检查 context/page 生命周期 |

### 健壮的重试模板

```
import time  
  
def retry(fn, max_retries=3, delay=2):  
    for attempt in range(max_retries):  
        try:  
            return fn()  
        except Exception as e:  
            if attempt == max_retries - 1:  
                raise  
            print(f"第 {attempt+1} 次失败: {e}，{delay}s 后重试...")  
            time.sleep(delay)  
  
# 使用  
retry(lambda: page.get_by_role("button", name="查询").click())
```

---

## 12. 完整流程模板

### 单任务模板（同步）

```
from playwright.sync_api import sync_playwright  
import time  
  
def run():  
    p = sync_playwright().start()  
    browser = p.chromium.launch(headless=True)  
    context = browser.new_context(  
        storage_state='session.json',  # 复用登录态  
        user_agent='Mozilla/5.0...',  
    )  
    page = context.new_page()  
    page.set_default_timeout(30000)  
  
    try:  
        page.goto('https://example.com/list', wait_until='domcontentloaded')  
        page.wait_for_load_state('networkidle')  
  
        # 你的业务逻辑  
        rows = page.locator("table tbody tr")  
        count = rows.count()  
        results = []  
  
        for i in range(count):  
            row = rows.nth(i)  
            results.append({  
                "name": row.locator("td:nth-child(1)").inner_text().strip(),  
                "status": row.locator("td:nth-child(2)").inner_text().strip(),  
            })  
  
        return results  
  
    except Exception as e:  
        page.screenshot(path=f"error_{int(time.time())}.png")  
        raise  
    finally:  
        context.close()  
        browser.close()  
        p.stop()  
  
if __name__ == "__main__":  
    data = run()  
    print(f"共采集 {len(data)} 条")
```

### 多账号并发模板（异步）

```
import asyncio  
from playwright.async_api import async_playwright  
  
ACCOUNTS = [  
    {"session": "session_a.json", "name": "账号A"},  
    {"session": "session_b.json", "name": "账号B"},  
]  
  
asyncdef process_account(browser, account):  
    context = await browser.new_context(storage_state=account["session"])  
    page = await context.new_page()  
    try:  
        await page.goto("https://example.com/dashboard")  
        await page.wait_for_load_state("networkidle")  
        # ... 业务逻辑  
        print(f"{account['name']} 完成")  
    finally:  
        await context.close()  
  
asyncdef main():  
    asyncwith async_playwright() as p:  
        browser = await p.chromium.launch(headless=True)  
        await asyncio.gather(*[  
            process_account(browser, acc) for acc in ACCOUNTS  
        ])  
        await browser.close()  
  
asyncio.run(main())
```

---

## 快速速查卡

```
# 导航  
page.goto(url)  
page.go_back() / page.go_forward() / page.reload()  
  
# 定位  
page.get_by_role("button", name="...")  
page.get_by_label("...") / page.get_by_placeholder("...")  
page.get_by_text("...") / page.get_by_test_id("...")  
page.locator("css or xpath")  
  
# 操作  
.click() / .dblclick() / .hover()  
.fill("text") / .type("text", delay=50) / .clear()  
.check() / .uncheck() / .select_option("value")  
.set_input_files("path")  
  
# 获取信息  
.inner_text() / .inner_html() / .input_value()  
.get_attribute("attr") / .is_visible() / .is_enabled()  
  
# 等待  
.wait_for(state="visible/hidden/attached/detached")  
page.wait_for_load_state("networkidle/load/domcontentloaded")  
page.wait_for_selector("css")  
page.wait_for_timeout(ms)  
  
# 截图  
page.screenshot(path="shot.png", full_page=True)  
page.locator(".el").screenshot(path="el.png")
```

---

## 13. 执行 JavaScript

当 Playwright 的 Python API 无法满足需求时，直接在页面上下文执行 JS 是最后的利器。

### 基础用法

```
# 执行 JS 并获取返回值  
title = page.evaluate("document.title")  
scroll_y = page.evaluate("window.scrollY")  
  
# 传参给 JS  
result = page.evaluate("(x) => x * 2", 21)  # → 42  
  
# 多行 JS  
data = page.evaluate("""  
    () => {  
        const rows = document.querySelectorAll('table tr');  
        return Array.from(rows).map(r => r.innerText);  
    }  
""")
```

### 操作 DOM（绕过 Playwright 限制）

```
# 强制修改输入框的值（对某些 React/Vue 受控组件有效）  
page.evaluate("""  
    (value) => {  
        const input = document.querySelector('#amount');  
        const nativeInputSetter = Object.getOwnPropertyDescriptor(  
            window.HTMLInputElement.prototype, 'value'  
        ).set;  
        nativeInputSetter.call(input, value);  
        input.dispatchEvent(new Event('input', { bubbles: true }));  
    }  
""", "9999")  
  
# 触发自定义事件  
page.evaluate("document.querySelector('#editor').dispatchEvent(new Event('change'))")  
  
# 滚动到页面底部  
page.evaluate("window.scrollTo(0, document.body.scrollHeight)")
```

### evaluate vs evaluate\_handle

```
# evaluate：返回 Python 原生值（字符串、数字、列表、字典）  
count = page.evaluate("document.querySelectorAll('li').length")  
  
# evaluate_handle：返回 JS 对象句柄（用于后续传给其他 JS 调用）  
list_handle = page.evaluate_handle("document.querySelectorAll('li')")  
# 可以把 handle 传回给 evaluate  
first_text = page.evaluate("(list) => list[0].innerText", list_handle)
```

### 在页面加载前注入脚本

```
# add_init_script：每次页面加载/刷新时都会执行，适合反检测  
page.add_init_script("""  
    Object.defineProperty(navigator, 'webdriver', { get: () => undefined });  
    Object.defineProperty(navigator, 'plugins', { get: () => [1, 2, 3] });  
""")  
  
# 也可以注入一个 JS 文件  
page.add_init_script(path="./stealth.js")
```

---

## 14. Shadow DOM 处理

Shadow DOM 是 Web Components 的封装机制，普通 CSS 选择器穿不透，但 Playwright 的大多数语义定位器（`get_by_role`、`get_by_text` 等）**默认可以穿透 Shadow DOM**。

### 推荐：直接用语义定位器

```
# Playwright 的 get_by_* 方法默认穿透 Shadow DOM，直接用即可  
page.get_by_role("button", name="提交").click()  
page.get_by_label("用户名").fill("zhangsan")
```

### 如果必须用 CSS 选择器

```
# 用 >> 穿透 Shadow DOM（旧语法，仍然有效）  
page.locator("my-component >> .inner-button").click()  
  
# 或者用 pierce/ 前缀（更明确）  
page.locator("pierce/.inner-button").click()
```

### 通过 JS 访问 Shadow Root

```
# 当以上方法都不行时，用 JS 访问  
text = page.evaluate("""  
    () => {  
        const host = document.querySelector('my-custom-element');  
        return host.shadowRoot.querySelector('.price').innerText;  
    }  
""")
```

---

## 15. 对话框处理（alert/confirm/prompt）

浏览器原生弹窗（`window.alert`、`window.confirm`、`window.prompt`）会阻塞页面，必须提前注册监听器处理，否则超时报错。

### 自动接受所有弹窗

```
# 注册监听器，自动点"确认"  
page.on("dialog", lambda dialog: dialog.accept())  
  
# 然后触发弹窗的操作  
page.get_by_role("button", name="删除").click()
```

### 根据类型分别处理

```
def handle_dialog(dialog):  
    print(f"弹窗类型: {dialog.type}")   # alert / confirm / prompt / beforeunload  
    print(f"弹窗内容: {dialog.message}")  
  
    if dialog.type == "confirm":  
        dialog.accept()         # 点确认  
    elif dialog.type == "prompt":  
        dialog.accept("我的输入")  # 输入内容并确认  
    else:  
        dialog.dismiss()        # 点取消/关闭  
  
page.on("dialog", handle_dialog)  
page.get_by_role("button", name="危险操作").click()
```

> **注意：** 监听器要在触发弹窗之前注册，不然来不及处理。

---

## 16. 文件下载

### 捕获下载事件

```
import os  
  
# 方法一：用 expect_download 上下文管理器（推荐）  
with page.expect_download() as download_info:  
    page.get_by_role("button", name="导出 Excel").click()  
  
download = download_info.value  
  
# 保存到指定路径  
download.save_as("./exports/report.xlsx")  
  
# 或查看临时路径  
print(download.path())          # 临时文件路径  
print(download.suggested_filename)  # 服务器建议的文件名
```

### 批量下载

```
import os  
  
os.makedirs("./downloads", exist_ok=True)  
  
rows = page.locator("table tbody tr")  
count = rows.count()  
  
for i in range(count):  
    row = rows.nth(i)  
    filename = row.locator("td:nth-child(1)").inner_text().strip()  
  
    with page.expect_download() as dl_info:  
        row.get_by_role("link", name="下载").click()  
  
    dl = dl_info.value  
    dl.save_as(f"./downloads/{filename}.pdf")  
    print(f"已下载: {filename}")
```

### 配置默认下载路径

```
# 在 context 层面设置下载目录  
context = browser.new_context(accept_downloads=True)  
# 也可以在 launch 时指定  
browser = p.chromium.launch_persistent_context(  
    user_data_dir="./user_data",  
    accept_downloads=True,  
    downloads_path="./downloads",  
)
```

---

## 17. Cookie 管理

### 读取 Cookie

```
# 获取所有 cookies  
cookies = context.cookies()  
for c in cookies:  
    print(f"{c['name']} = {c['value']}")  
  
# 获取特定域名的 cookies  
cookies = context.cookies(["https://example.com"])
```

### 添加 / 修改 Cookie

```
context.add_cookies([  
    {  
        "name": "token",  
        "value": "abc123xyz",  
        "domain": "example.com",  
        "path": "/",  
        "httpOnly": True,  
        "secure": True,  
        "sameSite": "Lax",  
    }  
])
```

### 清除 Cookie

```
context.clear_cookies()
```

### 导入外部 Cookie（如从浏览器导出）

```
import json  
  
# 假设你有从 EditThisCookie 等工具导出的 JSON  
with open("cookies.json") as f:  
    raw = json.load(f)  
  
# 标准化字段（不同工具导出格式可能略有不同）  
cookies = [{  
    "name": c["name"],  
    "value": c["value"],  
    "domain": c["domain"],  
    "path": c.get("path", "/"),  
    "secure": c.get("secure", False),  
    "httpOnly": c.get("httpOnly", False),  
} for c in raw]  
  
context.add_cookies(cookies)
```

---

## 18. 代理配置

### 全局代理

```
browser = p.chromium.launch(  
    proxy={  
        "server": "http://proxy.example.com:8080",  
        "username": "user",       # 如果代理需要认证  
        "password": "pass",  
    }  
)
```

### Context 级别代理（同一浏览器不同账号走不同代理）

```
context_a = browser.new_context(proxy={"server": "http://proxy1:8080"})  
context_b = browser.new_context(proxy={"server": "http://proxy2:8080"})
```

### SOCKS5 代理

```
browser = p.chromium.launch(  
    proxy={"server": "socks5://127.0.0.1:1080"}  
)
```

### 绕过特定域名（不走代理）

```
browser = p.chromium.launch(  
    proxy={  
        "server": "http://proxy:8080",  
        "bypass": "localhost,127.0.0.1,internal.company.com"  
    }  
)
```

---

## 19. 网络拦截进阶

### 监听所有请求/响应（只读，不拦截）

```
# 监听请求  
page.on("request", lambda req: print(f"→ {req.method} {req.url}"))  
  
# 监听响应  
page.on("response", lambda resp: print(f"← {resp.status} {resp.url}"))  
  
# 只关注特定接口  
def on_response(response):  
    if "/api/list" in response.url:  
        print(response.json())  
  
page.on("response", on_response)  
page.goto("https://example.com")
```

### 拦截并修改请求（Mock 数据）

```
def mock_api(route):  
    # 如果是目标接口，返回假数据  
    if "/api/user" in route.request.url:  
        route.fulfill(  
            status=200,  
            content_type="application/json",  
            body='{"name": "测试用户", "role": "admin"}'  
        )  
    else:  
        route.continue_()  
  
page.route("**/*", mock_api)
```

### 拦截并修改请求头

```
def add_auth_header(route):  
    headers = {**route.request.headers, "Authorization": "Bearer my-token"}  
    route.continue_(headers=headers)  
  
page.route("**/api/**", add_auth_header)
```

### 拦截并修改响应体

```
def modify_response(route):  
    # 先让请求正常发出，拿到真实响应  
    response = route.fetch()  
    body = response.json()  
  
    # 修改数据  
    body["data"]["vip"] = True  
  
    # 把修改后的数据返回给页面  
    route.fulfill(response=response, body=json.dumps(body))  
  
page.route("**/api/profile", modify_response)
```

### 采集接口数据（不解析 DOM）

```
collected = []  
  
def capture_list_api(response):  
    if"/api/items"in response.url and response.status == 200:  
        data = response.json()  
        collected.extend(data.get("items", []))  
  
page.on("response", capture_list_api)  
  
# 翻页操作，每翻一页接口数据自动被收集  
for page_num in range(1, 11):  
    page.goto(f"https://example.com/list?page={page_num}")  
    page.wait_for_load_state("networkidle")  
  
print(f"共采集 {len(collected)} 条")
```

---

## 20. 分页采集模板

### 按钮翻页（有"下一页"按钮）

```
results = []  
  
whileTrue:  
    # 采集当前页  
    rows = page.locator("table tbody tr")  
    for i in range(rows.count()):  
        row = rows.nth(i)  
        results.append({  
            "name": row.locator("td:nth-child(1)").inner_text().strip(),  
            "value": row.locator("td:nth-child(2)").inner_text().strip(),  
        })  
  
    # 检查下一页按钮  
    next_btn = page.get_by_role("button", name="下一页")  
    ifnot next_btn.is_visible() ornot next_btn.is_enabled():  
        break  
  
    next_btn.click()  
    page.wait_for_load_state("networkidle")  
    page.locator("table tbody tr").first.wait_for(state="visible")  
  
print(f"共采集 {len(results)} 条")
```

### URL 参数翻页

```
results = []  
page_num = 1  
  
whileTrue:  
    page.goto(f"https://example.com/list?page={page_num}&size=50")  
    page.wait_for_load_state("networkidle")  
  
    rows = page.locator("table tbody tr")  
    count = rows.count()  
    if count == 0:  
        break# 没有数据了  
  
    for i in range(count):  
        row = rows.nth(i)  
        results.append(row.locator("td:nth-child(1)").inner_text().strip())  
  
    # 如果返回行数小于每页大小，说明是最后一页  
    if count < 50:  
        break  
  
    page_num += 1
```

### 接口翻页（直接拿 API 数据）

```
import json  
  
results = []  
page_num = 1  
  
whileTrue:  
    with page.expect_response("**/api/list**") as resp_info:  
        page.goto(f"https://example.com/list?page={page_num}")  
  
    resp = resp_info.value.json()  
    items = resp.get("data", {}).get("list", [])  
    total_pages = resp.get("data", {}).get("totalPages", 1)  
  
    results.extend(items)  
    print(f"第 {page_num}/{total_pages} 页，已采集 {len(results)} 条")  
  
    if page_num >= total_pages:  
        break  
    page_num += 1
```

---

## 21. 无限滚动采集

有些页面不用翻页，而是滚动到底部自动加载更多内容。

### 基础无限滚动

```
results = set()  
last_count = 0  
no_new_count = 0# 连续几次没有新数据  
  
while no_new_count < 3:  
    # 采集当前可见的所有项目  
    items = page.locator(".item-card").all()  
    for item in items:  
        results.add(item.inner_text().strip())  
  
    current_count = len(results)  
    if current_count == last_count:  
        no_new_count += 1  
    else:  
        no_new_count = 0  
    last_count = current_count  
  
    # 滚动到页面底部  
    page.evaluate("window.scrollTo(0, document.body.scrollHeight)")  
    page.wait_for_timeout(1500)  # 等待新内容加载  
  
print(f"共采集 {len(results)} 条")
```

### 等待新内容出现后再滚动（更稳定）

```
results = []  
  
whileTrue:  
    current_items = page.locator(".item-card")  
    current_count = current_items.count()  
  
    # 采集当前页的所有项  
    for i in range(current_count):  
        text = current_items.nth(i).inner_text().strip()  
        if text notin [r["text"] for r in results]:  
            results.append({"text": text})  
  
    # 滚动到最后一个元素  
    current_items.last.scroll_into_view_if_needed()  
  
    # 等待新内容出现（数量增加）  
    try:  
        page.wait_for_function(  
            f"document.querySelectorAll('.item-card').length > {current_count}",  
            timeout=5000  
        )  
    except:  
        # 超时说明没有更多数据了  
        print("已到底部，采集完毕")  
        break  
  
print(f"共采集 {len(results)} 条")
```

---

## 22. 复杂组件处理（日期选择器/自定义下拉框）

### 日期选择器

原生 `<input type="date">` 直接用 `fill`：

```
page.get_by_label("开始日期").fill("2024-01-15")  # 格式：YYYY-MM-DD
```

基于 JS 库的日期选择器（无法直接 fill），用点击方式操作：

```
# 点击日期输入框打开日历  
page.get_by_placeholder("选择日期").click()  
  
# 等待日历弹出  
page.locator(".datepicker-popup").wait_for(state="visible")  
  
# 如果需要跳转到特定月份  
whileTrue:  
    month_label = page.locator(".datepicker-header .month").inner_text()  
    if"2024年3月"in month_label:  
        break  
    page.locator(".datepicker-prev").click()  
  
# 点击目标日期  
page.get_by_role("gridcell", name="15").click()
```

更简单的方案——直接用 JS 设置值：

```
# 找到背后隐藏的真实 input，直接设置值  
page.evaluate("""  
    () => {  
        const input = document.querySelector('input[name="startDate"]');  
        input.value = '2024-03-15';  
        input.dispatchEvent(new Event('change', { bubbles: true }));  
    }  
""")
```

### 自定义下拉框（非原生 `<select>`）

很多 UI 组件库（Ant Design、Element UI 等）的下拉框是 `<div>` 模拟的：

```
# 点击触发下拉框展开  
page.locator(".custom-select").click()  
  
# 等待选项列表出现  
page.locator(".select-dropdown").wait_for(state="visible")  
  
# 点击目标选项  
page.get_by_role("option", name="北京市").click()  
  
# 或者用 get_by_text 找选项  
page.locator(".select-dropdown").get_by_text("北京市").click()
```

### 富文本编辑器（如 Quill、TinyMCE）

```
# Quill 编辑器  
editor = page.locator(".ql-editor")  
editor.click()  
editor.fill("这是输入的内容")  
  
# TinyMCE（通常在 iframe 里）  
frame = page.frame_locator("iframe#content_ifr")  
frame.locator("#tinymce").fill("这是输入的内容")  
  
# 如果 fill 不生效，用 JS 注入  
page.evaluate("""  
    () => {  
        // Quill 实例  
        const quill = document.querySelector('.ql-editor').__quill;  
        quill.setText('这是内容');  
    }  
""")
```

### 滑块验证码（拖拽类）

```
# 找到滑块  
slider = page.locator(".slider-btn")  
box = slider.bounding_box()  
  
# 模拟拖拽（分步移动，模拟真人）  
page.mouse.move(box["x"] + box["width"] / 2, box["y"] + box["height"] / 2)  
page.mouse.down()  
  
# 分多步移动，不要一步到位  
import random  
target_x = box["x"] + 280# 目标位置（需根据实际调整）  
current_x = box["x"] + box["width"] / 2  
step = (target_x - current_x) / 20  
  
for i in range(20):  
    current_x += step + random.uniform(-1, 1)  # 加入随机抖动  
    page.mouse.move(current_x, box["y"] + box["height"] / 2 + random.uniform(-1, 1))  
    page.wait_for_timeout(random.randint(10, 30))  
  
page.mouse.up()
```

---

## 23. 数据导出（JSON/CSV/Excel）

### 导出为 JSON

```
import json  
  
results = [{"name": "张三", "age": 28}, {"name": "李四", "age": 32}]  
  
with open("output.json", "w", encoding="utf-8") as f:  
    json.dump(results, f, ensure_ascii=False, indent=2)  
  
print("已导出 output.json")
```

### 导出为 CSV

```
import csv  
  
results = [{"name": "张三", "score": 95}, {"name": "李四", "score": 87}]  
  
with open("output.csv", "w", newline="", encoding="utf-8-sig") as f:  
    # utf-8-sig 让 Excel 能正确识别中文  
    writer = csv.DictWriter(f, fieldnames=["name", "score"])  
    writer.writeheader()  
    writer.writerows(results)  
  
print("已导出 output.csv")
```

### 导出为 Excel（openpyxl）

```
# pip install openpyxl  
from openpyxl import Workbook  
from openpyxl.styles import Font, PatternFill, Alignment  
  
results = [  
    {"名称": "产品A", "价格": 199, "库存": 50},  
    {"名称": "产品B", "价格": 299, "库存": 30},  
]  
  
wb = Workbook()  
ws = wb.active  
ws.title = "采集结果"  
  
headers = list(results[0].keys())  
  
# 写表头（加粗+背景色）  
for col, header in enumerate(headers, 1):  
    cell = ws.cell(row=1, column=col, value=header)  
    cell.font = Font(bold=True)  
    cell.fill = PatternFill("solid", fgColor="DDEEFF")  
    cell.alignment = Alignment(horizontal="center")  
  
# 写数据  
for row_idx, record in enumerate(results, 2):  
    for col_idx, key in enumerate(headers, 1):  
        ws.cell(row=row_idx, column=col_idx, value=record[key])  
  
# 自动列宽  
for col in ws.columns:  
    max_len = max(len(str(cell.value or"")) for cell in col)  
    ws.column_dimensions[col[0].column_letter].width = max_len + 4  
  
wb.save("output.xlsx")  
print("已导出 output.xlsx")
```

### 增量写入（大量数据防止丢失）

```
import csv, os  
  
OUTPUT_FILE = "output.csv"  
HEADERS = ["name", "price", "url"]  
  
# 文件不存在时写表头  
write_header = not os.path.exists(OUTPUT_FILE)  
  
with open(OUTPUT_FILE, "a", newline="", encoding="utf-8-sig") as f:  
    writer = csv.DictWriter(f, fieldnames=HEADERS)  
    if write_header:  
        writer.writeheader()  
  
    # 每采集一条立即写入，不怕中途崩溃  
    for item in scrape_items():  
        writer.writerow(item)  
        f.flush()  # 强制刷新缓冲区
```

---

## 24. 调试工具

### Codegen：自动生成代码

录制你的浏览器操作，自动生成 Playwright 代码，非常适合快速上手一个新网站。

```
# 录制操作，输出 Python 代码  
playwright codegen https://example.com --output script.py  
  
# 指定语言  
playwright codegen https://example.com --lang python  
  
# 带已保存的登录态录制  
playwright codegen https://example.com --load-storage session.json
```

### Inspector：交互式调试

```
# 代码里加上这行，运行时会暂停并打开 Inspector 窗口  
page.pause()
```

或者通过环境变量启动：

```
PWDEBUG=1 python your_script.py
```

Inspector 里可以：逐步执行、实时查看元素、测试定位器表达式。

### Trace Viewer：事后回放

Trace 会记录操作的每一步截图、网络请求、DOM 快照，出错时查 trace 比看日志直观得多。

```
# 开启 trace 记录  
context.tracing.start(screenshots=True, snapshots=True, sources=True)  
  
# ... 你的操作 ...  
  
# 保存 trace  
context.tracing.stop(path="trace.zip")
```

```
# 在浏览器里回放 trace  
playwright show-trace trace.zip
```

### 页面控制台输出

```
# 监听页面 console.log 输出（调试 JS 逻辑很有用）  
page.on("console", lambda msg: print(f"[console.{msg.type}] {msg.text}"))  
  
# 监听页面报错  
page.on("pageerror", lambda err: print(f"[JS错误] {err}"))
```

---

## 25. 性能优化

### 按需加载资源（最有效）

```
# 屏蔽非必要资源，速度可以提升 2~5 倍  
BLOCKED_TYPES = {"image", "media", "font", "stylesheet"}  
  
def block_unnecessary(route):  
    if route.request.resource_type in BLOCKED_TYPES:  
        route.abort()  
    else:  
        route.continue_()  
  
page.route("**/*", block_unnecessary)
```

### 控制并发数量

```
import asyncio  
from playwright.async_api import async_playwright  
  
asyncdef process_url(browser, semaphore, url):  
    asyncwith semaphore:  # 限制同时运行的协程数  
        context = await browser.new_context()  
        page = await context.new_page()  
        try:  
            await page.goto(url)  
            # ... 处理逻辑  
        finally:  
            await context.close()  
  
asyncdef main():  
    urls = [f"https://example.com/item/{i}"for i in range(100)]  
    semaphore = asyncio.Semaphore(5)  # 最多 5 个并发  
  
    asyncwith async_playwright() as p:  
        browser = await p.chromium.launch(headless=True)  
        await asyncio.gather(*[process_url(browser, semaphore, url) for url in urls])  
        await browser.close()  
  
asyncio.run(main())
```

### 复用 Page 而不是每次新建

```
# 不好的做法：每个 URL 新建一个 page  
for url in urls:  
    page = context.new_page()  
    page.goto(url)  
    # ...  
    page.close()  # 也要关  
  
# 好的做法：复用同一个 page  
page = context.new_page()  
for url in urls:  
    page.goto(url)  
    # ...  
# 最后统一关  
page.close()
```

> 注意：如果页面跳转后状态有影响（如弹窗未关闭），复用 page 可能产生问题，按需选择。

### 使用 `domcontentloaded` 代替 `networkidle`

```
# networkidle 要等所有请求完成，通常需要 2~5 秒  
page.goto(url, wait_until="networkidle")  # 慢  
  
# domcontentloaded 只等 DOM 解析完，通常不到 1 秒  
page.goto(url, wait_until="domcontentloaded")  # 快  
# 然后再用更精确的元素等待  
page.locator(".data-table").wait_for(state="visible")
```

### 持久化 Context（保留缓存）

```
# launch_persistent_context 会保留磁盘缓存，重复访问同一网站更快  
context = p.chromium.launch_persistent_context(  
    user_data_dir="./browser_profile",  
    headless=True,  
    accept_downloads=True,  
)  
page = context.new_page()  
# 注意：persistent context 不需要再调用 browser.new_context()
```

### 性能 Checklist

| 场景 | 优化方案 |
| --- | --- |
| 采集大量页面 | 屏蔽 image/font/css + async 并发 |
| 只需要 API 数据 | 直接用 `expect_response` 拿接口数据，不解析 DOM |
| 需要登录 | 一次登录保存 session.json，后续复用 |
| 重复访问同站 | persistent context 利用磁盘缓存 |
| 跑在服务器上 | headless=True + no-sandbox + 关闭 GPU |
| 调试慢 | headless=False + slow\_mo=300 + page.pause() |