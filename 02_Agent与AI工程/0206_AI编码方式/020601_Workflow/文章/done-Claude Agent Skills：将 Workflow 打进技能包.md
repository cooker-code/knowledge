> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: Claude Agent Skills：将 Workflow 打进技能包
author: 奇舞精选
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg4MTYwMzY1Mw==&mid=2247517480&idx=1&sn=ebe6e5ef0d8675e721b93035ff08201b&chksm=ce709bd987f028d08979c34023e75fb4e44fae0154faa95f23a57378390776cfca367432f05a&mpshare=1&scene=24&srcid=1111Q5Vc5Md6bYhfqxOkgb6P&sharer_shareinfo=e0bf372d5da0f657211a0a06293d048c&sharer_shareinfo_first=e0bf372d5da0f657211a0a06293d048c#rd
---

在这个 AI 技术蓬勃发展的时代，每天都有新东西。每次浏览社交媒体，总能看到自媒体和 KOL 们极力的鼓吹着新产品、新技术，满口「革命」和「未来」，起初真的给我整 FOMO 了，但后来越来越感觉到厌烦，「Function Calling」、「Tool Use」、「MCP」、「Agent」、「Agentic AI」 …… 还有一大堆别的 AI 术语，你很难分清这些概念到底代表了技术的实质突破，还是同一件事被营销话术反复包装，跟前端火的那些年如出一辙，不禁让人感叹 AI 领域的概念也是通货膨胀了。

在 10 月中旬，Anthropic  悄然推出了 Agent Skills（后简称 Skills），不出意外，技术社区又炸锅了。有人发出质疑，认为又在炒概念，也有人认为 Skills 虽然简单，但却是比 MCP 还要强大的玩意。

最近我花了些时间深入研究 Skills，查阅官方文档，看了些 Youtube 视频，动手写了几个示例。结论是它没有某些 KOL 吹得那么神，但确实有其独到之处。而这背后，潜藏着 Anthropic 试图在 MCP 后进一步标准化 LLM 应用的野心。

本文将分享我对 Skills 的理解和思考，希望能为想要了解这项技术的朋友提供一些参考，有说的不对的地方欢迎大家批评指正。

## Why Skills？

实际上我认为 Skill 想要解决的问题很简单，解决 Prompt Engineering 的复用问题。

试想这样一个场景，你经常写技术文章，每次都要提醒 AI：

* 用轻松的语气，不要输出太多像列表一样的结构化数据
* 多用代码示例，少讲抽象概念
* 每段不超过 300 字

说实在这还是挺麻烦的，但是合理使用 AI 我们的还是能做的到的，但是你注意到了吗？这个 Workflow 很难迁移到你的下一次 AI 对话中，因为这些 Prompt 你要重新输入一遍。

Anthropic 给出的解法就是 Skills，Skills 就是把做某些重复性任务的 Prompt、资源文件（脚本、知识库之类的）打包在一起，在你的下一次对话中，像「Tool Use」一样按需载入，模型自主决定是否按照你给出的 Skills 行动。嘿，如果能够按照预期触发，我们的 workflow 就能变得更简单好用，不用准备那么多提示词了对吧～

当然 Skills 作为 Anthropic 也可以在社区分享，就像 MCP 刚出来就有了一堆 MCP Hub，现在也出现了很多 Skills Hub。

从技术角度看，Skills 确实有用，但别高估它。有人说 Skills 会革了 n8n、Dify 这些 Agent 平台的命，我是不太信的。Skills 本质上就是把你的 Prompt 和 workflow 打个包，让它能复用。这解决的是不想每次都重复输入指令的问题。但 n8n、Dify 这些平台解决的是什么？是要把十几个步骤串起来，中间还有条件判断、错误处理、定时触发这种复杂编排问题。这俩根本不是一个维度的东西。

Skills 适合那种重复性的、相对标准化的单个任务，比如代码审查、按规范生成文档、检查数据格式这种。你定义好规则，以后每次 Claude 自动按这套规则来，挺方便。但要是你的工作流程是多步骤、多服务的协同，Skills 就玩不转了。而且这些 Agent 平台有可视化界面，非技术人员也能配置，Skills 你还得写代码打包，门槛摆在那。

所以别被营销话术忽悠了。Skills 确实降低了一些简单场景的门槛，以前可能要搭个小 Agent 的事，现在一个 Skill 就搞定。但它不是什么革命性的东西，更不可能把这些平台干掉，顶多算个补充。

### Skills vs MCP

很多人搞不清楚为何有了 MCP 还需要 Skills，这里插一段解释下。

实际上两者的核心目标不一致，MCP 的目标是帮助 AI 连接外部世界，读取外部世界的数据，向外部世界输出影响，Anthropic 将其描述为「USB Hub」，强调的连接能力。而 Skills 的目标是封装特定领域的知识和 workflow，专注于执行特定任务，要比喻的话也是「瑞士军刀」。这俩一个主外，一个主内，是可以配合使用的。

### Skills vs Projects

还有一个相近的功能叫做 Projects，在 Claude 和 ChatGPT 上都有类似功能，简单来说就是可以创建一个 workspace，我们把做某件事（比如写现在的这篇文章）作为目标，将完成这件事的 Prompt 和 Context 聚拢在一起，AI 会记住项目背景，和当前的进度，帮我们延续思路，不用每次重新解释我们上次聊到哪了。

但 Skills 聚焦在「做事的方法」，更像是一套可复用的 workflow。它不关心你在做哪个项目，而是帮你保持统一的做事风格或标准。而且 Projects 通常是个人用的，没法直接分享；Skills 则可以打包成模板，分享给团队一起用。

从这个角度看，Projects 管项目，Skills 管方法，一个是工作空间，一个是工具箱。

## 什么是 Skills？

说了这么多，Skills 到底长什么样？它是如何实现「Prompt 复用」和「按需加载」的？接下来我们看看 Skills 的技术细节。

如前文所说 Skills 实际上就是包含了 Prompt 和资源文件的包，用于告知 AI 如何完成特定任务，并在不同会话之间复用，其基本结构是这样的：

```
my-skill.zip
  └── my-skill/
      ├── SKILL.md （必要）
      └── resources/（非必要的其他文件）
```

Anthropic 规定 Skill 包必须是一个 zip 包，且里面包含一个同名的文件夹，文件夹里面才是具体内容，其中的 `SKILL.md` 也是必须的描述了 Skill 的基本信息，并且里面内容也有特殊的规定，而其他文件就自己根据需求随意添加就好了。

`SKILL.md` 类似这样：

```
---
name: your-skill-name
description: Brief description of what this Skill does and when to use it
---

# Your Skill Name

## Instructions
[Clear, step-by-step guidance for Claude to follow]

## Examples
[Concrete examples of using this Skill]
```

可以看到 `SKILL.md`  分两部分，一部分是 YAML 的头部，用于存储元数据，其 `name` 和 `description` 是必须的。下半部分的 Markdown 就可以随意发挥了。

Claude 为 Skills 提供了代码执行环境。在 Claude.ai 中是隔离的虚拟机，而在 Claude Code 中则是直接在用户的本地环境执行，并提供了一套叫做 **渐进式披露**（progressive disclosure） 的调用策略来控制 Context。

1. Level 1: - 元数据 (始终加载)：也就是我们刚刚说的  `SKILL.md` 中的 YAML 头，会在 Claude 启动时载入并附加在 System Prompt 中，和「Tool Use」差不多
2. Level 2 - 指令（调用时加载）：也就是我们刚刚说的  `SKILL.md` 中的 Markdown 部分，当 Claude 确认要调用某个 Skill，就会载入全部的 `SKILL.md` 内容
3. Level 3 - 资源文件或代码（按需加载）：当指令让 Claude 采取了某些行为要访问这些文件或是执行代码时，才会载入这些资源文件

Anthropic 在 Claude 上提供了一些预制的技能可以开启，也支持用户自己上传 zip 包来使用：

## 实战开发

理解了 Skills 的原理，最好的学习方式就是动手写一个。虽说我们可以直接让 Claude 根据规范为我们创建一个 Skill（确实很方便），但自己从零开始写一遍，能更深入理解 Skills 的设计思想。

接下来我们创建一个实用的 Skill：**上传图床助手** ：平时在写完技术文章之后，发布的过程其实挺麻烦的，因为我经常使用 Ulysses 写作，导出为 Markdown 后，很多图片引用实际是在本地的，如果要发布一般还是要上传到图床上才能直接用。另外，我引用的一些图片的格式很多地方不支持，需要统一成 png 来上传。

我们的目录结构是这样：

```
article-publisher/
├── SKILL.md                      # Skill 定义和工作流程
└── scripts/
    └── upload_images.py      # 上传图片到图床脚本
```

首先，创建 `SKILL.md`，规定整个技能的定义和执行步骤。这里我用了比较直白的写法，虽然不完全符合官方的最佳实践（后面会说为什么），但为了稳定性做了一些妥协：

```
---
name: upload-article-images
description: Upload local images from Markdown files to image hosting. When user says "上传图床" or "upload images", immediately run the script at ~/.claude/skills/upload-article-images/scripts/upload_images.py
---

# Upload Article Images

To upload images from a Markdown file, execute:

`bash
python3 ~/.claude/skills/upload-article-images/scripts/upload_images.py <markdown_file_path>
`

## Critical: What NOT to do

- Do NOT read the Markdown file first
- Do NOT list or check image files
- Do NOT check if the script exists
- Do NOT search for files
- Do NOT create a new script
- Do NOT write any Python code
- Do NOT ask the user which image hosting service to use

## What to do

Run the command above. That's it.

The script at `~/.claude/skills/upload-article-images/scripts/upload_images.py` already exists and handles everything.

## Example

User: "帮我处理 index.md，上传图片到图床"

**Correct action:**
`bash
python3 ~/.claude/skills/upload-article-images/scripts/upload_images.py index.md
`

**Wrong actions:**
- ❌ Reading index.md first
- ❌ Listing PNG/TIFF files
- ❌ Checking if script exists
- ❌ Creating upload_images.py
- ❌ Asking about image hosting service

## Dependencies

If error: `pip install requests Pillow`
```

首先，`description` 里明确写了触发词（"上传图床"、"upload images”），然后直接在开头就给出执行命令，避免 Claude 自己瞎猜，还用了大量的 "Do NOT" 来约束行为（虽然官方不推荐，但实测这样更稳定）

之后的 `upload_images.py`  写我们主要的图片提取、上传、替换逻辑：

```
#!/usr/bin/env python3
"""
图片上传到图床并替换 Markdown 链接
使用方法：python upload_images.py <文章.md>
"""

import re
import os
import sys
import requests
from pathlib import Path
from PIL import Image
import tempfile


# 支持的图片格式（图床支持的格式）
SUPPORTED_FORMATS = {".png", ".jpg", ".jpeg", ".gif", ".webp", ".bmp"}

# 需要转换的格式
CONVERT_FORMATS = {".tiff", ".tif", ".svg", ".ico", ".heic"}


def convert_image_format(file_path):
    """
    将不支持的图片格式转换为 PNG

    Args:
        file_path: 原始图片路径

    Returns:
        str: 转换后的图片路径（如果需要转换），否则返回原路径
        bool: 是否进行了转换
    """
    file_path = Path(file_path)
    ext = file_path.suffix.lower()

    # 如果格式已支持，直接返回
    if ext in SUPPORTED_FORMATS:
        return str(file_path), False

    # 需要转换
    if ext in CONVERT_FORMATS or ext not in SUPPORTED_FORMATS:
        try:
            # 打开图片
            img = Image.open(file_path)

            # 如果是 RGBA 模式，保持透明度
            # 如果不是，转换为 RGB（PNG 支持两种模式）
            if img.mode in ("RGBA", "LA", "P"):
                # 保持透明度
                pass
            elif img.mode != "RGB":
                img = img.convert("RGB")

            # 创建临时 PNG 文件
            temp_dir = tempfile.gettempdir()
            temp_file = Path(temp_dir) / f"{file_path.stem}_converted.png"

            # 保存为 PNG
            img.save(temp_file, "PNG")

            print(f"   🔄 已转换 {ext} → .png")
            return str(temp_file), True

        except Exception as e:
            print(f"   ⚠️  格式转换失败: {e}")
            return str(file_path), False

    return str(file_path), False


def upload_file(file_path):
    """
    上传文件到图床

    Args:
        file_path: 文件路径

    Returns:
        str: 上传后的文件 URL

    Raises:
        Exception: 上传失败时抛出异常
    """
    url = "https://xxx.com/upload" # 这里记得替换成你的图床 API

    with open(file_path, "rb") as f:
        files = {"file": f}
        response = requests.post(url, files=files)

    if response.status_code != 200:
        raise Exception(f"上传失败，状态码: {response.status_code}")

    result = response.json()
    file_url = result.get("data", {}).get("url")

    if not file_url:
        error_msg = result.get("context", {}).get("message", "上传失败")
        raise Exception(error_msg)

    return file_url


def extract_local_images(md_content):
    """提取 Markdown 中的本地图片路径"""
    # 匹配 ![alt](path) 格式
    pattern = r"!\[([^\]]*)\]\(([^)]+)\)"
    matches = re.findall(pattern, md_content)

    local_images = []
    for alt_text, img_path in matches:
        # 排除已经是 URL 的图片
        if not img_path.startswith(("http://", "https://")):
            local_images.append((alt_text, img_path))

    return local_images


def replace_image_links(md_file, url_mapping):
    """替换 Markdown 中的图片链接"""
    with open(md_file, "r", encoding="utf-8") as f:
        content = f.read()

    for old_path, new_url in url_mapping.items():
        content = content.replace(f"]({old_path})", f"]({new_url})")

    with open(md_file, "w", encoding="utf-8") as f:
        f.write(content)


def main():
    if len(sys.argv) < 2:
        print("用法: python upload_images.py <Markdown文件>")
        sys.exit(1)

    md_file = sys.argv[1]

    if not os.path.exists(md_file):
        print(f"❌ 错误：文件不存在: {md_file}")
        sys.exit(1)

    print("🔍 扫描文章中的本地图片...")

    with open(md_file, "r", encoding="utf-8") as f:
        content = f.read()

    local_images = extract_local_images(content)

    if not local_images:
        print("✅ 没有找到需要上传的本地图片")
        return

    print(f"📤 找到 {len(local_images)} 张本地图片，开始上传...\n")

    url_mapping = {}
    md_dir = Path(md_file).parent
    success_count = 0
    temp_files = []  # 存储需要清理的临时文件

    for alt_text, img_path in local_images:
        # 转换为绝对路径
        full_path = (md_dir / img_path).resolve()

        if not full_path.exists():
            print(f"⚠️  跳过不存在的文件: {img_path}")
            continue

        try:
            # 尝试转换格式（如果需要）
            converted_path, was_converted = convert_image_format(str(full_path))
            if was_converted:
                temp_files.append(converted_path)

            # 上传图片
            new_url = upload_file(converted_path)
            url_mapping[img_path] = new_url
            success_count += 1
            print(f"✅ {img_path}")
            print(f"   → {new_url}\n")
        except Exception as e:
            print(f"❌ {img_path} 上传失败: {e}\n")

    if url_mapping:
        print("🔄 正在替换 Markdown 中的图片链接...")
        replace_image_links(md_file, url_mapping)
        print(f"✅ 完成！成功上传并替换 {success_count} 张图片的链接")
    else:
        print("❌ 没有成功上传任何图片")

    # 清理临时文件
    if temp_files:
        print("\n🧹 清理临时文件...")
        for temp_file in temp_files:
            try:
                Path(temp_file).unlink()
            except Exception as e:
                print(f"   ⚠️  清理失败 {temp_file}: {e}")


if __name__ == "__main__":
    main()
```

注意，这里 `https://xxx.com/upload` 是一个示例地址，你要修改成自己的图床 API。

我们将我们的 skill 整个文件夹移动到 `~/.claude/skills` 这样 Claude Code 就可以直接使用了。

我导出一份文档，使用 Cladue Code 测试一下看看：可以看到虽然经历了一些波折，但还是成功了。

检查原文可以看到，符合我们的预期，链接已经替换成功了！

## 写在最后

其实本文前面写得很快，最难的部分就是实战。不知道是我的 Prompt 水平不够还是 Skills 这个功能还不够成熟，即使按照官方推荐的写法，让 Claude 调用某个脚本，它还是经常选择自己写一个脚本而不是用我内置好的。

本来我规划了一个更复杂的 Skill：润色文章、上传图片、生成摘要和标签、再设计个封面，一套流程自动化。但测试下来发现稳定性太差，经常在某一步卡住或者跳过关键步骤，最后只好简化成现在这个单一功能的版本。

给想尝试写 Skills 的朋友几个建议：

1. **慢慢迭代**：别一上来就搞复杂流程，先做单一功能的 Skill
2. **description 可以适当写详细些**：跟官方推荐的不一样，但把一些重要内容放在这里似乎最有效
3. **英文 Prompt 效果更好**：我自己测试下来，感觉英文 Prompt 更稳定、速度也更快，大家也可以多对比下
4. **降低期望**：Skills 目前还是个新东西，还不够稳定好用

Skills 这东西设计理念不错，但目前的效果实在不尽如人意。希望 Anthropic 能在稳定性上多下功夫。不过话说回来，即使有这些问题，对于一些简单重复的任务，Skills 还是能省不少事的。

大家如果有什么使用 Skills 的小技巧，欢迎在评论区跟大家分享～

## 参考资料

* Claude Agent Skills 官方文档：https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview
* Anthropic Blog- Agent Skills：https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
* How to create custom Skills ：https://support.claude.com/en/articles/12512198-how-to-create-custom-skills
* The REAL POWER of Claude Agent SKILLS (Why Most Are Missing It) | Claude Skills Explained ：https://www.youtube.com/watch?v=m-5DjcgFmfQ
* Claude Agent Skills - 全新的技能包 ：https://www.youtube.com/watch?v=KBslHtyHRaU

-END -

**如果您关注前端+AI 相关领域可以扫码进群交流**

添加小编微信进群😊

## 关于奇舞团

奇舞团是 360 集团最大的大前端团队，非常重视人才培养，有工程师、讲师、翻译官、业务接口人、团队 Leader 等多种发展方向供员工选择，并辅以提供相应的技术力、专业力、通用力、领导力等培训课程。奇舞团以开放和求贤的心态欢迎各种优秀人才关注和加入奇舞团。