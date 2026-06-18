---
title: hvPlot：一个给你熟悉的 Pandas 加上一个超好用的可视化神器
author: 小白这样学Python
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkyMTU4MDIyMA==&mid=2247494886&idx=1&sn=6ebe30d1cb6c2c08b0bcd5077c498c8a&chksm=c0e8e9cdec644af05119728b1d46398892542a3e7bfadd6037c74963247f3117b706afa11ac7&mpshare=1&scene=24&srcid=0711JlxcEfcRI93VZMImD67T&sharer_shareinfo=6b4ddb94f8865fbb5c79c1dec0372526&sharer_shareinfo_first=6b4ddb94f8865fbb5c79c1dec0372526#rd
---

大家好！最近我刷到一个超好用的可视化神器——hvPlot，想跟你们唠唠。要是你跟我一样，平时用 Pandas、Xarray 然后还要切换到 Matplotlib、Plotly，写一堆冗长代码来回切换，肯定会心累。hvPlot 出来后：一行 `.hvplot()`，全搞定！

**hvPlot 到底是什么？**  
简单来说，hvPlot 是一个给你熟悉的 Pandas `.plot()` API 加上 HoloViz 生态系统的大招。

* • 支持数据源：Pandas、Polars、XArray、Dask、Streamz、Intake、GeoPandas、NetworkX……
* • 支持绘图后端：Bokeh、Matplotlib、Plotly
* • 提供交互式 API：`.interactive()`，自动给你加滑块、下拉菜单……

它的优势就在于“用你熟悉的接口，再给你大杀器”。你不需要学一堆新 API，就能直接享受 HoloViz 的交互能力。

**它解决了哪些痛点？**

1. 1. 多工具切换太麻烦  
   以前你要在 Pandas、Bokeh、Plotly、Matplotlib 里来回改代码，光改参数就够头疼。
2. 2. 写交互控件要手动写 Panel、ipywidgets  
   想在 Jupyter Notebook 做个简单的滑块过滤？Panel、ipywidgets 要写一堆模板。
3. 3. 图表样式不统一、难美化  
   各家默认 theme、配色差别大，整合报表的时候拼图很难看。

hvPlot 给你：

* • **统一接口**：`.hvplot(kind='scatter', color='species')`
* • **一键交互**：`.interactive()` 带控件
* • **主题一致**：HoloViz 默认风格，跟 Panel、Param、Datashader 无缝衔接

**核心代码示例**  
下面演示一下，从安装到画图，一气呵成。

```
# 安装  
conda install hvplot -c conda-forge  
# 或者  
pip install hvplot
```

```
import hvplot.pandas  
from bokeh.sampledata.penguins import data as df  
  
df.hvplot.scatter(x='bill_length_mm', y='bill_depth_mm', by='species')
```

**优缺点大盘点**

| 优点 | 说明 |
| --- | --- |
| 接口亲切 | 基于 Pandas/Xarray `.plot()` 扩展，学习成本几乎为零 |
| 多后端支持 | 切换 Bokeh、Matplotlib、Plotly 只要一行 `extension()` |
| 交互功能一键开启 | `.interactive()` 自动生成滑块、下拉……配合 Panel 秒变小仪表盘 |
| 生态整合 | 和 HoloViz 其他库（Datashader、Panel、Param）无缝协作 |
| 文档齐全 | 官方示例多，支持在线帮助 `hvplot.help(kind='scatter')` |

| 缺点 | 说明 |
| --- | --- |
| 定制化极限 | 复杂的自定义样式、动画得深入到底层的 Bokeh/Matplotlib API |
| 性能约束 | 超大数据量虽然支持 Dask/Datashader，但第一次上手要配置分片任务 |
| 依赖生态 | 要用最爽的交互得配合 Panel、Param，有点生态学习成本 |

**小结**  
总的来说，hvPlot 就是那种“你熟悉就好、我给你加料”的好工具。想快速做出漂亮交互图表？它就是你的首选。想低成本切换后端？它也能帮你搞定。要是你现在还在写一堆模板代码来回折腾，强烈推荐：动动手指，`pip install hvplot`，马上上线你的数据可视化新花样！

**项目地址**：https://github.com/holoviz/hvplot