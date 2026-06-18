> 已吸收至：[[04_OLAP与数据库/0404_搜索与索引/040401_Elasticsearch/040401_核心知识点/Elasticsearch搜索平台与文件检索边界|Elasticsearch搜索平台与文件检索边界]]
---
title: SpringBoot + Minio + ElasticSearch实现文件内容检索
author: 泉城IT圈子
date:
url: http://mp.weixin.qq.com/s?__biz=MzA4MDc3MzE1NQ==&mid=2653997417&idx=1&sn=cf8b6f659ef271f471dff134f5f4cf54&chksm=85489fd4d14cd7dac9228b87ec41d004a8aca60555b1b38dd1077b1d2d3e12d4d8d6e25063d6&mpshare=1&scene=24&srcid=1105GVSUN6Qrx0rtRb3Y3V2M&sharer_shareinfo=e0ace41994e9723e3cbb23912e348794&sharer_shareinfo_first=e0ace41994e9723e3cbb23912e348794#rd
---

## 序言

之前在工作上，碰到一个需求就是读取文件内容存储到es中，然后利用es的特性的进行文件的检索，所以今天抽出一些时间来写这篇文章
项目已经推送到了我的GitHub：github.com/maoshengyzx…SpringBoot依赖版本为 `2.3.4.RELEASE`，Minio依赖版本为 `8.0.2`

## 1.下载Minio

Minio是一个高新能、分布式的对象存储服务器，其是开源免费的，官方地址：github.com/minio/minioMinio下载后会是一个minio.exe的文件，我们使用cmd命令在当前文件夹下打开，然后输入命令`minio.exe server D:\`，将 `D:\`替换为你实际想要存储Minio文件的存储目录，启动完成后，访问http://127.0.0.1:9000, 账号密码都为 **minioadmin**。

## 2.配置Minio

下载启动完成之后，我们需要再Java当中配置Minio的信息，这样才能使Java程序连接到Minio服务器

minio:

```
  endpoint: http://127.0.0.1:9000  # 替换为自己的服务地址
  accessKey: 3x2K4oPLqIzvrEL6w2ya  #  公钥
  secretKey: xPx2PYoNbbxRmEYXgJVUmsPr5zf8OycyiJQJWzKl  # 私钥
```

配置完成后，创建MinioProp类，用于读取Minio配置

```
@Data@Component@ConfigurationProperties(prefix = "minio")public class MinioProp {    // minio 连接地址    private String endpoint;
    // 公钥    private String accessKey;
    // 私钥    private String secretKey;}
```

然后创建Minio的配置类

```
@Configurationpublic class MinioConfig {
    private final MinioProp minioProp;
    public MinioConfig(MinioProp minioProp) {        this.minioProp = minioProp;    }
    /**     * minio客户端     *     * @return     */    @Bean    public MinioClient minioClient() {        return MinioClient.builder()                .endpoint(minioProp.getEndpoint())                .credentials(minioProp.getAccessKey(), minioProp.getSecretKey())                .build();    }}
```

```

```

最后封装Minio的工具类，用于简化代码操作

```
@Componentpublic class MinioUtils {
    private final MinioClient minioClient;
    public MinioUtils(MinioClient minioClient) {        this.minioClient = minioClient;    }
    /**     * 创建桶     *     * @param bucketName     */    @SneakyThrows    public void createBucket(String bucketName) {        boolean found =                minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucketName).build());        if (!found) {            minioClient.makeBucket(                    MakeBucketArgs.builder()                            .bucket(bucketName)                            .region("cn-beijing")                            .build());        }    }
    /**     * 删除桶     *     * @param bucketName 桶名称     */    @SneakyThrows    public void removeBucket(String bucketName) {        minioClient.removeBucket(RemoveBucketArgs.builder().bucket(bucketName).build());    }

    /**     * 上传文件     *     * @param bucketName 桶名称     * @param objectName 文件名     * @param stream     流     * @param fileSize   文件大小     * @param type       文件类型     * @throws Exception     */    public void putObject(String bucketName, String objectName, InputStream stream, Long fileSize, String type) throws Exception {        minioClient.putObject(                PutObjectArgs.builder().bucket(bucketName).object(objectName).stream(                                stream, fileSize, -1)                        .contentType(type)                        .build());    }

    /**     * 判断文件夹是否存在     *     * @param bucketName 桶名称     * @param prefix     文件夹名字     * @return     */    @SneakyThrows    public Boolean folderExists(String bucketName, String prefix) {        Iterable<Result<Item>> results = minioClient.listObjects(ListObjectsArgs.builder().bucket(bucketName)                .prefix(prefix).recursive(false).build());        for (Result<Item> result : results) {            Item item = result.get();            if (item.isDir()) {                return true;            }        }        return false;    }
    /**     * 创建文件夹     *     * @param bucketName 桶名称     * @param path       路径     */    @SneakyThrows    public void createFolder(String bucketName, String path) {        minioClient.putObject(PutObjectArgs.builder().bucket(bucketName).object(path)                .stream(new ByteArrayInputStream(new byte[]{}), 0, -1).build());    }
    /**     * 获取文件在minio在服务器上的外链     *     * @param bucketName 桶名称     * @param objectName 文件名     * @return     */    @SneakyThrows    public String getObjectUrl(String bucketName, String objectName) {        return minioClient.getPresignedObjectUrl(                GetPresignedObjectUrlArgs.builder()                        .method(Method.GET)                        .bucket(bucketName)                        .object(objectName)                        .build());    }
    /**     * 从minio下载文件为流     *     * @param bucketName  桶名称     * @param objectName  文件名称     * @return     */    @SneakyThrows    public InputStream downloadObjectAsStream(String bucketName, String objectName) {        GetObjectArgs objectArgs = GetObjectArgs.builder().bucket(bucketName).object(objectName).build();        return minioClient.getObject(objectArgs);    }
}
```

```

```

到此Minio的准备工作就已经初步完成了，我们现在开始ElasticSearch的配置

## 3.ES配置

方便大家查看，我这里将YML文件展示出来

```
spring:  application:    name: SpringBoot-ElasticSearch  datasource:    driver-class-name: com.mysql.cj.jdbc.Driver    username: root    password: root    url: jdbc:mysql:///db7?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai  data:    elasticsearch:      client:        reactive:          endpoints: 127.0.0.1:9200  servlet:    multipart:      max-file-size: 20MB      max-request-size: 20MB
# MyBatis配置mybatis:  # 搜索指定包别名  typeAliasesPackage: com.ruoyi.**.domain  # 配置mapper的扫描，找到所有的mapper.xml映射文件  mapperLocations: classpath*:mapper/**/*Mapper.xml

server:  port: 8080
minio:  endpoint: http://127.0.0.1:9000  accessKey: 3x2K4oPLqIzvrEL6w2ya  secretKey: xPx2PYoNbbxRmEYXgJVUmsPr5zf8OycyiJQJWzKl
```

```

```

ES导入依赖和配置文件后，就已经可以使用了，具体的API大家可以参考官方文档 docs.spring.io/spring-data…接下来，我们就需要来配置ES中索引、文档等信息了，好在Spring为这些信息的创建提供了简介的方式--只需要在类和字段上添加注解就可以了

```
@Data@Document(indexName = "file_table")public class FileTable implements Serializable {    private static final long serialVersionUID = -84031829975035928L;    /**     * 文件表主键，自增 ID     */    @Id    private Long id;    /**     * 文件名     * text  可分词，不可以聚合     */    @Field(name = "fileName", type = FieldType.Text, analyzer = "ik_smart", searchAnalyzer = "ik_smart")    private String fileName;
    /**     * 文件内容     */    @Field(name = "fileContent", type = FieldType.Text, analyzer = "ik_max_word", searchAnalyzer = "ik_max_word")    private String fileContent;
    /**     * 文件类型     * keyword  不可分词  但是可以聚合     */    @Field(name = "fileType", type = FieldType.Keyword)    private String fileType;    /**     * 文件大小     */    @Field(name = "fileSize", type = FieldType.Long)    private Long fileSize;    /**     * 文件路径     */    @Field(name = "filePath", type = FieldType.Text, index = false)    private String filePath;    /**     * 逻辑删除标志，0 未删除  1 已删除     */    private Long isDeleted;}
```

```

```

实体类创建完毕后，我们需要创键`FileTableRepository`接口来继承`PagingAndSortingRepository`(由Spring提供的一个和ES交互操作的接口，提供了基本的CRUD操作)

```
public interface FileTableRepository extends PagingAndSortingRepository<FileTable,Long> {
       // 可以自定义方法(无需提供实现)，但是一般用不到
}
```

## 4.代码

以上操作都完成后，就已经可以开发代码了，FileTable的控制层代码如下：

```
@RestController@RequestMapping("/file")public class FileController {

    private final FileTableService fileTableService;
    public FileController(MinioUtils minioUtils, FileTableService fileTableService) {        this.fileTableService = fileTableService;    }
           /**     * 文件上传     * @param file  文件     * @param bucketName  桶名称     * @return     */    @GetMapping("/uploadFile")    public String uploadFile(@RequestParam("file") MultipartFile file, String bucketName) {        fileTableService.uploadFile(file, bucketName);        return "文件上传成功";    }
    /**     * 简单测试一个查询高亮     *     * @param id     * @return     */    @GetMapping("/getInfoHighlight")    public List<FileTable> getInfoHighlight(Long id) {        return fileTableService.getInfoHighlight(id);    }
}
```

```

```

sevice实现如下：

```
@Service("fileTableService")@AllArgsConstructorpublic class FileTableServiceImpl implements FileTableService {
    private final FileTableMapper fileTableMapper;
    private final FileTableRepository fileTableRepository;
    private final ElasticsearchOperations elasticsearchOperations;
    private final MinioUtils minioUtils;
    /**     * 实例化完成后创建索引     */    @PostConstruct    public void createIndex() {        IndexOperations operations = elasticsearchOperations.indexOps(FileTable.class);        if (!operations.exists()) {            operations.create();        }        Document document = operations.createMapping();        operations.putMapping(document);    }

    /**     * 新增数据     *     * @param fileTable 实例对象     * @return 实例对象     */    @Override    public FileTable insert(FileTable fileTable) {        this.fileTableMapper.insert(fileTable);        return fileTable;    }

    /**     * 获取file的文件内容，上次到es中来做检索     *     * @param file       文件对象     * @param bucketName 桶名称     */    @Override    @SneakyThrows    public void uploadFile(MultipartFile file, String bucketName) {        minioUtils.createBucket(bucketName);        // 生产服务器文件名        String objectName = bucketName + UUID.randomUUID().getLeastSignificantBits() + "_" + file.getOriginalFilename();        Assert.notNull(file.getOriginalFilename(), "文件名不能为空");        String fileType = file.getOriginalFilename().split("\.")[1];        minioUtils.putObject(bucketName, objectName, file.getInputStream(), file.getSize(), fileType);
        // 插入数据库数据        String fileUrl = minioUtils.getObjectUrl(bucketName, objectName);        FileTable fileTable = new FileTable();        fileTable.setFileName(objectName);        fileTable.setFilePath(fileUrl);        fileTable.setIsDeleted(0L);        fileTable.setFileType(fileType);        fileTable.setFileSize(file.getSize());        fileTableMapper.insert(fileTable);
        // 读取文件内容，上传到es，方便后续的检索  可以考虑使用消息队列，提高效率  因为读取文件内容比较耗时        // 这里为了演示，直接读取文件内容，上传到es        String fileContent = FileUtils.readFileContent(file.getInputStream(), fileType);        fileTable.setFileContent(fileContent);        fileTableRepository.save(fileTable);    }
    @Override    public List<FileTable> getInfoHighlight(Long id) {        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();        queryBuilder.withQuery(QueryBuilders.multiMatchQuery("手册", "fileName", "fileContent"));        queryBuilder.withQuery(QueryBuilders.termQuery("id", id));        // 设置高亮        HighlightBuilder highlightBuilder = new HighlightBuilder();        String[] fieldNames = {"fileName", "fileContent"};        for (String fieldName : fieldNames) {            highlightBuilder.field(fieldName);        }        highlightBuilder.preTags("<em>");        highlightBuilder.postTags("</em>");        highlightBuilder.order();        queryBuilder.withHighlightBuilder(highlightBuilder);//        queryBuilder.withHighlightFields(new HighlightBuilder.Field("fileName"));
        // 也可以添加分页和排序        SortBuilder<FieldSortBuilder> sortBuilder = new FieldSortBuilder("fileSize").order(SortOrder.DESC);        queryBuilder.withSort(sortBuilder).withPageable(PageRequest.of(0, 10)); // 表示第一页，每页10条
        NativeSearchQuery nativeSearchQuery = queryBuilder.build();
        SearchHits<FileTable> searchHits = elasticsearchOperations.search(nativeSearchQuery, FileTable.class);
        searchHits.forEach(item -> {            FileTable fileTable = item.getContent();            System.out.println("Highlighted FileName: " + item.getHighlightFields().get("fileName"));            System.out.println("Highlighted FileContent: " + item.getHighlightFields().get("fileContent"));        });
        ArrayList<FileTable> fileTables = new ArrayList<>();
        searchHits.forEach(item -> {            fileTables.add(item.getContent());        });
        return fileTables;    }}
```

```

```

FileUtils工具类代码，用于读取文件的内容

```
public class FileUtils {
    private static final List<String> FILE_TYPE;

    static {        FILE_TYPE = Arrays.asList("pdf", "doc", "docx", "text");    }

    @SneakyThrows    public static String readFileContent(InputStream inputStream, String fileType) {        if (!FILE_TYPE.contains(fileType)) {            return null;        }        // 使用PdfBox读取pdf文件内容        if ("pdf".equalsIgnoreCase(fileType)) {            return readPdfContent(inputStream);        } else if ("doc".equalsIgnoreCase(fileType) || "docx".equalsIgnoreCase(fileType)) {            return readDocOrDocxContent(inputStream);        } else if ("tex".equalsIgnoreCase(fileType)) {            return readTextContent(inputStream);        }
        return null;    }

    @SneakyThrows    private static String readPdfContent(InputStream inputStream) {        // 加载PDF文档        PDDocument pdDocument = PDDocument.load(inputStream);
        // 创建PDFTextStripper对象, 提取文本        PDFTextStripper textStripper = new PDFTextStripper();
        // 提取文本        String content = textStripper.getText(pdDocument);
        // 关闭PDF文档        pdDocument.close();        return content;    }

    private static String readDocOrDocxContent(InputStream inputStream) {        try {            // 加载DOC文档            XWPFDocument document = new XWPFDocument(inputStream);
            // 2. 提取文本内容            XWPFWordExtractor extractor = new XWPFWordExtractor(document);            return extractor.getText();        } catch (IOException e) {            e.printStackTrace();            return null;        }    }

    private static String readTextContent(InputStream inputStream) {        StringBuilder content = new StringBuilder();        try (InputStreamReader isr = new InputStreamReader(inputStream, StandardCharsets.UTF_8)) {            int ch;            while ((ch = isr.read()) != -1) {                content.append((char) ch);            }        } catch (IOException e) {            e.printStackTrace();            return null;        }        return content.toString();    }
}
```

作者：发愤图强的羔羊
链接：https://juejin.cn/post/7432942189849837594