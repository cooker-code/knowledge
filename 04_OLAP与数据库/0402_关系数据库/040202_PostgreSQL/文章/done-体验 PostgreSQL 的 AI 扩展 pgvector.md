> 已吸收至：[[04_OLAP与数据库/0402_关系数据库/040202_PostgreSQL/040202_核心知识点/PostgreSQLpgvector与多模态检索边界|PostgreSQLpgvector与多模态检索边界]]
---
title: 体验 PostgreSQL 的 AI 扩展 pgvector
author: 老码泽云数据工程随笔
date:
url: https://mp.weixin.qq.com/s?__biz=MzA5NTk4NjAyNw==&mid=2247483784&idx=1&sn=56c1e69605b6dc773acb15078467076a&chksm=91555ef14efc8027873a984456b7e83d560c85eccb0cd6216ecfea11499da61e70fcbde714f6&mpshare=1&scene=24&srcid=10112lk6rNOnjyMJ1lVy6dcD&sharer_shareinfo=b9415c05f51d219a1a990300afdf16a5&sharer_shareinfo_first=b9415c05f51d219a1a990300afdf16a5#rd
---

了解概念 - 向量 Vector 与 嵌入 Embeddings

向量vector ，表示n个有序的数值序列。在AI模型中，学习是基于特征向量的算法处理过程。因此vector可以用于存储任意的AI特征向量。现阶段pgvector的主要应用场景中，是用于存储行记录的语义嵌入(embedding)向量。

怎么理解Embedding这个词？em有放入（to put into）的意思， bed是床，引申为承载体或基座，因此将embed翻译为嵌入，embedding是动词的名词形式，表示嵌入以后形成的状态，仍然简单翻译为嵌入，在AI语境下，翻译为嵌入或嵌入向量。它首先表达的嵌入技术，也往往用于表达嵌入处理后的形成的向量。就像将宝石嵌入（embed）底座一样，Embedding技术是将一个词、一个商品ID、一段话、一张图片、一段声音等任意待理解的数据压缩投射到一个预设的连续向量空间中。这个空间为所有数据提供了存在和相互比较的“场所”。好的嵌入不仅是压缩投射，更要保留甚至凸显对象间的内在关系。在深度学习中，一个好的Embedding会使语义相近的数据在向量空间中的位置也很接近。

因此，Embedding技术的价值在于它让计算机能够以数值化方式理解和计算对象之间的复杂关系，其应用非常广泛，如信息检索、图像分类、自然语言处理等等。在pgvector应用中，往往使用名为embedding的字段来存放使用AI模型提供的嵌入处理生成的结果向量。SQL建表语句类似如下(m代表向量维数)

```
create table T1 (id bigserial primary key,...embedding vector(m)
);
```

常见的应用场景

pgvector 将向量数据与结构化业务数据无缝融合存储，通过针对向量相似性的索引实现实时检索。其核心优势在于继承 PostgreSQL 的事务特性和 SQL 标准，允许实现标准SQL语句与向量相似性搜索的结合，直接支撑语义推荐、搜索、知识库检索等业务场景，无需引入额外复杂系统，为中小规模含有AI场景的应用系统提供了成本效益极高的单一数据库解决方案。

其具体的功能是，文本、图像、音视频等信息或特征的嵌入表示存储、索引和相似性检索。比如，在电商系统中，实现基于商品描述语义的相似产品推荐，而不仅仅是关键词匹配；为内部知识库或客服系统构建智能问答功能，通过语义搜索快速找到最相关的文档片段来生成答案。

安装

docker方式是最快捷的，以pg15为例。

```
docker volume create pgdata15docker run -d --name pgvectorpg15 \-p 5432:5432 \-e TZ=PRC \-e POSTGRES_PASSWORD=secret \-v pgdata15:/var/lib/postgresql/data \pgvector/pgvector:pg15
```

如此，得到一个后台运行的pg docker容器。

安装了pgvector以后，数据库系统中增加了哪些内容呢？另外启动一个bash进程来了解所创建容器内的相关目录和文件信息,

```
docker exec -it pgvectorpg15 bashls /usr/lib/postgresql/15/lib | grep vectorls /usr/share/postgresql/15/extension | grep vector
```

核心是动态库 `/usr/lib/postgresql/15/lib/vector.so` 和 相应的扩展描述信息和脚本信息。在目录`/usr/share/postgresql/15/extension`, 文件 `pgvector.control`元信息, 脚本 `vector--0.8.0.sql`，创建（注册）类型、函数、操作符等，将扩展所实现的功能注册到postgres系统中。

```
$ cat/usr/share/postgresql/15/extension/vector.controlcomment = 'vector data type and ivfflat and hnsw access methods'default_version = '0.8.0'module_pathname = '$libdir/vector'relocatable = true 
```

简单体验

另开一个psql命令窗口，创建数据库、含vector字段的表，然后体验相关的操作。

```
docker exec -it pgvectorpg15 su -l postgres -c psqlcreate database testpgvector;\c testpgvector;
```

在该数据库中注册vector扩展，并通过 `\dx` 查看已安装的扩展。

```
create extension if not exists vector;\dxList of installed extensions  Name   | Version |   Schema   |                     Description                      ---------+---------+------------+------------------------------------------------------ plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language vector  | 0.8.0   | public     | vector data type and ivfflat and hnsw access methods(2 rows)
```

由此可知，核心的数据类型是 vector，核心的访问方法（索引）是 ivfflat和hnsw。

由于AI应用中，向量的维数往往很高，不方便手工体验，这里以仅有5个元素的示例向量来演示操作。

假设有一个产品表，有图片和图片的嵌入向量 (正常情况下该嵌入向量的维数不少于512维)。

```
create table products(   id bigserial primary key,  name text,  photo_url text,   photo_embeddings vector(5));insert into products(name, photo_url, photo_embeddings) values ('Pa', 's3://path/to/a.jpg','[0,2,3,4,5]'), ('Pb', 's3://path/to/b.jpg', '[1,2,3,3,3]'), ('Pc', 's3://path/to/c.jpg', '[3,2,3,4,5]'),('Pd', 's3://path/to/d.jpg', '[4,2,3,4,5]'),('Pe', 's3://path/to/e.jpg', '[5,2,3,4,5]'),('Pf', 's3://path/to/f.jpg', '[6,2,3,4,5]'),('Pg', 's3://path/to/g.jpg', '[7,2,3,4,5]'),('Ph', 's3://path/to/h.jpg', '[8,2,3,4,5]'),('Pi', 's3://path/to/i.jpg', '[9,2,3,4,5]');
```

执行以上语句，会创建一张表并插入10条记录。

在加入了初期的数据以后，就可以为该字段建立语义检索索引，

```
CREATE INDEX ON products USING hnsw (photo_embeddings vector_l2_ops);
```

索引的类型是 hnsw, 针对的相似性方法是 vector\_l2\_ops。hnsw是类似于在地图上定位某个购物中心的方法 -- 先定位一个大的范围，再定位其中的一个小的范围，再进一步深入定位到具体的位置的过程。vector\_l2\_ops指的是使用L2方法来度量两个向量的相似性。所谓L2方法，其实质就是对两个向量，逐个元素相减的平方（2的来源）之和，作为度量两个向量差异(相似）的数值。很显然当两个向量完全相同时，这个差值为0。当两个向量的元素之间的距离大时，其整体的距离也相应变大。因此这个值越小，代表它们的距离越近，相似度越高。

现在，我们想知道和商品ID为X的商品的图片相似的商品的前5个，其查询写法如下,

```
SELECT * FROM products WHERE id != x ORDER BY photo_embeddings <->  (SELECT photo_embeddings FROM products WHERE id = x)  LIMIT 5;
```

小结与展望

以上，我们简单介绍了如何快速搭建一个装有pgvector的postgres数据库容器环境，并体验其数据插入、建立索引并执行查询的过程。

实际的AI项目中，数据集往往庞大，如何配置相应规模的硬件投入，如何调优系统的参数设置和资源分配，从而保障查询的准确率和系统的响应速度，如何持续观测系统运行情况从而及时调优等，都需要对任务特点、技术细节等有更深入的学习理解和实践。