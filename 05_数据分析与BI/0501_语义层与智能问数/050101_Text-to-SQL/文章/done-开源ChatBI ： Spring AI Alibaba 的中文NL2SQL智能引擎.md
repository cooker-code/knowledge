---
title: 开源ChatBI ： Spring AI Alibaba 的中文NL2SQL智能引擎
author: AI大模型机器人
date:
url: https://mp.weixin.qq.com/s?__biz=MzkxMzM0NzY0Nw==&mid=2247483855&idx=1&sn=77b80c4c2b0637cf0588ff050fe128cd&chksm=c0c4396bba9bf21131e1ea8490876f083ae9db1694cef1334f55f7add92c8ae43baa7fc871cd&mpshare=1&scene=24&srcid=10292FoOnL9szIchFti7xxm1&sharer_shareinfo=18c5b10d8d1259bbe20fbcf65a884841&sharer_shareinfo_first=18c5b10d8d1259bbe20fbcf65a884841#rd
---


> 已吸收至：[[05_数据分析与BI/0501_语义层与智能问数/050101_Text-to-SQL/050101_核心知识点/Text-to-SQL工程架构与SchemaLinking|Text-to-SQL工程架构与SchemaLinking]]

阿里在NL2SQL（自然语言到SQL）领域有多个开源项目和解决方案，以下是对其NL2SQL智能技术的详细介绍：

一、核心开源项目

  1. Spring AI Alibaba NL2SQL

      • 项目地址：https://github.com/alibaba/spring-ai-alibaba（null）

      • 项目简介：Spring AI Alibaba NL2SQL是阿里云（https://baike.baidu.com/item/%E9%98%BF%E9%87%8C%E4%BA%91/297128）析言GBI产品的开源延伸，致力于打造一套轻量、高效、可扩展的NL2SQL框架。它基于Spring AI Alibaba框架，深度集成百炼平台，支持ChatBot、工作流、多智能体应用开发模式，让Java程序员可以快速构建和集成自然语言查询系统。

      • 核心功能：

          • Schema智能召回：提供强大的语义相似度计算能力和多策略召回机制，能够在海量表结构中精准匹配出最可能涉及的数据库Schema和字段信息。

          • SQL智能生成与优化：基于Qwen等主流大语言模型的强大推理能力，实现从自然语言到结构化SQL的一键生成，支持多种数据库方言和复杂函数能力。

          • SQL自动执行与结果反馈：生成的SQL语句可以直接调度并安全执行，返回结构化结果，并提供丰富的错误处理机制。

  2. 析言GBI

      • 项目简介：析言GBI是阿里云百炼官方推出的智能数据分析产品，基于大模型的ChatBI技术，帮助用户轻松实现自然语言交互的数据分析。

      • 核心功能：

          • NL2SQL：支持用户通过自然语言查询数据库，生成对应的SQL语句。

          • 数据问答：提供数据问答功能，支持用户通过自然语言提问，获取数据洞察。

          • 云端服务：提供丰富的云端服务支持，助力企业实现高效的数据管理与分析。

二、技术特点与优势

  1. 轻量模块化设计：Spring AI Alibaba NL2SQL采用高度解耦的设计理念，将Schema召回、SQL生成、SQL执行三个环节进行模块化封装，开发者可以根据自身需求灵活组合，适配不同的业务场景。

  2. 强大的语义理解能力：通过多模态语义理解和动态权重计算等技术，显著提升Schema匹配的准确性，确保生成的SQL语句符合用户意图。

  3. 支持多种数据库方言：支持MySQL、PostgreSQL等多种数据库方言，满足不同业务场景的需求。

  4. 丰富的错误处理机制：提供丰富的错误处理机制，确保即使在执行失败时也能给出清晰的提示和建议。

  5. 开源与社区共建：阿里积极推动NL2SQL技术的开源与社区共建，希望与开发者共同推动NL2SQL技术在企业级场景中的广泛应用。

三、应用场景与案例

  1. 企业数据分析：企业可以使用NL2SQL技术构建智能数据分析解决方案，支持业务人员通过自然语言查询数据库，获取数据洞察，提升数据分析效率。

  2. 金融行业应用：在金融行业，NL2SQL技术可以应用于客服、营销支持、金融交易等业务领域，帮助业务人员快速查询和分析数据，提升业务决策效率。

  3. 教育行业应用：在教育行业，NL2SQL技术可以应用于学生信息管理、课程安排等场景，支持管理人员通过自然语言查询数据库，获取相关信息。