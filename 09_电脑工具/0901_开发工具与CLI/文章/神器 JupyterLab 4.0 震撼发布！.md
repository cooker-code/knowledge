---
title: 神器 JupyterLab 4.0 震撼发布！
author: 涛哥聊Python
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA5MTkxNTMzNg==&mid=2650297357&idx=1&sn=74cc19c884f9ef8398ad5418463998be&chksm=8879e12abf0e683c9c637fbafef8e1bf99cfecbc849a3252adf3d3ba3ec18c6223ad7eead523&mpshare=1&scene=24&srcid=0613pK96DLVc8BlhTUBARc9M&sharer_sharetime=1686659395438&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

JupyterLab 是 Jupyter Notebook 的下一代版本，它提供了更强大的功能和更灵活的用户界面，6月6日，官方发布了**JupyterLab 4.0**的说明，并且说该版本是下一个主要的版本。

JupyterLab的主要改进是:

1. **用户界面**：Jupyter Notebook 使用单个文档界面，以逐个标签的方式显示打开的笔记本。每个标签对应一个笔记本。而 JupyterLab 则提供了一个更灵活的多文档界面，可以在同一个窗口中同时打开多个笔记本、终端、文本文件和其他插件。
2. **布局**：Jupyter Notebook 的布局是固定的，用户只能在每个单元格之间垂直滚动。JupyterLab 允许用户在一个窗口中自由地拖放和重新排列笔记本、代码编辑器、输出窗口和其他面板，使布局更加自定义化和灵活。
3. **文件浏览器**：JupyterLab 内置了一个侧边栏文件浏览器，方便用户管理文件和文件夹。这个功能在 Jupyter Notebook 中是通过在命令行中进行操作实现的。
4. **扩展性**：JupyterLab 的架构更加模块化和可扩展，使用户可以添加自定义插件和扩展功能。这意味着开发人员可以根据自己的需求添加新的功能和工具，使 JupyterLab 更适应特定的工作流程。
5. **终端**：JupyterLab 具有内置的终端功能，可以直接在界面中运行命令行命令，而无需打开额外的终端窗口。这对于需要在交互式计算环境中执行命令行任务的用户来说非常方便。

对于，JupyterLab 4.0来说，最大的更新就是更快了，这要通过CSS规则优化、CodeMirror 6、MathJax 3和等改进。在处理大型笔记本时，JupyterLab 4比JupyterLab 3要高效得多。

为了优化性能，将实时协作(RTC)移到了一个单独的包jupyter\_collaboration中，该包的1.0.0版本现在已经可以使用。这样如果我们单机使用的话就不需要再装这些不需要的内容了。如果想在JupyterLab 4中使用RTC，则需要安装jupyter\_collaboration包。

在JupyterLab 4中，还包含了一个新的扩展管理器，这样就可以直接从PyPI安装，不需要再本地的编译了，这样对于我们安装也方便很多。

以上就是JupyterLab 4.0的简单总结，完整官方发布在这里：https://blog.jupyter.org/jupyterlab-4-0-is-here-388d05e03442

总的来说，JupyterLab 提供了更丰富的功能和更灵活的用户界面，使用户能够更好地组织和管理笔记本和其他相关工具。但是对于现在的AI辅助编程，包括ChatGPT和Copilot，JupyterLab 已经落后太多了。如果只需要简单地运行和共享笔记本，使用VS code+Copilot运行Notebook 仍然是一个很好的选择。

---END---

往期链接：[5月琐碎但值得的事情](http://mp.weixin.qq.com/s?__biz=MzA5MTkxNTMzNg==&mid=2650297115&idx=1&sn=2da17fd75c60c258f6c1fee109d8bc8a&chksm=8879e63cbf0e6f2a13332c25ba86e0150bd93835b026fe47e0aff76db0f24dd6555da5cee7ec&scene=21#wechat_redirect)

我的爬虫课了解： [成就感爆棚](http://mp.weixin.qq.com/s?__biz=MzA5MTkxNTMzNg==&mid=2650296584&idx=1&sn=e85c680797b667a3dd048e991089620e&chksm=8879e42fbf0e6d3910ed080fb090a7e4c0bed9b8900fa3f45f5173d18e328d38dbd1263cd0f4&scene=21#wechat_redirect)

> 我叫彭涛，做Python开发&技术博主，在成都有个小团队，
>
> 朋友圈日常分享一些个人思考创业干货，
>
> 欢迎加我vx，围观朋友圈，做个点赞之交。