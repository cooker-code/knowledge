---
title: DolphinScheduler 3.0新增对FlinkSQL任务的支持！
author: 大数据技术与架构
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247515299&idx=1&sn=d649845735baff361d26213fb8fe0725&chksm=fd3efa36ca497320de5dee7e187916900bf49645b09d4b1b1fefc9f0fd17c6a467fd378c4fb7&mpshare=1&scene=24&srcid=0822Z8zu07zyiGbcOqCZBVMN&sharer_sharetime=1661168473225&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> **[全网最全大数据面试提升手册！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247514568&idx=1&sn=659d7464a3b61ebf8397d2597069f4ab&chksm=fd3eff5dca49764beef9ab14400e96715b86b3394f05e161181c03424fa579cdbc242a401411&scene=21#wechat_redirect)**

2022 年 8 月 10 日，在经过 3.0.0 alpha、3.0.0-beta-1、3.0.0-beta-2 不断验证之后，Apache DolphinScheduler 终于正式发布第三个大版本。3.0.0 正式版本发生了自发版以来的最大幅度变动，新增了众多全新功能和特性。

经过迭代的 3.0.0 正式版与此前 3.0.0 alpha 版本更新文中所描述的主要功能和特性更新、优化项和 Bug 修复大致一致，包括“更快、更强、更现代化、更易维护”这四个关键词总结。

* 更快：重构了 UI 界面，新 UI 不仅用户响应速度提高数十倍，开发者构建速度提高数百倍；
* 更强：带来了许多振奋人心的新功能，如数据质量保证、自定义时区、新增多个任务支持和多个告警插件；
* 更现代化：新 UI 除了更快外，大到页面布局，细到图标样式都更加现代化；
* 更易维护：后端服务拆分更加符合容器化和微服务化的发展趋势，还能明确各个服务的职责，让维护更加简单。

### 新功能和新特性包括：

* 全新 UI
* AWS 支持
* 服务拆分

所有的服务都可以通过`bin/dolphinscheduler-daemon.sh`的方式进行启动或者停止。

#### 数据质量保证

此版本中，用户们从 2.0.0 开始就期待已久的数据质量保证应用功能上线，解决了从源头同步的数据条数准确性，单表或多表周均、月均波动超过阈值告警等数据质量问题。Apache DolphinScheduler 此前版本解决了将任务以特定顺序和时间运行的问题，但数据运行完之后对数据的质量一直没有较为通用的衡量标准，用户需要付出额外的开发成本。

现在，3.0.0 已经实现了数据质量原生支持，用户可以直接通过配置的方式，轻松实现数据质量监控，在保证工作流运行的前提下，保证运行结果的准确性。

**另外有一个非常重要的Feature就是对Flink SQL任务的支持。**

#### 支持 Flink 任务类型

在该版本中，Apache DolphinScheduler 扩展了 Flink 任务类型，使其支持运行 Flink SQL 任务，其使用 sql-client.sh 提交任务。在此前的版本中, 我们仅支持通过 flink cli 的方式提交任务, 这种方式需要结合资源中心, 将资源文件提交到资源中心, 然后在任务定义页面引用改资源, 对于版本化和用户透明都不是十分友好. 随着 flink sql 逐渐成为 flink 使用者的主流, 加之直接在编辑页面写 sql 更加用户透明, 我们采纳了向社区贡献的 flink sql 功能. 3.0.0 以后的版本用户可以更加方便的使用 flink 任务了。

如图所示：

核心的实现类包含下面几个：

大概的源代码如下：

```
public class FlinkTask extends AbstractYarnTask {  
    /**     * flink parameters     */    private FlinkParameters flinkParameters;  
    /**     * taskExecutionContext     */    private TaskExecutionContext taskExecutionContext;  
    public FlinkTask(TaskExecutionContext taskExecutionContext) {        super(taskExecutionContext);        this.taskExecutionContext = taskExecutionContext;    }  
    @Override    public void init() {  
        logger.info("flink task params {}", taskExecutionContext.getTaskParams());  
        flinkParameters = JSONUtils.parseObject(taskExecutionContext.getTaskParams(), FlinkParameters.class);  
        if (flinkParameters == null || !flinkParameters.checkParameters()) {            throw new RuntimeException("flink task params is not valid");        }        flinkParameters.setQueue(taskExecutionContext.getQueue());  
        if (ProgramType.SQL != flinkParameters.getProgramType()) {            setMainJarName();        }    }  
    /**     * create command     *     * @return command     */    @Override    protected String buildCommand() {        List<String> args = new ArrayList<>();  
        if (ProgramType.SQL != flinkParameters.getProgramType()) {            // execute flink run [OPTIONS] <jar-file> <arguments>            args.add(FlinkConstants.FLINK_COMMAND);            args.add(FlinkConstants.FLINK_RUN);            args.addAll(populateFlinkOptions());        } else {            // execute sql-client.sh -f <script file>            args.add(FlinkConstants.FLINK_SQL_COMMAND);            args.addAll(populateFlinkSqlOptions());        }        String command = ParameterUtils.convertParameterPlaceholders(String.join(" ", args), taskExecutionContext.getDefinedParams());        logger.info("flink task command : {}", command);        return command;    }  
    /**     * build flink options     *     * @return argument list     */    private List<String> populateFlinkOptions() {        List<String> args = new ArrayList<>();  
        String deployMode = StringUtils.isNotEmpty(flinkParameters.getDeployMode()) ? flinkParameters.getDeployMode() : FlinkConstants.DEPLOY_MODE_CLUSTER;  
        if (!FlinkConstants.DEPLOY_MODE_LOCAL.equals(deployMode)) {            populateFlinkOnYarnOptions(args);        }  
        // -p        int parallelism = flinkParameters.getParallelism();        if (parallelism > 0) {            args.add(FlinkConstants.FLINK_PARALLELISM);            args.add(String.format("%d", parallelism));        }  
        /**         * -sae         *         * If the job is submitted in attached mode, perform a best-effort cluster shutdown when the CLI is terminated abruptly.         * The task status will be synchronized with the cluster job status.         */        args.add(FlinkConstants.FLINK_SHUTDOWN_ON_ATTACHED_EXIT);  
        // -s -yqu -yat -yD -D        String others = flinkParameters.getOthers();        if (StringUtils.isNotEmpty(others)) {            args.add(others);        }  
        // -c        ProgramType programType = flinkParameters.getProgramType();        String mainClass = flinkParameters.getMainClass();        if (programType != ProgramType.PYTHON && StringUtils.isNotEmpty(mainClass)) {            args.add(FlinkConstants.FLINK_MAIN_CLASS);            args.add(flinkParameters.getMainClass());        }  
        ResourceInfo mainJar = flinkParameters.getMainJar();        if (mainJar != null) {            args.add(mainJar.getRes());        }  
        String mainArgs = flinkParameters.getMainArgs();        if (StringUtils.isNotEmpty(mainArgs)) {            // combining local and global parameters            Map<String, Property> paramsMap = ParamUtils.convert(taskExecutionContext, getParameters());            if (MapUtils.isEmpty(paramsMap)) {                paramsMap = new HashMap<>();            }            if (MapUtils.isNotEmpty(taskExecutionContext.getParamsMap())) {                paramsMap.putAll(taskExecutionContext.getParamsMap());            }            args.add(ParameterUtils.convertParameterPlaceholders(mainArgs, ParamUtils.convert(paramsMap)));        }  
        return args;    }  
    private void populateFlinkOnYarnOptions(List<String> args) {        // -m yarn-cluster        args.add(FlinkConstants.FLINK_RUN_MODE);        args.add(FlinkConstants.FLINK_YARN_CLUSTER);  
        // -ys        int slot = flinkParameters.getSlot();        if (slot > 0) {            args.add(FlinkConstants.FLINK_YARN_SLOT);            args.add(String.format("%d", slot));        }  
        // -ynm        String appName = flinkParameters.getAppName();        if (StringUtils.isNotEmpty(appName)) {            args.add(FlinkConstants.FLINK_APP_NAME);            args.add(ArgsUtils.escape(appName));        }  
        /**         * -yn         *         * Note: judge flink version, the parameter -yn has removed from flink 1.10         */        String flinkVersion = flinkParameters.getFlinkVersion();        if (flinkVersion == null || FlinkConstants.FLINK_VERSION_BEFORE_1_10.equals(flinkVersion)) {            int taskManager = flinkParameters.getTaskManager();            if (taskManager > 0) {                args.add(FlinkConstants.FLINK_TASK_MANAGE);                args.add(String.format("%d", taskManager));            }        }  
        // -yjm        String jobManagerMemory = flinkParameters.getJobManagerMemory();        if (StringUtils.isNotEmpty(jobManagerMemory)) {            args.add(FlinkConstants.FLINK_JOB_MANAGE_MEM);            args.add(jobManagerMemory);        }  
        // -ytm        String taskManagerMemory = flinkParameters.getTaskManagerMemory();        if (StringUtils.isNotEmpty(taskManagerMemory)) {            args.add(FlinkConstants.FLINK_TASK_MANAGE_MEM);            args.add(taskManagerMemory);        }  
        // -yqu        String others = flinkParameters.getOthers();        if (StringUtils.isEmpty(others) || !others.contains(FlinkConstants.FLINK_QUEUE)) {            String queue = flinkParameters.getQueue();            if (StringUtils.isNotEmpty(queue)) {                args.add(FlinkConstants.FLINK_QUEUE);                args.add(queue);            }        }    }  
    /**     * build flink sql options     *     * @return argument list     */    private List<String> populateFlinkSqlOptions() {        List<String> args = new ArrayList<>();        List<String> defalutOptions = new ArrayList<>();  
        String deployMode = StringUtils.isNotEmpty(flinkParameters.getDeployMode()) ? flinkParameters.getDeployMode() : FlinkConstants.DEPLOY_MODE_CLUSTER;  
        /**         * Currently flink sql on yarn only supports yarn-per-job mode         */        if (!FlinkConstants.DEPLOY_MODE_LOCAL.equals(deployMode)) {            populateFlinkSqlOnYarnOptions(defalutOptions);        } else {            // execution.target            defalutOptions.add(String.format(FlinkConstants.FLINK_FORMAT_EXECUTION_TARGET, FlinkConstants.EXECUTION_TARGET_LOACL));        }  
        // parallelism.default        int parallelism = flinkParameters.getParallelism();        if (parallelism > 0) {            defalutOptions.add(String.format(FlinkConstants.FLINK_FORMAT_PARALLELISM_DEFAULT, parallelism));        }  
        // -i        args.add(FlinkConstants.FLINK_SQL_INIT_FILE);        args.add(generateInitScriptFile(StringUtils.join(defalutOptions, FlinkConstants.FLINK_SQL_NEWLINE).concat(FlinkConstants.FLINK_SQL_NEWLINE)));  
        // -f        args.add(FlinkConstants.FLINK_SQL_SCRIPT_FILE);        args.add(generateScriptFile());  
        String others = flinkParameters.getOthers();        if (StringUtils.isNotEmpty(others)) {            args.add(others);        }        return args;    }  
    private void populateFlinkSqlOnYarnOptions(List<String> defalutOptions) {        // execution.target        defalutOptions.add(String.format(FlinkConstants.FLINK_FORMAT_EXECUTION_TARGET, FlinkConstants.EXECUTION_TARGET_YARN_PER_JOB));  
        // taskmanager.numberOfTaskSlots        int slot = flinkParameters.getSlot();        if (slot > 0) {            defalutOptions.add(String.format(FlinkConstants.FLINK_FORMAT_TASKMANAGER_NUMBEROFTASKSLOTS, slot));        }  
        // yarn.application.name        String appName = flinkParameters.getAppName();        if (StringUtils.isNotEmpty(appName)) {            defalutOptions.add(String.format(FlinkConstants.FLINK_FORMAT_YARN_APPLICATION_NAME, ArgsUtils.escape(appName)));        }  
        // jobmanager.memory.process.size        String jobManagerMemory = flinkParameters.getJobManagerMemory();        if (StringUtils.isNotEmpty(jobManagerMemory)) {            defalutOptions.add(String.format(FlinkConstants.FLINK_FORMAT_JOBMANAGER_MEMORY_PROCESS_SIZE, jobManagerMemory));        }  
        // taskmanager.memory.process.size        String taskManagerMemory = flinkParameters.getTaskManagerMemory();        if (StringUtils.isNotEmpty(taskManagerMemory)) {            defalutOptions.add(String.format(FlinkConstants.FLINK_FORMAT_TASKMANAGER_MEMORY_PROCESS_SIZE, taskManagerMemory));        }  
        // yarn.application.queue        String others = flinkParameters.getOthers();        if (StringUtils.isEmpty(others) || !others.contains(FlinkConstants.FLINK_QUEUE)) {            String queue = flinkParameters.getQueue();            if (StringUtils.isNotEmpty(queue)) {                defalutOptions.add(String.format(FlinkConstants.FLINK_FORMAT_YARN_APPLICATION_QUEUE, queue));            }        }    }  
    private String generateInitScriptFile(String parameters) {        String initScriptFileName = String.format("%s/%s_init.sql", taskExecutionContext.getExecutePath(), taskExecutionContext.getTaskAppId());  
        File file = new File(initScriptFileName);        Path path = file.toPath();  
        if (!Files.exists(path)) {            Set<PosixFilePermission> perms = PosixFilePermissions.fromString(RWXR_XR_X);            FileAttribute<Set<PosixFilePermission>> attr = PosixFilePermissions.asFileAttribute(perms);            try {                if (OSUtils.isWindows()) {                    Files.createFile(path);                } else {                    if (!file.getParentFile().exists()) {                        file.getParentFile().mkdirs();                    }                    Files.createFile(path, attr);                }  
                // Flink sql common parameters are written to the script file                logger.info("common parameters : {}", parameters);                Files.write(path, parameters.getBytes(), StandardOpenOption.APPEND);  
                // Flink init script is written to the script file                if (StringUtils.isNotEmpty(flinkParameters.getInitScript())) {                    String script = flinkParameters.getInitScript().replaceAll("\\r\\n", "\n");                    flinkParameters.setInitScript(script);                    logger.info("init script : {}", flinkParameters.getInitScript());                    Files.write(path, flinkParameters.getInitScript().getBytes(), StandardOpenOption.APPEND);                }            } catch (IOException e) {                throw new RuntimeException("generate flink sql script error", e);            }        }        return initScriptFileName;    }  
    private String generateScriptFile() {        String scriptFileName = String.format("%s/%s_node.sql", taskExecutionContext.getExecutePath(), taskExecutionContext.getTaskAppId());  
        File file = new File(scriptFileName);        Path path = file.toPath();  
        if (!Files.exists(path)) {            String script = flinkParameters.getRawScript().replaceAll("\\r\\n", "\n");            flinkParameters.setRawScript(script);  
            logger.info("raw script : {}", flinkParameters.getRawScript());            logger.info("task execute path : {}", taskExecutionContext.getExecutePath());  
            Set<PosixFilePermission> perms = PosixFilePermissions.fromString(RWXR_XR_X);            FileAttribute<Set<PosixFilePermission>> attr = PosixFilePermissions.asFileAttribute(perms);            try {                if (OSUtils.isWindows()) {                    Files.createFile(path);                } else {                    if (!file.getParentFile().exists()) {                        file.getParentFile().mkdirs();                    }                    Files.createFile(path, attr);                }                // Flink sql raw script is written to the script file                Files.write(path, flinkParameters.getRawScript().getBytes(), StandardOpenOption.APPEND);            } catch (IOException e) {                throw new RuntimeException("generate flink sql script error", e);            }        }        return scriptFileName;    }  
    @Override    protected void setMainJarName() {        ResourceInfo mainJar = flinkParameters.getMainJar();        String resourceName = getResourceNameOfMainJar(mainJar);        mainJar.setRes(resourceName);        flinkParameters.setMainJar(mainJar);    }  
    @Override    public AbstractParameters getParameters() {        return flinkParameters;    }}
```

### Release note

* `GitHub`

https://github.com/apache/dolphinscheduler/releases/tag/3.0.0

* `下载`:

https://dolphinscheduler.apache.org/en-us/download/download.html

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！

[2022年全网首发|大数据专家级技能模型与学习指南(胜天半子篇)](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247510040&idx=1&sn=aa335f25965975731173916f012d56f4&chksm=fd3eee8dca49679b82f632048fb64d21fac01497d1a0fe33917edb01e194caf0f9a1930a55ce&scene=21#wechat_redirect)

[互联网最坏的时代可能真的来了](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247508317&idx=1&sn=0bcb7fb6997b42306994b890eaa0d47f&chksm=fd3ee7c8ca496ede347a2971002c6ea68dcfd70abeeb90b43c8b02a2a60ebc7cec3a87ef7d52&scene=21#wechat_redirect)

[我在B站读大学，大数据专业](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247507860&idx=1&sn=807ac5003762f29c127bc4071dcebe33&chksm=fd3e9901ca4910178cfc816043ea86c29f881f70cd943d252de9ae2e21cb7e8ba80bff162e1b&scene=21#wechat_redirect)

[我们在学习Flink的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247499604&idx=1&sn=d938dfb30d221774704982d2938b30c1&chksm=fd3eb9c1ca4930d76a391241333de461ca22d2aa27472eab3cffab1564872ae37f1b48fe2d3c&scene=21#wechat_redirect)

[193篇文章暴揍Flink，这个合集你需要关注一下](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504856&idx=2&sn=6f62e2a0c756ce56773eed253d2f41ee&chksm=fd3e954dca491c5b732b7e2aa46db32efbc3f690e528522098659bb3c6e78e1582ab4529b5dc&scene=21#wechat_redirect)

[Flink生产环境TOP难题与优化，阿里巴巴藏经阁YYDS](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504742&idx=1&sn=8765c198d8ad66219a7bcabb221e4a23&chksm=fd3e95f3ca491ce52d0724b9e4154a47af0f5ea349e1e184c8fbc65aa17c0000242aa9b58429&scene=21#wechat_redirect)

[Flink CDC我吃定了耶稣也留不住他！| Flink CDC线上问题小盘点](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)

[我们在学习Spark的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=1&sn=1f693e549f76622725b936041ff8896e&chksm=e9bff2fbdec87bedecbdeed4547d7d612c72fc8d4918614a26a4e31666cf0128fd57b537f347&scene=21#wechat_redirect)

[在所有Spark模块中，我愿称SparkSQL为最强！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504834&idx=1&sn=46ca1a3924b8fdb89ac1c2acb0319d6c&chksm=fd3e9557ca491c41d837b917639ea62007ea16d3c2cc6c0c02d1f8a8c6e44318f5183b787c46&scene=21#wechat_redirect)

[硬刚Hive | 4万字基础调优面试小总结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247502750&idx=1&sn=bd9a9173d060dc4e4ebd49c8efc6acfe&chksm=fd3e8d0bca49041dea84da93910e5efdc4935e520525c09887c986691377aeb48e5cf7fb5667&scene=21#wechat_redirect)

[数据治理方法论和实践小百科全书](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504382&idx=2&sn=550b78802acfe727e0e77cd9195f8784&chksm=fd3e976bca491e7db2b8b2446d231736df01bbf13653804d680ac4390597ec17fa466ad4ae83&scene=21#wechat_redirect)

[标签体系下的用户画像建设小指南](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503741&idx=1&sn=e5039be93123f2e337013756a818bfc3&chksm=fd3e89e8ca4900fe603b63c5722a6fb8a32bd63d6ba23e0028851948a71b877eb1f742d95087&scene=21#wechat_redirect)

[4万字长文 | ClickHouse基础&实践&调优全视角解析](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503675&idx=1&sn=3ee6af64d0126c78b48cad219308f81e&chksm=fd3e89aeca4900b8b8954e9569ee3c0877881fac8c792bfafc22e7e9d3e8524da8eb860d33d8&scene=21#wechat_redirect)

[【面试&个人成长】2021年过半，社招和校招的经验之谈](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=2&sn=57ecc77718f1b6f2f262d62e8318dcc9&chksm=e9bff2fbdec87bedb1986765bfae7dbabb9aece45b0b2af147bbad1ac5d79ebc934c64df40ea&scene=21#wechat_redirect)

[大数据方向另一个十年开启 |《硬刚系列》第一版完结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504478&idx=1&sn=14efef3868ba42bd044f9618745a7fdc&chksm=fd3e94cbca491dddb961b7b8b93105b2869c5bcf4c03f9f0dc83ad62be0dce4c124b52ed0ba3&scene=21#wechat_redirect)

[我写过的关于成长/面试/职场进阶的文章](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504410&idx=1&sn=7e81ab5a324395eb0f12397c40247ca1&chksm=fd3e948fca491d9946456acbd93b2d651ae4d7a7d0127a211981e08f50f377e60d1b7c0d7b3a&scene=21#wechat_redirect)

[当我们在学习Hive的时候在学习什么？「硬刚Hive续集」](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504783&idx=1&sn=72aed147a459368ed934a007a2df3bc9&chksm=fd3e951aca491c0cade212390011eca7d8a68951fee6aa2844b8f4d14bcbd5b3ed26305d69dc&scene=21#wechat_redirect)