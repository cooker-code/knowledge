---
title: Spring Boot 集成 MinIO 与 KKFile 实现文件预览功能
author: 新生代码农
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkwNDY2Mjg3NA==&mid=2247485734&idx=2&sn=0f10fa3a924cda537d50efc97ead9413&chksm=c15618b9b646eef2bf05926984bac8252a99c6e96393edbf0dc874e325551b83d3151099436e&mpshare=1&scene=24&srcid=0919lj1R3i63cqC9rFnfHjPw&sharer_shareinfo=12f318ae8e4ef5713bd09301286b7c6e&sharer_shareinfo_first=12f318ae8e4ef5713bd09301286b7c6e#rd
---

*点击关注公众号，更多**资讯**及时推送↓*

在现代的 Web 应用中，文件预览功能是提升用户体验的重要部分，尤其是在文档管理系统中。本文将带你逐步实现如何在 Spring Boot 项目中集成 MinIO（一个对象存储系统）与 KKFileView（一个开源文件预览工具），以实现对各种类型文件的在线预览。

### **项目准备**

在开始之前，请确保你已经安装和配置好了以下工具：

* **Java 11+**
* **Spring Boot**
* **MinIO 服务器**
* **KKFileView**

### **第一步：搭建 MinIO 服务器**

首先，我们需要配置 MinIO 作为对象存储服务器。你可以在本地或服务器上运行 MinIO。

1. **下载 MinIO**  
   从 MinIO 官方网站 下载适用于你操作系统的版本。
2. **运行 MinIO 服务器**在安装完 MinIO 后，可以使用以下命令启动 MinIO：

```
minio server /data --console-address ":9001"
```

1. /data 是存储文件的路径，9001 是 MinIO 控制台的端口。
2. **访问 MinIO 控制台**通过浏览器访问 http://localhost:9001，并使用默认的 access key 和 secret key 登录。你可以在 MinIO 控制台中创建一个 bucket 用来存储文件。

### **第二步：集成 Spring Boot 与 MinIO**

在 Spring Boot 项目中，我们将使用 MinIO SDK 来上传和下载文件。

1. **添加依赖**在 pom.xml 中添加 MinIO SDK 的依赖：

```
<dependency>    <groupId>io.minio</groupId>    <artifactId>minio</artifactId>    <version>8.0.3</version></dependency>
```

2. **配置 MinIO**在 application.properties 中添加 MinIO 的配置信息：

```
minio.endpoint=http://localhost:9000minio.accessKey=minioadminminio.secretKey=minioadminminio.bucketName=files
```

3. **创建 MinIO Service**编写一个 MinIOService 类，用于文件的上传和下载：

```
@Servicepublic class MinioService {    @Value("${minio.endpoint}")    private String minioEndpoint;    @Value("${minio.accessKey}")    private String minioAccessKey;    @Value("${minio.secretKey}")    private String minioSecretKey;    @Value("${minio.bucketName}")    private String bucketName;  
    private MinioClient minioClient;  
    @PostConstruct    public void init() {        minioClient = MinioClient.builder()            .endpoint(minioEndpoint)            .credentials(minioAccessKey, minioSecretKey)            .build();    }  
    public String uploadFile(MultipartFile file) throws Exception {        String fileName = System.currentTimeMillis() + "-" + file.getOriginalFilename();        minioClient.putObject(            PutObjectArgs.builder()                .bucket(bucketName)                .object(fileName)                .stream(file.getInputStream(), file.getSize(), -1)                .contentType(file.getContentType())                .build()        );        return fileName;    }  
    public String getFileUrl(String fileName) {        return minioClient.getPresignedObjectUrl(            GetPresignedObjectUrlArgs.builder()                .bucket(bucketName)                .object(fileName)                .build()        );    }}
```

### **第三步：集成 KKFileView 实现文件预览**

KKFileView 支持多种格式的文件预览，例如 PDF、Word、Excel 等。

1. **下载并运行 KKFileView**从 KKFileView 官方仓库 下载源码或使用 Docker 安装：

```
docker run -d -p 8012:8012 --name kkfileview keking/kkfileview
```

2. **配置 KKFileView 服务**KKFileView 提供了一个 REST API，用于文件预览。我们可以将文件的 URL 传递给 KKFileView，实现在线预览。

在 application.properties 中配置 KKFileView 服务的地址：

```
kkfileview.server=http://localhost:8012
```

3. **实现预览接口**在 MinioService 中创建一个方法，用于生成文件的预览 URL：

```
public String getPreviewUrl(String fileName) {    String fileUrl = getFileUrl(fileName);    return kkFileViewServer + "/onlinePreview?url=" + URLEncoder.encode(fileUrl, StandardCharsets.UTF_8);}
```

4.**Controller 实现文件上传和预览**创建一个 FileController，实现文件上传和预览的接口：

```
@RestController@RequestMapping("/files")public class FileController {    @Autowired    private MinioService minioService;  
    @PostMapping("/upload")    public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {        try {            String fileName = minioService.uploadFile(file);            return ResponseEntity.ok(fileName);        } catch (Exception e) {            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(e.getMessage());        }    }  
    @GetMapping("/preview/{fileName}")    public ResponseEntity<String> previewFile(@PathVariable String fileName) {        String previewUrl = minioService.getPreviewUrl(fileName);        return ResponseEntity.ok(previewUrl);    }}
```

### **第四步：测试文件上传与预览**

1. **文件上传**  
   使用 Postman 或前端页面上传文件，访问 /files/upload 接口。
2. **文件预览**  
   访问 /files/preview/{fileName}，即可获取文件的预览链接，打开后即可在浏览器中预览文件。

### **总结**

通过本文的介绍，你已经成功地集成了 MinIO 作为文件存储系统，并结合 KKFileView 实现了文件的在线预览功能。这种组合非常适合文档管理系统和需要处理大量文件的企业级应用。你可以根据需要对 MinIO 和 KKFileView 进行进一步的优化和扩展。

```
```
```
获取方式：点“在看”，关注公众号并回复【Java、1024、AI、游戏】领取，更多内容陆续奉上。
```
```

```
PS：因公众号平台更改了推送规则，如果不想错过内容，记得读完点一下“在看”，加个“星标”，这样每次新文章推送才会第一时间出现在你的订阅列表里。

点“在看”支持码农呀，谢谢啦
```
```