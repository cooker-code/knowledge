> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020602_VibeCode/020602_核心知识点/VibeCode边界与交付判断准则|VibeCode边界与交付判断准则]]
---
title: AI代码评审CodeReview第20节：集成字节组件库的界面设计
author: 无处不在的技术
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648933662&idx=1&sn=403ec36c54aa18cc15ad76b536d62000&chksm=f1edd28331136997c170e5075587ea75663acf109d9c6bfbc715d8e48a778fd446b93b780c53&mpshare=1&scene=24&srcid=1209gxCeBXDTv9u4ScZ5NPJa&sharer_shareinfo=1e73b976c662c38af0997e315f0c421f&sharer_shareinfo_first=1e73b976c662c38af0997e315f0c421f#rd
---

### 持续内容输出，点击蓝字关注我吧

**01**

**前言**

来看一段AI大模型对它的介绍

上篇文章中，我分享了基于AI Hub这个AI客户端工具的源码搭建过程，同时开发了一个HelloWord的流程演示，在AI时代，程序员的AI大模型应用开发能力、AI工具的使用能力、AI的创新能力非常重要，通过AI可以帮助程序员在个人和企业的工作中实现提效。AI Hub工具也是基于Vue、Electron跨端技术、字节跳动Arco-design组件库、AI大模型等等技术为一体的一个客户端工具，学会如何基于这个工具进行二次开发，将有助于我们开发出一个属于自己的跨端类的AI客户端软件。

本节主要分享这个工具中的Arco-design的界面设计和基本使用，希望每个人都学会并设计出属于自己的AI工具，Arco-design是字节跳动出品的企业级设计系统，我们用的是Vue版本。

07高峰论坛会

**02**

**正文**

下面先分享分享这个语言的东西，可以仔细看看。

1、在编写代码之前，我们先来分析下本节我们需要做的内容：

**开发一个AI代码评审的设置界面，用于维护AI代码评审的相关配置信息****开发一个AI代码评审的主界面的功能**

2、首先来分析下我们的主界面元素：

在这个主界面中，上面是基本的描述区域，然后是Tab区域，然后左侧的内容配置区域，以及右侧的数据预览区域，本节主要分享界面的布局以及，上方右侧的设置界面。

再来看下设置界面：

设置界面由几个文本框构成。

在开发界面之前，先在VS Code中安装个格式统一的插件，有了这个插件后，我们写的代码格式化的时候，就可以按照统一的配置进行格式化，避免出现很多黄色的警告：

我们先来看下主界面的布局开发，首先我们在src目录下的renderer下的src下的components下的views目录下新建ai-codereview文件夹：

然后创建CodeReview.vue的文件，文件中增加如下代码：

代码如下所示：

```
<script setup lang="ts">import { Message, Modal } from '@arco-design/web-vue'</script><template>  <div class="codereview">    <!-- 头部 -->    <div class="codereview-header">      <div class="codereview-header-title">AI大模型CodeReview</div>      <div class="codereview-header-setting">        <a-button class="no-drag-area" type="primary" size="small">          <a-space :size="5">            <icon-settings />            <span>AI代码评审设置</span>          </a-space>        </a-button>      </div>    </div>  </div></template><style scoped lang="less">.codereview {  flex-grow: 1;  display: flex;  flex-direction: column;  overflow: hidden;  position: relative;  .codereview-header {    height: 55px;    border-bottom: 1px solid var(--color-border-1);    box-sizing: border-box;    padding: 15px;    display: flex;    align-items: center;    justify-content: space-between;    .codereview-header-title {      flex-grow: 1;      font-size: var(--font-size-lg);      font-weight: 500;    }    .codereview-header-setting {      flex-shrink: 0;    }  }  .codereview-body {    padding: 20px;    flex-grow: 1;    min-height: 0;    display: flex;    gap: 10px;    box-sizing: border-box;    padding: 15px;    .input-area {      flex: 1;      min-height: 0;      height: 100%;      display: flex;      flex-direction: column;      gap: 10px;    }    .preview-area {      flex: 1;      min-height: 0;      height: 100%;      display: flex;      flex-direction: column;      gap: 10px;    }  }}</style>
```

然后修改App.vue文件，在上边引入这个文件，以及在下面替换上回的HelloWorld区域：

在下面的内容区域更改：

这样更改后的内容的界面如下所示：

接下来我们分析下这个界面中我们学习到的前端组件：

```
 <a-button class="no-drag-area" type="primary" size="small">          <a-space :size="5">            <icon-settings />            <span>AI代码评审设置</span>          </a-space>        </a-button>
```

这里主要用到了按钮组件a-button、间距组件a-space，这两个组件的地址如下：

https://arco.design/vue/component/button

https://arco.design/vue/component/space

建议小伙伴们看下文档学习下基本使用，比如按钮可以设置不同的样式：

接下来我们给这个右上角的按钮，增加个单击事件：

这里我用到了Message这个组件，这个组件的用法的文档如下：

https://arco.design/vue/component/message

然后我们运行这个效果如下所示：

可以看到这个按钮的单击事件已经成功的响应了。

接下来我们就画一下页面设置的内容，首先我们分析下这个页面的组成结构：

这几个组件的文档如下所示：

对话框组件：https://arco.design/vue/component/modal

输入框组件：https://arco.design/vue/component/input

文本域组件：https://arco.design/vue/component/textarea

栅格组件：https://arco.design/vue/component/grid

熟悉了这几个组件后，我们在当前目录下新增文件：CodeReviewSetting.vue，在这个文件中通过a-modal组件定义为对话框，然后通过a-row标签作为页面上按行布局，然后每一行中使用a-col进行列布局，同时分别使用a-input组件进行输入框的定义，截图如下所示：

这段的文件代码如下所示：

```
<script setup lang="ts">import { useCodeReviewSettingStore } from './store/codereview'const codeReviewSettingStore = useCodeReviewSettingStore()const modalVisible = defineModel<boolean>('modalVisible', { default: () => false })</script><template>  <div class="codereview-setting-modal">    <!-- CodeReviewModal -->    <a-modal v-model:visible="modalVisible" unmount-on-close title-align="start" width="80vw" :footer="false">      <template #title>AI代码评审设置</template>      <!-- 提醒页 -->      <div class="codereview-setting-page">        <a-space direction="vertical" :size="5" fill class="setting-tab-content">          <a-row class="grid-demo" :gutter="24" justify="space-between">            <a-col :span="12">              <a-space direction="vertical" :size="10" fill>                <div>Gitlab的仓库地址</div>                <a-input v-model="codeReviewSettingStore.gitlabUrl" size="small"                  placeholder="请输入Gitlab仓库的API根地址,例如：https://www.gitlab.com.cn/api/v4" />              </a-space>            </a-col>            <a-col :span="12">              <a-space direction="vertical" :size="10" fill>                <div>Gitlab的访问令牌</div>                <a-input v-model="codeReviewSettingStore.gitlabToken" size="small" placeholder="请输入Gitlab访问令牌" />              </a-space>            </a-col>          </a-row>          <a-row class="grid-demo" :gutter="24" justify="space-between">            <a-col :span="12">              <a-space direction="vertical" :size="10" fill>                <div>大模型的API地址(系统会自动拼接/chat/completions)</div>                <a-input v-model="codeReviewSettingStore.modelUrl" size="small"                  placeholder="请输入大模型的API地址,例如：http://ip:port/" />              </a-space>            </a-col>            <a-col :span="12">              <a-space direction="vertical" :size="10" fill>                <div>大模型的访问KEY</div>                <a-input v-model="codeReviewSettingStore.modelKey" size="small" placeholder="请输入大模型的访问key" />              </a-space>            </a-col>          </a-row>          <a-space direction="vertical" :size="10" fill>            <div>飞书通知Webhook机器人</div>            <a-input v-model="codeReviewSettingStore.feishuWebhook" size="small" placeholder="飞书群机器人API回调地址" />          </a-space>          <a-space direction="vertical" :size="25" fill>            <a-space direction="vertical" :size="10" fill>              <div>默认System指令提示词</div>              <a-textarea v-model="codeReviewSettingStore.systemPrompt" allow-clear style="height:200px;" />            </a-space>          </a-space>        </a-space>      </div>    </a-modal>  </div></template><style lang="less" scoped>.codereview-setting-page {  height: 60vh;  overflow-y: auto;  font-size: var(--font-size-default);}</style>
```

这段代码的上方有些代码会报错，这个是因为我准备引入数据存储的相关内容，我们知道当前开发的是一个设置界面，那么对于这个设置界面，最终我们需要的时候它可以存储起来，这里我们需要定义一个store文件，因此我们需要在当前目录下再定义一个store文件:

这里我们用到了状态管理 - Pinia这个组件，全局状态管理是一个大型系统不可避免的存在，因为经常有一些需要全局共享的信息需要存储。有了这个东西后，我们就可以在设置界面中来引入这个store，然后再HTML标签中通过v-model的方式来关联这个配置，这样当我们在界面中输入好具体的数值以后，就会通过pinia组件内部的功能来持久化我们的设置数据，截图如下所示：

接下来我们在CodeReview.vue文件中引入这个弹窗组件，实现单击按钮的显示弹窗，然后可以设置弹窗数据，关闭进程在打开后数据仍然存在，代码截图如下所示：

这里的代码如下：

```
<script setup lang="ts">import { reactive, toRefs } from 'vue'import { Message, Modal } from '@arco-design/web-vue'import CodeReviewSetting from '@renderer/components/views/ai-codereview/CodeReviewSetting.vue'import { useCodeReviewSettingStore } from '@renderer/components/views/ai-codereview/store/codereview'// storeconst codeReviewSettingStore = useCodeReviewSettingStore()const data = reactive({  // 代码评审设置modal  codeSettingModalVisible: false})const { codeSettingModalVisible } = toRefs(data)//  显示设置的方法函数const showSetting = () => {  data.codeSettingModalVisible = true;}</script><template>  <div class="codereview">    <!-- 头部 -->    <div class="codereview-header">      <div class="codereview-header-title">AI大模型CodeReview</div>      <div class="codereview-header-setting">        <a-button class="no-drag-area" type="primary" size="small" @click="showSetting">          <a-space :size="5">            <icon-settings />            <span>AI代码评审设置</span>          </a-space>        </a-button>      </div>    </div>    <div>显示内容为：{{ codeReviewSettingStore.gitlabUrl }}</div>    <!-- 提示词列表modal -->    <CodeReviewSetting v-model:modal-visible="codeSettingModalVisible" />  </div></template><style scoped lang="less">.codereview {  flex-grow: 1;  display: flex;  flex-direction: column;  overflow: hidden;  position: relative;  .codereview-header {    height: 55px;    border-bottom: 1px solid var(--color-border-1);    box-sizing: border-box;    padding: 15px;    display: flex;    align-items: center;    justify-content: space-between;    .codereview-header-title {      flex-grow: 1;      font-size: var(--font-size-lg);      font-weight: 500;    }    .codereview-header-setting {      flex-shrink: 0;    }  }  .codereview-body {    padding: 20px;    flex-grow: 1;    min-height: 0;    display: flex;    gap: 10px;    box-sizing: border-box;    padding: 15px;    .input-area {      flex: 1;      min-height: 0;      height: 100%;      display: flex;      flex-direction: column;      gap: 10px;    }    .preview-area {      flex: 1;      min-height: 0;      height: 100%;      display: flex;      flex-direction: column;      gap: 10px;    }  }}</style>
```

这样当我们运行效果后，就可以单击弹窗的时候，显示了设置界面：

同时我们在界面中设置的数据，在主界面中都可以展示出来。

接下来我们再看下主界面，主界面的第二部分是一个Tab页，这个组件的文档如下所示：

https://arco.design/vue/component/tabs

截图如下所示：

这里我们可以通过a-tabs组件来定义这个是一个标签页的配置，然后再定义a-tab-pane组件为具体的标签页中的内容，因此我们的标签页可以添加到我们的代码评审的设置界面中：

这个增加完成后，我们的界面效果如下所示：

接下来我们在添加2个组件，项目ID和MR ID的输入框组件，截图如下所示：

代码如下所示：

```
 <a-tabs>      <a-tab-pane key="1" title="基于MergeRequest的代码评审">        <div class="codereview-body">          <div class="input-area">            <div>显示内容为：{{ codeReviewSettingStore.gitlabUrl }}</div>            <a-space direction="vertical" :size="25" fill>              <a-space direction="horizontal" :size="10" fill>                <div>Gitlab 项目ID</div>                <a-input :style="{ width: '100%' }" size="small" v-model="data.projectId" placeholder="请输入项目ID" />                <div>Gitlab Merge Request ID</div>                <a-input size="small" v-model="data.mergeRequestId" placeholder="请输入Merge Request ID" />              </a-space>            </a-space>          </div>        </div>      </a-tab-pane>    </a-tabs>
```

这个里面需要两个属性，因此在脚本区域，我们增加上2个属性的配置：

这样我们的界面如下所示：

接下来我们在添加一个单选框组件，实现模型的选择，单选框的文档如下：

https://arco.design/vue/component/radio

通过文档可以知道，使用起来也是很简单的，增加如下代码：

```
<!--模型的选择区域-->            <a-space direction="vertical" :size="10">              <div>模型名称</div>              <a-radio-group v-model="data.modelName" type="button">                <a-radio value="glm-4-flash">智普AI-glm4-flash</a-radio>                <a-radio value="qwen2:7b">开源模型:阿里云Qwen2:7b</a-radio>                <a-radio value="gemma2:9b">开源模型:谷歌Gemma2:9b</a-radio>              </a-radio-group>            </a-space>
```

这样界面如下所示：

剩下的内容，本文这里先不分享了，下次再分享。感兴趣的小伙伴可以自己研究下把剩下的内容画到界面上。

自动化未来01秋季疾病预防秋季是呼吸道疾病的高发季节，如感冒、咳嗽、支气管炎等。这是因为气温的变化使得人体的呼吸道黏膜抵抗力下降，病毒和细菌更容易侵入。02胃肠道疾病秋季是胃肠道疾病的多发期。经过炎热的夏季，人们的脾胃功能相对较弱，再加上气温适宜，细菌繁殖加快，食物容易变质。03心脑血管疾病秋季早晚温差较大，血管收缩和舒张的变化较为明显，容易导致血压波动，从而增加心脑血管疾病的发病风险。07高峰论坛会

**03**

**总结**

本文分享了字节跳动的Arco-design组件库的几个场景的界面设计的元素：

文本输入框组件、文本域组件、按钮组件、单选框组件、间距组件、栅格组件、弹窗组件、消息组件、标签页组件。

感兴趣的小伙伴可以深入研究和学习，这样就可以自己设计出符合自己业务场景的界面。大部分的组件的使用官方文档都提供了说明。

最近也看到有人问如何学习AI，这里分享几个资料如下：

1、通往AGI之路的知识库飞书云文档:

https://waytoagi.feishu.cn/wiki/QPe5w5g7UisbEkkow8XcDmOpn8e

2、掘金的AI知识库的飞书云文档

https://agijuejin.feishu.cn/wiki/UvJPwhfkiitMzhkhEfycUnS9nAm?table=blk3RfZtR7Nh73tO

3、极客时间的AI知识库的飞书云文档

https://geek-agi.feishu.cn/wiki/B9rYwwg6xidZYJkbrlscxTQFnOc

4、LangGPT社区的飞书云文档（结构化提示词等等）：

https://langgptai.feishu.cn/wiki/RXdbwRyASiShtDky381ciwFEnpe

5、一站式AI产品经理飞书知识库:

https://v11enp9ok1h.feishu.cn/wiki/KiIvwdFOciiqqNkwKzTcmn88ndL

**喜欢本文的，可以关注、收藏、点赞、转发、分享到朋友圈哦。**

**本专题系列文章：**

**[AI代码评审CodeReview教程第1节：基本介绍](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648931978&idx=1&sn=cf7b09a38680d395592bd6a748f81a00&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第2节：私有化部署开源AI大模型](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648931993&idx=1&sn=f1997915ae630b017fb6ea35154ff447&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第3节：智普AI申请与代码评审体验](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932016&idx=1&sn=dbd6720492d0a711806d7844c889cc05&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第4节：AI大模型API接口的基本格式](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932032&idx=1&sn=aa89d0091d0a7440b2750c58a58c44f2&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第5节：企业级Gitlab环境安装与CI环境配置](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932088&idx=1&sn=2134b8dd843dbbabd7089ba428594c54&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第6节：基于Gitlab API与CI的AI代码评审实现](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932129&idx=1&sn=200e6c4c624c9689aebc4d99fc6eaa79&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第7节：基于MergeRequest变更代码AI评审实现](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932189&idx=1&sn=c3777b9abf4008a2bcb748415dd17b63&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第8节：项目的思考与Gitlab CI的内容分离](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932200&idx=1&sn=dffa3bdf0a6dd34259fc4f34fa590539&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第9节：Gitlab提交API的评审实现与飞书消息通知](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932226&idx=1&sn=bfb852923893ab3226736bba02ac13f4&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第10节：集成DeepSeekR1实现AI评审代码](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932245&idx=1&sn=9071658a5e33be3ed44ecf07df510c74&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第11节：手写TypeScript的AI大模型接口对接](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932272&idx=1&sn=ab48682008785dfef15d60c08328b429&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第12节：TypeScript的Gitlab仓库代码评审设计实现](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932336&idx=1&sn=4eea2f62dba222cf63669a0ef5eae5d8&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第13节：部署OneAPI统一调用大模型](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932640&idx=1&sn=2d8dc1bc116616ed9b24f72d7111c8cb&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第14节：评审消息飞书卡片对接设计实现](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932716&idx=1&sn=80798758191e1286123844489920c64f&scene=21#wechat_redirect)**

**[AI代码评审CodeReview教程第15节：Gitlab行级评论审查的对接设计](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648932743&idx=1&sn=6309f20201fa92fd875c1a95524568fe&scene=21#wechat_redirect)**

**[AI代码评审CodeReview第16节：评审提示词模板的设计实现与优化](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648933504&idx=1&sn=20a1b0e9b283080b297ce0be67f3e2fa&scene=21#wechat_redirect)**

**[AI代码评审CodeReview第17节：提升评审结果质量的16个想法](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648933510&idx=1&sn=2e86e289993fb1540d0cb9bb857a0b6a&scene=21#wechat_redirect)**

**[AI代码评审CodeReview第18节：代码评审的上下文的设计实现](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648933525&idx=1&sn=c51bee2288415353260118a3fe50b60f&scene=21#wechat_redirect)**

**[AI代码评审CodeReview第19节：开源AI Hub工具源码搭建介绍](https://mp.weixin.qq.com/s?__biz=MzIzNDAwODMxNA==&mid=2648933633&idx=1&sn=2ee9ea0b9047f7f5cfe24040f1d8d901&scene=21#wechat_redirect)**

- END -

喜欢的可以加入我的免费知识星球：觉醒的新世界程序员，或者我的付费知识星球：AI技术&MCP落地实践，随时与我沟通，交流技术与想法。

喜欢的也可以关注我的公众号：无处不在的技术，与我一起学习成长、共同进步，在技术的道路上越走越远。

**喜欢就点个****在看****呗 👇**