---
title: 5分钟部署自己的Kafka可视化平台
author: 小小平头哥
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg4Mzg5OTA2Ng==&mid=2247484050&idx=1&sn=16f9d2f29f46dcc621a793cd8eef7539&chksm=cf412d47f836a451b0e8c7ef99c631cc4cd0f5af73e5b01145054f70d9659029f7d66f318900&mpshare=1&scene=24&srcid=0119HxmrACRPaXym59hummXW&sharer_shareinfo=28b1928a8d0d79e829a1220d2c3a2120&sharer_shareinfo_first=28b1928a8d0d79e829a1220d2c3a2120#rd
---

Kafka是目前使用最广泛的消息队列之一，但Kafka的调试、可视化却困扰了一部分开发人员，这里给大家推荐一个小而美的Kafka UI工具：UI for Apache Kafka。它是完全开源的，目前在github有7.6k star。

***点关注👇👇👇不迷路***

我们可以使用K8S实现“一键部署”，以下是部署所使用的yaml文件（请将KAFKA\_CLUSTERS\_0\_BOOTSTRAPSERVERS替换为自己Kafka环境的地址）：

```
apiVersion: apps/v1                # Kubernetes API版本kind: Deployment                   # 部署资源类型metadata:  name: kafka-ui-deployment         # Deployment的名称  #namespace: kafka-ui               # 命名空间   labels:    app: kafka-ui                   # 标签以标识此Deployment属于kafka-ui应用spec:  replicas: 1                       # 副本数为1  selector:    matchLabels:      app: kafka-ui                 # 选择与此标签匹配的Pod作为副本  template:    metadata:      labels:        app: kafka-ui               # 在Pod模板中设置与Deployment相同的标签    spec:      containers:      - name: kafka-ui               # 容器名称        image: provectuslabs/kafka-ui:latest  # 容器映像        env:                                  # 数组格式 可设置多组        - name: KAFKA_CLUSTERS_0_NAME         # Kafka集群0的名称变量          value: "K8 Kafka Cluster"           # 注意：请自定义名称        - name: KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS   #Kafka集群的服务器地址变量          value: kafka-1.kafka.svc.cluster.local:9092 #注意：需根据自身情况填写此Kafka访问地址        imagePullPolicy: Always        # 总是拉取最新的容器映像        resources:          requests:            memory: "256Mi"            # 容器请求的内存资源            cpu: "100m"                # 容器请求的CPU资源          limits:            memory: "1024Mi"           # 容器可使用的最大内存资源            cpu: "1000m"               # 容器可使用的最大CPU资源        ports:        - containerPort: 8080         # 容器暴露的端口---apiVersion: v1                      # Kubernetes API版本kind: Service                      # 服务资源类型metadata:  name: kafka-ui-service            # 服务的名称spec:  selector:    app: kafka-ui                   # 选择与此标签匹配的Pod作为服务的后端  type: NodePort                    # 服务类型为NodePort，允许从节点外部访问  ports:    - protocol: TCP                 # 端口协议为TCP      port: 8080                    # 服务监听的端口      targetPort: 8080              # 将流量转发到Pod的端口      nodePort: 31080               # 暴露给外部访问的端口
```

我们将以上内容定义为kafka-ui.yaml文件，接下来使用以下命令就可以完成部署：

```
kubectl apply -f kafka-ui.yaml
```

然后确认Pod状态正常：

最后我们就可以使用IP:31080的方式访问UI，如下图所示：

在Topics-Messages菜单下我们就可以看到Kafka中的消息，如下图所示：

参考内容：

1.https://docs.kafka-ui.provectus.io/

2.https://www.felpfe.com/2023/09/13/kafka-streamcraft-a-dive-into-liquid-data/

**欢迎大家私信批评、指正。**

**如果您觉得文章还不错，欢迎您点赞、分享、点击“在看”，更期待您的打赏****👇******