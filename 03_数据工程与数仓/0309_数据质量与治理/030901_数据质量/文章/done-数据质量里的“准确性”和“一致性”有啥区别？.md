> 已吸收至：[[03_数据工程与数仓/0309_数据质量与治理/030901_数据质量/030901_核心知识点/数据质量准确性与一致性边界|数据质量准确性与一致性边界]]
---
title: 数据质量里的“准确性”和“一致性”有啥区别？
author: 大数据架构师
date:
url: http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247501150&idx=1&sn=54d35601e4f14cc222c7f067e16bcd50&chksm=c325144bf4529d5dea16c54db968d7c72099ce33e0309f31a54f3bdfe401bce8b2c1221bf368&mpshare=1&scene=24&srcid=0422zQ30tMd7B7LoBEqpgyXs&sharer_sharetime=1650600624266&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

彭友们好，我是老彭啊。前两天群里讨论了一个很有意思的话题：

1、提问：

“为什么数据质量维度同时包括准确性和一致性，不应该是准确性包括一致性吗，总感觉在中文字面上这两个词有重叠的地方”

2、第一眼看见这个问题，感觉这是一道语文题，有点咬文嚼字的感觉，就是问的“准确性”和“一致性”的各是什么意思嘛？

3、先看看百度的解释：

4、再看看DAMA-DMBOK2的解释：

*DAMA-DMBOK2 中文版 P353*

*DAMA-DMBOK2 中文版 P353*

*DAMA-DMBOK2 中文版 P354*

5、以上文字描述恐怕大部分人看完都是不太理解或一脸懵，特别是DAMA-DMBOK2的解释，毕竟那些文字都是从英文原版书中直译过来的。

6、我的理解：一致性关注点在数据是否合规，即是否负责遵循统一的规范和是否符合逻辑；而准确性则侧重于关注数据的真实性，是否正确，是否存在异常。

7、举个例子吧，比如某通讯录表中数据如下：

一般人看这行数据并没有问题，但是有经验的人可能可以一样发现这行数据存在的问题。

这行数据的“联系电话”为“13800138000”，“一致性”是没问题的，因为符合手机号的格式，也是一个正常的手机号，但是准确性就有问题了，因为众所周知“13800138000”在早些年是中国移动手机充值卡充值电话，后在2015年10月1日起停止服务

（http://www.chinamobile.com/aboutus/news/pannounce/gx/index\_771\_771\_detail\_29736.html），即便是停止服务了，该号码也应该属于中国移动内部保留号码，不会向公众开放选用，所以数据中这个值是肯定不正确的，符合”一致性”但有违“准确性”。

8、再举个例子，比如某用户信息数据如下：

以上主要关注“联系电话”和“有效期”两个字段值。

直观的可以看出，联系电话是不准确的，且不符合正常电话号的规则，除了满足中国大陆手机号的位数，即“联系电话”违反“一致性”和“准确性”，如果要防止此类脏数据入库，可能上游系统需要优化联系电话的校验规则（如选用更通用的正则表达式），不能仅仅是11位数字就让通过校验。

再看“有效期”，从挨着的“注册日期”字段可以分析出，这里的日期类型存储的值为“yyyyMMdd”格式的字符串，而“有效期”的值“99999999”其实是不符合日期类型取值逻辑的，因为9999年99月99日，年为9999可以，月、日为99明显不符合逻辑，但是这条数据就是对的，因为通过相关文档可以了解，有效期默认就是“99999999”，由此看来，它在此处并不违反“一致性”，因为有约定。

那为什么说“联系电话”符合11位数字又不算符合“一致性”呢，笔者认为，这应该属于一个常识吧。

9、综上，同一场景下，违反一致性的数据一定违反准确性，违反准确性的数据不一定违反一致性，但准确性的可解释性有点复杂，同样的数据，在A看来是正确的，而在B看呢，又是错误的，公说公有理婆说婆有理。

很多时候，数据质量的相关维度需要各个组织内部提前提炼和定义好，做好基于自己组织的合理解释，而后再开展各项活动。

10、以上，不知您看完本文后，能否区分开“准确性”和“一致性”呢，如有不妥或不明之处，欢迎留言指正或讨论。

更多精彩：

[**为什么要做数仓分层，不做行吗？**](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247501096&idx=1&sn=fb2644820eb640f5f45aa7f99fc494cd&chksm=c325143df4529d2be89c0168cc869a33a0b75f72363919ad6ba597523cba5eec84aef5dc4b5d&scene=21#wechat_redirect)

[**CRM数据质量怎么控？来，全球500强的经验分享给你！**](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247487710&idx=1&sn=95c88db2cb83308f41031cd38cd1bf1d&chksm=c326c1cbf45148dd019df3d3adb279ddf8434898f384269d01922ebe614e3ab3d79122a81073&scene=21#wechat_redirect)

[**数据治理工作的8种推进套路（下）**](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247500567&idx=1&sn=7cd45531352e76ccbaca809303646985&chksm=c3251202f4529b147ce39067cb7f79faf034eda38eb51dcb98565549c7d0d96c6f4bf43728c5&scene=21#wechat_redirect)

[**数据治理工作的8种推进套路（上）**](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247500374&idx=1&sn=ea235fca49a4bb3c15ae5559f9d865b2&chksm=c3251343f4529a55500ab3c7fe04a40e7cb4b5552564d8abdfd68b3fc2d8238d28884a132db4&scene=21#wechat_redirect)

[**数据中台之OneID (ID-Mapping)架构设计细节全解**](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247500331&idx=1&sn=2098c190277ab503c6822c7f1477dcb4&chksm=c325133ef4529a289b28e22665ec64a4d2e9c7cf4cca8b611dfb0c6c106638ee53440cd708a4&scene=21#wechat_redirect)

[**什么是数据治理的第一性原理？**](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247498676&idx=1&sn=09173d5b60dbc27d0d41e58f4f806018&chksm=c3252aa1f452a3b7bca488c109834dfb435c3e5c2716fe531169da82624b897b4fa6205f2e33&scene=21#wechat_redirect)

[**数据管理和数据治理那个大？**](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247494229&idx=1&sn=9d5f3eb916bba84dd2e5999b69bd6918&chksm=c3253b40f452b256d966de5c78e49ea4f1a59854ba4006620ff17c722103adc5a9df9e601a9d&scene=21#wechat_redirect)

[**为什么数据治理项目会失败？**](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247492039&idx=1&sn=c2e623bc48f11b38eeb6df2ed81c898a&chksm=c32530d2f452b9c4aca7a96de0a3c19e10e8685e2768ae2d8137da3d7e91210226407f5158e2&scene=21#wechat_redirect)

[**主数据怎么治理？**](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247491375&idx=1&sn=85bb90508931f77e20bcd013449ba434&chksm=c326ce3af451472c6661a251e917269f6d95d2213515324d34a5cf149e4049ca365f2418acb3&scene=21#wechat_redirect)

**[【66页PPT】部委、集团级数据治理项目经验分享](http://mp.weixin.qq.com/s?__biz=Mzk0NDI0NDg1OA==&mid=2247487750&idx=1&sn=c3df955cfcfd1bb2daceffd40f3da277&chksm=c326c013f4514905f934c8016aa80c6f0d0c37b41641f369b0ca5e880281d5addbbfbfa3d80e&scene=21#wechat_redirect)**

**排版** | 老彭

**审校** | 老彭  **主编** | 志明