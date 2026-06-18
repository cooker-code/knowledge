---
title: 万星|开源|用 React 代码"写"视频，超级生产力
author: Lin夕的AI沉思录
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUyNTk1NTM2Ng==&mid=2247488059&idx=1&sn=0c947bf10f2cbebb01c3dab3c523c37d&chksm=fbe9d86c175ad8485a6c09ef3e0ac64612b30424fc2075405b1b1d5bb67ffaa84b3529fdd3be&mpshare=1&scene=24&srcid=0414ImihPtVL7mfz4oqR3EAw&sharer_shareinfo=87db049f61d39f52c77d0e564bd5d0b8&sharer_shareinfo_first=87db049f61d39f52c77d0e564bd5d0b8#rd
---

# 

> **导读**：当GitHub Unwrapped刷爆朋友圈时，你可能不知道，一个叫Remotion 的开源框架正在悄然改变视频制作的未来。

---

## 一、缘起：为什么需要"用代码做视频"？

### 1.1 传统视频编辑的痛点

如果你用过 Adobe Premiere 或 After Effects，一定经历过这些崩溃时刻：

* **学习曲线陡峭**：新手入门至少需要几周时间熟悉界面和功能
* **重复劳动繁琐**：要生成 100 个相似视频？手动调整 100 次参数吧
* **版本管理噩梦**：想回退到昨天的版本？祝你好运
* **动态数据无力**：要根据 Excel 表格生成个性化视频？基本不可能
* **协作效率低下**：多人同时编辑一个项目？想都别想

一位资深视频编辑曾吐槽："我花了 3 个小时调整一个文字动画的位置，结果客户说要改文案……一切重来。"

### 1.2 程序化视频的崛起

2025 年，GitHub Unwrapped 横空出世，数百万开发者收到了属于自己的年度代码总结视频。这些视频质量精良、高度个性化，但违背常理的是？**它们不是人工剪辑的，而是代码自动生成的**。

背后的功臣就是 **Remotion** —— 一个"用 React 编程创建视频"的开源框架。

> **项目地址**：https//github.com/remotion-dev/remotion  
> **Stars**：超过 35,000 ⭐  
> **NPM 月下载量**：40 万+  
> **许可证**：Remotion License（公司 4 人以上需商业授权）

---

## 二、Remotion 是什么？

### 2.1 核心理念

用官方的一句话概括：

> **"🎥 Make videos programmatically with React"**  
> （用 React 程序化地制作视频）

听起来很玄？其实逻辑非常简单：

```
你写 React 组件 → Remotion 逐帧渲染 → 合成 MP4 视频
```

想象一下，你把视频的每一帧都当成一个 React 组件的"状态"，通过改变状态（props、state、时间轴）来控制画面变化，最后 Remotion 帮你把这些"状态"渲染成真正的视频文件。

### 2.2 技术架构

Remotion 的核心工作流程：

```
graph LR  
    A[React 组件] --> B[Remotion Studio 预览]  
    B --> C[渲染引擎]  
    C --> D[Headless Chrome]  
    D --> E[帧序列]  
    E --> F[FFmpeg 编码]  
    F --> G[MP4 输出]
```

**关键技术栈**：

* **React**：组件化视频结构
* **TypeScript**：类型安全的开发体验
* **Headless Chrome**：渲染每一帧画面
* **FFmpeg**：将帧序列编码为视频
* **Node.js/Bun**：运行时环境

### 2.3 为什么选择 React？

创始人 Jonny Burger 解释了选择 React 的原因：

1. **复用 Web 技术**：CSS、Canvas、SVG、WebGL 全部可用
2. **编程能力加持**：变量、函数、API、数学算法都能创造特效
3. **React 生态优势**：可复用组件、强大组合能力、快速刷新、海量 npm 包

用一位开发者的话说："**我会用 React 组件，做视频就很轻松。**"

---

## 三、快速上手：10 分钟创建第一个视频

### 3.1 环境准备

需要安装 Node.js 16+ 或 Bun 1.0.3+：

```
# 检查 Node.js 版本  
node -v  
  
# Linux 用户需要 Libc 2.35+  
ldd --version
```

### 3.2 创建项目

使用官方脚手架工具（推荐新手选择 Hello World 模板）：

```
# 使用 npm  
npx create-video@latest  
  
# 使用 pnpm  
pnpm create video  
  
# 使用 Yarn  
yarn create video  
  
# 使用 Bun  
bun create video
```

按照提示选择模板后，进入项目目录：

```
cd your-video-project  
npm run dev
```

### 3.3 启动 Remotion Studio

```
npm run remotion
```

浏览器会自动打开 `http://localhost:3000`，你会看到一个可视化的视频预览界面。

### 3.4 第一个视频组件

打开 `src/HelloWorld.tsx`，你会看到类似这样的代码：

```
import { AbsoluteFill, useCurrentFrame, spring } from 'remotion';  
  
export const HelloWorld: React.FC = () => {  
  const frame = useCurrentFrame();  
  const opacity = spring({ frame, fps: 30 });  
  
  return (  
    <AbsoluteFill style={{ backgroundColor: 'white', justifyContent: 'center', alignItems: 'center' }}>  
      <h1 style={{ fontSize: 100, opacity }}>Hello, World!</h1>  
    </AbsoluteFill>  
  );  
};
```

**代码解析**：

* `useCurrentFrame()`：获取当前帧数（类似时间轴位置）
* `spring()`：应用弹簧物理动画，让过渡更自然
* `AbsoluteFill`：Remotion 提供的布局组件，占满整个画面

修改代码后，Studio 会**实时热更新**预览效果，就像开发网页一样流畅。

### 3.5 渲染导出

预览满意后，导出为 MP4：

```
npx remotion render out/video.mp4 HelloWorld
```

渲染完成后，`out/video.mp4` 就是你的第一个程序化视频！

---

## 四、核心功能详解

### 4.1 时间轴控制

Remotion 将视频视为时间的函数，通过帧来控制内容：

```
import { useCurrentFrame, interpolate } from 'remotion';  
  
const frame = useCurrentFrame();  
  
// 在第 0-30 帧（1 秒）内，透明度从 0 渐变到 1  
const opacity = interpolate(frame, [0, 30], [0, 1]);  
  
// 在第 30-60 帧，文字从左侧滑入  
const translateX = interpolate(frame, [30, 60], [-500, 0]);
```

**关键 API**：

* `useCurrentFrame()`：当前帧号
* `useVideoConfig()`：获取视频配置（分辨率、帧率、时长）
* `interpolate()`：值插值，实现平滑过渡
* `spring()`：弹簧动画，更自然的物理效果

### 4.2 组件化视频结构

像搭积木一样构建视频：

```
// 可复用的标题组件  
const Title: React.FC<{ text: string }> = ({ text }) => {  
  return (  
    <div style={{ fontSize: 80, color: '#333' }}>  
      {text}  
    </div>  
  );  
};  
  
// 主视频组合  
export const MyVideo: React.FC = () => {  
  return (  
    <AbsoluteFill>  
      <Background color="white" />  
      <Title text="第一章：开篇" />  
      <ContentSection />  
      <Footer />  
    </AbsoluteFill>  
  );  
};
```

**优势**：

* ✅ 组件复用：一次编写，多处使用
* ✅ 版本控制：Git 管理视频"源码"
* ✅ 测试驱动：单元测试验证视频逻辑
* ✅ 团队协作：多人并行开发不同组件

### 4.3 动态数据驱动

这是 Remotion 最强大的能力之一 —— **根据数据批量生成视频**：

```
interface UserData {  
  name: string;  
  avatar: string;  
  totalCommits: number;  
  topLanguage: string;  
}  
  
interface Props {  
  user: UserData;  
}  
  
export const PersonalizedVideo: React.FC<Props> = ({ user }) => {  
  return (  
    <AbsoluteFill>  
      <img src={user.avatar} style={{ width: 200, borderRadius: '50%' }} />  
      <h1>{user.name} 的年度总结</h1>  
      <p>全年提交了 {user.totalCommits} 次代码</p>  
      <p>最常用的语言：{user.topLanguage}</p>  
    </AbsoluteFill>  
  );  
};  
  
// 批量渲染 1000 个个性化视频  
const users = await fetchUsersFromAPI();  
users.forEach(user => {  
  renderMedia({  
    composition: PersonalizedVideo,  
    inputProps: { user },  
    output: `output/${user.name}.mp4`  
  });  
});
```

**应用场景**：

* 📊 个性化年度报告（GitHub Unwrapped 模式）
* 🎯 定制化营销视频（电商商品推广）
* 📈 数据可视化报告（KPI 动态展示）
* 🎓 在线教育证书视频（学员专属）

### 4.4 多媒体支持

Remotion 支持丰富的媒体类型：

```
import { Img, Video, Audio } from 'remotion';  
  
// 图片  
<Img src="https://example.com/image.jpg" style={{ width: '100%' }} />  
  
// 视频（可作为背景或画中画）  
<Video src={videoUrl} style={{ width: '100%' }} />  
  
// 音频（背景音乐、配音）  
<Audio src={audioUrl} />  
  
// SVG 矢量图  
<SvgComponent />  
  
// Canvas 绘图  
<canvas ref={canvasRef} />  
  
// WebGL 3D 效果  
<WebGLShader />
```

**注意**：嵌入视频可能是性能瓶颈，建议结合 FFmpeg 构建高效流水线。

### 4.5 动画系统

Remotion 提供多种动画工具：

#### 弹簧动画（Spring）

```
import { spring, useCurrentFrame } from 'remotion';  
  
const frame = useCurrentFrame();  
const scale = spring({  
  frame,  
  fps: 30,  
  config: { damping: 20, stiffness: 100 }  
});  
  
<div style={{ transform: `scale(${scale})` }} />
```

#### 插值动画（Interpolate）

```
import { interpolate, useCurrentFrame } from 'remotion';  
  
const frame = useCurrentFrame();  
  
// 线性插值  
const opacity = interpolate(frame, [0, 60], [0, 1]);  
  
// 缓入缓出  
const rotate = interpolate(frame, [0, 120], [0, 360], {  
  extrapolateRight: 'clamp',  
  easing: 'easeInOut'  
});
```

#### 序列动画

```
import { Sequence } from 'remotion';  
  
<Sequence>  
  {/* 0-2 秒：显示标题 */}  
  <Title durationInFrames={60} />  
  
  {/* 2-5 秒：显示内容 */}  
  <Sequence from={60}>  
    <Content durationInFrames={90} />  
  </Sequence>  
  
  {/* 5-7 秒：淡出 */}  
  <Sequence from={150}>  
    <FadeOut />  
  </Sequence>  
</Sequence>
```

---

## 五、实战案例：Remotion 能做什么？

### 5.1 GitHub Unwrapped 模式

**场景**：为每个用户生成个性化的年度总结视频

**技术要点**：

* 从 GitHub API 获取用户数据
* 动态生成图表动画
* 批量渲染数千个视频

**效果**：单个视频渲染时间 < 30 秒，成本仅为传统制作的 1/10

### 5.2 电商商品推广视频

**场景**：电商平台有 10,000 个商品，需要为每个商品生成推广视频

**传统方案**：

* 设计师制作模板：3 天
* 手动替换商品信息：10,000 × 5 分钟 = 833 小时
* 总成本：约 50 万元

**Remotion 方案**：

```
interface Product {  
  id: string;  
  name: string;  
  price: number;  
  images: string[];  
  description: string;  
}  
  
const ProductVideo: React.FC<{ product: Product }> = ({ product }) => {  
  return (  
    <AbsoluteFill>  
      <Slideshow images={product.images} />  
      <Title>{product.name}</Title>  
      <Price>{product.price}</Price>  
      <Description>{product.description}</Description>  
      <CTAButton>立即购买</CTAButton>  
    </AbsoluteFill>  
  );  
};  
  
// 批量渲染  
products.forEach(product => {  
  renderMedia({  
    composition: ProductVideo,  
    inputProps: { product },  
    output: `videos/${product.id}.mp4`  
  });  
});
```

**结果**：

* 开发时间：2 天
* 批量渲染：10,000 个视频 / 12 小时（云端并行）
* 总成本：约 2 万元
* **节省 96% 成本** ⚡

### 5.3 企业培训视频自动化

**案例**：某金融科技公司需要每周更新产品培训视频

**挑战**：

* 产品信息频繁变化
* 需要多语言版本（中/英/日）
* 合规要求高，不能出错

**Remotion 解决方案**：

```
const TrainingVideo: React.FC<{  
  productData: ProductInfo;  
  language: 'zh' | 'en' | 'ja';  
}> = ({ productData, language }) => {  
  const translations = getTranslations(language);  
  
  return (  
    <>  
      <Header title={translations.welcome} />  
      <ProductSpecs data={productData} />  
      <ComplianceDisclaimer text={translations.disclaimer} />  
      <Quiz questions={generateQuiz(productData)} />  
    </>  
  );  
};
```

**成效**：

* 更新时间从 3 天缩短到 3 小时
* 多语言版本同步生成
* 100% 准确性（数据驱动，无人工错误）

### 5.4 社交媒体批量内容

**场景**：MCN 机构需要为 50 个博主生成每日短视频

**工作流**：

1. 从 CMS 获取当日热点话题
2. 调用 AI 生成文案
3. Remotion 自动匹配素材库
4. 批量渲染并上传到抖音/B 站/YouTube

**代码示例**：

```
const SocialMediaVideo: React.FC<{  
  topic: TrendingTopic;  
  creator: CreatorProfile;  
}> = ({ topic, creator }) => {  
  return (  
    <VerticalVideo format="9:16">  
      <HookText text={topic.hook} />  
      <ContentSlides slides={topic.slides} />  
      <CreatorBranding profile={creator} />  
      <CallToAction action="follow" />  
    </VerticalVideo>  
  );  
};
```

**产出**：每天自动生成 200+ 条视频，人工只需审核

### 5.5 数据可视化视频

**场景**：将 dashboard 报表转为可分享的视频

```
const ChartAnimation: React.FC<{ data: DataPoint[] }> = ({ data }) => {  
  const frame = useCurrentFrame();  
  
  return (  
    <AbsoluteFill>  
      {data.map((point, index) => {  
        const height = interpolate(frame, [0, 60], [0, point.value]);  
        return (  
          <Bar  
            key={index}  
            height={height}  
            delay={index * 10} // 依次出现  
          />  
        );  
      })}  
    </AbsoluteFill>  
  );  
};
```

**应用**：

* 月度 KPI 报告视频
* 融资路演数据演示
* 政府公开数据可视化

---

## 六、Remotion vs 传统视频编辑

### 6.1 对比表格

| 维度 | Remotion | Premiere Pro | After Effects |
| --- | --- | --- | --- |
| **学习曲线** | 低（会 React 即可） | 高（数周入门） | 极高（数月精通） |
| **批量生产** | ✅ 代码循环自动生成 | ❌ 手动重复操作 | ❌ 手动重复操作 |
| **版本控制** | ✅ Git 完美支持 | ⚠️ 二进制文件难管理 | ⚠️ 二进制文件难管理 |
| **动态数据** | ✅ 原生支持 | ❌ 几乎不可能 | ❌ 需要插件 |
| **协作效率** | ✅ 多人并行开发 | ⚠️ 单文件锁定 | ⚠️ 单文件锁定 |
| **自动化测试** | ✅ 单元测试 | ❌ 不支持 | ❌ 不支持 |
| **成本（企业）** | $500/年（商业许可） | ¥3888/年 | ¥3888/年 |
| **适用场景** | 程序化、批量化、数据驱动 | 影视剪辑、纪录片 | 特效、动画 |

### 6.2 何时选择 Remotion？

✅ **适合场景**：

* 需要批量生成相似视频
* 视频内容依赖动态数据
* 团队已有 React 开发能力
* 需要版本控制和 CI/CD
* 追求自动化和可扩展性

❌ **不适合场景**：

* 一次性创意视频（如电影短片）
* 大量实拍素材剪辑
* 复杂的手绘动画
* 需要精细音频处理
* 团队无编程基础

---

## 七、性能优化指南

### 7.1 渲染速度优化

**问题**：渲染太慢怎么办？

**解决方案**：

#### 1. 使用 RAM 磁盘

```
# Linux 创建 RAM 磁盘  
mount -t tmpfs -o size=4G tmpfs /mnt/ramdisk  
  
# 设置 Remotion 临时目录  
export REMOTION_TEMP_DIR=/mnt/ramdisk/remotion
```

**效果**：比 SSD 快 1.5 倍，比机械硬盘快 4 倍

#### 2. GPU 加速渲染

```
# 启用 GPU 渲染  
npx remotion render --gpu out/video.mp4 Composition
```

**注意**：需要 NVIDIA 显卡 + CUDA 驱动

#### 3. 并发渲染

```
import { renderMediaInParallel } from '@remotion/lambda';  
  
// 在 AWS Lambda 上并行渲染  
await renderMediaInParallel({  
  compositions: [{ id: 'Comp1', props: {...} }],  
  concurrencyPerLambda: 4, // 每个 Lambda 并发 4 个  
  memorySizeInMb: 3009,    // 3 vCPU  
});
```

#### 4. 缓存策略

```
import { useMemo } from 'react';  
  
const ExpensiveComponent: React.FC<{ data: Data }> = ({ data }) => {  
  // 避免重复计算  
  const processed = useMemo(() => {  
    return heavyComputation(data);  
  }, [data]);  
  
  return <div>{processed}</div>;  
};
```

### 7.2 内存优化

**常见问题**：渲染大文件时内存溢出

**解决技巧**：

1. **降低预览分辨率**

```
import { Config } from 'remotion';  
Config.setPreviewSize({ width: 1280, height: 720 });
```

2. **分阶段渲染**

```
# 先渲染序列 0-100  
npx remotion render --sequence=0-100 out/part1.mp4 Comp  
  
# 再渲染序列 100-200  
npx remotion render --sequence=100-200 out/part2.mp4 Comp  
  
# 用 FFmpeg 合并  
ffmpeg -i part1.mp4 -i part2.mp4 -filter_complex concat final.mp4
```

3. **图片格式选择**

* JPEG：更快，但不支持透明
* PNG：支持透明，但较慢
* WebP：平衡选择（实验性支持）

### 7.3 调试工具

Remotion 内置性能分析：

```
# 启用详细日志  
npx remotion render --log-level=verbose out/video.mp4 Comp  
  
# 生成性能报告  
npx remotion profile out/video.mp4 Comp
```

查看每帧渲染时间，定位瓶颈组件。

---

## 八、部署与云端渲染

### 8.1 本地部署

**最低配置**：

* CPU：4 核
* 内存：8GB
* 存储：SSD 优先
* 系统：Linux（推荐 Ubuntu 20.04+）

**Docker 部署**：

```
FROM node:18-alpine  
  
# 安装依赖  
RUN apk add --no-cache chromium ffmpeg  
  
WORKDIR /app  
COPY package*.json ./  
RUN npm ci  
COPY . .  
  
CMD ["npm", "run", "render"]
```

### 8.2 云端渲染（AWS Lambda）

Remotion 官方提供 Lambda 渲染服务：

```
# 创建 Lambda 站点  
npx remotion lambda sites create --site-name=my-render  
  
# 上传并渲染  
npx remotion lambda render my-render CompositionId \  
  --delete-after="7-days"
```

**定价**：按实际渲染时间计费，通常 $0.01-0.05/分钟视频

**优势**：

* 无需管理服务器
* 弹性伸缩
* 并行渲染速度快

### 8.3 自建渲染农场

大规模生产场景：

```
┌─────────────┐  
│ 任务队列     │ (Redis/RabbitMQ)  
└──────┬──────┘  
       │  
   ┌───┴───┐  
   │       │  
┌──▼──┐ ┌──▼──┐  
│节点 1│ │节点 2│ ... N 个节点  
└──┬──┘ └──┬──┘  
   │       │  
   └───┬───┘  
       │  
┌──────▼──────┐  
│ 共享存储     │ (S3/NAS)  
└─────────────┘
```

**参考架构**：

* Kubernetes 编排容器
* S3 存储素材和输出
* Redis 任务调度
* 自动扩缩容

---

## 九、生态系统与社区

### 9.1 官方资源

* **文档**：https://www.remotion.dev/docs
* **API 参考**：https://www.remotion.dev/api
* **案例展示**：https://www.remotion.dev/showcase
* **Discord 社区**：活跃开发者交流

### 9.2 第三方库

**remotion-bits**：视频构建模块库

```
npm install @remotion/bits
```

**remotion-lottie**：支持 Lottie 动画

```
npm install @remotion/lottie
```

**remotion-google-fonts**：谷歌字体集成

```
npm install @remotion/google-fonts
```

### 9.3 学习资源

* **官方教程视频**：YouTube 频道 "Remotion"
* **中文社区**：掘金、CSDN 活跃博主
* **示例项目**：GitHub 搜索 "remotion-example"

---

## 十、结语：你的视频，你来定义

Remotion 不仅仅是一个工具，它代表了一种新的创作范式：

* **从手动到自动**：解放重复劳动
* **从静态到动态**：数据驱动内容
* **从孤立到协作**：代码即协作语言
* **从艺术到工程**：视频也可测试、可版本控制

当然，Remotion 不会完全取代传统视频编辑软件。就像 Photoshop 和 CSS 的关系——各有适用场景，相互补充。

**关键问题是**：你想把时间花在重复操作上，还是创造力上？

---

## 附录：快速参考

### 常用命令

```
# 创建项目  
npx create-video@latest  
  
# 启动预览  
npm run dev  
  
# 启动 Studio  
npm run remotion  
  
# 渲染视频  
npx remotion render out/video.mp4 CompositionId  
  
# GPU 加速渲染  
npx remotion render --gpu out/video.mp4 CompositionId  
  
# 云端渲染  
npx remotion lambda render my-site CompositionId  
  
# 性能分析  
npx remotion profile out/video.mp4 CompositionId
```

### 核心 API 速查

```
import {  
  useCurrentFrame,      // 当前帧号  
  useVideoConfig,       // 视频配置  
  interpolate,          // 值插值  
  spring,               // 弹簧动画  
  Sequence,             // 序列容器  
  AbsoluteFill,         // 占满容器  
  Img, Video, Audio,    // 媒体组件  
  Config,               // 全局配置  
} from 'remotion';
```

### 学习路径建议

**第 1 周**：完成 Hello World，理解帧和时间的关系  
**第 2 周**：学习动画 API（spring、interpolate）  
**第 3 周**：掌握组件化和 props 传递  
**第 4 周**：尝试动态数据驱动  
**第 2 个月**：第一个生产级项目上线

---

## 参考资料

1. Remotion 官方文档：https://www.remotion.dev
2. GitHub 仓库：https://github.com/remotion-dev/remotion
3. GitHub Unwrapped 案例：https://github-unwrapped.com
4. Remotion 最佳实践：https://www.remotion.dev/docs/performance
5. 社区教程合集：https://www.remotion.dev/showcase

---

> **互动话题**：你对程序化视频感兴趣吗？有什么想做的视频项目？欢迎在评论区留言讨论～

---

**支持开源**：如果 Remotion 帮到了你，考虑给项目点个 Star 或赞助作者！⭐