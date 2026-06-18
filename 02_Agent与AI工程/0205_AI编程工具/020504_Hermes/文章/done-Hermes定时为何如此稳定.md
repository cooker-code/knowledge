> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: Hermes定时为何如此稳定
author: ai4u
date: ai4uai4u
url: https://mp.weixin.qq.com/s?__biz=MzkzNjg2MDQ5Mg==&mid=2247483747&idx=1&sn=6e8da2c289bfb4932e7a182846688ecb&chksm=c3ecfb8994803e11e8add2fe1b9677a4a580c04276ea13d77cc08ba83faa56580214d895041e&mpshare=1&scene=24&srcid=0501cegnwkvSTiixmRyVwRql&sharer_shareinfo=f3f93596b3258784ddc09784f7edef0e&sharer_shareinfo_first=f3f93596b3258784ddc09784f7edef0e#rd
---

Hermes Cron 是 Hermes Agent 内置的定时任务调度系统，运行在 Gateway 进程的后台线程中，每60秒 tick 一次检查到期任务。它与系统 crontab 无关，完全由 Hermes 自身管理。  

【架构】Gateway主循环监听消息，cron\_ticker后台线程循环执行 tick()。每次 tick 先加文件锁(flock)，读取 ~/.hermes/cron/jobs.json 获取到期任务，先推进 next\_run\_at 再执行(at-most-once保障)，然后通过 ThreadPoolExecutor并行执行多个任务。  

【run\_job流程】每个任务在独立线程中经历4个阶段：① \_build\_job\_prompt() 注入skills内容、cron投递指令；② 重新加载.env和config.yaml，解析模型/Provider；③ 创建 AIAgent 实例并执行 run\_conversation(prompt)；④ 后处理：保存输出到 output/{id}/{time}.md，自动投递到飞书等平台，更新 jobs.json 记录状态。  

【上下文沙盒】cron 模式使用独立纯净会话：load\_soul\_identity=True 载入SOUL.md保持身份，但 skip\_memory=True 跳过所有持久记忆(防止cron产物污染用户画像)，skip\_context\_files=True 不加载项目AGENTS.md。每次运行都是全新的LLM会话。同时禁用 cronjob/messaging/clarify 工具集，防止递归创建任务或向用户提问。  

【超时控制】10分钟无活动(inactivity timeout)自动kill，每5秒poll一次 agent 的活动状态。