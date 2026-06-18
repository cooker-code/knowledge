---
title: Solon AI MCP：超越单一工具，全面支持三大原语的创新实践
author: Event Stack
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODcyODI3OA==&mid=2247483827&idx=1&sn=70547702b06380d0658c00c286f4dcf3&chksm=97bf68aa4b1b25d44dc50adace4f20aa6af4ff596983d3d5b8bfed5e573725b1b96045863f09&mpshare=1&scene=24&srcid=0930Rb2sbU2t4hr1tx884wfx&sharer_shareinfo=d073a4e9cb7f3655968e258b3bbb44b3&sharer_shareinfo_first=d073a4e9cb7f3655968e258b3bbb44b3#rd
---

# 引言

在AI工具服务开发领域，Spring AI提供了基础的@Tool注解支持，但这仅仅是冰山一角。Solon AI MCP协议通过完整支持Tool、Prompt、Resource三大原语，实现了对Spring AI的全面超越，为开发者提供了更强大、更灵活的AI工具服务开发体验。

# Solon MCP vs Spring AI

| 能力维度 | Spring AI | Solon MCP | 优势说明 |
| --- | --- | --- | --- |
| 工具调用支持 | ✅ | ✅ | 两者均支持基础工具调用 |
| 提示语管理 | ❌ | ✅ | Solon独有的@PromptMapping专业支持 |
| 资源抽象层 | ❌ | ✅ | Solon的@ResourceMapping统一资源访问 |
| 动态工具管理 | ❌ | ✅ | Solon支持运行时添加/更新/移除工具 |
| 多端点支持 | ❌ | ✅ | Solon支持按业务划分多个服务端点 |
| 协议转换能力 | ❌ | ✅ | Solon轻松实现SSE⇌Stdio协议转换 |
| 资源模板支持 | ❌ | ✅ | Solon支持带参数的资源路径 |
| 二进制资源支持 | ❌ | ✅ | Solon可直接返回图片/文件等二进制数据 |
| 统一服务架构 | ❌ | ✅ | Solon单协议支持三大原语协同工作 |

下面我们深入分析Solon MCP如何通过三大原语支持完胜Spring AI。

### 一、Tool实现：不只是基础调用

#### Spring AI的局限

Spring AI仅提供基本的工具注册能力：

```
@Tool  
public String getWeather(@Param String location) {  
    // 简单返回天气数据  
}
```

这种实现方式存在明显局限：

* • 工具固定，无法动态更新
* • 缺乏上下文访问能力
* • 无法进行细粒度控制

#### Solon MCP的全面超越

```
@McpServerEndpoint(sseEndpoint = "/weather/sse")  
public class WeatherService {  
    // 基础工具实现  
    @ToolMapping(description = "精准天气预报")  
    public WeatherData getWeather(  
        @Param(description = "城市编码") String cityCode,  
        Context context) { // 访问请求上下文  
          
        // 基于IP地址的简单限流  
        if (isHighFrequency(context.realIp())) {  
            throw new RuntimeException("请求过于频繁");  
        }  
          
        return weatherDao.getByCode(cityCode);  
    }  
      
    // 动态工具管理  
    @Scheduled(fixedRate = 1_800_000)// 每30分钟更新  
    public void updateTools() {  
        McpServerEndpointProvider provider = context.getBean();  
        provider.removeTool("getWeather");  
        provider.addTool(createWeatherToolWithCache());  
    }  
}
```

**核心优势：**

* • 支持请求上下文访问
* • 运行时动态更新工具
* • 细粒度的频率控制
* • 多端点并行支持

### 二、Prompt实现：Spring AI的空白领域

Spring AI完全缺失提示语的专业化管理能力，而Solon MCP通过@PromptMapping提供了完整解决方案：

```
@McpServerEndpoint(sseEndpoint = "/prompt/sse")  
public class EnterprisePromptService {  
    // 企业级提示语版本管理  
    private final AtomicInteger version = new AtomicInteger(1);  
      
    @PromptMapping(description = "客户服务标准话术")  
    public String customerServicePrompt() {  
        return loadPromptTemplate("v" + version.get() + "/customer.md");  
    }  
      
    // 提示语热更新  
    @Mapping("/prompt/upgrade")  
    public void upgrade() {  
        version.incrementAndGet();  
        notifyClients(); // 主动通知客户端更新  
    }  
      
    // 参数化提示语  
    @PromptMapping(description = "个性化营销文案")  
    public List<ChatMessage> marketingCopy(  
            @Param(description = "客户等级") String tier,  
            @Param(description = "产品线") String productLine) {  
          
        String template = loadPromptTemplate("marketing/" + productLine + ".md");  
        return Arrays.asList(  
            ChatMessage.ofSystem(template),  
            ChatMessage.ofUser("客户等级: {tier}").replace("{tier}", tier)  
        );  
    }  
}
```

**核心优势：**

* • 企业级提示语版本控制
* • 热更新能力
* • 结构化参数支持
* • 主动通知机制

### 三、Resource实现：Solon的独家能力

Spring AI完全没有资源抽象层概念，而Solon MCP的@ResourceMapping创造了统一资源访问新范式：

```
@McpServerEndpoint(sseEndpoint = "/resource/sse")  
public class EnterpriseResourceHub {  
    // 配置资源  
    @ResourceMapping(uri = "config://risk-rules",   
                     mimeType = "application/json")  
    public RiskRules getRiskRules() {  
        return riskService.getCurrentRules();  
    }  
      
    // 二进制资源支持  
    @ResourceMapping(uri = "doc://contract-templates/{type}",  
                     mimeType = "application/pdf")  
    public InputStream getContractTemplate(  
        @Param(description = "合同类型") String type) {  
        return storageService.getTemplate(type);  
    }  
      
    // 动态资源更新  
    @Scheduled(cron = "0 0 3 * * ?")// 每天凌晨3点更新  
    public void refreshResources() {  
        McpServerEndpointProvider provider = context.getBean();  
        provider.removeResource("config://risk-rules");  
        provider.addResource(this::getUpdatedRiskRules);  
    }  
}
```

客户端统一调用：

```
// 获取资源配置  
RiskRules rules = client.readResource("config://risk-rules", RiskRules.class);  
  
// 获取合同模板  
InputStream pdf = client.readResourceAsStream(  
    "doc://contract-templates/lease"  
);
```

**核心优势：**

* • 统一资源URI抽象
* • 二进制资源原生支持
* • 资源热更新能力
* • 类型安全访问机制

## 企业级场景对比

### 金融风控系统实现

**Spring AI方案局限：**

* • 规则更新需重新部署
* • 无法动态调整
* • 缺乏资源集成

```
@Tool  
public boolean riskCheck(@Param String userId) {  
    // 硬编码规则  
    return riskDao.check(userId);  
}
```

**Solon MCP完整方案：**

```
@McpServerEndpoint(sseEndpoint = "/risk/sse")  
public class RiskSystem {  
    @ResourceMapping(uri = "config://risk-rules")  
    public RiskRules getRules() { /* 动态加载规则 */ }  
      
    @ToolMapping(description = "智能风控检查")  
    public RiskResult check(  
        @Param(description = "用户ID") String userId,  
        @Param(description = "交易金额") double amount) {  
          
        RiskRules rules = getRules(); // 获取最新规则  
        return engine.evaluate(userId, amount, rules);  
    }  
      
    // 规则更新监听  
    @EventListen  
    public void onRuleUpdate(RuleUpdateEvent event) {  
        // 动态更新资源  
        getProvider().removeResource("config://risk-rules");  
        getProvider().addResource(this::getRules);  
    }  
}
```

**方案优势对比：**

* • Solon实现动态规则更新（无需重启）
* • 资源与工具协同工作
* • 事件驱动的实时更新
* • 完整的风控生态

### 架构优势：Solon MCP的三大突破

#### 1. 统一协议，多原语协同

```
统一MCP协议

客户端

Solon MCP服务端

Tool 工具服务

Prompt 提示语服务

Resource 资源服务

业务系统
```

#### 2. 动态服务网格

```
// 创建金融工具服务网格  
McpServerEndpointProvider financeGrid = McpServerEndpointProvider.builder()  
    .name("finance-grid")  
    .sseEndpoint("/finance/sse")  
    .addProvider(stockService)  // 股票服务  
    .addProvider(forexService)  // 外汇服务  
    .addProvider(riskService)   // 风控服务  
    .build();  
  
// 动态添加加密货币服务  
cryptoWatcher.subscribe(event -> {  
    financeGrid.addProvider(cryptoService);  
});
```

#### 3. 跨协议转换网关

```
// 将SSE服务转为Stdio服务  
@McpServerEndpoint(channel = McpChannel.STDIO)  
public class SseToStdioGateway implements ToolProvider {  
    private final McpClientProvider sseClient;  
      
    public SseToStdioGateway() {  
        sseClient = McpClientProvider.builder()  
            .apiUrl("http://finance-api:8080/finance/sse")  
            .build();  
    }  
      
    @Override  
    public Collection<FunctionTool> getTools() {  
        // 透传所有工具  
        return sseClient.getTools();  
    }  
}
```

#### Solon MCP的优势

* • 完整原语支持：一站式解决Tool+Prompt+Resource需求
* • 动态能力：运行时更新工具、提示语和资源
* • 企业级特性：版本控制、热更新、灰度发布
* • 轻量高效：Solon核心仅0.1MB，启动速度比Spring快3-5倍
* • 多协议支持：SSE、Stdio、Streaming（规划中）自由转换
* • 开放生态：不绑定特定AI引擎，兼容各种大模型

## 项目地址

```
# GitHub：  
https://github.com/noear/solon  
  
# 相关案例：  
https://solon.noear.org/article/ai-mcp
```

## 结语

Spring AI提供了基础的AI工具调用能力，但在企业级应用场景中显得力不从心。Solon MCP通过三大原语的完整支持、动态服务管理和统一资源抽象，解决了Spring AI的诸多痛点：打破单一工具调用的局限，填补提示语专业管理的空白，创建资源访问的统一范式，实现真正的动态服务能力。

无论是构建简单的AI工具服务，还是开发复杂的企业级AI系统，Solon MCP都提供了比Spring AI更全面、更灵活的解决方案。

---

  
**这个公众号会发布很多有趣的开源项目，分享编程知识，AI实战案例，欢迎关注。**