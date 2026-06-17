# Maven 项目管理与模块边界

## 目标

这个知识点要回答：

> Java 项目如何用 Maven 管理代码、模块、依赖、版本和构建生命周期。

当前收藏文章对 Maven 覆盖不足，所以本文先建立准则骨架，后续再用文章补证。

## 单模块与多模块选择

| 项目类型 | 推荐结构 | 判断 |
|---|---|---|
| 小型 CRUD / 原型 | 单模块 | 业务简单、部署单一、团队小 |
| 中型业务系统 | 单仓多包，必要时多模块 | 业务开始有清晰边界，但还不需要拆服务 |
| 核心业务系统 | Maven 多模块 | 需要隔离 domain、application、infrastructure、interfaces |
| 平台型系统 | 多模块 + 多应用 | 有多个启动应用、公共能力、插件或 SDK |

## 推荐多模块结构

```text
project-root/
  pom.xml                 # 父工程：版本、插件、模块声明
  app-bootstrap/          # 启动模块：Spring Boot main、配置装配
  app-interfaces/         # REST、消息、定时任务入口
  app-application/        # 用例编排、事务、端口
  app-domain/             # 领域模型、领域服务、领域仓储接口
  app-infrastructure/     # 数据库、外部服务、消息、缓存实现
  app-common/             # 谨慎使用：只放真正通用的基础对象
```

## 模块依赖方向

```text
app-bootstrap
  -> app-interfaces
  -> app-application
  -> app-domain

app-infrastructure
  -> app-application
  -> app-domain
```

| 规则 | 说明 |
|---|---|
| domain 不依赖其他业务模块 | 领域核心不能被框架和基础设施污染 |
| application 依赖 domain | 应用服务编排领域对象和端口 |
| infrastructure 实现 application/domain 定义的端口 | 技术实现不能反向定义业务 |
| interface 调用 application | 入口适配只触发用例 |
| bootstrap 负责装配 | 启动类、配置、Bean 装配集中在启动模块 |

## 父工程应该管理什么

| 内容 | 应放位置 | 原因 |
|---|---|---|
| Java 版本 | 父 `pom.xml` | 保证所有模块一致 |
| Spring Boot 版本 | 父 `pom.xml` 或 BOM | 避免子模块版本漂移 |
| 依赖版本 | `dependencyManagement` | 子模块只声明依赖，不重复写版本 |
| 插件版本 | `pluginManagement` | 编译、测试、打包行为一致 |
| 模块声明 | `<modules>` | 明确构建顺序 |

## 子模块应该做什么

| 子模块 | 只应该声明 |
|---|---|
| domain | 领域所需的最小依赖，尽量不依赖 Spring |
| application | 事务、校验、用例接口、端口接口 |
| infrastructure | JPA/MyBatis、Redis、HTTP Client、MQ SDK |
| interfaces | Spring Web、Controller、DTO、消息监听 |
| bootstrap | Spring Boot Starter、配置文件、启动类 |

## 当前缺口

| 缺口 | 后续需要文章补充 |
|---|---|
| Maven 生命周期 | compile/test/package/install/deploy 的真实使用边界 |
| BOM 与 dependencyManagement | Spring Boot BOM、公司内部 BOM 的管理方式 |
| 多模块循环依赖治理 | 如何发现和避免模块反向依赖 |
| 插件管理 | compiler、surefire、failsafe、jacoco、shade、spring-boot-maven-plugin |
| 私服与发布 | SNAPSHOT、release、制品仓库、版本号策略 |
