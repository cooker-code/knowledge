---
title: 使用 Cursor 重构 Java 旧项目：手把手教学
author: 码农跃迁指南
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4MTE2NjUxMw==&mid=2247483711&idx=1&sn=720ea7bfbec00ed8556872625a0fe3d6&chksm=ce1df23a18e3697697acaf3b19294a6c199d0a5c03d1b1aef678e6f18b14f5a5cff7ff64eb23&mpshare=1&scene=24&srcid=1126XHNSyNAIKGQ8rnQjjIOY&sharer_shareinfo=f3a2fcda4616a75dc983fbe609f95bde&sharer_shareinfo_first=f3a2fcda4616a75dc983fbe609f95bde#rd
---

## **1. 引言**

Java 旧项目重构是许多开发者面临的挑战。随着业务需求的变化和技术的发展，旧代码可能存在以下问题：

* **代码耦合严重**

  （业务逻辑、数据访问、控制器混杂）
* **缺乏模块化**

  （单一巨大的类文件）
* **性能低下**

  （未优化的 SQL 查询、内存泄漏）
* **缺乏测试**

  （难以维护和扩展）
* **过时的依赖**

  （如旧版 Spring、Hibernate）

传统的重构方式需要大量手动修改，而 **Cursor** 作为一款 AI 辅助的代码编辑器，可以极大提升重构效率。本文将详细介绍如何使用 **Cursor** 对一个 **Java Spring Boot 旧项目** 进行重构，包括：

* **项目分析与规划**
* **代码结构优化（MVC、DDD）**
* **依赖升级（Spring Boot 2.x → 3.x）**
* **性能改进（JPA 优化、缓存）**
* **单元测试（JUnit 5 + Mockito）**
* **自动化重构技巧**

---

## **2. 准备工作**

### **2.1 安装 Cursor**

Cursor 是一款基于 VS Code 的 AI 辅助编辑器，支持 Java 重构。下载地址：  
🔗https://www.cursor.com/

### **2.2 示例项目介绍**

我们将重构一个 **旧版 Spring Boot 博客系统**，其结构如下：

```
old-blog/  
├── src/  
│   ├── main/  
│   │   ├── java/com/example/blog/  
│   │   │   ├── BlogController.java    
                 # 控制器，混杂业务逻辑  
│   │   │   ├── BlogRepository.java     
                 # 原始 JDBC 操作  
│   │   │   ├── BlogService.java        
                 # 部分业务逻辑  
│   │   │   └── Blog.java               
                 # 实体类，缺乏验证  
│   │   └── resources/  
│   │       ├── application.properties  
│   │       └── templates/               
              # Thymeleaf 模板  
│   └── test/                           
        # 无测试  
└── pom.xml                            
  # 依赖混乱，Spring Boot 1.5
```

**问题：**

* **耦合严重**

  ：`BlogController` 直接调用 `BlogRepository` 的 JDBC 代码。
* **缺乏层次**

  ：没有明确的 Service 层、DTO 层。
* **性能差**

  ：JDBC 原始 SQL，未使用 JPA/Hibernate。
* **无测试**

  ：难以保证重构后的正确性。

---

## **3. 使用 Cursor 进行重构**

### **3.1 第一步：项目分析**

在重构前，我们需要理解项目的当前状态。Cursor 可以帮助我们快速分析代码结构。

**操作：**

1. 打开 `BlogController.java`，选中整个文件，右键选择 **"Ask Cursor"**（或按 `Cmd+K` / `Ctrl+K`）。
2. 输入提示：

**"分析这段 Spring Boot 控制器代码的问题，并给出重构建议。"**

**Cursor 可能的回复：**

**问题：**

1. **耦合过高**

   ：控制器直接调用 `BlogRepository`

       的 JDBC 方法，违反了 **分层架构** 原则。

2. **缺乏 DTO**

   ：直接返回 `Blog` 实体，暴露了数据库结构。
3. **无输入验证**

   ：`@PostMapping` 没有对请求参数进行验证。
4. **异常处理缺失**

   ：数据库操作未处理 `SQLException`。
5. **代码重复**

   ：多个方法重复了相同的数据库查询逻辑。

**建议：**

1. **引入分层架构**

   ：

+ **Controller**

  （处理 HTTP 请求）
+ **Service**

  （业务逻辑）
+ **Repository**

  （数据访问，使用 **Spring Data JPA**）

2. **使用 DTO**

   ：创建 `BlogDTO` 隔离实体和 API 层。
3. **添加验证**

   ：使用 `@Valid` + `javax.validation`。
4. **统一异常处理**

   ：使用 `@ControllerAdvice`。
5. **升级依赖**

   ：迁移到 **Spring Boot 3.x** + **Jakarta EE 9**。

---

### **3.2 第二步：引入分层架构**

我们将把单一的 `BlogController` 拆分为 **Controller、Service、Repository** 三层。

#### **1. 创建**`BlogDTO`**（数据传输对象）**

**操作：**

1. 新建 `BlogDTO.java`，问 Cursor：

**"为**`Blog`**实体创建一个 DTO 类，包含**

`id`**、**`title`**、**`content`**、**`createdAt`**字段，**

**并添加**`@NotBlank`**验证。"**

**Cursor 生成的**`BlogDTO.java`**：**

```
package com.example.blog.dto;  
  
import jakarta.validation.constraints.NotBlank;  
import java.time.LocalDateTime;  
  
public class BlogDTO {  
    private Long id;  
  
    @NotBlank(message = "标题不能为空")  
    private String title;  
  
    @NotBlank(message = "内容不能为空")  
    private String content;  
  
    private LocalDateTime createdAt;  
  
    // Getters & Setters  
    public Long getId() { return id; }  
    public void setId(Long id)  
              { this.id = id; }  
    public String getTitle()   
              { return title; }  
    public void setTitle(String title)   
              { this.title = title; }  
    public String getContent()   
              { return content; }  
    public void setContent(String content)   
              { this.content = content; }  
    public LocalDateTime getCreatedAt()  
              { return createdAt; }  
    public void setCreatedAt(LocalDateTime createdAt)   
             { this.createdAt = createdAt; }  
}
```

#### **2. 重构**`BlogRepository`**（使用 Spring Data JPA）**

**操作：**

1. 打开 `BlogRepository.java`，选中 JDBC 代码，问 Cursor：

**"将这段 JDBC 代码重构为 Spring Data JPA 仓库，并添加基本的 CRUD 方法。"**

**Cursor 生成的**`BlogRepository.java`**：**

```
package com.example.blog.repository;  
  
import com.example.blog.model.Blog;  
import org.springframework.data.jpa.  
               repository.JpaRepository;  
import org.springframework.stereotype.Repository;  
  
@Repository  
public interface BlogRepository extends   
             JpaRepository<Blog, Long> {  
    // 自动继承 save(), findById(),   
    //findAll(), deleteById() 等方法  
    // 可添加自定义查询，如：  
    List<Blog> findByTitleContaining(String keyword);  
}
```

#### **3. 创建**`BlogService`**（业务逻辑层）**

**操作：**

1. 新建 `BlogService.java`，问 Cursor：

**"创建一个博客服务类，使用**`BlogRepository`**和**`BlogDTO`**，实现博客的增删改查逻辑。"**

**Cursor 生成的**`BlogService.java`**：**

```
package com.example.blog.service;  
  
import com.example.blog.dto.BlogDTO;  
import com.example.blog.model.Blog;  
import com.example.blog.repository  
             .BlogRepository;  
import org.springframework.beans.  
             factory.annotation.Autowired;  
import org.springframework.  
              stereotype.Service;  
import org.springframework.transaction.  
              annotation.Transactional;  
  
import java.time.LocalDateTime;  
import java.util.List;  
import java.util.stream.Collectors;  
  
@Service  
public class BlogService {  
  
    @Autowired  
    private BlogRepository blogRepository;  
  
    // DTO 转 Entity  
    private Blog toEntity(BlogDTO blogDTO) {  
        Blog blog = new Blog();  
        blog.setId(blogDTO.getId());  
        blog.setTitle(blogDTO.getTitle());  
        blog.setContent(blogDTO.getContent());  
        blog.setCreatedAt(  
             LocalDateTime.now());  
        return blog;  
    }  
  
    // Entity 转 DTO  
    private BlogDTO toDTO(Blog blog) {  
        BlogDTO blogDTO = new BlogDTO();  
        blogDTO.setId(blog.getId());  
        blogDTO.setTitle(blog.getTitle());  
        blogDTO.setContent(blog.getContent());  
        blogDTO.setCreatedAt(  
                blog.getCreatedAt());  
        return blogDTO;  
    }  
  
    // 获取所有博客  
    public List<BlogDTO> getAllBlogs() {  
        return blogRepository.  
               findAll().stream()  
                .map(this::toDTO)  
                .collect(Collectors.toList());  
    }  
  
    // 创建博客  
    @Transactional  
    public BlogDTO createBlog(BlogDTO blogDTO) {  
        Blog blog = toEntity(blogDTO);  
        blog = blogRepository.save(blog);  
        return toDTO(blog);  
    }  
  
    // 更新博客  
    @Transactional  
    public BlogDTO updateBlog(Long id  
                       , BlogDTO blogDTO) {  
        Blog blog = blogRepository.findById(id)  
                .orElseThrow(() ->   
                    new RuntimeException("博客不存在"));  
        blog.setTitle(blogDTO.getTitle());  
        blog.setContent(blogDTO.getContent());  
        blog = blogRepository.save(blog);  
        return toDTO(blog);  
    }  
  
    // 删除博客  
    @Transactional  
    public void deleteBlog(Long id) {  
        blogRepository.deleteById(id);  
    }  
}
```

#### **4. 重构**`BlogController`**（仅处理 HTTP 请求）**

**操作：**

1. 打开 `BlogController.java`，问 Cursor：

**"重构这个控制器，使用**`BlogService`**和**`BlogDTO`**，**

**并添加**`@Valid`**验证和全局异常处理。"**

**Cursor 生成的**`BlogController.java`**：**

```
package com.example.blog.controller;  
  
import com.example.blog.dto.BlogDTO;  
import com.example.blog.service.BlogService;  
import jakarta.validation.Valid;  
import org.springframework.beans  
              .factory.annotation.Autowired;  
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind.annotation.*;  
  
import java.util.List;  
  
@RestController  
@RequestMapping("/api/blogs")  
public class BlogController {  
  
    @Autowired  
    private BlogService blogService;  
  
    // 获取所有博客  
    @GetMapping  
    public ResponseEntity<List<BlogDTO>>  
                  getAllBlogs() {  
        return ResponseEntity.ok(  
                 blogService.getAllBlogs());  
    }  
  
    // 创建博客  
    @PostMapping  
    public ResponseEntity<BlogDTO>   
        createBlog(@Valid   
            @RequestBody BlogDTO blogDTO) {  
        return ResponseEntity.ok(  
            blogService.createBlog(blogDTO));  
    }  
  
    // 更新博客  
    @PutMapping("/{id}")  
    public ResponseEntity<BlogDTO> updateBlog(  
            @PathVariable Long id,  
            @Valid @RequestBody   
                BlogDTO blogDTO) {  
        return ResponseEntity.ok(  
            blogService.updateBlog(id, blogDTO));  
    }  
  
    // 删除博客  
    @DeleteMapping("/{id}")  
    public ResponseEntity<Void> deleteBlog  
                 (@PathVariable Long id) {  
        blogService.deleteBlog(id);  
        return ResponseEntity.noContent().build();  
    }  
}
```

---

### **3.3 第三步：添加全局异常处理**

**操作：**

1. 新建 `GlobalExceptionHandler.java`，问 Cursor：

**"创建一个全局异常处理器，处理**

`MethodArgumentNotValidException`

**和**`RuntimeException`**，返回统一的错误响应。"**

**Cursor 生成的**`GlobalExceptionHandler.java`**：**

```
package com.example.blog.exception;  
  
import org.springframework.http.HttpStatus;  
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind  
              .MethodArgumentNotValidException;  
import org.springframework.web.bind  
              .annotation.ControllerAdvice;  
import org.springframework.web.bind  
              .annotation.ExceptionHandler;  
  
import java.util.HashMap;  
import java.util.Map;  
  
@ControllerAdvice  
public class GlobalExceptionHandler {  
  
    // 处理验证异常  
    @ExceptionHandler(  
        MethodArgumentNotValidException.class)  
    public ResponseEntity<Map<String, String>>   
       handleValidationExceptions(  
            MethodArgumentNotValidException ex) {  
        Map<String, String> errors =   
                       new HashMap<>();  
        ex.getBindingResult().getFieldErrors()  
              .forEach(error ->  
            errors.put(error.getField(),   
                error.getDefaultMessage()));  
        return ResponseEntity.  
                     badRequest().body(errors);  
    }  
  
    // 处理运行时异常  
    @ExceptionHandler(RuntimeException.class)  
    public ResponseEntity<String>   
        handleRuntimeException(  
            RuntimeException ex) {  
        return ResponseEntity.status(  
            HttpStatus.INTERNAL_SERVER_ERROR)  
                .body(ex.getMessage());  
    }  
}
```

---

### **3.4 第四步：升级依赖（Spring Boot 2.x → 3.x）**

**操作：**

1. 打开 `pom.xml`，问 Cursor：

**"将这个 Spring Boot 1.5 项目的依赖升级到**

**Spring Boot 3.x，并替换**`javax.persistence`

**为**`jakarta.persistence`**。"**

**Cursor 生成的**`pom.xml`**修改部分：**

```
<parent>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-parent</artifactId>  
    <version>3.1.0</version> <!-- 升级到 3.x -->  
</parent>  
<properties>  
    <java.version>17</java.version>  
  <!-- Spring Boot 3 需要 Java 17+ -->  
</properties>  
<dependencies>  
    <!-- 替换 javax.persistence 为 jakarta.persistence -->  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-data-jpa</artifactId>  
    </dependency>  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
    </dependency>  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-validation</artifactId>  
    </dependency>  
    <!-- 数据库驱动（例如 MySQL） -->  
    <dependency>  
        <groupId>com.mysql</groupId>  
        <artifactId>mysql-connector-j</artifactId>  
    </dependency>  
    <!-- 测试 -->  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-test</artifactId>  
        <scope>test</scope>  
    </dependency>  
</dependencies>
```

**注意：**

* Spring Boot 3.x 使用 **Jakarta EE 9**（包名从 `javax.*` 改为 `jakarta.*`）。
* 需要将 `import javax.persistence.*` 替换为 `import jakarta.persistence.*`（Cursor 可以批量替换）。

---

### **3.5 第五步：添加单元测试**

**操作：**

1. 在 `test/` 目录下创建 `BlogServiceTest.java`，问 Cursor：

**"为**`BlogService`**生成 JUnit 5 + Mockito 的单元测试，覆盖所有方法。"**

**Cursor 生成的**`BlogServiceTest.java`**：**

```
package com.example.blog.service;  
  
import com.example.blog.dto.BlogDTO;  
import com.example.blog.model.Blog;  
import com.example.blog.repository.BlogRepository;  
import org.junit.jupiter.api.Test;  
import org.junit.jupiter.api.extension.ExtendWith;  
import org.mockito.InjectMocks;  
import org.mockito.Mock;  
import org.mockito.junit.jupiter.MockitoExtension;  
  
import java.time.LocalDateTime;  
import java.util.Arrays;  
import java.util.List;  
import java.util.Optional;  
  
import static org.junit.jupiter.api.Assertions.*;  
import static org.mockito.ArgumentMatchers.any;  
import static org.mockito.Mockito.*;  
  
@ExtendWith(MockitoExtension.class)  
class BlogServiceTest {  
  
    @Mock  
    private BlogRepository blogRepository;  
  
    @InjectMocks  
    private BlogService blogService;  
  
    @Test  
    void testGetAllBlogs() {  
        // Mock 数据  
        Blog blog1 = new Blog(1L, "Title 1",   
                    "Content 1", LocalDateTime.now());  
        Blog blog2 = new Blog(2L, "Title 2",   
                    "Content 2", LocalDateTime.now());  
        when(blogRepository.findAll()).  
            thenReturn(Arrays.asList(blog1, blog2));  
  
        // 调用服务  
        List<BlogDTO> blogs = blogService.getAllBlogs();  
  
        // 验证  
        assertEquals(2, blogs.size());  
        assertEquals("Title 1", blogs.get(0).getTitle());  
    }  
  
    @Test  
    void testCreateBlog() {  
        BlogDTO blogDTO = new BlogDTO();  
        blogDTO.setTitle("New Blog");  
        blogDTO.setContent("Content");  
  
        Blog savedBlog = new Blog(1L, "New Blog",  
                        "Content", LocalDateTime.now());  
        when(blogRepository.save(any(Blog.class)))  
               .thenReturn(savedBlog);  
  
        BlogDTO result = blogService.createBlog(blogDTO);  
  
        assertNotNull(result.getId());  
        assertEquals("New Blog", result.getTitle());  
    }  
  
    @Test  
    void testUpdateBlog_Success() {  
        Long id = 1L;  
        BlogDTO blogDTO = new BlogDTO();  
        blogDTO.setTitle("Updated Title");  
        blogDTO.setContent("Updated Content");  
  
        Blog existingBlog = new Blog(id,   
            "Old Title", "Old Content", LocalDateTime.now());  
        when(blogRepository.findById(id))  
            .thenReturn(Optional.of(existingBlog));  
        when(blogRepository.save(any(Blog.class)))  
            .thenAnswer(i -> i.getArguments()[0]);  
  
        BlogDTO result = blogService.updateBlog(id, blogDTO);  
  
        assertEquals("Updated Title", result.getTitle());  
        verify(blogRepository, times(1)).save(existingBlog);  
    }  
  
    @Test  
    void testUpdateBlog_NotFound() {  
        Long id = 999L;  
        BlogDTO blogDTO = new BlogDTO();  
        blogDTO.setTitle("Updated Title");  
  
        when(blogRepository.findById(id))  
            .thenReturn(Optional.empty());  
  
        assertThrows(RuntimeException.class, () -> {  
            blogService.updateBlog(id, blogDTO);  
        });  
    }  
  
    @Test  
    void testDeleteBlog() {  
        Long id = 1L;  
        doNothing().when(blogRepository).deleteById(id);  
  
        blogService.deleteBlog(id);  
  
        verify(blogRepository, times(1)).deleteById(id);  
    }  
}
```

---

### **3.6 第六步：性能优化（JPA + 缓存）**

**操作：**

1. 打开 `Blog.java`，问 Cursor：

**"为这个实体添加 JPA 二级缓存（使用 EhCache）和**`@Index`**注解优化查询。"**

**Cursor 生成的**`Blog.java`**修改部分：**

```
package com.example.blog.model;  
  
import jakarta.persistence.*;  
import org.hibernate.annotations.Cache;  
import org.hibernate.annotations.CacheConcurrencyStrategy;  
import java.time.LocalDateTime;  
  
@Entity  
@Table(name = "blogs", indexes = {  
    @Index(name = "idx_title", columnList = "title") // 添加索引  
})  
@Cacheable  
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE) // 二级缓存  
public class Blog {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column(nullable = false)  
    private String title;  
  
    @Column(nullable = false, length = 5000)  
    private String content;  
  
    @Column(name = "created_at", nullable = false)  
    private LocalDateTime createdAt;  
  
    // Getters & Setters  
}
```

2. 在 `application.properties` 中配置缓存：

```
# 开启 JPA 二级缓存  
spring.jpa.properties.javax.  
     persistence.sharedCache.mode=ALL  
spring.jpa.properties.hibernate.  
     cache.use_second_level_cache=true  
spring.jpa.properties.hibernate.  
     cache.region.factory_class=jcache  
spring.jpa.properties.hibernate.  
     jcache.provider_class=org.ehcache  
     .jsr107.EhcacheCachingProvider
```

---

### **3.7 第七步：自动化重构**

Cursor 支持 **批量重构**，例如：

* **重命名类/方法**
* **提取公共方法**
* **统一代码风格**

#### **1. 批量重命名**

**操作：**

1. 选中 `BlogController` 中的

`blogRepo`（假设旧代码使用此名），问 Cursor：

**"在整个项目中将**`blogRepo`**重命名为**`blogRepository`**。"**

#### **2. 提取公共方法**

**操作：**

1. 选中 `BlogService` 中重复的 `toDTO` 逻辑，问 Cursor：

**"将这段 DTO 转换代码提取为一个名为**`BlogMapper`

**的单独类，并使用 MapStruct 自动生成映射。"**

**Cursor 生成的**`BlogMapper.java`**：**

```
package com.example.blog.mapper;  
  
import com.example.blog.dto.BlogDTO;  
import com.example.blog.model.Blog;  
import org.mapstruct.Mapper;  
import org.mapstruct.Mapping;  
  
@Mapper(componentModel = "spring")  
public interface BlogMapper {  
    BlogDTO toDTO(Blog blog);  
    Blog toEntity(BlogDTO blogDTO);  
}
```

然后在 `BlogService` 中注入 `BlogMapper`：

```
@Autowired  
private BlogMapper blogMapper;  
  
// 替换手动映射  
public BlogDTO toDTO(Blog blog) {  
    return blogMapper.toDTO(blog);  
}
```

---

## **4. 验证与部署**

### **4.1 本地测试**

1. 运行应用：

```
mvn spring-boot:run
```

2. 使用 Postman 或 curl 测试 API：

```
curl -X POST http://localhost:8080/api/blogs   
     -H "Content-Type: application/json"   
     -d '{"title":"Test","content":"Hello"}'
```

### **4.2 运行单元测试**

```
mvn test
```

### **4.3 性能对比**

* 使用 **Spring Boot Actuator** 监控 API 性能：

```
# application.properties  
management.endpoints.web.exposure.include=*  
management.endpoint.health.show-details=always
```

* 访问 `http://localhost:8080/actuator/metrics` 查看指标。

---

## **5. 总结**

通过 **Cursor**，我们成功重构了一个 **Java Spring Boot 旧项目**，实现了：  
✅**分层架构**（Controller、Service、Repository）  
✅**DTO 模式**（隔离实体和 API）  
✅**输入验证**（`@Valid` + `jakarta.validation`）  
✅**全局异常处理**（`@ControllerAdvice`）  
✅**依赖升级**（Spring Boot 3.x + Jakarta EE 9）  
✅**单元测试**（JUnit 5 + Mockito）  
✅**性能优化**（JPA 缓存、索引）  
✅**自动化重构**（MapStruct、批量重命名）

**Cursor 的优势：**

* **快速理解代码**

  ：AI 分析代码结构，给出重构建议。
* **自动生成代码**

  ：减少手动编写重复代码的工作。
* **批量重构**

  ：安全地重命名、提取、优化代码。
* **学习辅助**

  ：可以向 AI 询问 Java 最佳实践。

---

## **6. 进一步改进建议**

1. **微服务化**

   ：将博客系统拆分为 **用户服务、文章服务、评论服务**。
2. **API 文档**

   ：集成 **SpringDoc OpenAPI** 自动生成 Swagger 文档。
3. **容器化**

   ：添加 `Dockerfile` 和 `docker-compose.yml`。
4. **CI/CD**

   ：集成 **GitHub Actions** 或 **Jenkins** 自动化部署。
5. **前端现代化**

   ：使用 **React/Vue** 替换 Thymeleaf。

---

## **7. 常见问题解答**

**Q1：Cursor 重构 Java 代码会不会引入 Bug？**  
A：Cursor 的建议通常正确，但 **Java 是静态语言**，重构后需：

* 编译检查（`mvn compile`）
* 运行单元测试（`mvn test`）
* 手动验证关键功能

**Q2：如何处理大型遗留 Java 项目？**  
A：

1. **分阶段重构**

   ：

+ 第一阶段：引入分层架构（Controller/Service/Repository）。
+ 第二阶段：升级依赖（Spring Boot 3.x）。
+ 第三阶段：添加测试和缓存。

2. **逐步替换**

   ：保留旧接口，新功能用新架构实现。
3. **使用 Feature Flags**

   ：逐步切换新旧逻辑。

**Q3：Cursor 能否重构 Android/Kotlin 项目？**  
A：可以！Cursor 支持 **Kotlin、Android、JavaFX** 等。提示词可调整为：

**"将这个 Android Activity 重构为 MVVM 架构，使用 ViewModel 和 LiveData。"**

---

## **8. 结语**

使用 **Cursor** 进行 Java 旧项目重构，可以显著提升效率和代码质量。

通过 **AI 辅助分析、自动生成代码、批量重构**，我们能够：

* **降低重构风险**
* **提高代码可维护性**
* **加速项目现代化**

希望这篇教程能帮助你顺利完成 Java 旧项目的重构！ 🚀

---

**附录：Cursor 高效提示词模板（Java 版）**

|  |  |
| --- | --- |
| 场景 | 提示词示例 |
| **代码分析** | "分析这个 Spring Boot 控制器的问题，并给出重构为分层架构的建议。" |
| **JDBC → JPA** | "将这段 JDBC 代码重构为 Spring Data JPA 仓库，并添加自定义查询方法。" |
| **添加 DTO** | "为 `User` 实体创建一个 DTO 类，并使用 MapStruct 生成映射方法。" |
| **单元测试** | "为这个服务类生成 JUnit 5 + Mockito 的单元测试，覆盖成功和失败场景。" |
| **异常处理** | "创建一个全局异常处理器，处理 `MethodArgumentNotValidException` 和自定义业务异常。" |
| **依赖升级** | "将这个 Spring Boot 2.x 项目的 `pom.xml` 升级到 3.x，并替换 `javax` 为 `jakarta`。" |
| **性能优化** | "为这个 JPA 实体添加二级缓存（EhCache）和数据库索引。" |
| **批量重构** | "在整个项目中将 `findById` 方法的返回类型从 `Optional<User>` 改为 `User`，并处理空指针。" |

##