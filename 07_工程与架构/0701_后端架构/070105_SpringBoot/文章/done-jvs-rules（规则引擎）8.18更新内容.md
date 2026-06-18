> 已吸收至：[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_核心知识点/SpringBoot来源校准与扩展边界|SpringBoot来源校准与扩展边界]]、[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_知识地图|070105_SpringBoot知识地图]]

---
title: jvs-rules（规则引擎）8.18更新内容
author: 软开企服
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg4NzY5Nzc1MA==&mid=2247492041&idx=1&sn=20d170a3b1172b715fde5e799e678bd0&chksm=cf84d9f0f8f350e6e1de45383335d1ed5eed23624eb325ffbc59063d85f06e63c7018c025529&mpshare=1&scene=24&srcid=0823Y3qD3KA10D9qCnP7jGFA&sharer_sharetime=1692757699845&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**jvs-rules更新内容**

1.复合变量新增数据补充节点，实现请求回来的数据再以入参方式请求其他数据进行数据补充（例如通过参数A，请求回数据B，再以数据B为入参，请求回数据C）

2.规则流结束节点支持新增、新建、引入变量；结束节点可直接选择或新增某个变量进行结果输出

3.评分卡新增条件一键添加和导入功能，实现快速导入复杂评分规则

4.复合变量,数据筛选增加条件导入功能，实现快速导入多个筛选规则

5.复合变量,数据筛选增加条件快捷查看方式，实现查看多个筛选规则方式

**优化部分**

1.优化规则流保存效率

2.复合变量，横向连接节点增加示例说明

3.修复数据源操作按钮权限控制

4.修复数据筛选无文本框情况

规则引擎在线demo：http://rules.bctools.cn/

开源地址：https://gitee.com/software-minister/jvs-rules

微信：ruanjbz      rkqf11

24h联系电话：13983558167