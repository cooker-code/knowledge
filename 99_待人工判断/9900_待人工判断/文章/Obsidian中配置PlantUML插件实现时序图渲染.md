---
title: Obsidian中配置PlantUML插件实现时序图渲染
author: Agent工程化
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg2MjIwODc3Mw==&mid=2247518319&idx=1&sn=7e777feac0a2ef66c54a70a600610c3b&chksm=cfa5495f83715ea27ac0bdc04ff454898bf0f3a2a22e6e3fbf7a2a24e7b5e3064ac04199ae3c&mpshare=1&scene=24&srcid=0422IWBZOuRJyM659fzo0vXV&sharer_shareinfo=6e2f37912cf2ca4e234f4b1763e8b170&sharer_shareinfo_first=6e2f37912cf2ca4e234f4b1763e8b170#rd
---

# 目录

* 1. 背景[#1-背景]
* 2. 问题复现[#2-问题复现]
* 3. 解决方案[#3-解决方案]

+ 3.1 安装PlantUML插件[#31-安装plantuml插件]
+ 3.2 启用插件[#32-启用插件]
+ 3.3 文件格式转换[#33-文件格式转换]
+ 3.4 解决渲染空白问题[#34-解决渲染空白问题]

* 4. 最终方案总结[#4-最终方案总结]
* 5. 避坑指南[#5-避坑指南]
* 6. 最佳实践[#6-最佳实践]
* 参考文献[#参考文献]

# 1. 背景

在使用 AI 编程助手进行代码分析时，常需要生成 PlantUML 时序图来可视化系统架构和交互流程。生成的 `.puml` 源文件存储在 Obsidian vault 中，需要在 Obsidian 中直接预览这些时序图，实现"源码可编辑 + 渲染可预览"的工作流。

**环境信息：**

* Obsidian 版本：v1.5+
* Java 环境：JDK 17+
* PlantUML 源文件：`.puml` 格式（含中文标注）
* 插件版本：obsidian-plantuml v1.8.0

# 2. 问题复现

整个配置过程遇到了四个递进式问题：

**问题一：.puml 文件在 Obsidian 中不可见**  
Obsidian 默认只显示 `.md` 等标准文件类型，`.puml` 文件不会出现在文件列表中。需要修改 `app.json` 配置：

* 1
* 2
* 3
* 4

```
{  "showUnsupportedFiles": true,  "attachmentFolderPath": "images"}
```

**问题二：点击 .puml 文件无法预览**  
即使文件可见，Obsidian 原生不支持渲染 PlantUML 语法，需要安装社区插件。

**问题三：安装插件后仍不渲染**  
`obsidian-plantuml` 插件只渲染 Markdown 中的 ```` ```plantuml `````代码块，不支持独立的`.puml` 文件。

**问题四：转换为 .md 后渲染空白**  
代码块 ```` ```plantuml ```` ` 之前缺少空行，导致 Obsidian 无法正确识别代码块边界。

# 3. 解决方案

## 3.1 安装PlantUML插件

通过命令行直接下载插件文件到 Obsidian 插件目录：

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12

```
# 创建插件目录mkdir -p ".obsidian/plugins/obsidian-plantuml"  
# 下载插件文件curl -sL "https://github.com/joethei/obsidian-plantuml/releases/latest/download/main.js" \  -o ".obsidian/plugins/obsidian-plantuml/main.js"  
curl -sL "https://github.com/joethei/obsidian-plantuml/releases/latest/download/manifest.json" \  -o ".obsidian/plugins/obsidian-plantuml/manifest.json"  
curl -sL "https://github.com/joethei/obsidian-plantuml/releases/latest/download/styles.css" \  -o ".obsidian/plugins/obsidian-plantuml/styles.css"
```

**下载完成后验证文件完整性：**

* 1
* 2
* 3
* 4

```
ls -la .obsidian/plugins/obsidian-plantuml/# main.js       ~1MB# manifest.json ~255B# styles.css    ~724B
```

## 3.2 启用插件

创建 `community-plugins.json` 注册插件：

* 1
* 2
* 3

```
[  "obsidian-plantuml"]
```

重启 Obsidian 后，会弹出安全提示，点击 **"Trust author and enable"** 完成启用。

## 3.3 文件格式转换

`obsidian-plantuml` 插件只识别 Markdown 文件中的 PlantUML 代码块，不支持独立 `.puml` 文件。需要两步转换：

**第一步：文件重命名**

* 1
* 2
* 3

```
for f in *.puml; do  mv "$f" "${f%.puml}.md"done
```

**第二步：内容包裹**  
在文件头部和尾部分别添加代码块标记。转换前后对比：

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12
* 13
* 14
* 15

```
转换前（.puml 格式）：  
@startumltitle 时序图标题Alice -> Bob: Hello@enduml  
转换后（.md 格式）：  
```plantuml@startumltitle 时序图标题Alice -> Bob: Hello@enduml```
```

批量转换脚本：

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9

```
for f in *.md; do  content=$(cat "$f")  if [[ "$content" == @startuml* ]]; then    echo '```plantuml' > "$f.tmp"    cat "$f" >> "$f.tmp"    echo '```' >> "$f.tmp"    mv "$f.tmp" "$f"  fidone
```

## 3.4 解决渲染空白问题

转换完成后，打开文件切换到阅读视图仍显示空白。原因是 ```` ```plantuml ```` ` 代码块之前缺少空行，Obsidian 解析器无法正确识别代码块边界。

**修复方法：** 在文件顶部添加空行。

* 1
* 2
* 3

```
for f in ch*.md; do  sed -i '1s/^/\n/' "$f"done
```

**修复后的正确格式：**

* 1
* 2
* 3
* 4
* 5
* 6
* 7

```
（空行）```plantuml@startumltitle 时序图标题...@enduml```
```

# 4. 最终方案总结

完整的文件结构：

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12

```
vault/├── .obsidian/│  ├──app.json#showUnsupportedFiles: true│   ├── community-plugins.json # ["obsidian-plantuml"]│   └── plugins/│       └── obsidian-plantuml/ # 插件文件├── images/#PNG 渲染图片│   ├── diagram_01.png│   └── ...└── plantuml/#PlantUML源码（.md 格式）    ├── diagram_01.md    └── ...
```

单个 PlantUML 源码文件的正确格式：

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12
* 13
* 14
* 15
* 16
* 17
* 18
* 19
* 20
* 21

```
```plantuml@startumltitle 标题autonumber  
skinparam defaultFontName Microsoft YaHei  
skinparam sequence {    ParticipantBackgroundColor #E3F2FD    ParticipantBorderColor #1976D2    ParticipantFontColor #333333    ArrowColor #1976D2}  
participant "参与者A" as Aparticipant "参与者B" as B  
A -> B : 请求B --> A : 响应@enduml```
```

关键要素：

* 文件扩展名必须为 `.md`（不是 `.puml`）
* ```` ```plantuml ```` ` 之前必须有至少一个空行
* 使用 `skinparam defaultFontName Microsoft YaHei` 确保中文正常渲染

# 5. 避坑指南

**坑点一：直接使用 .puml 文件**  
`obsidian-plantuml` 插件不支持独立 `.puml` 文件渲染。必须转换为 `.md` 并用 ```` ```plantuml ```` ` 代码块包裹。

**坑点二：代码块前缺少空行**  
Obsidian 的 Markdown 解析器要求代码块之前有空行分隔。缺少空行会导致代码块不被识别，渲染结果为空白。这是最常见的"安装了插件但不出图"的原因。

**坑点三：中文字体乱码**  
PlantUML 默认字体不支持中文。必须在源码中声明：

同时生成 PNG 时使用 UTF-8 编码：

* 1

```
java -Dfile.encoding=UTF-8 -jar plantuml.jar -charset UTF-8 "*.puml"
```

**坑点四：插件依赖网络**  
`obsidian-plantuml` 插件默认使用公共 PlantUML 服务器（`www.plantuml.com/plantuml`）渲染。在内网环境下，需要在插件设置中配置本地渲染模式或自建 PlantUML 服务器。

# 6. 最佳实践

**源码与图片分离管理**

* 1
* 2

```
plantuml/    ← 可编辑的 PlantUML 源码（.md 格式，Obsidian 可预览）images/      ← 渲染后的 PNG 图片（供 Markdown 文档引用）
```

* `plantuml/` 目录存放 `.md` 格式的 PlantUML 源码，既可在 Obsidian 中预览，又可用命令行生成 PNG
* `images/` 目录存放渲染好的 PNG 图片，供文档通过 `![](images/xxx.png)` 引用

**命令行批量生成 PNG**  
当修改了 PlantUML 源码后，需要重新生成 PNG：

* 1
* 2

```
java -Dfile.encoding=UTF-8 -jar plantuml.jar -charset UTF-8 "plantuml/*.md"mv plantuml/*.png images/
```

**Git 版本管理建议**

* 1
* 2
* 3

```
images/*.png        # PNG 可从源码重新生成，无需版本管理plantuml/*.md       # PlantUML 源码需要版本管理*.md                # 文档需要版本管理
```

# 参考文献

[1] obsidian-plantuml 插件：https://github.com/joethei/obsidian-plantuml

[2] PlantUML 官方文档：https://plantuml.com/zh/sequence-diagram

[3] Obsidian 社区插件开发指南：https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin

[4] PlantUML 中文字体配置：https://plantuml.com/zh/font