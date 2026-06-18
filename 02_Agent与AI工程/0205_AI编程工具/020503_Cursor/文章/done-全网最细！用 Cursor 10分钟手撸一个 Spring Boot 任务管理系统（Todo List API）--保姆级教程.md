> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: 全网最细！用 Cursor 10分钟手撸一个 Spring Boot 任务管理系统（Todo List API）--保姆级教程
author: 码农跃迁指南
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg4MTE2NjUxMw==&mid=2247483705&idx=1&sn=88e1537d7fe1eff41d06018de1650c1b&chksm=ce84c009c7fce88985a80460d010d5358adb94ef7c330ab464605aeab6310cbfe0761ee58b3b&mpshare=1&scene=24&srcid=1122M91vpip1MM7gEQUbAjvK&sharer_shareinfo=eb4bfaf4e3614965df63d73687e6e44f&sharer_shareinfo_first=eb4bfaf4e3614965df63d73687e6e44f#rd
---

重点突出了**Cursor 的实操流程**和**Prompt（提示词）技巧**。

---

# 在这个 AI 编程爆发的时代，如果你还在一个字符一个字符地敲代码，那你可能已经慢了。

今天，带大家实战体验一下最近火出圈的代码编辑器——**Cursor**。我们将从零开始，完全依靠 AI 辅助，搭建一个可运行的 Java Spring Boot 后端项目。

**目标**：构建一个任务管理系统（Todo List API），支持增删改查。
**工具**：Cursor + JDK 17 + Maven。

---

## 🚀 为什么是 Cursor？

Cursor 并不是简单的“代码补全”工具，它是一个 Fork 自 VS Code 的编辑器，内置了 GPT-4/Claude 3.5 等强力模型。它最大的杀手锏是**理解上下文**——它知道你的整个项目结构，而不仅仅是当前打开的文件。

## 🛠️ 准备工作

1. **下载安装 Cursor**

   ：官网直接下载，操作习惯和 VS Code 完全一致。
2. **环境准备**

   ：确保本地已安装 JDK (推荐 17+) 和 Maven。

---

## 第一步：初始化项目结构

打开 Cursor，不需要去 Spring Initializr 网站点点点。我们直接在 Cursor 的空文件夹里操作。

1. **打开 Cursor 终端**

   (`Ctrl + ~`)。
2. **使用**`Ctrl + L`**打开 AI 对话框**

   。
3. **输入 Prompt（提示词）**

   ：

**Prompt:**
请帮我生成一个 Spring Boot 项目的目录结构和 pom.xml 文件。
要求：

1. Java 版本：17
2. 构建工具：Maven
3. 依赖：Spring Web, Spring Data JPA, H2 Database, Lombok
4. 项目名为：cursor-demo

**Cursor 的反应**：
它会立刻列出 `pom.xml` 的完整代码。你只需要点击代码块右上角的 **"Apply"** 或者手动创建文件粘贴即可。

*(注：如果 Cursor 没有自动创建文件权限，你可以手动创建*`pom.xml`*并粘贴内容，然后由 Maven 自动加载依赖)*

---

## 第二步：创建核心实体类 (Entity)

在 `src/main/java/com/example/demo/model` 下（如果目录不存在，让 Cursor 创建），我们需要一个 `Task` 类。

1. 创建一个空文件 `Task.java`。
2. **按下**`Ctrl + K`**(这是 Cursor 的神技：行内代码生成)**

   。
3. **输入 Prompt**

   ：

**Prompt:**
创建一个 Task 实体类，包含 id (主键自增), title (标题), description (描述), completed (是否完成), createdAt (创建时间)。使用 Lombok 注解。

**Cursor 生成的代码：**

```
package com.example.demo.model;

import jakarta.persistence.*;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@Entity
@Table(name = "tasks")
public class Task {
    @Id
    @GeneratedValue
    (strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String description;
    private boolean completed;

    private LocalDateTime createdAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
}
```

*（这是完美符合要求的标准代码，连*`@PrePersist`*自动生成时间都考虑到了。）*

---

## 第三步：一键生成 Repository, Service 和 Controller

这是 Cursor 最震撼的地方。我们不需要一个文件一个文件去写。

1. **按下**`Ctrl + L`**打开 Chat 面板**

   。
2. **输入 Prompt（注意使用 @ 符号引用上下文）**

   ：

**Prompt:**
@Task.java 
根据这个实体类，请帮我生成对应的 

Repository 接口、Service 接口及实现类、

以及一个 RESTful 风格的 Controller。
Controller 需要包含：

* 查询所有任务
* 根据 ID 查询任务
* 创建任务
* 更新任务
* 删除任务

**Cursor 的操作：**
它会一次性输出三个文件的代码。你只需要点击每个代码块上方的文件名旁边的 **"Apply to Current File"** 或者直接复制新建文件。

**生成的 Controller 代码片段预览：**

```
@RestController
@RequestMapping("/api/tasks")
public class TaskController {

    @Autowired
    private TaskService taskService;

    @GetMapping
    public List<Task> getAllTasks() {
        return taskService.getAllTasks();
    }

    @PostMapping
    public Task createTask
        (@RequestBody Task task) {
        return taskService.createTask(task);
    }

    // ... 其他 CRUD 方法自动生成
}
```

---

## 第四步：配置数据库

我们需要配置 H2 数据库以便运行。

1. 打开 `src/main/resources/application.properties`。
2. **按**`Ctrl + K`

   。
3. **输入 Prompt**

   ：

**Prompt:**
配置 H2 内存数据库，开启 H2 控制台访问，设置端口为 8080。

**Cursor 生成的代码：**

```
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform
=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
server.port=8080
```

---

## 第五步：运行与测试

到了见证奇迹的时刻。

1. **启动项目**

   ：找到主启动类（`DemoApplication.java`），点击 Main 方法旁边的 "Run" 按钮。
2. **测试接口**

   ：我们不需要下载 Postman，直接用 Cursor。

打开一个新的文件，命名为 `test.http` (或者直接在 Cursor 终端测试)。

**让 Cursor 写测试脚本 (**`Ctrl + K`**)：**

**Prompt:**
生成几个 curl 命令来测试刚才的 /api/tasks 接口：创建一个任务，然后查询列表。

**Cursor 输出：**

```
# 创建任务
curl -X POST http://localhost:8080/api/tasks \
-H "Content-Type: application/json" \
-d '{"title":"学习Cursor",
"description":"完成一篇博客文章","completed":false}'

# 查询列表
curl http://localhost:8080/api/tasks
```

把这些命令粘贴到 Cursor 内置终端运行，你会看到数据库返回了 JSON 数据！

---

## 💡 进阶技巧：如何用 Cursor 改 Bug？

假设你想修改需求：**“创建任务时，如果标题为空，抛出异常。”**

1. 打开 `TaskServiceImpl.java`。
2. 选中 `createTask` 方法的代码。
3. **按**`Ctrl + K`

   。
4. **输入 Prompt**

   ：`修改这个方法，如果 title 为空或 null，抛出 IllegalArgumentException。`

Cursor 会立即展示修改前后的对比（Diff View），你确认无误后点击 **Accept** 即可。

---

## 📝 总结

通过这篇文章，我们只用了不到 10 分钟，几乎没有手写一行核心逻辑代码，就完成了一个标准的 Spring Boot CRUD 项目。

**Cursor 的核心优势总结：**

1. **Ctrl + K (Edit)**

   ：局部代码生成与修改，精准控制。
2. **Ctrl + L (Chat)**

   ：全局架构设计，多文件联动。
3. **@Context**

   ：能够感知整个项目上下文，而不是瞎编代码。

**📢****建议：** 尽管 AI 很强大，但作为开发者，**阅读和审查代码的能力**变得比“写代码”更重要。Cursor 是你的副驾驶，但方向盘（业务逻辑和架构判断）依然在你手中。

---

*如果你觉得这篇文章对你有帮助，欢迎点赞、收藏、转发！下期我们来讲讲如何用 Cursor 进行旧项目重构。*