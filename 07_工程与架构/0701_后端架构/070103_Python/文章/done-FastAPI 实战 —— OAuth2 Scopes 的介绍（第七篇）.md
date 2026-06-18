> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: FastAPI 实战 —— OAuth2 Scopes 的介绍（第七篇）
author: D先生的自白
date:
url: https://mp.weixin.qq.com/s?__biz=MzYzNTE0NDIyMg==&mid=2247483695&idx=1&sn=dedf79403d2840912a4470712cd6fe2b&chksm=f18e56ff0152bea73f7bc46689bbadd0fc8653e4f05bfd394807e6bcc364c2f2b569d39a7d16&mpshare=1&scene=24&srcid=1121A5mNXmIGHRGn6yAO7bDO&sharer_shareinfo=4b342c61da9583833c599f007f1814d7&sharer_shareinfo_first=4b342c61da9583833c599f007f1814d7#rd
---

> 本文将介绍OAuth2 Scopes的基本概念，理解为什么需要scopes。

---

## 🎯 **阅读收获**

通过本教程，你将掌握：

✅ 理解 OAuth2 Scopes 的基本概念

---

## 🧩 **OAuth2 Scopes 是什么？**

### 🏠 **把 Scopes 想象成「门禁卡权限」**

假设你有一张公司的门禁卡，但这张卡不是哪里都能刷开的。

#### **基本概念**

* **你的门禁卡** = OAuth2 的访问令牌（Access Token）
* **门禁卡的权限** = Scopes（作用域）

所以会有不同级别的权限：

```
# 你的门禁卡可能有这些权限：
scopes = {
    "进入大楼": "可以刷开公司大门",
    "进入办公室": "可以进入办公区域", 
    "使用打印机": "可以刷卡使用打印机",
    "进入财务室": "可以进入财务部门"
}

# 小王的权限
小王_scopes = "进入大楼 进入办公室 使用打印机"

# 小李的权限  
小李_scopes = "进入大楼 进入办公室 使用打印机 进入财务室"

@app.get("/公司大门")
def enter_company(需要有: "进入大楼"权限):
    return"欢迎来到公司"

@app.get("/打印机") 
def use_printer(需要有: "使用打印机"权限):
    return"打印完成"

@app.get("/财务室")
def enter_finance(需要有: "进入财务室"权限):
    return"这里是财务部"
```

* 小王尝试进入财务室时，需要 `进入财务室` 权限，但 `小王_scopes` 中没有这个权限，所以提示无权限。
* 小李尝试进入财务室时，`小李_scopes` 中有 `进入财务室` 权限，所以可以正常访问。

#### **为什么这样设计？**

* **安全性**：避免「一刀切」

+ ❌ 不好的做法：要么全有权限，要么全没权限
+ ✅ 好的做法：细粒度控制，按需分配

* **灵活性**：不同角色不同权限

+ 实习生：只有基本权限
+ 正式员工：更多权限
+ 管理员：所有权限

## 🚀 下一篇预告

📌 我们将继续深入 FastAPI OAuth2 实战，讲解如何在实际项目中配置和使用 Scopes。