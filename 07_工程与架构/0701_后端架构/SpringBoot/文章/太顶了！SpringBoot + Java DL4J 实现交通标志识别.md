---
title: 太顶了！SpringBoot + Java DL4J 实现交通标志识别
author: 方志朋
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxNjk4ODE4OQ==&mid=2247540791&idx=1&sn=f757685173e8e156e0c889875be5bfff&chksm=9a726bbe18c5c493d219191fb72dbb92ed22819be9e8fe4146bcb7742bb92630b8b32f19a550&scene=0&xtrack=1&key=daf9bdc5abc4e8d005e103dfa1478a08b830d166cdb11cf0c4ad015a6f3326a348babe1b20704d9d32043bb174b7f8cd22756e8cd1687f13383b4f01156555caa437a7351cb0bad22ef145752099cdaae022593d1446a99fd207679b3b4d889e86be7ff45d00e310def1e3eb9cc5ed3f5d5142a9f795bfde558f99eaa660a3fc&ascene=15&uin=MjAwNTI2NjEyMw%3D%3D&devicetype=iMac+Mac15%2C6+OSX+OSX+14.4+build(23E214)&version=13080712&nettype=WIFI&lang=zh_CN&session_us=gh_a2736e68a652&countrycode=CN&fontScale=100&exportkey=n_ChQIAhIQuN50tEWiDdjruUGraXtouRKPAgIE97dBBAEAAAAAAC5FKiSDASIAAAAOpnltbLcz9gKNyK89dVj0dVdYHsfQwcNxXTd52cdpDfs6m6Fxjg1R%2F4g1CKnUxTaaAaAHyvV%2BFYzgz%2B8HxFJYn4CXEfdZLoOb6JS0m0iJjr%2BwTQj4UgKgh0%2FU7g8vrvE%2Fr4XtsQgD8B81dmBuJkZXa9tiXRN5SkTPBmbheTCf0qfsiUrl2RkZ54ea9vta0KfL%2FKccY%2F%2FZHRQZ%2FtEI3DX5gVHfuokxAG42S8hObtfoCKRl2V%2F79bloz%2FdWFcZih0Ei2D5tvOUfpkS6Sa3SyuMBftWhQEpTSAWSzG8iKC7fMWYJW9cnxOEGqCKiSWuVu8aMehBDRCVCu%2FU%3D&acctmode=0&pass_ticket=u4Q%2BPhWdF4dsRHQm4A2HvhO%2Bp%2BbeflE47jEoVZ%2FvJHl92XHmekcqeppeNmzYun2Q&wx_header=0&fasttmpl_type=0&fasttmpl_fullversion=7512981-zh_CN-zip&fasttmpl_flag=1
---

在当今科技飞速发展的时代，自动驾驶技术成为了热门的研究领域。交通标志识别是自动驾驶系统中的关键环节之一，它能够帮助汽车准确地理解道路状况，遵守交通规则。

> 本文将介绍如何使用 Spring Boot 整合 Java Deeplearning4j 来构建一个交通标志识别系统。

## 一、技术概述

### 1. 神经网络选择

在这个交通标志识别系统中，我们选择使用卷积神经网络（Convolutional Neural Network，CNN）。CNN 在图像识别领域具有卓越的性能，主要原因如下：

* **局部连接：** CNN 中的神经元只与输入图像的局部区域相连，这使得网络能够捕捉图像中的局部特征，如边缘、纹理等。对于交通标志这种具有特定形状和颜色特征的对象，局部连接能够有效地提取关键信息。
* **权值共享：** CNN 中的滤波器在整个图像上共享权值，这大大减少了参数数量，降低了模型的复杂度，同时也提高了模型的泛化能力。
* **层次结构：** CNN 通常由多个卷积层、池化层和全连接层组成，这种层次结构能够逐步提取图像的高级特征，从而实现对复杂图像的准确识别。

### 2. 数据集格式

我们使用的交通标志数据集通常包含以下格式：

* **图像文件：** 数据集由大量的交通标志图像组成，图像格式可以是常见的 JPEG、PNG 等。每个图像文件代表一个交通标志。
* **标签文件：** 与图像文件相对应的标签文件，用于标识每个图像所代表的交通标志类别。标签可以是数字编码或文本描述。

以下是一个简单的数据集目录结构示例：

```
traffic_sign_dataset/  
├── images/  
│   ├── sign1.jpg  
│   ├── sign2.jpg  
│   ├──...  
├── labels/  
│   ├── sign1.txt  
│   ├── sign2.txt  
│   ├──...
```

在标签文件中，可以使用数字编码来表示不同的交通标志类别，例如：0 表示限速标志，1 表示禁止标志，2 表示指示标志等。

### 3. 技术栈

**Spring Boot：** 用于构建企业级应用程序的开源框架，它提供了快速开发、自动配置和易于部署的特性。

**Java Deeplearning4j：** 一个基于 Java 的深度学习库，支持多种神经网络架构，包括 CNN、循环神经网络（Recurrent Neural Network，RNN）等。它提供了高效的计算引擎和丰富的工具，方便开发者进行深度学习应用的开发。

## 二、Maven 依赖

在项目的 pom.xml 文件中，需要添加以下 Maven 依赖：

```
<dependency>  
    <groupId>org.deeplearning4j</groupId>  
    <artifactId>deeplearning4j-core</artifactId>  
    <version>1.0.0-beta7</version>  
</dependency>  
<dependency>  
    <groupId>org.deeplearning4j</groupId>  
    <artifactId>deeplearning4j-nn</artifactId>  
    <version>1.0.0-beta7</version>  
</dependency>  
<dependency>  
    <groupId>org.deeplearning4j</groupId>  
    <artifactId>deeplearning4j-ui</artifactId>  
    <version>1.0.0-beta7</version>  
</dependency>  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
</dependency>
```

这些依赖将引入 Deeplearning4j 和 Spring Boot 的相关库，以便我们在项目中使用它们进行交通标志识别。

## 三、代码示例

### 1. 数据加载与预处理

首先，我们需要加载交通标志数据集，并进行预处理。以下是一个示例代码：

```
import org.datavec.image.loader.NativeImageLoader;  
import org.deeplearning4j.datasets.iterator.impl.ListDataSetIterator;  
import org.nd4j.linalg.api.ndarray.INDArray;  
import org.nd4j.linalg.dataset.DataSet;  
import org.nd4j.linalg.dataset.api.preprocessor.DataNormalization;  
import org.nd4j.linalg.dataset.api.preprocessor.ImagePreProcessingScaler;  
  
import java.io.File;  
import java.util.ArrayList;  
import java.util.List;  
  
public class DataLoader {  
  
    public static ListDataSetIterator loadData(String dataDirectory) {  
        // 加载图像文件  
        File imageDirectory = new File(dataDirectory + "/images");  
        NativeImageLoader imageLoader = new NativeImageLoader(32, 32, 3);  
        List<INDArray> images = new ArrayList<>();  
        for (File imageFile : imageDirectory.listFiles()) {  
            INDArray image = imageLoader.asMatrix(imageFile);  
            images.add(image);  
        }  
  
        // 加载标签文件  
        File labelDirectory = new File(dataDirectory + "/labels");  
        List<Integer> labels = new ArrayList<>();  
        for (File labelFile : labelDirectory.listFiles()) {  
            // 假设标签文件中每行只有一个数字，表示标签类别  
            int label = Integer.parseInt(FileUtils.readFileToString(labelFile));  
            labels.add(label);  
        }  
  
        // 创建数据集  
        DataSet dataSet = new DataSet(images.toArray(new INDArray[0]), labels.stream().mapToDouble(i -> i).toArray());  
  
        // 数据归一化  
        DataNormalization scaler = new ImagePreProcessingScaler(0, 1);  
        scaler.fit(dataSet);  
        scaler.transform(dataSet);  
  
        return new ListDataSetIterator(dataSet, 32);  
    }  
}
```

在这个示例中，我们使用`NativeImageLoader`加载图像文件，并将其转换为INDArray格式。然后，我们读取标签文件，获取每个图像的标签类别。最后，我们创建一个DataSet对象，并使用`ImagePreProcessingScaler`进行数据归一化。

### 2. 模型构建与训练

接下来，我们构建一个卷积神经网络模型，并使用加载的数据进行训练。以下是一个示例代码：

```
import org.deeplearning4j.nn.conf.ConvolutionMode;  
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;  
import org.deeplearning4j.nn.conf.layers.ConvolutionLayer;  
import org.deeplearning4j.nn.conf.layers.DenseLayer;  
import org.deeplearning4j.nn.conf.layers.OutputLayer;  
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;  
import org.deeplearning4j.nn.weights.WeightInit;  
import org.nd4j.linalg.activations.Activation;  
import org.nd4j.linalg.lossfunctions.LossFunctions;  
  
public class TrafficSignRecognitionModel {  
  
    public static MultiLayerNetwork buildModel() {  
        NeuralNetConfiguration.Builder builder = new NeuralNetConfiguration.Builder()  
               .seed(12345)  
               .weightInit(WeightInit.XAVIER)  
               .updater(org.deeplearning4j.nn.weights.WeightInit.XAVIER)  
               .l2(0.0005)  
               .list();  
  
        // 添加卷积层  
        builder.layer(0, new ConvolutionLayer.Builder(5, 5)  
               .nIn(3)  
               .stride(1, 1)  
               .nOut(32)  
               .activation(Activation.RELU)  
               .convolutionMode(ConvolutionMode.Same)  
               .build());  
  
        // 添加池化层  
        builder.layer(1, new org.deeplearning4j.nn.conf.layers.SubsamplingLayer.Builder(org.deeplearning4j.nn.conf.layers.PoolingType.MAX)  
               .kernelSize(2, 2)  
               .stride(2, 2)  
               .build());  
  
        // 添加更多卷积层和池化层  
        builder.layer(2, new ConvolutionLayer.Builder(5, 5)  
               .nOut(64)  
               .activation(Activation.RELU)  
               .convolutionMode(ConvolutionMode.Same)  
               .build());  
        builder.layer(3, new org.deeplearning4j.nn.conf.layers.SubsamplingLayer.Builder(org.deeplearning4j.nn.conf.layers.PoolingType.MAX)  
               .kernelSize(2, 2)  
               .stride(2, 2)  
               .build());  
  
        // 添加全连接层  
        builder.layer(4, new DenseLayer.Builder()  
               .nOut(1024)  
               .activation(Activation.RELU)  
               .build());  
  
        // 添加输出层  
        builder.layer(5, new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)  
               .nOut(10) // 假设共有 10 种交通标志类别  
               .activation(Activation.SOFTMAX)  
               .build());  
  
        return new MultiLayerNetwork(builder.build());  
    }  
  
    public static void trainModel(MultiLayerNetwork model, ListDataSetIterator iterator) {  
        model.init();  
        for (int epoch = 0; epoch < 10; epoch++) {  
            model.fit(iterator);  
            iterator.reset();  
        }  
    }  
}
```

在这个示例中，我们使用`NeuralNetConfiguration.Builder`构建一个卷积神经网络模型。模型包含多个卷积层、池化层、全连接层和输出层。我们使用`WeightInit.XAVIER`初始化权重，并设置了一些超参数，如学习率、正则化系数等。

然后，我们使用`MultiLayerNetwork`的fit方法对模型进行训练。

### 3. 预测与结果展示

最后，我们可以使用训练好的模型对新的交通标志图像进行预测，并展示结果。以下是一个示例代码：

```
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;  
import org.nd4j.linalg.api.ndarray.INDArray;  
import org.nd4j.linalg.dataset.api.preprocessor.DataNormalization;  
import org.nd4j.linalg.dataset.api.preprocessor.ImagePreProcessingScaler;  
import org.nd4j.linalg.factory.Nd4j;  
  
import java.io.File;  
  
public class Prediction {  
  
    public static int predict(MultiLayerNetwork model, File imageFile) {  
        // 加载图像并进行预处理  
        NativeImageLoader imageLoader = new NativeImageLoader(32, 32, 3);  
        INDArray image = imageLoader.asMatrix(imageFile);  
        DataNormalization scaler = new ImagePreProcessingScaler(0, 1);  
        scaler.transform(image);  
  
        // 进行预测  
        INDArray output = model.output(image);  
        return Nd4j.argMax(output, 1).getInt(0);  
    }  
}
```

在这个示例中，我们使用`NativeImageLoader`加载新的交通标志图像，并进行数据归一化。然后，我们使用训练好的模型对图像进行预测，返回预测的标签类别。

## 四、单元测试

为了确保代码的正确性，我们可以编写一些单元测试。以下是一个测试数据加载和模型训练的示例：

```
import org.deeplearning4j.datasets.iterator.impl.ListDataSetIterator;  
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;  
import org.junit.jupiter.api.BeforeEach;  
import org.junit.jupiter.api.Test;  
  
import static org.junit.jupiter.api.Assertions.assertNotNull;  
  
public class TrafficSignRecognitionTest {  
  
    private MultiLayerNetwork model;  
  
    @BeforeEach  
    public void setup() {  
        model = TrafficSignRecognitionModel.buildModel();  
    }  
  
    @Test  
    public void testLoadData() {  
        String dataDirectory = "path/to/your/dataset";  
        ListDataSetIterator iterator = DataLoader.loadData(dataDirectory);  
        assertNotNull(iterator);  
    }  
  
    @Test  
    public void testTrainModel() {  
        String dataDirectory = "path/to/your/dataset";  
        ListDataSetIterator iterator = DataLoader.loadData(dataDirectory);  
        TrafficSignRecognitionModel.trainModel(model, iterator);  
        assertNotNull(model);  
    }  
}
```

在这个测试中，我们首先构建一个模型，然后测试数据加载和模型训练的方法。我们使用`assertNotNull`断言来确保数据加载和模型训练的结果不为空。

## 五、预期输出

当我们运行交通标志识别系统时，预期的输出是对输入的交通标志图像进行准确的分类。例如，如果输入一个限速标志的图像，系统应该输出对应的标签类别，如“限速标志”。

## 六、参考资料文献

> * Deeplearning4j 官方文档
> * Spring Boot 官方文档
> * 《深度学习》（Ian Goodfellow、Yoshua Bengio、Aaron Courville 著）

> 作者：月下独码
>
> 来源：blog.csdn.net/lilinhai548/article/details/142851333