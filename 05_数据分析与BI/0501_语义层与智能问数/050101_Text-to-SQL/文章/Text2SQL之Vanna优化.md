---
title: Text2SQL之Vanna优化
author: ToTensor
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkzMDY0NjkzNQ==&mid=2247484031&idx=1&sn=40e83ca5349db01c39df594c96f5cc9d&chksm=c33b893119060dc6d6ac5da753094a52d4d45c71596ca8e4c1292041e00e92a19e25b4bd47d9&mpshare=1&scene=24&srcid=0807LvAGkYv2AA4fMMSaVgYw&sharer_shareinfo=823b0b5850b75c3535558326f438f447&sharer_shareinfo_first=823b0b5850b75c3535558326f438f447#rd
---

# 前言

前阵子写了篇Text2SQL的简单介绍，发现其也是RAG只会，写下了Text2SQL之不装了，我也是RAG

最近也一直在做Text2SQL的优化，于是把自己的一些心得，总结于这篇文章。

# 一、优化方向

既然本质是RAG，那顺着RAG的优化方向走，准没错。

* 文档增强：

+ 对文档进行摘要：先对摘要进行检索，如果有必要，才深入文档细节
+ 对文档进行QA生成：同时检索文档和生成的QA
+ 数据清洗，去除文档中存在的特殊字符或不相关信息

* 元数据过滤：例如对query和文档分类，对应类别的query只查找对应类别的文档，能有效提升检索效率和召回率
* 混合检索：语义检索 + 关键词检索
* query改写：让LLM对query改写，在嵌入空间中，看似相同的两个问题并不一定很相似

+ 同义词替换：例如，将“LLM”、“大语言模型”和“大模型”标准化为通用术语。
+ 缩写替换

* query分解：将复杂query分解成多个子问题，逐个进行匹配，然后汇总文档，生成答案
* HyDE: 用LLM直接生成query的回答，将生成的答案与query 拼接，再进行RAG
* 文档分块优化：

+ 内容重叠分块：在分块时，保持相邻文本块之间有一定的内容重叠。

* 文档压缩：对检索到的相关上下文，让LLM总结一遍，再丢给RAG
* T-RAG: 构造实体树，若query中存在实体，则对实体进行检索
* CRAG: 检索到相关文档后，使用 LLMs 进行评估，分类三类 Correct、Incorrect、Ambiguous。
* SELF-RAG:

+ 判断是否需要额外检索事实性信息（retrieve on demand），仅当有需要时才召回
+ 处理每个片段：生产 prompt + 一个片段 的生成结果
+ 使用反思字段，检查输出是否相关，选择最符合需要的片段
+ 再重复检索
+ 生成结果会引用相关片段，以及输出结果是否符合该片段，便于查证事实。

这是找了一些资料做的总结，凑合着看吧。

这篇文章主要针对**文档增强**部分的**对文档进行QA生成**进行赘述

# 二、干就完了

想必大家肯定做过从文档中抽取问答对，这是RAG中丰富知识库的一种非常常规的手段，在`RAG`中是生成`QA`对，那么，在`text2sql`中，生成的便是`query-sql`对了。

我们可以对每个表的部分数据，进行`Query-SQL` 对生成，在构建知识库时，将此部分数据`embedding`，参与检索，当用户的问题与该部分数据相似时，则检索完成后，这部分`Query-SQL`对会被添加到`prompt`中，对`LLM`来说，这部分数据不但可以约束`LLM`的输出格式，还能辅助当前的`SQL`生成。

并且，只对一张表进行SQL生成，比从多张表里找最相似的表进行SQL生成要容易得多。预生成`Query-SQL`对，就是对一张表进行`SQL`生成，实际落地，是根据用户的问题，检索出最相关的`topn` 个表进行`query`生成，很明显，前者难度更小。因此，生成的`SQL`也会更准确。

我这里做了两手准备:

* 一次性生成多个`Question-SQL`对
* 先生成一个问题，再根据`DDL`和业务数据生成`SQL`

之所以这样做，是考虑到一次性生成多个`Question-SQL`对时准确率可能会比较低，事实上也确实如此，在做`RAG`时就发现了这个问题。

不过，生成的SQL都是需要校验的，如下：

## 一次性生成多个Question-SQL对

```
    def optimize_question(self, sql_ddl, sql_desc, question):        logger.info(f"优化用户问题: {question}")        system_prompt = f"""以下DDL为{sql_desc} 表, DDL：{sql_ddl} 。请你根据以上信息对用户的问题进行优化，使问题更加明确。请记住，你只需要输出优化后的问题，不要输出其它额外的内容。  
EXAMPLES  
INPUT:统计最常见的五个用户常用词OUTPUT:IOS输入法中，该用户最常用的五个常用词是什么？INPUT:统计搜索次数最多的前两个搜索内容OUTPUT:用户在Safari浏览器中搜索次数最多的前两个搜索内容是什么？"""        user_prompt = f"INPUT:\n{question}\nOUTPUT:"        message_log = [            self.vn.system_message(system_prompt),            self.vn.user_message(user_prompt)        ]        llm_response = self.vn.submit_prompt(message_log)                return llm_response  
  
    def json_correct(self, input_text):        logger.info(f"纠正json: {input_text}")        system_prompt = "你非常善于对一段格式有误的 JSON 进行修改。请你对用户提供的格式有误的JSON进行修正，你需要把它严格按照 JSON 标准进行改正为可以直接进行解析的代码，并重新输出。\n\n请记住：回答时不要做任何解释，只回答以 [ 开头的 JSON。"        user_prompt = f"输入：{input_text}"        message_log = [            self.vn.system_message(system_prompt),            self.vn.user_message(user_prompt)        ]        llm_response = self.vn.submit_prompt(message_log)        question_sqls = re.findall(r"```json\n(.*)```", llm_response, re.DOTALL)        if question_sqls:            llm_response = question_sqls[-1]  
        return llm_response  
    def pre_generate_question_sql_pair(self):                n_questions = 5        files = get_dir_files_path(self.data_dir, '.csv')        for file in files:            table_name = self.file_table_map[file]            ddl = self.vn.run_sql(f"SELECT sql FROM sqlite_master WHERE name = '{table_name}' AND sql IS NOT NULL;")             sql_ddl = ddl["sql"].to_list()[0]                        match = re.search(r'--(.+)', sql_ddl, re.MULTILINE)            sql_desc = match.group(1).strip()                        # print(sql_ddl)            # print(sql_desc)            df = pd.read_csv(file).head()            # print(df.to_markdown())  
            message_log = [                self.vn.system_message(                    f"你是一个SQLite专家。"                ),                self.vn.user_message(                    f"根据用户提供的SQLite表信息和数据样例，生成 {n_questions} 个 Question-SQL 对。\n\n" +                    f"表描述：{sql_desc} 表，SQLite建表语句：{sql_ddl}\n\n 数据样例：{df.to_markdown()}\n\n" +                    """必须以JSON格式回答。格式如下：	```json	[	    {"Question": "生成的第一个问题", "SQL": "第一个问题对应的SQL语句"}	    {"Question": "生成的第二个问题", "SQL": "第二个问题对应的SQL语句"}	]	```回答时不要做任何解释，只回答JSON。请记住，生成的问题必须明确，生成的SQL必须是SELECT语句，必须要符号SQLite的语法规范，尽量不要生成 `SELECT *`开头的SQL语句。不要试图生成错误的SQL和JSON，否则会造成严重的BUG。"""                )            ]            llm_response = self.vn.submit_prompt(message_log)            try:                question_sqls = re.findall(r"```json\n(.*)```", llm_response, re.DOTALL)                if question_sqls:                    question_sql_json = json.loads(question_sqls[-1])            except Exception as e:                try:                    logger.warning(f"生成JSON 失败，即将纠正: {e}")                    correct_json = self.json_correct(llm_response)                    print(correct_json)                    print('----'*20)                    question_sql_json = json.loads(correct_json)                    logger.info("纠正JSON成功...")                except Exception as e1:                    logger.warning(f"纠正JSON失败: {e}")                    continue                        for question_sql in question_sql_json:                question = question_sql["Question"]                sql = question_sql["SQL"]                sql_status = self.val_sql(sql)                if sql_status:                    optimized_question = self.optimize_question(sql_ddl, sql_desc, question)                    logger.info(f"优化后的Question: {optimized_question}")                else:                    logger.info(f"SQL：{sql} 无效...")                    continue                                meta = {                    "Question": optimized_question,                    "SQL": sql                }                                with jsonlines.open(self.pre_general_question_sql_path, 'a') as f:                    f.write(meta)
```

在一次性生成多个SQL时，发现生成的问题容易模糊不清，缺乏明确的主语，举个在`RAG`中比较容易碰到的例子，生成的问题如：

```
实验的目的是什么？论文的研究背景是什么？...
```

这一类问题，不利于检索，因此再让`LLM`优化了一下。

并且，在生成`SQL`时，还对`SQL`进行了有效性校验以及执行校验，如果执行报错了，那肯定是错误的`SQL`，放进`prompt`中也是错误的信息，对结果是有害的。如果想更严格一点，可以考虑使用更大的`LLM`对生成的`SQL`进行正确性判断，类似`GPT4`裁判的做法，这里就不展开叙述了。

## 先生成一个问题，再根据DDL和业务数据生成SQL

```
    def pre_generate_question(self):        n_questions = 5        files = get_dir_files_path(self.data_dir, '.csv')        for file in files:            table_name = self.file_table_map[file]            ddl = self.vn.run_sql(f"SELECT sql FROM sqlite_master WHERE name = '{table_name}' AND sql IS NOT NULL;")             sql_ddl = ddl["sql"].to_list()[0]            match = re.search(r'--(.+)', sql_ddl, re.MULTILINE)            sql_desc = match.group(1).strip()            df = pd.read_csv(file).head()  
            message_log = [                self.vn.system_message(                    f"You are a helpful data assistant."                ),                self.vn.user_message(                    f"""                    SQLite中：{sql_desc} 表的建表语句：\n\n{sql_ddl} \n\n以及该表的一部分数据：{df.to_markdown()}                    请你根据以上数据，生成 {n_questions} 个问题，每行一个，生成的问题必须能够通过以上数据找到答案，请记住，回答时不要做任何额外的解释，只回答问题。                    请记住，生成的问题必须能够通过SQL查询语句找到答案，生成的每个问题应该明确主谓宾，不要生成模糊不清的问题，因为模糊的问题对于检索有非常严重的问题。                    请记住，生成的问题里面，必须包含 {sql_desc} 中的内容，否则，当我有很多表时，我无法明确你的问题的目的。                    请记住，不要生成和序号相关的问题，因为这一列没有任何作用。                    """                )            ]  
            llm_response = self.vn.submit_prompt(message_log)  
            numbers_removed = re.sub(r"^\d+\.\s*", "", llm_response, flags=re.MULTILINE)            generate_questions = numbers_removed.split("\n")  
            logger.info(f"根据SQL数据示例生成的Question: \n {generate_questions}")            for question in generate_questions:                message_log = [                    self.vn.system_message(                        f"你是一位资深的SQLite专家，非常擅长将自然语言转换成符合SQLite语法的SQL语句。"                    ),                    self.vn.user_message(                        f"""                        SQLite中：{sql_desc} 表的建表语句：\n\n{sql_ddl} \n\n以及该表的一部分数据：{df.to_markdown()}                        请你根据以上数据，将问题：{question} 转换成对应的SQL语句，以便我后续执行SQL语句。                        请记住，你的回答只有SQL语句，无需回答额外的解释，否则会产生严重的BUG。                        """                    )                ]                llm_response = self.vn.submit_prompt(message_log)                sql_statement = self.vn.extract_sql(llm_response)                sql_status = self.val_sql(sql_statement)                if sql_status:                    logger.info(f"根据问题: {question} 生成的SQL: {sql_statement}")                else:                    logger.info(f"SQL：{sql_statement} 无效...")                    continue                meta = {                    "Question": question,                    "SQL": sql_statement                }                                with jsonlines.open(self.pre_general_question_sql_path, 'a') as f:                    f.write(meta)
```

这里的做法，就是针对`DDL`和部分数据，只生成一个问题，不生成`SQL`，再让LLM根据生成的问题和`DDL`，去生成`SQL`，可以理解为是对一次性生成`Question-SQL`对的做法进行拆解，将复杂步骤拆解成多个步骤，降低`LLM`生成难度。当然，这里的`SQL`校验也是有必要的。

# 总结

提升RAG的效果，能一定程度上提升Text2SQL的效果，剩下的，就看LLM的能力了。