---
title: cursor进阶篇--rules总览
author: AIGC小牛独立开发日记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwNzg4MjgzNQ==&mid=2247484180&idx=1&sn=4f0b8b5230f018bafeff0f7a1aca5d44&chksm=c14e72df2cd85c8d82688ed816aab7fb75c2c495e2dba90e3f168fb187a5fb16491a680e080b&mpshare=1&scene=24&srcid=11139uN5QcJgx9Q1w9gHJQgd&sharer_shareinfo=3848cd42f5e40c1d7995b971af7ab621&sharer_shareinfo_first=3848cd42f5e40c1d7995b971af7ab621#rd
---

前两篇我们分享了关于cursor的注册、使用规范及基础功能等内容，这一篇主要补充rules的内容。

# 一、为什么要单独分享rules？

Rules之所以这么重要，是因为它能让我们更精准地控制项目代码往我们想要的方向去生成，这是和“AI幻觉”做对抗的有效手段之一。

# 二、rules都有哪些？

Cursor中有三种rules–User Rules、cursorrules、Project Rules。

## 2.1 User Rules

User Rules是一种系统级规则，对全部开发项目、全部对话生效，是一种通用规则。常见的比如可以在User Rules中指定输出语言、响应长度、以及输出样式等。具体路径：settings–Rules & Memories–User Rules，点击Add Rule，在空白处输入规则即可。

## 2.2 cursorrules

cursorrules是仅对某一个项目有效的规则，也就是你新开一个项目，那你就需要重新配置一次。不过官方说明中该项目根目录中的文件仍受支持，但终究会将被弃用。建议迁移到项目规则，以获得更多控制、灵活性和可见性。

以下是一个示例，其中包括项目初始化、需求理解、UI和样式设计、代码编写(技术栈选择)、问题解决、迭代优化等内容。但由于它每部分没有很细致，所以在代码生成的控制上就没有那么精细。所以，这份.cursorrules是可以继续优化的，就是对每部分进行更详细的展开，而且每部分只专注做好自己部分的工作。

```
```
# 角色   
你是一名精通**iOS应用开发**的高级工程师，拥有10年以上的**移动应用**开发经验，熟悉**Xcode、Swift、Objective-C、UIKit、SwiftUI、Core Data、Combine、CocoaPods**等开发工具和技术栈。你的任务是帮助用户设计和开发易用且易于维护的**iOS应用开发**。始终遵循最佳实践，并坚持干净代码和健壮架构的原则。   
# 目标   
你的目标是以用户容易理解的方式帮助他们完成**iOS应用开发**的设计和开发工作，确保应用功能完善、性能优异、用户体验良好。   
# 要求   
在理解用户需求、设计UI、编写代码、解决问题和项目迭代优化时，你应该始终遵循以下原则：   
## 项目初始化  
在项目开始时，首先仔细阅读项目目录下的README.md文件并理解其内容，包括项目的目标、功能架构、技术栈和开发计划，确保对项目的整体架构和实现方式有清晰的认识；   
- 如果还没有README.md文件，请主动创建一个，用于后续记录该应用的功能模块、页面结构、数据流、依赖库等信息。   
## 需求理解  
充分理解用户需求，站在用户角度思考，分析需求是否存在缺漏，并与用户讨论完善需求；   
- 选择最简单的解决方案来满足用户需求，避免过度设计。   
## UI和样式设计  
使用现代UI框架进行样式设计（例如**UIKit**或**swiftUI**，遵循苹果的**Human Interface Guidelines**设计规范；   
- 在不同平台上实现一致的设计和响应式模式   
## 代码编写  
技术选型：根据项目需求选择合适的技术栈（例如**swift**用于主要开发语言，**UIKit**用于构建传统UI，**swiftUI**用于构建声明式UI，**Core Data**用于数据持久化，**Combine**用于响应式编程）    
- **swift**：用于主要开发语言，遵循面向协议编程原则，确保代码结构清晰且易于扩展。     
- **UIKit**：用于构建传统UI，遵循苹果的NVC架构模式。确保UI与业务逻辑分离。     
- **swiftUI**：用于声明式UI设计，遵循 View 协议来创建自定义视图，确保代码简洁且易于维护。     
- **Core Data**：用于数据持久化，遵循数据模型与视图分离的原则。确保数据管理高效且安全。     
- **Combine**：两于响应式编程，遵循数据流管理最佳实践，确保数据流清晰且易于管理。   
- 代码结构：强调代码的清晰性、模块化、可维护性，遵循最佳实践（如DRY原则、最小权限原则、响应式设计等）   
- 代码安全性：在编写代码时，始终考虑安全性，避免引入漏洞，确保用户输入的安全处理   
- 性能优化：优化代码的性能，减少资源占用，提升加载速度，确保项目的高效运行  
- 测试与文档：编写单元测试，确保代码的健壮性，并提供清晰的中文注释和文档，方便后续阅读和维护    
## 问题解决  
全面阅读相关代码，理解**iOS应用开发**的工作原理  
- 根据用户的反馈分析问题的原因，提出解决问题的思路  
- 确保每次代码变更不会破坏现有功能，且尽可能保持最小的改动   
## 迭代优化  
与用户保持密切沟通，根据反馈调整功能和设计，确保应用符合用户需求  
- 在不确定需求时，主动询问用户以澄清需求或技术细节  
- 每次迭代都需要更新README.md文件，包括功能说明和优化建议   
## 方法论  
系统2思维：以分析严谨的方式解决问题。将需求分解为更小、可管理的部分，并在实施前仔细考虑每一步  
- 思维树：评估多种可能的解决方案及其后果。使用结构化的方法探索不同的路径，并选择最优的解决方案  
- 迭代改进：在最终确定代码之前，考虑改进、边缘情况和优化。通过潜在增强的迭代，确保最终解决方案是健壮的
```
```

## 2.3 Project Rules

Project Rules相较于cursorrules更强大且灵活，它可以对项目中不同位置的AI行为进行更细粒度的控制，可以使用路径模式来限定它们的作用域，手动调用，或者根据相关性包含它们。所有的Project Rules都存储与该文件夹`.cursor/rules`。使用项目规则可以：对有关代码库的特定于域的知识进行编码；自动执行特定于项目的工作流程或模板；标准化样式或体系结构决策。

具体设置路径：settings–Rules & Memories–Project Rules–Add Rule；或者使用快捷键Cmd+Shift+P调用命令面板，搜索New Cursor Rule进行创建。

# 三、如何使用这些rules？

## 3.1 User Rules

User Rules目前使用的比较少，一般我会添加以下内容：

1、Always respond in 中文

2、代码注释

-使用 JSDoc注释

3、代码生成

-严格遵循正确的代码格式

## 3.2 cursorrules

cursorrules有个基本的设计大纲，主要内容有：角色扮演，目标，项目初始化、需求理解、UI和样式设计、代码编写(技术栈选择)、问题解决、迭代优化等。如果制作一个rules的模板的话，可以像下面的文字格式：

```
```
# 角色   
你是一名精通** **的高级工程师，拥有10年以上的**  **开发经验，熟悉** **等开发工具和技术栈。你的任务是帮助用户设计和开发易用且易于维护的** **。始终遵循最佳实践，并坚持干净代码和健壮架构的原则。   
# 目标   
你的目标是以用户容易理解的方式帮助他们完成** **的设计和开发工作，确保应用功能完善、性能优异、用户体验良好。   
# 要求   
在理解用户需求、设计UI、编写代码、解决问题和项目迭代优化时，你应该始终遵循以下原则：   
## 项目初始化  
- 在项目开始时，首先仔细阅读项目目录下的README.md文件并理解其内容，包括项目的目标、功能架构、技术栈和开发计划，确保对项目的整体架构和实现方式有清晰的认识；   
- 如果还没有README.md文件，请主动创建一个，用于后续记录该应用的功能模块、页面结构、数据流、依赖库等信息。   
## 需求理解  
- 充分理解用户需求，站在用户角度思考，分析需求是否存在缺漏，并与用户讨论完善需求；   
- 选择最简单的解决方案来满足用户需求，避免过度设计。   
## UI和样式设计  
- 使用现代UI框架进行样式设计（例如** **设计规范；   
- 在不同平台上实现一致的设计和响应式模式   
## 代码编写  
- 技术选型：根据项目需求选择合适的技术栈（例如** **用于 ，需要针对技术栈分点展开，介绍某个技术栈用在什么地方，以及遵循什么最佳实践）    
- 代码结构：强调代码的清晰性、模块化、可维护性，遵循最佳实践（如DRY原则、最小权限原则、响应式设计等）   
- 代码安全性：在编写代码时，始终考虑安全性，避免引入漏洞，确保用户输入的安全处理   
- 性能优化：优化代码的性能，减少资源占用，提升加载速度，确保项目的高效运行  
- 测试与文档：编写单元测试，确保代码的健壮性，并提供清晰的中文注释和文档，方便后续阅读和维护    
## 问题解决  
- 全面阅读相关代码，理解** **的工作原理  
- 根据用户的反馈分析问题的原因，提出解决问题的思路  
- 确保每次代码变更不会破坏现有功能，且尽可能保持最小的改动   
## 迭代优化  
- 与用户保持密切沟通，根据反馈调整功能和设计，确保应用符合用户需求  
- 在不确定需求时，主动询问用户以澄清需求或技术细节  
- 每次迭代都需要更新README.md文件，包括功能说明和优化建议   
## 方法论  
- 系统2思维：以分析严谨的方式解决问题。将需求分解为更小、可管理的部分，并在实施前仔细考虑每一步  
- 思维树：评估多种可能的解决方案及其后果。使用结构化的方法探索不同的路径，并选择最优的解决方案  
- 迭代改进：在最终确定代码之前，考虑改进、边缘情况和优化。通过潜在增强的迭代，确保最终解决方案是健壮的
```
```

## 3.3 Project Rules

当在Project Rules中点击Add Rule后会弹出添加项目名称的窗口，这就是给Project Rules命个名，按照自己的喜好来就行，命名完成后会有一个新的界面，有四个部分的内容需要关注。

Always：这个规则会自动附加到每个聊天和命令上

Specific Files：文件模式匹配，能应用到特定的文件或文件夹上

Intelligently：根据描述判断某事具有相关性时

Manual：这个规则需要被手动@ 才能被包含在上下文

接下来就是添加描述，用来描述什么时候用这条Project Rules，不用写很复杂，简洁清晰即可，因为它是写给Agent看的。在Cursor的Agent模式下，Agent可以看到这条描述，并根据任务需求，决定是否阅读完整的Project Rules；描述也可以像下面这种稍微长一点的，只要能清晰表达自己的规则什么时候用就行。

```
```
# Chrome 扩展开发专家指南  
  
## 核心架构模式  
  
### Manifest V3 配置模板  
```json  
{  
  "manifest_version": 3,  
  "name": "__MSG_extensionName__",  
  "version": "1.0.0",  
  "description": "__MSG_extensionDescription__",  
  "default_locale": "en",  
  
  "permissions": [  
    "storage",  
    "activeTab",  
    "scripting"  
  ],  
  
  "host_permissions": [  
    "https://api.example.com/*"  
  ],  
  
  "background": {  
    "service_worker": "src/background/sw.js",  
    "type": "module"  
  },  
  
  "content_scripts": [  
    {  
      "matches": ["https://*.example.com/*"],  
      "js": ["src/content/main.js"],  
      "css": ["src/content/styles.css"],  
      "run_at": "document_idle"  
    }  
  ],  
  
  "action": {  
    "default_popup": "src/popup/popup.html",  
    "default_title": "__MSG_popupTitle__"  
  },  
  
  "web_accessible_resources": [  
    {  
      "resources": ["assets/*"],  
      "matches": ["https://*.example.com/*"]  
    }  
  ],  
  
  "content_security_policy": {  
    "extension_pages": "script-src 'self'; object-src 'self'"  
  }  
}  
```  
  
## 类型安全架构  
  
### TypeScript 类型定义  
```typescript  
// types/messaging.ts  
interface MessagePayload {  
  type: string;  
  payload?: unknown;  
  timestamp: number;  
  source: 'popup' | 'content' | 'background';  
}  
  
interface TabState {  
  tabId: number;  
  isActive: boolean;  
  lastUpdated: number;  
  data: Record<string, unknown>;  
}  
  
type StorageSchema = {  
  userPreferences: UserPreferences;  
  sessionData: SessionData;  
  cachedResponses: CachedResponse[];  
};  
  
// types/api.ts  
interface ApiResponse<T> {  
  data: T;  
  success: boolean;  
  error?: string;  
  timestamp: number;  
}  
```  
  
## 核心模块实现  
  
### 后台服务工作者 (Service Worker)  
```typescript  
// src/background/sw.ts  
import { handleMessage, sendMessageToTabs } from '../utils/messaging';  
import { initializeStorage, getStorageItem, setStorageItem } from '../utils/storage';  
import { createAlarm, clearAlarm } from '../utils/alarms';  
  
// 初始化扩展  
const initializeExtension = async (): Promise<void> => {  
  try {  
    await initializeStorage();  
    await setupMessageHandlers();  
    await scheduleBackgroundTasks();  
  
    console.log('Extension initialized successfully');  
  } catch (error) {  
    console.error('Failed to initialize extension:', error);  
  }  
};  
  
// 消息处理  
const setupMessageHandlers = (): void => {  
  chrome.runtime.onMessage.addListener(  
    (message: MessagePayload, sender, sendResponse) => {  
      handleMessage(message, sender)  
        .then(sendResponse)  
        .catch((error) => {  
          console.error('Message handling error:', error);  
          sendResponse({ success: false, error: error.message });  
        });  
  
      return true; // 保持消息通道开放  
    }  
  );  
};  
  
// 定时任务管理  
const scheduleBackgroundTasks = async (): Promise<void> => {  
  await createAlarm('dataSync', { periodInMinutes: 60 });  
  await createAlarm('cleanup', { periodInMinutes: 1440 }); // 每天  
};  
  
// 定时任务处理  
chrome.alarms.onAlarm.addListener(async (alarm) => {  
  switch (alarm.name) {  
    case 'dataSync':  
      await performDataSync();  
      break;  
    case 'cleanup':  
      await performCleanup();  
      break;  
  }  
});  
  
// 标签页状态管理  
const manageTabState = (): void => {  
  chrome.tabs.onActivated.addListener(async (activeInfo) => {  
    const tabState = await getStorageItem<TabState>(`tab-${activeInfo.tabId}`);  
    await updateTabState(activeInfo.tabId, { ...tabState, isActive: true });  
  });  
  
  chrome.tabs.onRemoved.addListener(async (tabId) => {  
    await clearTabState(tabId);  
  });  
};  
```  
  
### 安全的通信工具  
```typescript  
// src/utils/messaging.ts  
interface MessageHandler {  
  (message: MessagePayload, sender: chrome.runtime.MessageSender): Promise<unknown>;  
}  
  
const messageHandlers = new Map<string, MessageHandler>();  
  
export const registerMessageHandler = (type: string, handler: MessageHandler): void => {  
  messageHandlers.set(type, handler);  
};  
  
export const handleMessage = async (  
  message: MessagePayload,   
  sender: chrome.runtime.MessageSender  
): Promise<unknown> => {  
  const handler = messageHandlers.get(message.type);  
  
  if (!handler) {  
    throw new Error(`No handler registered for message type: ${message.type}`);  
  }  
  
  // 验证消息来源  
  if (!isValidSender(sender)) {  
    throw new Error('Invalid message sender');  
  }  
  
  return await handler(message, sender);  
};  
  
export const sendMessageToTab = async (  
  tabId: number,   
  message: MessagePayload  
): Promise<unknown> => {  
  try {  
    return await chrome.tabs.sendMessage(tabId, {  
      ...message,  
      timestamp: Date.now(),  
      source: 'background'  
    });  
  } catch (error) {  
    console.error(`Failed to send message to tab ${tabId}:`, error);  
    throw error;  
  }  
};  
  
const isValidSender = (sender: chrome.runtime.MessageSender): boolean => {  
  // 实现发送者验证逻辑  
  return Boolean(sender.tab || sender.id === chrome.runtime.id);  
};  
```  
  
### 安全存储管理  
```typescript  
// src/utils/storage.ts  
export const initializeStorage = async (): Promise<void> => {  
  const defaults = {  
    userPreferences: {  
      theme: 'light',  
      notifications: true,  
      language: 'en'  
    },  
    sessionData: {},  
    cachedResponses: []  
  };  
  
  await chrome.storage.local.set(defaults);  
};  
  
export const getStorageItem = async <T>(key: string): Promise<T | null> => {  
  try {  
    const result = await chrome.storage.local.get(key);  
    return result[key] || null;  
  } catch (error) {  
    console.error(`Failed to get storage item ${key}:`, error);  
    return null;  
  }  
};  
  
export const setStorageItem = async <T>(key: string, value: T): Promise<void> => {  
  try {  
    await chrome.storage.local.set({ [key]: value });  
  } catch (error) {  
    console.error(`Failed to set storage item ${key}:`, error);  
    throw error;  
  }  
};  
  
export const secureStorage = {  
  encrypt: (data: string): string => {  
    // 实现加密逻辑 (使用 Web Crypto API)  
    return btoa(unescape(encodeURIComponent(data)));  
  },  
  
  decrypt: (encryptedData: string): string => {  
    // 实现解密逻辑  
    return decodeURIComponent(escape(atob(encryptedData)));  
  }  
};  
```  
  
## 内容脚本最佳实践  
  
```typescript  
// src/content/main.ts  
import { waitForElement, observeDOMChanges } from '../utils/dom';  
import { sendMessageToBackground } from '../utils/messaging';  
  
const initializeContentScript = async (): Promise<void> => {  
  try {  
    await waitForPageLoad();  
    await injectUIComponents();  
    await setupEventListeners();  
    await observePageChanges();  
  
    console.log('Content script initialized');  
  } catch (error) {  
    console.error('Content script initialization failed:', error);  
  }  
};  
  
const waitForPageLoad = (): Promise<void> => {  
  return new Promise((resolve) => {  
    if (document.readyState === 'loading') {  
      document.addEventListener('DOMContentLoaded', () => resolve());  
    } else {  
      resolve();  
    }  
  });  
};  
  
const injectUIComponents = async (): Promise<void> => {  
  const targetElement = await waitForElement('.main-content');  
  
  if (!targetElement) {  
    throw new Error('Target element not found');  
  }  
  
  const widget = createExtensionWidget();  
  targetElement.prepend(widget);  
};  
  
const createExtensionWidget = (): HTMLElement => {  
  const widget = document.createElement('div');  
  widget.className = 'extension-widget';  
  widget.innerHTML = `  
    <div class="widget-header">  
      <h3>Extension</h3>  
      <button class="close-btn" aria-label="Close widget">×</button>  
    </div>  
    <div class="widget-content">  
      <p>Extension content loaded</p>  
    </div>  
  `;  
  
  return widget;  
};  
  
const setupEventListeners = (): void => {  
  document.addEventListener('click', handleDocumentClick);  
  window.addEventListener('beforeunload', handlePageUnload);  
};  
  
const observePageChanges = (): void => {  
  observeDOMChanges(document.body, (mutations) => {  
    mutations.forEach((mutation) => {  
      if (mutation.type === 'childList') {  
        handleDOMUpdate(mutation);  
      }  
    });  
  });  
};  
```  
  
## 弹出窗口 UI 组件  
  
```typescript  
// src/popup/popup.ts  
import { getStorageItem, setStorageItem } from '../utils/storage';  
import { sendMessageToBackground } from '../utils/messaging';  
  
interface PopupState {  
  isLoading: boolean;  
  hasError: boolean;  
  data: unknown;  
}  
  
const initializePopup = async (): Promise<void> => {  
  try {  
    await loadUserPreferences();  
    await setupPopupEventListeners();  
    await loadInitialData();  
  
    updateUIState({ isLoading: false });  
  } catch (error) {  
    handlePopupError(error);  
  }  
};  
  
const loadUserPreferences = async (): Promise<void> => {  
  const preferences = await getStorageItem('userPreferences');  
  
  if (preferences) {  
    applyUserPreferences(preferences);  
  }  
};  
  
const setupPopupEventListeners = (): void => {  
  const saveButton = document.getElementById('save-btn');  
  const refreshButton = document.getElementById('refresh-btn');  
  const settingsButton = document.getElementById('settings-btn');  
  
  saveButton?.addEventListener('click', handleSave);  
  refreshButton?.addEventListener('click', handleRefresh);  
  settingsButton?.addEventListener('click', handleSettings);  
};  
  
const handleSave = async (event: Event): Promise<void> => {  
  event.preventDefault();  
  
  const formData = collectFormData();  
  await setStorageItem('userPreferences', formData);  
  
  showNotification('Settings saved successfully', 'success');  
};  
  
const updateUIState = (state: Partial<PopupState>): void => {  
  const loadingElement = document.getElementById('loading');  
  const errorElement = document.getElementById('error');  
  const contentElement = document.getElementById('content');  
  
  if (state.isLoading && loadingElement) {  
    loadingElement.style.display = 'block';  
    contentElement.style.display = 'none';  
  } else if (state.hasError && errorElement) {  
    errorElement.style.display = 'block';  
    contentElement.style.display = 'none';  
  } else {  
    loadingElement.style.display = 'none';  
    errorElement.style.display = 'none';  
    contentElement.style.display = 'block';  
  }  
};  
```  
  
## 构建和开发工具配置  
  
### Webpack 配置示例  
```javascript  
// webpack.config.js  
const path = require('path');  
const CopyPlugin = require('copy-webpack-plugin');  
  
module.exports = {  
  mode: process.env.NODE_ENV || 'development',  
  entry: {  
    background: './src/background/sw.ts',  
    content: './src/content/main.ts',  
    popup: './src/popup/popup.ts'  
  },  
  output: {  
    path: path.resolve(__dirname, 'dist'),  
    filename: '[name]/[name].js',  
    clean: true  
  },  
  module: {  
    rules: [  
      {  
        test: /\.ts$/,  
        use: 'ts-loader',  
        exclude: /node_modules/  
      },  
      {  
        test: /\.css$/,  
        use: ['style-loader', 'css-loader']  
      }  
    ]  
  },  
  resolve: {  
    extensions: ['.ts', '.js']  
  },  
  plugins: [  
    new CopyPlugin({  
      patterns: [  
        { from: 'src/manifest.json', to: 'manifest.json' },  
        { from: 'src/popup/popup.html', to: 'popup/popup.html' },  
        { from: 'assets', to: 'assets' },  
        { from: '_locales', to: '_locales' }  
      ]  
    })  
  ],  
  devtool: 'cheap-source-map'  
};  
```  
  
## 测试和调试工具  
  
```typescript  
// src/utils/debug.ts  
interface DebugConfig {  
  enabled: boolean;  
  level: 'error' | 'warn' | 'info' | 'debug';  
  modules: string[];  
}  
  
class ExtensionDebugger {  
  private config: DebugConfig;  
  
  constructor() {  
    this.config = this.loadConfig();  
  }  
  
  log(module: string, message: string, data?: unknown): void {  
    if (this.shouldLog(module)) {  
      console.log(`[${module}] ${message}`, data || '');  
    }  
  }  
  
  error(module: string, message: string, error?: Error): void {  
    console.error(`[${module}] ERROR: ${message}`, error || '');  
  
    // 发送错误报告到后台  
    this.reportError(module, message, error);  
  }  
  
  private shouldLog(module: string): boolean {  
    return this.config.enabled &&   
           this.config.modules.includes(module) &&  
           this.isDevEnvironment();  
  }  
  
  private isDevEnvironment(): boolean {  
    return process.env.NODE_ENV === 'development';  
  }  
  
  private reportError(module: string, message: string, error?: Error): void {  
    chrome.runtime.sendMessage({  
      type: 'ERROR_REPORT',  
      payload: {  
        module,  
        message,  
        error: error?.stack,  
        timestamp: Date.now(),  
        userAgent: navigator.userAgent  
      }  
    }).catch(() => {  
      // 静默处理发送失败  
    });  
  }  
}  
  
export const debug = new ExtensionDebugger();  
```  
  
## 安全最佳实践  
  
```typescript  
// src/utils/security.ts  
export const sanitizeHTML = (input: string): string => {  
  const div = document.createElement('div');  
  div.textContent = input;  
  return div.innerHTML;  
};  
  
export const validateURL = (url: string): boolean => {  
  try {  
    const parsed = new URL(url);  
    return ['http:', 'https:'].includes(parsed.protocol);  
  } catch {  
    return false;  
  }  
};  
  
export const createCSPNonce = (): string => {  
  const array = new Uint8Array(16);  
  crypto.getRandomValues(array);  
  return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');  
};  
  
export const safeEval = (code: string, context: object = {}): unknown => {  
  // 使用 Function 构造函数而不是 eval  
  try {  
    return Function(`"use strict"; return (${code})`).call(context);  
  } catch (error) {  
    console.error('Safe eval failed:', error);  
    return null;  
  }  
};  
```  
  
## 国际化支持  
  
### 本地化文件结构  
```  
_locales/  
  en/  
    messages.json  
  es/  
    messages.json  
  fr/  
    messages.json  
```  
  
```json  
// _locales/en/messages.json  
{  
  "extensionName": {  
    "message": "My Extension",  
    "description": "Extension name"  
  },  
  "popupTitle": {  
    "message": "Extension Controls",  
    "description": "Popup window title"  
  }  
}  
```  
  
```typescript  
// src/utils/i18n.ts  
export const getMessage = (key: string, substitutions?: string | string[]): string => {  
  return chrome.i18n.getMessage(key, substitutions) || key;  
};  
  
export const getUILanguage = (): string => {  
  return chrome.i18n.getUILanguage();  
};  
  
export const detectRTLLanguage = (): boolean => {  
  const rtlLanguages = ['ar', 'he', 'fa', 'ur'];  
  return rtlLanguages.includes(getUILanguage());  
};  
```  
  
这些准则和代码示例体现了现代 Chrome 扩展开发的最佳实践，包括：  
  
- **Manifest V3 合规性**：服务工作者、最小权限原则  
- **类型安全**：完整的 TypeScript 支持  
- **安全架构**：CSP、输入验证、安全通信  
- **性能优化**：有效的资源管理和缓存策略  
- **用户体验**：Material Design、无障碍支持、国际化  
- **可维护性**：模块化架构、清晰的错误处理、完整的测试支持
```
```

使用Project Rules的方法也很简单，因为Project Rules就是一个具体文件，支持将多个Proiect Rules串联起来使用。在对话中用@rules，选择你所写Project Rules即可。但是Project Rules种类很多，后期可以单独用一篇文章来分享。

# 四、注意事项

给大家分享以下我主要关注的Project Rules网站https://cursor.directory/， 每一个卡片就是一个rules，可以直接拷贝使用。