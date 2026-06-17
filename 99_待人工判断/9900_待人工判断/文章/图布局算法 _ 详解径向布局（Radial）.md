---
title: 图布局算法 | 详解径向布局（Radial）
author: 数据可视化 AntV
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg3NTU4OTc3OA==&mid=2247496667&idx=1&sn=6394ae271bd3de8e7369681c73fc3c28&chksm=ce79dc7dca5d15fce9bc2f8c204fcf6015d4ee3f17bba61e659ee506fae3217584993b5d42d3&mpshare=1&scene=24&srcid=1218Km5ljrt3YOWrfXtLg7d2&sharer_shareinfo=203341bda63d01ff94275d0d0f995b08&sharer_shareinfo_first=203341bda63d01ff94275d0d0f995b08#rd
---

Radial 布局是一种将节点围绕某个中心节点（focusNode）以辐射状排列的图布局，清晰地展现节点之间的层次结构和连接关系。这种布局方式特别**「适用于具有层次关系的场景」**，如组织结构图、树形结构等。

## 基本原理

Radial 布局的核心思想是围绕中心节点按节点与中心的连接关系将节点分配到不同的环形层上。通过将与中心节点直接连接的节点放置在内环上，并将与中心节点间接连接的节点置于外环，可以清晰地显示出节点的层级深度。

| 核心术语 | 解释 |
| --- | --- |
| **focusNode** | 辐射的中心点 |
| **unitRadius** | 每一圈距离上一圈的距离。默认填充整个画布，即根据图的大小决定 |
| **linkDistance** | 边长度 |
| **strictRadial** | 是否必须是严格的 Radial 布局，及每一层的节点严格布局在一个环上 |
| **preventOverlap** | 是否防止重叠 |

preventOverlap = false.   strictRadial = true

preventOverlap = true     strictRadial = true

preventOverlap = true.     strictRadial = false

## 算法实现

G6 的 Radial 布局实现基于多个图算法的结合，实现该布局的步骤如下：

1. **「确定中心节点」**：通常，中心节点是图中比较重要的节点，衡量节点重要性的方式有很多，例如中介中心性、接近中心性等，这里不做过多赘述。在此算法中，直接由用户指定。
2. **「计算节点之间的最短路径」**：利用 Floyd-Warshall 算法来计算图中所有节点对的最短路径。通过构建了一个邻接矩阵，矩阵中的每个值代表图中任意两个节点之间的最短路径长度。
3. **「分配节点到环上」**：根据每个节点与中心节点的最短路径（即“度数”），将节点分配到不同的环上。最近的节点位于最内层环，随着距离的增加，节点分布到更外层的环上。
4. **「排列节点」**：节点被分配到环之后，按照一定的排列规则（如节点权重或某种排序属性）在环上进行排列。若启用防止节点重叠的能力，优化算法会根据节点的大小和节点之间的间距增加一定的斥力，调整节点的位置。

   ### floydWarshall 动态规划算法

Floyd-Warshall 算法是一种用于**「计算图中所有节点对之间最短路径」**的动态规划算法。它适用于有向图和无向图，并且可以处理带有负权边的图（前提是没有负权环）。其核心思想是利用动态规划的方法，不断比较并更新路径长度。

算法的步骤如下：

* 初始化：首先，创建一个邻接矩阵 `dist`，其中 `dist[i][j]` 表示节点 i 到节点 j 的初始距离。对于每一条边 (i, j)，将 `dist[i][j]` 设置为边的权重；对于没有直接连接的节点，设置为 。对于每个节点自身，设置
* 迭代更新：使用三层循环，逐步考虑每个节点 k 作为可能的中间节点。对于每一对节点 (i, j)，更新最短路径：

  这意味着，如果通过节点 k 到达节点 j 的路径比直接从 i 到 j 的路径更短，则更新 `dist[i][j]`。
* 完成：经过所有节点的迭代后，`dist[i][j]` 就包含了从节点 i 到节点 j 的最短路径长度。

Floyd-Warshall 算法的时间复杂度为，其中 𝑉 是图中的节点数。这使得它在处理密集图时非常有效。

## 优缺点

### 优点

* 直观，以中心节点为核心，辐射状排列使得图形结构一目了然。
* 层次分明，节点按照与中心节点的度数关系排列，有助于快速识别各节点之间的关系。

### 缺点

* 布局效果高度依赖于中心节点的选择。
* 当节点数量较多时，外层环的节点可能会过于密集，影响整体可读性。
* 需要计算节点的度数和距离，特别是在大规模图中，计算可能会变得复杂。

## 参考文献

* Ulrik Brandes, Christian Pich. More Flexible Radial Layout.
* Floyd Warshall Algorithm (https://www.geeksforgeeks.org/floyd-warshall-algorithm-dp-16/)

AntV 数据可视化官网

https://antv.antgroup.com/

AntV 数据可视化 GitHub

https://github.com/antvis

✨ 关注我们，了解更多~