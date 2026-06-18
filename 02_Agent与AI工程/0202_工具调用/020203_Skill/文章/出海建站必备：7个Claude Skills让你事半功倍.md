---
title: 出海建站必备：7个Claude Skills让你事半功倍
author: niro
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5ODQzOTk1Mg==&mid=2247484436&idx=1&sn=158520f8445ba934eb5e0afc86842b73&chksm=91b25ca36f86ef67e2d0c14b3ce56b43b9f1e994b55d10974e7a6da04e5e796a235ee10d766f&mpshare=1&scene=24&srcid=0106GuWJgh8y0WdTAYosGnnP&sharer_shareinfo=06a7b540e394eda14f1901686dc959c3&sharer_shareinfo_first=06a7b540e394eda14f1901686dc959c3#rd
---

# 

## 写在前面

上周末，我的朋友老李找到我：「我想做个SaaS出海，预算只有5万，能帮我推荐个靠谱的外包团队吗？」

我反问他：「为什么一定要外包？」

他说：「我一个后端，不懂前端设计，更不懂海外用户喜欢什么风格。找设计师要2万，找前端开发又要2万，还得找运维部署……算下来5万都不够。」

**这是大多数独立开发者出海时面临的困境：**

* ❌ 设计不专业，网站一看就是套的模板，转化率惨淡
* ❌ 技术栈太复杂，一个人根本搞不定前后端+运维+文档
* ❌ 时间紧、预算少，试错成本高，失败了就血本无归

三天后，老李用Claude Code和7个Skills，周末搞定了MVP，第二周就开始收到第一批海外用户的付费。他的建站成本？**接近0**。

今天我就来分享这7个Skills，以及两套完整的建站方案：**快速MVP**和**专业出海站**。

---

## 什么是Claude Skills？

在讲具体Skills之前，先说说什么是Claude Skills。

简单来说，\*\*Claude Skills = Claude的"插件系统"\*\*。就像VSCode有插件、Chrome有扩展一样，Skills让Claude具备了专业领域的能力。

**和ChatGPT插件的区别：**

* ChatGPT插件：需要手动调用，适合单次任务
* Claude Skills：**自动识别场景**，比如你说"帮我设计一个登录页"，Claude会自动调用frontend-design

**获取渠道：**

* 官方Skills：github.com/anthropics/skills
* 社区Skills：skillsmp.com（目前有44,000+个Skills）

**安装很简单：**

```
# 1. 克隆仓库到本地  
git clone https://github.com/xxx/skill-name.git  
  
# 2. 复制到Claude目录  
cp -r skill-name ~/.claude/skills/  
  
# 3. 重启Claude Code，自动生效
```

接下来，我们看看具体的Skills和使用方法。

---

## 快速建站MVP方案（周末就能上线）

这套方案适合：

* ✅ 预算有限的独立开发者
* ✅ 需要快速验证想法的产品经理
* ✅ 想要在周末上线第一个版本的创业者

**核心目标：最快速度上线，验证市场需求。**

---

### 1. frontend-design：告别千篇一律的AI设计

**⭐ 来源：** Anthropic官方 | **⭐ Stars：** 47.9k

#### 为什么需要它？

你有没有发现，很多AI生成的网站看起来都一个样？圆角卡片、渐变按钮、大标题小副标题……用户一眼就能看出是"AI做的"。

**frontend-design解决的就是这个问题。**

它不是简单地套模板，而是：

* 🎨 **分析你的品牌定位**，生成符合目标用户审美的设计
* 🎯 **避免AI常见的设计陷阱**（比如过度使用渐变、圆角半径不统一）
* 💎 **生成production-ready的代码**，不是那种"看起来很美，实际无法使用"的demo

#### 实际案例

**场景：** 你要做一个面向欧美用户的SaaS工具，比如"项目管理工具"。

**使用方法：**

```
我要做一个项目管理工具的落地页，目标用户是欧美的中小企业主。  
请帮我设计一个专业、现代、值得信赖的风格。
```

**Claude会问你：**

* 你的核心价值主张是什么？（比如："让项目管理像聊天一样简单"）
* 有没有竞品参考？（比如："类似Asana，但更轻量"）
* 品牌色调？（比如："专业蓝 + 活力橙"）

**生成结果：**

```
// 不是简单的模板，而是根据你的需求定制的React组件  
export function Hero() {  
  return (  
    <section className="relative overflow-hidden bg-slate-50">  
      <div className="mx-auto max-w-7xl px-6 py-24 sm:py-32 lg:px-8">  
        <div className="mx-auto max-w-2xl lg:mx-0 lg:max-w-xl">  
          <h1 className="text-4xl font-bold tracking-tight text-slate-900 sm:text-6xl">  
            Project management,  
            <span className="text-blue-600"> as simple as chatting</span>  
          </h1>  
          <p className="mt-6 text-lg leading-8 text-slate-600">  
            Stop juggling spreadsheets and endless Slack threads.  
            Manage your team's work in one place—without the complexity.  
          </p>  
          <div className="mt-10 flex items-center gap-x-6">  
            <Button size="lg" variant="primary">  
              Start free trial  
            </Button>  
            <Button size="lg" variant="ghost">  
              Watch demo <PlayIcon className="ml-2 h-4 w-4" />  
            </Button>  
          </div>  
        </div>  
        <div className="mx-auto mt-16 max-w-2xl sm:mt-24 lg:ml-10 lg:mr-0 lg:mt-0 lg:max-w-none">  
          {/* 产品截图 */}  
        </div>  
      </div>  
    </section>  
  )  
}
```

**注意这些细节：**

* ✅ 使用了情感词汇："as simple as chatting"
* ✅ 针对痛点："Stop juggling spreadsheets"
* ✅ 明确CTA："Start free trial" vs "Watch demo"
* ✅ 专业的间距和字体层级

**使用技巧：**

1. **提供参考，但不要照搬**

   ```
   ❌ "帮我做一个和Linear一模一样的设计"  
   ✅ "我喜欢Linear的极简风格，但希望更温暖一些，适合非技术团队"
   ```
2. **多迭代几轮**

   ```
   第一轮："设计一个Hero区"  
   第二轮："再加一个Features对比表"  
   第三轮："把CTA按钮改成更有紧迫感的文案"
   ```
3. **让Claude解释设计决策**

   ```
   "为什么你选择蓝色作为主色调？"  
   "为什么这里用了卡片布局而不是列表？"
   ```

**适用场景：**

* ✅ 产品落地页
* ✅ 营销网站
* ✅ SaaS应用界面
* ✅ 博客和内容站
* ❌ 电商网站（需要更多专业设计）
* ❌ 复杂的Dashboard（建议用成熟组件库）

---

### 2. landing-page-guide-v2：高转化落地页的11个要素

**⭐ 来源：** 社区（基于DESIGNNAS框架）

#### 为什么需要它？

**一个残酷的事实：** 90%的落地页转化率不到1%。

不是因为产品不好，而是因为落地页缺少关键要素。DESIGNNAS总结了高转化落地页必备的11个元素，这个Skill会帮你检查并生成完整的落地页。

**11个必备要素：**

1. ✅ **独特价值主张（UVP）** - 3秒内让用户知道你是干什么的
2. ✅ **英雄图/演示视频** - 可视化展示产品
3. ✅ **核心功能（3-5个）** - 聚焦主要卖点
4. ✅ **工作原理** - 降低理解成本
5. ✅ **社会证明** - 客户评价、使用人数、知名客户
6. ✅ **对比表** - 和竞品或传统方案对比
7. ✅ **价格表** - 透明定价，降低决策门槛
8. ✅ **FAQ** - 提前回答用户疑虑
9. ✅ **明确CTA** - 每屏都有清晰的下一步
10. ✅ **信任标识** - 安全认证、媒体报道
11. ✅ **紧迫感** - 限时优惠、剩余名额

#### 实际案例

**使用方法：**

```
我要做一个AI写作工具的落地页，帮我检查是否包含了所有高转化要素。  
  
产品信息：  
- 名称：WriteFlow  
- 价值主张：10分钟写出SEO友好的博客文章  
- 目标用户：内容创作者、市场营销人员  
- 核心功能：AI大纲生成、SEO关键词优化、多语言支持  
- 定价：免费试用 → $29/月
```

**Claude会生成完整的落地页结构：**

```
## 1. Hero区（折叠上方）  
[大标题] Write SEO-ready blog posts in 10 minutes  
[副标题] Stop spending hours on research. Let AI handle the heavy lifting while you focus on your unique insights.  
[CTA] Start writing for free →  
[信任标识] ⭐⭐⭐⭐⭐ 4.8/5 from 1,240 creators | No credit card required  
  
## 2. 社会证明  
"WriteFlow cut my content production time by 70%. I'm publishing 3x more without sacrificing quality."  
— Sarah Chen, Content Lead @ TechStartup  
  
[显示] 10,000+ articles written | 2,500+ active users  
  
## 3. 核心功能  
  
### 🎯 AI-Powered Outlines  
Input your topic, get a structured outline in seconds. Based on top-ranking articles in your niche.  
  
### 🔍 Built-in SEO Optimization  
Real-time keyword suggestions, readability scores, and meta description generation.  
  
### 🌍 Multi-language Support  
Write in English, translate to 12+ languages with native-level quality.  
  
## 4. 工作原理（3步）  
1️⃣ Enter your topic and target keywords  
2️⃣ AI generates outline + research → You add your insights  
3️⃣ One-click optimization for SEO and readability  
  
[动画演示视频]  
  
## 5. 对比表：传统方式 vs WriteFlow  
  
| 传统方式 | WriteFlow |  
|---------|-----------|  
| 研究竞品文章：2小时 | AI分析Top 10：2分钟 |  
| 手动优化SEO：1小时 | 实时SEO建议：0分钟 |  
| 总耗时：4-6小时 | 总耗时：10-20分钟 |  
  
## 6. 定价  
  
### Free Trial  
- 3 articles/month  
- All core features  
[CTA] Start free →  
  
### Pro - $29/month  
- Unlimited articles  
- Priority support  
- Advanced SEO tools  
[CTA] Start 14-day trial →  
  
## 7. FAQ  
Q: 会不会生成"AI味"很重的内容？  
A: 不会。WriteFlow只生成大纲和研究，核心内容由你撰写，保持你的独特风格。  
  
Q: 支持中文吗？  
A: 支持中文、英文、日文等12种语言。  
  
## 8. 最后的CTA  
Ready to 10x your content output?  
[CTA] Start your free trial — No credit card required →  
  
[信任标识] 🔒 SSL Secured | 💳 Cancel anytime | 📧 24/7 Support
```

**使用技巧：**

1. **用数据说话，不要空洞描述**

   ```
   ❌ "WriteFlow很快"  
   ✅ "10分钟完成，比传统方式快6倍"
   ```
2. **每个区块都要有明确的CTA**

   ```
   Hero区：Start free trial  
   功能区：See how it works  
   定价区：Choose your plan  
   底部：Last chance to try free
   ```
3. **针对不同用户群体，准备多个版本**

   ```
   版本A：强调速度（"10分钟完成"）  
   版本B：强调质量（"SEO友好，排名更高"）  
   A/B测试，看哪个转化率更高
   ```

**适用场景：**

* ✅ SaaS产品落地页
* ✅ 数字产品（课程、电子书）
* ✅ App下载页
* ❌ 纯内容网站
* ❌ 企业官网（需求更复杂）

---

### 3. docs-write：让文档成为你的增长引擎

**⭐ 来源：** Metabase | **⭐ Stars：** 44.7k

#### 为什么需要它？

很多人以为，文档只是"给用户看的使用说明"。

**错了。好的文档是增长引擎。**

数据显示：

* 📈 有完善文档的产品，试用转化率提升**40%**
* 📈 SEO流量中，\*\*60%\*\*来自文档和教程页面
* 📈 好的文档能减少\*\*80%\*\*的客服工作量

**docs-write做什么：**

* ✍️ 用对话式风格写文档，不是冷冰冰的"技术手册"
* 🎯 SEO优化，让文档成为流量入口
* 📚 自动生成目录、导航、搜索功能

#### 实际案例

**场景：** 你做了一个API工具，需要写API文档。

**传统方式（枯燥）：**

```
## Authentication  
  
使用Bearer Token进行身份验证。  
  
请求示例：  
```http  
GET /api/users  
Authorization: Bearer YOUR_TOKEN
```

```
**用docs-write（吸引人）：**
```

我要写一个API认证的文档，让开发者快速上手。

```
**Claude生成：**  
  
```markdown  
# 5分钟接入认证  
  
## 你需要什么  
  
在开始之前，确保你有：  
- ✅ 一个WriteFlow账号（[免费注册](链接)）  
- ✅ 你的API密钥（在 设置 → API 中获取）  
  
## 快速开始  
  
### 1. 获取你的API密钥  
  
登录后，进入 **设置 → API**，点击"生成新密钥"。  
  
⚠️ **重要：** 请妥善保管你的密钥，不要提交到GitHub！  
  
### 2. 发起第一个请求  
  
用你喜欢的工具（curl、Postman、或任何HTTP客户端），发送这个请求：  
  
```bash  
curl -X GET "https://api.writeflow.com/users/me" \  
  -H "Authorization: Bearer YOUR_API_KEY"
```

**期待的响应：**

```
{  
  "id": "user_abc123",  
  "email": "you@example.com",  
  "plan": "pro",  
  "credits_remaining": 950  
}
```

### 3. 常见问题

#### ❓ 收到401错误？

* 检查API密钥是否复制正确（不要有多余空格）
* 确认账号状态是否正常

#### ❓ 请求速率限制？

* 免费用户：100次/小时
* Pro用户：1000次/小时
* 需要更多？联系我们

## 下一步

现在你已经成功认证了！接下来：

* 📝 创建你的第一篇文章
* 🔍 探索所有API端点
* 💬 加入开发者社区

---

**还有问题？** 在我们的Discord社区提问，通常10分钟内就有回复。

```
**看到区别了吗？**  
  
| 传统文档 | docs-write风格 |  
|---------|--------------|  
| "使用Bearer Token" | "5分钟接入认证" |  
| 冷冰冰的代码 | 对话式引导 |  
| 假设用户都懂 | 提前解答疑惑 |  
| 没有下一步 | 明确引导："接下来做什么" |  
  
**使用技巧：**  
  
1. **用第二人称，像和朋友聊天**
```

❌ "用户应该在设置中找到API密钥" ✅ "你可以在 设置 → API 中找到你的密钥"

```
2. **提前回答用户的疑问**
```

每写完一段，问自己："用户看到这里会有什么疑问？" 然后用FAQ或提示框解答

```
3. **多用视觉元素**
```

✅ 用emoji表示检查项 ⚠️ 用警告框标注重要信息 💡 用提示框给出最佳实践

```
4. **SEO优化：每个标题都是搜索词**
```

❌ "认证" ✅ "如何获取WriteFlow API密钥并认证"

```
**适用场景：**  
- ✅ API文档  
- ✅ 产品使用手册  
- ✅ 教程和最佳实践  
- ✅ 博客文章  
- ❌ 法律条款（需要严谨表述）  
  
---  
  
### 4. git-advanced-workflows：一个人也能专业版本管理  
  
**⭐ Stars：** 24.2k  
  
#### 为什么需要它？  
  
很多独立开发者的Git使用：  
```bash  
git add .  
git commit -m "更新"  
git push
```

**这样做的问题：**

* ❌ 提交历史混乱，回滚困难
* ❌ 多功能并行开发时，容易冲突
* ❌ 出了Bug不知道是哪次提交导致的

**git-advanced-workflows教你：**

* 🌿 Feature Branch工作流（不同功能在不同分支开发）
* 🔄 Rebase保持提交历史整洁
* 🐛 Git Bisect快速定位Bug
* 💾 Worktrees同时维护多个版本

#### 实际案例

**场景：** 你正在开发"用户认证"功能，但突然有个紧急Bug需要修复。

**糟糕的做法：**

```
# 在当前分支修改，提交历史混在一起  
git commit -m "修了个Bug，还加了认证功能"
```

**用git-advanced-workflows：**

```
我正在开发用户认证功能（feature/auth分支），  
但现在有个紧急Bug需要修复，怎么处理？
```

**Claude会教你：**

```
# 1. 保存当前工作（不提交）  
git stash push -m "认证功能开发中"  
  
# 2. 切换到main分支  
git checkout main  
  
# 3. 创建hotfix分支  
git checkout -b hotfix/login-error  
  
# 4. 修复Bug并提交  
git commit -m "fix: 修复登录时的空指针异常"  
  
# 5. 合并到main  
git checkout main  
git merge hotfix/login-error  
  
# 6. 部署修复  
  
# 7. 回到feature分支继续开发  
git checkout feature/auth  
git stash pop
```

**更进阶的用法：Rebase保持提交历史整洁**

```
我的feature分支有10个提交，但很多是"修复拼写错误"、"忘记加分号"，  
怎么整理成几个清晰的提交？
```

**Claude教你Interactive Rebase：**

```
# 1. 查看最近10个提交  
git log --oneline -10  
  
# 2. 交互式rebase  
git rebase -i HEAD~10  
  
# 3. 在编辑器中，把零碎提交合并（squash）  
pick abc123 添加用户认证基础结构  
squash def456 修复拼写错误  
squash ghi789 忘记加分号  
pick jkl012 完成JWT验证逻辑  
squash mno345 修复测试用例  
  
# 4. 保存后，整理提交信息
```

**结果：** 10个杂乱的提交 → 2个清晰的提交

**使用技巧：**

1. **养成好习惯：每个功能一个分支**

   ```
   feature/user-auth        # 用户认证  
   feature/payment          # 支付功能  
   hotfix/critical-bug      # 紧急修复
   ```
2. **提交信息遵循约定**

   ```
   feat: 添加用户注册功能  
   fix: 修复登录失败的Bug  
   docs: 更新API文档  
   refactor: 重构认证逻辑
   ```
3. **定期rebase，保持分支同步**

   ```
   # 每天开始工作前  
   git checkout main  
   git pull  
   git checkout feature/your-feature  
   git rebase main
   ```

**适用场景：**

* ✅ 独立开发者（一个人也要专业）
* ✅ 小团队协作
* ✅ 需要频繁回滚的项目
* ❌ 简单的静态网站（用不上）

---

## 专业出海网站方案（可持续增长）

如果你的MVP验证成功，开始有付费用户了，就该考虑升级到"专业方案"。

这套方案适合：

* ✅ 月收入超过$1000的产品
* ✅ 需要频繁更新内容（博客、产品更新）
* ✅ 计划长期运营，不是"一锤子买卖"

**核心目标：可扩展、易维护、全球化。**

---

### 5. payload：开源CMS，内容管理不求人

**⭐ Stars：** 39.0k

#### 为什么需要它？

MVP阶段，你可能直接把文案写在代码里：

```
<h1>我们的产品很棒</h1>
```

**但当你需要：**

* ✏️ 每周更新博客
* 📢 发布产品更新日志
* 🌍 添加多语言支持
* 👥 让市场团队自己更新文案

**这时候，你需要一个CMS（内容管理系统）。**

**Payload的优势：**

* 🆓 开源免费（WordPress太老，Contentful太贵）
* ⚡ 性能优秀（基于Node.js + MongoDB）
* 🎨 界面现代化（不像WordPress那样老旧）
* 🔌 API优先（方便集成到任何前端框架）

#### 实际案例

**场景：** 你的SaaS工具需要一个博客，SEO引流。

**步骤1：安装Payload**

```
npx create-payload-app my-blog  
cd my-blog  
npm run dev
```

**步骤2：定义内容结构**

```
帮我创建一个博客的Collection，包含：  
- 标题、作者、正文、发布时间  
- SEO元数据（meta description、keywords）  
- 特色图片
```

**Claude生成配置：**

```
// collections/Posts.ts  
export const Posts = {  
  slug: 'posts',  
  admin: {  
    useAsTitle: 'title',  
  },  
  fields: [  
    {  
      name: 'title',  
      type: 'text',  
      required: true,  
    },  
    {  
      name: 'author',  
      type: 'relationship',  
      relationTo: 'users',  
    },  
    {  
      name: 'content',  
      type: 'richText',  
      required: true,  
    },  
    {  
      name: 'featuredImage',  
      type: 'upload',  
      relationTo: 'media',  
    },  
    {  
      name: 'publishedAt',  
      type: 'date',  
    },  
    {  
      name: 'seo',  
      type: 'group',  
      fields: [  
        {  
          name: 'metaTitle',  
          type: 'text',  
        },  
        {  
          name: 'metaDescription',  
          type: 'textarea',  
        },  
        {  
          name: 'keywords',  
          type: 'text',  
        },  
      ],  
    },  
  ],  
}
```

**步骤3：在前端调用**

```
// 在Next.js中获取博客文章  
const posts = await fetch('http://localhost:3000/api/posts')  
  .then(res => res.json())  
  
// 渲染  
posts.docs.map(post => (  
  <article key={post.id}>  
    <h2>{post.title}</h2>  
    <p>{post.author.name}</p>  
    <img src={post.featuredImage.url} alt={post.title} />  
    <div dangerouslySetInnerHTML={{ __html: post.content }} />  
  </article>  
))
```

**效果：**

* ✅ 市场团队可以自己发布文章，不用找开发
* ✅ 自动生成SEO元数据
* ✅ 图片自动压缩和CDN加速
* ✅ 支持草稿和定时发布

**使用技巧：**

1. **规划好内容模型**

   ```
   不要一股脑把所有字段都加上  
   先列出核心字段，后续再扩展
   ```
2. **利用Hooks自动化**

   ```
   // 发布文章时，自动发送到社交媒体  
   hooks: {  
     afterChange: [  
       async ({ doc }) => {  
         if (doc.published) {  
           await postToTwitter(doc.title, doc.url)  
         }  
       }  
     ]  
   }
   ```
3. **配置权限**

   ```
   编辑人员：可以写文章，但不能发布  
   管理员：可以发布和删除
   ```

**适用场景：**

* ✅ 博客和内容营销
* ✅ 产品文档
* ✅ 多语言网站
* ❌ 简单的静态页面（用不上）
* ❌ 电商（建议用Shopify）

---

### 6. terraform-module-library：多云部署，全球化无忧

**⭐ Stars：** 24.2k

#### 为什么需要它？

**MVP阶段：** 你可能把网站部署在Vercel或Netlify，点点鼠标就搞定。

**但当你出海后，会遇到这些问题：**

* 🌍 欧洲用户访问很慢（服务器在美国）
* 💸 单一云厂商锁定（想换很麻烦）
* 🔒 数据合规要求（欧盟GDPR要求数据必须存在欧洲）

**terraform-module-library帮你：**

* 🌐 多云部署（AWS + Cloudflare + Vercel）
* 🚀 一键部署到全球多个节点
* 📝 基础设施即代码（方便团队协作和版本管理）

#### 实际案例

**场景：** 你的SaaS工具有美国、欧洲、亚洲用户，需要部署到多个区域。

**传统方式：**

* 打开AWS控制台 → 手动创建EC2
* 配置负载均衡器
* 配置CDN
* 配置数据库
* 重复以上步骤，分别在us-east-1、eu-west-1、ap-southeast-1...

**用Terraform（自动化）：**

```
我要部署一个Next.js应用到AWS，  
需要在美国、欧洲、亚洲各部署一个节点。
```

**Claude生成Terraform配置：**

```
# main.tf  
module "us_deployment" {  
  source = "./modules/nextjs-app"  
  
  region = "us-east-1"  
  app_name = "myapp-us"  
  domain = "us.myapp.com"  
}  
  
module "eu_deployment" {  
  source = "./modules/nextjs-app"  
  
  region = "eu-west-1"  
  app_name = "myapp-eu"  
  domain = "eu.myapp.com"  
}  
  
module "asia_deployment" {  
  source = "./modules/nextjs-app"  
  
  region = "ap-southeast-1"  
  app_name = "myapp-asia"  
  domain = "asia.myapp.com"  
}  
  
# 配置Cloudflare负载均衡  
resource "cloudflare_load_balancer" "main" {  
  zone_id = var.cloudflare_zone_id  
  name    = "myapp.com"  
  
  default_pool_ids = [  
    cloudflare_load_balancer_pool.us.id,  
    cloudflare_load_balancer_pool.eu.id,  
    cloudflare_load_balancer_pool.asia.id,  
  ]  
  
  # 根据用户地理位置自动路由  
  geo_balancing = {  
    enabled = true  
  }  
}
```

**一键部署：**

```
# 1. 初始化  
terraform init  
  
# 2. 预览变更  
terraform plan  
  
# 3. 执行部署  
terraform apply
```

**结果：**

* 🌍 美国用户访问美国节点
* 🇪🇺 欧洲用户访问欧洲节点
* 🌏 亚洲用户访问亚洲节点
* ⚡ 延迟降低70%

**使用技巧：**

1. **从模板开始，不要从零写**

   ```
   # 使用社区模板  
   git clone https://github.com/terraform-aws-modules/terraform-aws-vpc.git
   ```
2. **分环境管理**

   ```
   terraform/  
   ├── dev/       # 开发环境  
   ├── staging/   # 测试环境  
   └── prod/      # 生产环境
   ```
3. **用变量管理敏感信息**

   ```
   # 不要直接写死API密钥  
   variable "aws_access_key" {  
     type = string  
     sensitive = true  
   }
   ```
4. **配置自动备份**

   ```
   # 每天自动备份数据库  
   resource "aws_db_instance" "main" {  
     backup_retention_period = 7  
     backup_window          = "03:00-04:00"  
   }
   ```

**适用场景：**

* ✅ 全球化产品
* ✅ 需要多区域部署
* ✅ 团队协作（大家用同一套配置）
* ❌ 简单的单服务器部署
* ❌ 完全依赖无服务器平台（Vercel/Netlify）

---

## 实战：用这7个Skills从0到1建站

让我们模拟一个真实场景：**7天做一个AI写作工具的完整网站。**

### Day 1-2：设计和落地页

**目标：** 完成品牌设计和高转化落地页

**步骤：**

```
# 1. 用frontend-design设计品牌风格  
"帮我设计一个AI写作工具的品牌风格，目标用户是内容创作者"  
  
# 2. 用landing-page-guide-v2生成落地页结构  
"根据DESIGNNAS框架，生成完整的落地页"  
  
# 3. 迭代优化  
"这个Hero区的文案不够吸引人，帮我改得更有冲击力"
```

**产出：**

* ✅ 品牌配色、字体、Logo
* ✅ 落地页的React组件代码
* ✅ 响应式设计（移动端适配）

---

### Day 3-4：开发和CMS

**目标：** 搭建产品原型和博客系统

**步骤：**

```
# 1. 初始化项目  
npx create-next-app@latest myapp  
cd myapp  
  
# 2. 集成Payload CMS  
npx create-payload-app blog  
cd blog && npm run dev  
  
# 3. 配置Content Collections  
"帮我创建Blog Post的Collection，包含SEO字段"  
  
# 4. 连接前端  
"在Next.js中调用Payload API，渲染博客文章列表"
```

**产出：**

* ✅ Next.js前端应用
* ✅ Payload CMS后台
* ✅ 博客系统（支持发布文章）

---

### Day 5-6：部署和文档

**目标：** 部署到生产环境，编写用户文档

**步骤：**

```
# 1. 用Terraform配置基础设施  
"帮我写一个Terraform配置，部署到AWS"  
  
# 2. 执行部署  
terraform init  
terraform apply  
  
# 3. 用docs-write编写文档  
"帮我写一个快速开始文档，教用户如何使用AI写作工具"  
  
# 4. 配置Git工作流  
"帮我设置Feature Branch工作流，配置CI/CD"
```

**产出：**

* ✅ 网站部署到生产环境
* ✅ 用户文档和API文档
* ✅ 自动化部署流程

---

### Day 7：上线和优化

**目标：** 最后检查，正式上线

**检查清单：**

* [ ] 落地页转化元素齐全（UVP、CTA、社会证明）
* [ ] 博客至少有3篇SEO文章
* [ ] 文档完整（快速开始、API文档、FAQ）
* [ ] 全球CDN加速
* [ ] 移动端适配
* [ ] SSL证书配置
* [ ] Google Analytics埋点

**上线：**

```
# 切换DNS  
# 监控网站性能  
# 在Product Hunt发布
```

---

## 安装和使用指南

### 1. 安装Claude Code

**方式一：官网下载**

* 访问 claude.ai/code
* 下载对应系统版本（Mac/Windows/Linux）
* 安装并登录

**方式二：通过包管理器（推荐）**

```
# macOS  
brew install --cask claude-code  
  
# Windows  
winget install Anthropic.ClaudeCode  
  
# Linux  
curl -fsSL https://claude.ai/install.sh | sh
```

---

### 2. 安装Skills

**方式一：从GitHub克隆**

```
# 1. 创建Skills目录  
mkdir -p ~/.claude/skills  
  
# 2. 克隆Skill仓库  
cd ~/.claude/skills  
  
# 安装frontend-design  
git clone https://github.com/anthropics/claude-code.git  
cp -r claude-code/plugins/frontend-design/skills/frontend-design ./  
  
# 安装其他Skills  
git clone https://github.com/xxx/landing-page-guide-v2.git  
git clone https://github.com/xxx/docs-write.git  
# ...
```

**方式二：通过Claude Code插件市场（如果支持）**

```
# 在Claude Code中执行  
/plugin install frontend-design  
/plugin install landing-page-guide-v2  
/plugin install docs-write  
/plugin install git-advanced-workflows  
/plugin install payload  
/plugin install terraform-module-library
```

---

### 3. 验证安装

启动Claude Code，输入：

```
列出我已安装的所有Skills
```

你应该看到：

```
已安装的Skills：  
✅ frontend-design  
✅ landing-page-guide-v2  
✅ docs-write  
✅ git-advanced-workflows  
✅ payload  
✅ terraform-module-library
```

---

### 4. 常见问题

**Q：Skill不生效怎么办？**

```
# 1. 检查目录结构  
ls -la ~/.claude/skills/frontend-design/  
# 应该有 SKILL.md 文件  
  
# 2. 重启Claude Code  
  
# 3. 手动加载  
/skills reload
```

**Q：可以同时使用多个Skills吗？**

可以！Claude会自动识别场景，调用合适的Skill。

**Q：Skills会收费吗？**

大部分Skills是开源免费的。使用Skills只消耗你的Claude API额度。

**Q：可以自己开发Skills吗？**

可以！参考官方文档：docs.anthropic.com/skills

---

## 写在最后

回到文章开头的问题：**出海建站，5万预算够不够？**

**用这7个Skills，你连5000块都不用花。**

**快速建站MVP方案（成本：接近0）：**

* frontend-design：设计费省下2万
* landing-page-guide-v2：文案策划省下5000
* docs-write：技术写作省下8000
* git-advanced-workflows：版本管理规范，避免返工

**专业出海网站方案（成本：服务器费用）：**

* payload：CMS订阅费省下每月$299
* terraform-module-library：运维成本降低70%

**更重要的是时间成本：**

* 传统方式：找外包2-3个月
* 用Skills：7天MVP，14天专业站

---

## 选择适合你的方案

**如果你是：**

### 🚀 首次创业者 / 想快速验证想法

→ **选快速MVP方案**

* 先用frontend-design + landing-page-guide-v2做落地页
* 用docs-write写1-2篇SEO文章
* 跑通第一批用户后，再考虑升级

### 💼 有产品经验 / 计划长期运营

→ **直接上专业方案**

* 前期多花3-5天，搭建Payload CMS
* 配置Terraform多区域部署
* 建立规范的Git工作流

### 🎯 技术团队 / 需要可扩展性

→ **两个方案都用**

* MVP阶段：快速验证
* 成长阶段：升级基础设施
* 成熟阶段：多云、多区域、CDN

---

## 下一步

**立即行动：**

1. **安装Claude Code** → claude.ai/code
2. **安装7个Skills** → 按照上面的指南操作
3. **找个周末，做个MVP** → 7天后，你会有自己的产品

**进阶学习：**

* 📚 skillsmp.com - 探索更多Skills（44,000+个）
* 📖 Claude Code文档 - 深入学习
* 💬 Discord社区 - 和其他开发者交流

---

**最后，记住一句话：**

> "最好的产品不是最完美的，而是最快上线的那个。"

不要等一切就绪才开始。用这7个Skills，这个周末就动手吧。

你的第一个出海产品，正在等着你。🚀

---

如果这篇文章对你有帮助，欢迎分享给更多出海创业者。

有问题？在评论区留言，我会尽快回复。👇