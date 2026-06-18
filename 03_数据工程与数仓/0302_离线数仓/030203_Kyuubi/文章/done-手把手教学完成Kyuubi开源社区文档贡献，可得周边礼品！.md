---
title: 手把手教学完成Kyuubi开源社区文档贡献，可得周边礼品！
author: Apache Kyuubi
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247493584&idx=1&sn=b09b1463681224e6725a3c921b9cfcee&chksm=ce29027bf95e8b6dd741ea53e89c76e84904c92ae290afefec53171ae12ff95c657508272f6e&mpshare=1&scene=24&srcid=1027Pq2dgBQyyqyHGAMjdfOY&sharer_sharetime=1666836636170&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030203_Kyuubi/030203_知识地图|知识地图]]


“

Apache Kyuubi 开源社区 **文档贡献活动及保姆级教程**已经全面上线，成功完成贡献的同学即可**领取周边礼品！**

”

✦

**新手小白的疑问**

✦

大家好，这里是 Kyuubi 小助手，最近有同学过来问小助手：

“ 我是新手小白，不懂得敲代码，还能加入 Apache Kyuubi 开源社区吗？”

“当然可以！”

Apache Kyuubi 开源社区欢迎所有对 Kyuubi 感兴趣的小伙伴，新手小白不会敲代码也能够在社区里获得满满的参与感！

这是因为 Apache Kyuubi 开源社区中不仅需要代码，还需要：

1.完善的安装部署文档、技术博客、官网文档指引以及视频教程；

2.组织社区活动、推广开源项目、撰写宣传文案、设计精美的社区图案；

3.参与用户交流、在社区内进行QA答疑、社区投票；

这些部分都需要社区的小伙伴们一起努力，同学们可以根据自己的爱好和特长，选择自己感兴趣的部分。

为了满足萌新们尽快参与社区建设的愿望，社区上线了 Kyuubi 文档贡献活动和配套的保姆级文档贡献教程，保证上手简单，包教包会！

顺利完成社区贡献的同学可以获得周边礼品！

‍

✦  **文档贡献活动**  ✦

本次社区文档贡献活动面向 Apache Kyuubi 全体社区成员，社区的同学快来跟着小助手了解活动的具体事项吧~

首先，社区中关于文档贡献的任务，可以在Kyuubi项目的GitHub Issue列表

https://github.com/apache/incubator-kyuubi/issues 

通过 kind:documentation 标签进行搜索

大家可以看到，在以下两个 Umbrella 下有很多撰写文档的子任务，同学们可以根据自己的兴趣自行选择相应的任务。

https://github.com/apache/incubator-kyuubi/issues/3059

https://github.com/apache/incubator-kyuubi/issues/3100

以 “ [Umbrella] Client side Documentation improvement ” 为例，

https://github.com/apache/incubator-kyuubi/issues/3039，

目前发布的任务有：

* JDBC Drivers

+ Kyuubi Hive JDBC Driver
+ Hive JDBC Driver
+ MySQL Connectors

* Command Line Interface(CLI)s

+ Kyuubi Beeline
+ Hive Beeline

* Business Intelligence Tools and SQL IDEs

+ Apache Superset
+ Cloudera Hue
+ DataGrip
+ DBeaver
+ PowerBI
+ Tableau
+ ...

* ODBC Drivers
* Thrift APIs
* RESTful APIs and Clients

+ batch api
+ batch sdk
+ batch command line(kyuubi-ctl batch part)
+ SPNEGO
+ BASIC
+ Authentication
+ Admin
+ SQL
+ Batch

* Web UI

+ Spark UI Extension
+ Kyuubi Server UI
+ Kyuubi Engine UI

* Python DB-APIs

+ PyHive

* Client Commons

+ Using Different Kyuubi Engines
+ Sharing and Isolation for Kyuubi Engines
+ Setting Time to Live for Kyuubi Engines
+ Enabling Kyuubi Engine Pool
+ Running Scala Snippets
+ Plan Only Execution Mode
+ Client Configuration Guide
+ Logging
+ Access Kerberized Kyuubi with Beeline & BI Tools
+ Advanced Features

值得注意的是，无论选择哪个任务，在编写文档的过程中，同学们都需要遵循以下原则：

General ones

http://www.cypressmedia.net/articles/article/26/six\_principles\_of\_technical\_writing

·Using markdown or reStructuredText(recommended)

·More about how users or developers shall use an API, feature etc, less about what it is.

想要参与社区文档贡献的同学可以 扫描下方二维码 进入文档贡献活动微信群

**人航天日**

✦

**文档贡献教程**

✦

接下来，小助手准备了文档贡献的详细教程，手把手教大家如何做社区文档贡献者，参与到 Kyuubi 开源社区的建设中来。

在本次的教程中，我们尝试：**向 Kyuubi 项目增加 Kyuubi Spark TPC-DS 连接器文档**。

**01**

**Fork仓库**

**步骤1：打开官网对应的GitHub链接**：https://github.com/apache/incubator-kyuubi

**步骤2：点击Fork按钮，把官网的项目Fork到自己的仓库中**      

**02**

**将项目克隆到本地，并设置追踪远程仓库**

**步骤1：克隆上游项目 apache/incubator-kyuubi 到本地，并将此远程仓库命名为 apache**

cd ~/Projects

git clone git@github.com:apache/incubator-kyuubi.git --origin apache

此时项目将位于：~/Projects/incubator-kyuubi

**步骤2：打开自己建立的 fork 仓库，如**

**https://github.com/pan3793/kyuubi**

进入到已克隆的项目目录中，如 ~/Projects/incubator-kyuubi，添加自己的远程仓库并命名为 origin

git@github.com:pan3793/kyuubi.git

此时关联的远程仓库共有2个，分别是 apache(上游)和 origin(个人)仓库，可以通过 git remote -v 确认。

apache git@github.com:apache/incubator-kyuubi.git (fetch)

apache git@github.com:apache/incubator-kyuubi.git (push)

origin git@github.com:pan3793/kyuubi.git (fetch)

origin git@github.com:pan3793/kyuubi.git (push)

**03**

**创建临时分支，修改并提交**

**步骤1：同步上游最新master分支**

Git 是一个适用于多人协作的版本控制工具，在 Kyuubi 的项目分支管理中，新更改都会合入 master 分支，同时master 是一个只会前进的公共分支。为避免冲突，在做修改前，请同步上游最新的 master 分支。

第二步操作完成后，当前本地默认即为 master 分支。同时，本地 master 分支关联到远程上游 apache/master 分支，在本地 master 分支上执行 git pull，将会同步远程分支最新代码到本地。

**步骤2：****从****master 分支****检出****临时特性分支**

执行如下命令从当前 master 分支检出并切换到 tpcds-doc 临时分支

git checkout -b tpcds-doc

**04**

**修改和验证**

**步骤1：用编辑器打开项目，编辑文档（本例中使用 IDEA 作为编辑器）**

本例中要添加的 Kyuubi Spark TPC-DS 连接器文档位于 docs/connector/spark/tpcds.rst，打开之后，使用reStructuredText 格式编辑文档。Markdown 格式也支持！

**步骤2：构建和预览文档**

当对文档做出修改后，为了验证语法正确，并确定渲染正常，需要在进行构建和预览。

首先请参考 Kyuubi 文档安装和配置文档构建环境：

https://kyuubi.readthedocs.io/en/latest/develop\_tools/build\_document.html

进入 docs 目录执行 make html，如果构建成功，将会输出

build succeeded, 62 warnings.

The HTML pages are in \_build/html.

进入该目录，并找到修改文档页面，使用浏览器打开预览。

**05**

**提交修改**

**步骤1：当确认修改无误后，提交修改**

git add .

git commit -m '[DOCS] Add documents for Kyuubi Spark TPC-DS connector'

**步骤2：将提交推送到远程个人仓库**

若直接执行 git push，会被提示远程仓库不存在与本地对应分支

根据提示信息执行 git push --set-upstream origin tpcds-doc 即可推送成功，并将本地的 tpcds-doc 与个人远程仓库中的 origin/tpcds-doc 分支相关联，后续直接使用 git push 即可推送成功。

**06**

**创建Pull Request**

**步骤1(方式1)：**Push完毕后，根据上图 push 成功后的提示信息，直接打开链接创建 Pull Request。

https://github.com/pan3793/kyuubi/pull/new/tpcds-doc

**步骤1(方式2)：**从GitHub页面创建Pull Request。

选择将个人仓库的 tpcds-doc 分支，合并到上游仓库的 master 分支。

**步骤2：按照模板填写 PR 描述**

一定要按照模板将自己的【修改内容】、【修改原因】、【验证过程】描述清晰，这将加速项目维护者对 PR 的审查速度，使得 PR 有较大的概率被合入！

**步骤3：创建 PR**

当内容确认无误后，点击 Create Pull Request 即可创建成功。

**07**

**等待Review和Merge**

（1）提交 PR 成功后，需要等待项目维护者进行内容 Review。如果提交的内容有问题，官方开发同学会在 PR 上进行留言，根据反馈进行修改文档内容。改动过程**重复第四步和第五步，在当前分支上做改动、提交，并推送到远程**。

（2）如无修改，Reviewer 会 Approve 并 Merge 该 PR。Merge 成功后，会收到邮件通知。

（3）PR 被 Merge 后，状态也会由 Open 转换成 Closed。

**08**

**查看官方文档，验证改动生效**

当文档内容变更后，Kyuubi 项目的持续构建工作流会自动将内容同步官方文档站点。因此 PR merge 后稍等片刻，即可在官方文档中看到自己的更改内容。

https://kyuubi.readthedocs.io/en/latest/index.html

✦

**成为Contributor**

✦

dangdang！成功完成社区贡献的同学能够成为 Apache Kyuubi 开源社区的贡献者，你的头像会出现在 GitHub 的 Contributor 当中，成为更多萌新的指路人。

✦ 成为Apache Kyuubi Committer ✦

除此之外，在社区积极贡献的同学还有机会成为社区的 **Committer**。

事实上，任何在 CoPDoC 中做出充分贡献的成员都可以成为 committer-ship 的候选人，并最终投票成为 Kyuubi Committer。**CoPDoC 是指**：

* **Community** - 你可以通过我们的邮件列表、问题跟踪器、讨论页面加入我们，与社区成员互动，分享愿景和知识
* **Project** - 需要清晰的愿景和共识
* **Documentation** - 没有文档，这些东西只会留在作者的脑海中
* **Code**- 没有代码，讨论就无从谈起

想要成为 Apache Kyuubi Committer 的同学可以参考：https://kyuubi.apache.org/become\_committer.html

以上就是本次文档贡献活动的主要内容，社区的同学们快快行动起来，一起为 Apache Kyuubi 开源社区建设做贡献！

********END********

**Apache Kyuubi****推特账号****现已开通**

**推特搜索****Apache Kyuubi****或 浏览器****打开下方链接****即可关注~**

**https://twitter.com/KyuubiApache**

**还可以加入****Apache Kyuubi Slack**

**https://join.slack.com/t/apachekyuubi/shared\_invite/zt-1e1qw68g4-yE5HJsVVDin~ABtZISyuxg**

**和海外开发者交流互动哦~**

**最后**

**Kyuubi 在这里提醒大家**

**文明上网 科学上网**

**往期精彩:**

[有了它！爱奇艺加速 Hive SQL 迁移 Spark](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247491661&idx=1&sn=6bb13f9fc7d3a034e122903f8fb77499&chksm=ce2905e6f95e8cf0b5e0e4303667d0f72efc15d2d3c2a999e47debd280241cdf1a48cd5e98eb&scene=21#wechat_redirect)

[如何优化 Spark 小文件，Kyuubi 一步搞定！](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247491506&idx=1&sn=ec070e4d00fd3cc91d95b5e579a05d99&chksm=ce2afa19f95d730fbef721003505bcf06e03eeece8e6a4a5421b7acaf27de0279c0e94843764&scene=21#wechat_redirect)

[Kyuubi on Kubernetes 模式从 0 到 1 部署](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247488092&idx=1&sn=24b40dc0457c09b83518b1d2ed08ab22&chksm=ce2af7f7f95d7ee1f55a47808ad87905383913e76ef2fa886eb149cbce00b05c6d8d4fb14b5a&scene=21#wechat_redirect)

[Apache Kyuubi 1.5.0-incubating发布: 40+贡献者, 300+提交, 支持 Flink/Trino](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247488088&idx=1&sn=6f6104175b70d5edcd7c98784be5b27f&chksm=ce2af7f3f95d7ee51b521a631ce6b3d681e010d2267f97e9960ec2eca4ac9a1a8e488bacf984&scene=21#wechat_redirect)

[Become A Committer of Apache Kyuubi](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247487924&idx=1&sn=40e868e4deabd789a03ef4d0196f7ce9&chksm=ce2af41ff95d7d096b086eb4949a2fe543356b727da817355e58ca48347c74be12fa03cfc225&scene=21#wechat_redirect)

看到这里记得多多点赞、评论、收藏

还可以把 Kyuubi 分享给更多朋友~

点击"阅读原文"即可跳转到站点~