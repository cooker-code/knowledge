---
title: SpringBoot的3种六边形架构应用方式
author: 风象南
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU3NTgwOTE4NQ==&mid=2247484387&idx=1&sn=ff69e82f8375a5c674a95c5f924ed0d1&chksm=fc266227fac878c8951e313cff40816fcfe772e18a2a3bf197b1710b9490c041e1e28e98250c&mpshare=1&scene=24&srcid=0807BwdJHF7iHG1dJN6FIa6a&sharer_shareinfo=cc5141e2e2b6edc308f9cf3ef3b8448f&sharer_shareinfo_first=cc5141e2e2b6edc308f9cf3ef3b8448f#rd
---

六边形架构，也被称为端口与适配器架构或洋葱架构，是一种将业务逻辑与外部依赖解耦的架构模式。

本文将介绍在SpringBoot中实现六边形架构的三种不同方式。

# 一、六边形架构基本原理

### 1.1 核心概念

六边形架构由Alistair Cockburn于2005年提出，其核心思想是将应用程序的内部业务逻辑与外部交互隔离开来。这种架构主要由三部分组成：

**领域（Domain）** ：包含业务逻辑和领域模型，是应用程序的核心  
**端口（Ports）** ：定义应用程序与外部世界交互的接口  
**适配器（Adapters）** ：实现端口接口，连接外部世界与应用程序

### 1.2 端口分类

端口通常分为两类：

**输入端口（Primary/Driving Ports）** ：允许外部系统驱动应用程序，如REST API、命令行接口  
**输出端口（Secondary/Driven Ports）** ：允许应用程序驱动外部系统，如数据库、消息队列、第三方服务

### 1.3 六边形架构的优势

**业务逻辑独立性**：核心业务逻辑不依赖于特定技术框架  
**可测试性**：业务逻辑可以在没有外部依赖的情况下进行测试  
**灵活性**：可以轻松替换技术实现而不影响业务逻辑  
**关注点分离**：明确区分了业务规则和技术细节  
**可维护性**：使代码结构更加清晰，便于维护和扩展

# 二、经典六边形架构实现

### 2.1 项目结构

经典六边形架构严格遵循原始设计理念，通过明确的包结构分离领域逻辑和适配器：

```
src/main/java/com/example/demo/  
├── domain/                  # 领域层  
│   ├── model/               # 领域模型  
│   ├── service/             # 领域服务  
│   └── port/                # 端口定义  
│       ├── incoming/        # 输入端口  
│       └── outgoing/        # 输出端口  
├── adapter/                 # 适配器层  
│   ├── incoming/            # 输入适配器  
│   │   ├── rest/            # REST API适配器  
│   │   └── scheduler/       # 定时任务适配器  
│   └── outgoing/            # 输出适配器  
│       ├── persistence/     # 持久化适配器  
│       └── messaging/       # 消息适配器  
└── application/             # 应用配置  
    └── config/              # Spring配置类
```

### 2.2 代码实现

#### 2.2.1 领域模型

```
// 领域模型  
package com.example.demo.domain.model;  
  
public class Product {  
    private Long id;  
    private String name;  
    private double price;  
    private int stock;  
      
    // 构造函数、getter和setter  
      
    // 领域行为  
    public boolean isAvailable() {  
        return stock > 0;  
    }  
      
    public void decreaseStock(int quantity) {  
        if (quantity > stock) {  
            throw new IllegalArgumentException("Not enough stock");  
        }  
        this.stock -= quantity;  
    }  
}
```

#### 2.2.2 输入端口

```
// 输入端口（用例接口）  
package com.example.demo.domain.port.incoming;  
  
import com.example.demo.domain.model.Product;  
import java.util.List;  
import java.util.Optional;  
  
public interface ProductService {  
    List<Product> getAllProducts();  
    Optional<Product> getProductById(Long id);  
    Product createProduct(Product product);  
    void updateStock(Long productId, int quantity);  
}
```

#### 2.2.3 输出端口

```
// 输出端口（存储库接口）  
package com.example.demo.domain.port.outgoing;  
  
import com.example.demo.domain.model.Product;  
import java.util.List;  
import java.util.Optional;  
  
public interface ProductRepository {  
    List<Product> findAll();  
    Optional<Product> findById(Long id);  
    Product save(Product product);  
    void deleteById(Long id);  
}
```

#### 2.2.4 领域服务实现

```
// 领域服务实现  
package com.example.demo.domain.service;  
  
import com.example.demo.domain.model.Product;  
import com.example.demo.domain.port.incoming.ProductService;  
import com.example.demo.domain.port.outgoing.ProductRepository;  
import org.springframework.stereotype.Service;  
import org.springframework.transaction.annotation.Transactional;  
  
import java.util.List;  
import java.util.Optional;  
  
@Service  
public class ProductServiceImpl implements ProductService {  
      
    private final ProductRepository productRepository;  
      
    public ProductServiceImpl(ProductRepository productRepository) {  
        this.productRepository = productRepository;  
    }  
      
    @Override  
    public List<Product> getAllProducts() {  
        return productRepository.findAll();  
    }  
      
    @Override  
    public Optional<Product> getProductById(Long id) {  
        return productRepository.findById(id);  
    }  
      
    @Override  
    public Product createProduct(Product product) {  
        return productRepository.save(product);  
    }  
      
    @Override  
    @Transactional  
    public void updateStock(Long productId, int quantity) {  
        Product product = productRepository.findById(productId)  
            .orElseThrow(() -> new RuntimeException("Product not found"));  
          
        product.decreaseStock(quantity);  
        productRepository.save(product);  
    }  
}
```

#### 2.2.5 输入适配器（REST API）

```
// REST适配器  
package com.example.demo.adapter.incoming.rest;  
  
import com.example.demo.domain.model.Product;  
import com.example.demo.domain.port.incoming.ProductService;  
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind.annotation.*;  
  
import java.util.List;  
  
@RestController  
@RequestMapping("/api/products")  
public class ProductController {  
      
    private final ProductService productService;  
      
    public ProductController(ProductService productService) {  
        this.productService = productService;  
    }  
      
    @GetMapping  
    public List<Product> getAllProducts() {  
        return productService.getAllProducts();  
    }  
      
    @GetMapping("/{id}")  
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {  
        return productService.getProductById(id)  
            .map(ResponseEntity::ok)  
            .orElse(ResponseEntity.notFound().build());  
    }  
      
    @PostMapping  
    public Product createProduct(@RequestBody Product product) {  
        return productService.createProduct(product);  
    }  
      
    @PutMapping("/{id}/stock")  
    public ResponseEntity<Void> updateStock(  
            @PathVariable Long id,  
            @RequestParam int quantity) {  
        productService.updateStock(id, quantity);  
        return ResponseEntity.ok().build();  
    }  
}
```

#### 2.2.6 输出适配器（持久化）

```
// JPA实体  
package com.example.demo.adapter.outgoing.persistence.entity;  
  
import javax.persistence.*;  
  
@Entity  
@Table(name = "products")  
public class ProductEntity {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
    private String name;  
    private double price;  
    private int stock;  
      
    // 构造函数、getter和setter  
}  
  
// JPA仓库  
package com.example.demo.adapter.outgoing.persistence.repository;  
  
import com.example.demo.adapter.outgoing.persistence.entity.ProductEntity;  
import org.springframework.data.jpa.repository.JpaRepository;  
  
public interface JpaProductRepository extends JpaRepository<ProductEntity, Long> {  
}  
  
// 持久化适配器  
package com.example.demo.adapter.outgoing.persistence;  
  
import com.example.demo.adapter.outgoing.persistence.entity.ProductEntity;  
import com.example.demo.adapter.outgoing.persistence.repository.JpaProductRepository;  
import com.example.demo.domain.model.Product;  
import com.example.demo.domain.port.outgoing.ProductRepository;  
import org.springframework.stereotype.Component;  
  
import java.util.List;  
import java.util.Optional;  
import java.util.stream.Collectors;  
  
@Component  
public class ProductRepositoryAdapter implements ProductRepository {  
      
    private final JpaProductRepository jpaRepository;  
      
    public ProductRepositoryAdapter(JpaProductRepository jpaRepository) {  
        this.jpaRepository = jpaRepository;  
    }  
      
    @Override  
    public List<Product> findAll() {  
        return jpaRepository.findAll().stream()  
            .map(this::mapToDomain)  
            .collect(Collectors.toList());  
    }  
      
    @Override  
    public Optional<Product> findById(Long id) {  
        return jpaRepository.findById(id)  
            .map(this::mapToDomain);  
    }  
      
    @Override  
    public Product save(Product product) {  
        ProductEntity entity = mapToEntity(product);  
        ProductEntity savedEntity = jpaRepository.save(entity);  
        return mapToDomain(savedEntity);  
    }  
      
    @Override  
    public void deleteById(Long id) {  
        jpaRepository.deleteById(id);  
    }  
      
    private Product mapToDomain(ProductEntity entity) {  
        Product product = new Product();  
        product.setId(entity.getId());  
        product.setName(entity.getName());  
        product.setPrice(entity.getPrice());  
        product.setStock(entity.getStock());  
        return product;  
    }  
      
    private ProductEntity mapToEntity(Product product) {  
        ProductEntity entity = new ProductEntity();  
        entity.setId(product.getId());  
        entity.setName(product.getName());  
        entity.setPrice(product.getPrice());  
        entity.setStock(product.getStock());  
        return entity;  
    }  
}
```

### 2.3 优缺点分析

**优点：**

* • 结构清晰，严格遵循六边形架构原则
* • 领域模型完全独立，不受技术框架影响
* • 适配器隔离了所有外部依赖
* • 高度可测试，可以轻松模拟任何外部组件

**缺点：**

* • 代码量较大，需要编写更多的接口和适配器
* • 对象映射工作增加，需要在适配器中转换领域对象和持久化对象
* • 可能感觉过度设计，特别是对简单应用程序
* • 学习曲线较陡峭，团队需要深入理解六边形架构

### 2.4 适用场景

* • 复杂的业务领域，需要清晰隔离业务规则
* • 长期维护的核心系统
* • 团队已熟悉六边形架构原则
* • 需要灵活替换技术实现的场景

# 三、基于DDD的六边形架构

### 3.1 项目结构

基于DDD的六边形架构结合了领域驱动设计的概念，进一步丰富了领域层：

```
src/main/java/com/example/demo/  
├── domain/                  # 领域层  
│   ├── model/               # 领域模型  
│   │   ├── aggregate/       # 聚合  
│   │   ├── entity/          # 实体  
│   │   └── valueobject/     # 值对象  
│   ├── service/             # 领域服务  
│   └── repository/          # 仓储接口  
├── application/             # 应用层  
│   ├── port/                # 应用服务接口  
│   │   ├── incoming/        # 输入端口  
│   │   └── outgoing/        # 输出端口  
│   └── service/             # 应用服务实现  
├── infrastructure/          # 基础设施层  
│   ├── adapter/             # 适配器  
│   │   ├── incoming/        # 输入适配器  
│   │   └── outgoing/        # 输出适配器  
│   └── config/              # 配置类  
└── interface/               # 接口层  
    ├── rest/                # REST接口  
    ├── graphql/             # GraphQL接口  
    └── scheduler/           # 定时任务
```

### 3.2 代码实现

#### 3.2.1 领域模型

```
// 值对象  
package com.example.demo.domain.model.valueobject;  
  
public class Money {  
    private final BigDecimal amount;  
    private final String currency;  
      
    public Money(BigDecimal amount, String currency) {  
        this.amount = amount;  
        this.currency = currency;  
    }  
      
    public Money add(Money other) {  
        if (!this.currency.equals(other.currency)) {  
            throw new IllegalArgumentException("Cannot add different currencies");  
        }  
        return new Money(this.amount.add(other.amount), this.currency);  
    }  
      
    // 其他值对象方法  
}  
  
// 实体  
package com.example.demo.domain.model.entity;  
  
import com.example.demo.domain.model.valueobject.Money;  
  
public class Product {  
    private ProductId id;  
    private String name;  
    private Money price;  
    private int stock;  
      
    // 构造函数、getter和setter  
      
    // 领域行为  
    public boolean isAvailable() {  
        return stock > 0;  
    }  
      
    public void decreaseStock(int quantity) {  
        if (quantity > stock) {  
            throw new IllegalArgumentException("Not enough stock");  
        }  
        this.stock -= quantity;  
    }  
}  
  
// 聚合根  
package com.example.demo.domain.model.aggregate;  
  
import com.example.demo.domain.model.entity.Product;  
import com.example.demo.domain.model.valueobject.Money;  
  
import java.util.ArrayList;  
import java.util.List;  
  
public class Order {  
    private OrderId id;  
    private CustomerId customerId;  
    private List<OrderLine> orderLines = new ArrayList<>();  
    private OrderStatus status;  
    private Money totalAmount;  
      
    // 构造函数、getter和setter  
      
    // 领域行为  
    public void addProduct(Product product, int quantity) {  
        if (!product.isAvailable() || product.getStock() < quantity) {  
            throw new IllegalArgumentException("Product not available");  
        }  
          
        OrderLine orderLine = new OrderLine(product.getId(), product.getPrice(), quantity);  
        orderLines.add(orderLine);  
        recalculateTotal();  
    }  
      
    public void confirm() {  
        if (orderLines.isEmpty()) {  
            throw new IllegalStateException("Cannot confirm empty order");  
        }  
        this.status = OrderStatus.CONFIRMED;  
    }  
      
    private void recalculateTotal() {  
        this.totalAmount = orderLines.stream()  
            .map(OrderLine::getSubtotal)  
            .reduce(Money.ZERO, Money::add);  
    }  
}
```

#### 3.2.2 领域仓储接口

```
// 仓储接口  
package com.example.demo.domain.repository;  
  
import com.example.demo.domain.model.aggregate.Order;  
import com.example.demo.domain.model.aggregate.OrderId;  
  
import java.util.Optional;  
  
public interface OrderRepository {  
    Optional<Order> findById(OrderId id);  
    Order save(Order order);  
    void delete(OrderId id);  
}
```

#### 3.2.3 应用服务接口

```
// 应用服务接口  
package com.example.demo.application.port.incoming;  
  
import com.example.demo.application.dto.OrderRequest;  
import com.example.demo.application.dto.OrderResponse;  
  
import java.util.List;  
import java.util.Optional;  
  
public interface OrderApplicationService {  
    OrderResponse createOrder(OrderRequest request);  
    Optional<OrderResponse> getOrder(String orderId);  
    List<OrderResponse> getCustomerOrders(String customerId);  
    void confirmOrder(String orderId);  
}
```

#### 3.2.4 应用服务实现

```
// 应用服务实现  
package com.example.demo.application.service;  
  
import com.example.demo.application.dto.OrderRequest;  
import com.example.demo.application.dto.OrderResponse;  
import com.example.demo.application.port.incoming.OrderApplicationService;  
import com.example.demo.application.port.outgoing.ProductRepository;  
import com.example.demo.domain.model.aggregate.Order;  
import com.example.demo.domain.model.aggregate.OrderId;  
import com.example.demo.domain.model.entity.Product;  
import com.example.demo.domain.repository.OrderRepository;  
import org.springframework.stereotype.Service;  
import org.springframework.transaction.annotation.Transactional;  
  
import java.util.List;  
import java.util.Optional;  
import java.util.stream.Collectors;  
  
@Service  
public class OrderApplicationServiceImpl implements OrderApplicationService {  
      
    private final OrderRepository orderRepository;  
    private final ProductRepository productRepository;  
      
    public OrderApplicationServiceImpl(OrderRepository orderRepository, ProductRepository productRepository) {  
        this.orderRepository = orderRepository;  
        this.productRepository = productRepository;  
    }  
      
    @Override  
    @Transactional  
    public OrderResponse createOrder(OrderRequest request) {  
        // 创建订单领域对象  
        Order order = new Order(new CustomerId(request.getCustomerId()));  
          
        // 添加产品  
        for (OrderRequest.OrderItem item : request.getItems()) {  
            Product product = productRepository.findById(new ProductId(item.getProductId()))  
                .orElseThrow(() -> new RuntimeException("Product not found"));  
              
            order.addProduct(product, item.getQuantity());  
              
            // 减少库存  
            product.decreaseStock(item.getQuantity());  
            productRepository.save(product);  
        }  
          
        // 保存订单  
        Order savedOrder = orderRepository.save(order);  
          
        // 返回DTO  
        return mapToDto(savedOrder);  
    }  
      
    @Override  
    public Optional<OrderResponse> getOrder(String orderId) {  
        return orderRepository.findById(new OrderId(orderId))  
            .map(this::mapToDto);  
    }  
      
    @Override  
    public List<OrderResponse> getCustomerOrders(String customerId) {  
        return orderRepository.findByCustomerId(new CustomerId(customerId)).stream()  
            .map(this::mapToDto)  
            .collect(Collectors.toList());  
    }  
      
    @Override  
    @Transactional  
    public void confirmOrder(String orderId) {  
        Order order = orderRepository.findById(new OrderId(orderId))  
            .orElseThrow(() -> new RuntimeException("Order not found"));  
          
        order.confirm();  
        orderRepository.save(order);  
    }  
      
    private OrderResponse mapToDto(Order order) {  
        // 映射逻辑  
    }  
}
```

#### 3.2.5 输入适配器（REST控制器）

```
// REST控制器  
package com.example.demo.interface.rest;  
  
import com.example.demo.application.dto.OrderRequest;  
import com.example.demo.application.dto.OrderResponse;  
import com.example.demo.application.port.incoming.OrderApplicationService;  
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind.annotation.*;  
  
import java.util.List;  
  
@RestController  
@RequestMapping("/api/orders")  
public class OrderController {  
      
    private final OrderApplicationService orderService;  
      
    public OrderController(OrderApplicationService orderService) {  
        this.orderService = orderService;  
    }  
      
    @PostMapping  
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {  
        OrderResponse response = orderService.createOrder(request);  
        return ResponseEntity.ok(response);  
    }  
      
    @GetMapping("/{orderId}")  
    public ResponseEntity<OrderResponse> getOrder(@PathVariable String orderId) {  
        return orderService.getOrder(orderId)  
            .map(ResponseEntity::ok)  
            .orElse(ResponseEntity.notFound().build());  
    }  
      
    @GetMapping("/customer/{customerId}")  
    public List<OrderResponse> getCustomerOrders(@PathVariable String customerId) {  
        return orderService.getCustomerOrders(customerId);  
    }  
      
    @PostMapping("/{orderId}/confirm")  
    public ResponseEntity<Void> confirmOrder(@PathVariable String orderId) {  
        orderService.confirmOrder(orderId);  
        return ResponseEntity.ok().build();  
    }  
}
```

#### 3.2.6 输出适配器（JPA持久化）

```
// JPA实体  
package com.example.demo.infrastructure.adapter.outgoing.persistence.entity;  
  
import javax.persistence.*;  
import java.math.BigDecimal;  
import java.util.ArrayList;  
import java.util.List;  
  
@Entity  
@Table(name = "orders")  
public class OrderJpaEntity {  
    @Id  
    private String id;  
      
    private String customerId;  
      
    @Enumerated(EnumType.STRING)  
    private OrderStatusJpa status;  
      
    private BigDecimal totalAmount;  
      
    private String currency;  
      
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)  
    @JoinColumn(name = "order_id")  
    private List<OrderLineJpaEntity> orderLines = new ArrayList<>();  
      
    // 构造函数、getter和setter  
}  
  
// JPA仓库  
package com.example.demo.infrastructure.adapter.outgoing.persistence.repository;  
  
import com.example.demo.infrastructure.adapter.outgoing.persistence.entity.OrderJpaEntity;  
import org.springframework.data.jpa.repository.JpaRepository;  
import org.springframework.data.jpa.repository.Query;  
  
import java.util.List;  
  
public interface OrderJpaRepository extends JpaRepository<OrderJpaEntity, String> {  
    List<OrderJpaEntity> findByCustomerId(String customerId);  
}  
  
// 适配器实现  
package com.example.demo.infrastructure.adapter.outgoing.persistence;  
  
import com.example.demo.domain.model.aggregate.Order;  
import com.example.demo.domain.model.aggregate.OrderId;  
import com.example.demo.domain.model.entity.CustomerId;  
import com.example.demo.domain.repository.OrderRepository;  
import com.example.demo.infrastructure.adapter.outgoing.persistence.entity.OrderJpaEntity;  
import com.example.demo.infrastructure.adapter.outgoing.persistence.repository.OrderJpaRepository;  
import org.springframework.stereotype.Repository;  
  
import java.util.List;  
import java.util.Optional;  
import java.util.stream.Collectors;  
  
@Repository  
public class OrderRepositoryAdapter implements OrderRepository {  
      
    private final OrderJpaRepository jpaRepository;  
      
    public OrderRepositoryAdapter(OrderJpaRepository jpaRepository) {  
        this.jpaRepository = jpaRepository;  
    }  
      
    @Override  
    public Optional<Order> findById(OrderId id) {  
        return jpaRepository.findById(id.getValue())  
            .map(this::mapToDomain);  
    }  
      
    @Override  
    public Order save(Order order) {  
        OrderJpaEntity entity = mapToJpaEntity(order);  
        OrderJpaEntity savedEntity = jpaRepository.save(entity);  
        return mapToDomain(savedEntity);  
    }  
      
    @Override  
    public void delete(OrderId id) {  
        jpaRepository.deleteById(id.getValue());  
    }  
      
    @Override  
    public List<Order> findByCustomerId(CustomerId customerId) {  
        return jpaRepository.findByCustomerId(customerId.getValue()).stream()  
            .map(this::mapToDomain)  
            .collect(Collectors.toList());  
    }  
      
    private Order mapToDomain(OrderJpaEntity entity) {  
        // 映射逻辑  
    }  
      
    private OrderJpaEntity mapToJpaEntity(Order order) {  
        // 映射逻辑  
    }  
}
```

### 3.3 优缺点分析

**优点：**

* • 结合DDD概念，更丰富的领域模型
* • 更精确地表达业务规则和约束
* • 领域模型与持久化完全分离
* • 支持复杂业务场景和领域行为

**缺点：**

* • 架构复杂度进一步增加
* • 学习曲线陡峭，需要同时掌握DDD和六边形架构
* • 对象映射工作更加繁重
* • 可能过度设计，特别是对简单领域

### 3.4 适用场景

* • 复杂业务领域，有丰富的业务规则和约束
* • 大型企业应用，特别是核心业务系统
* • 团队熟悉DDD和六边形架构
* • 长期维护的系统，需要适应业务变化

# 四、简化版架构实现

### 4.1 项目结构

简化架构采用更轻量级的方式实现六边形架构的核心理念，减少接口数量，简化层次结构：

```
src/main/java/com/example/demo/  
├── service/                 # 服务层  
│   ├── business/            # 业务服务  
│   ├── model/               # 数据模型  
│   └── exception/           # 业务异常  
├── integration/             # 集成层  
│   ├── database/            # 数据库集成  
│   ├── messaging/           # 消息集成  
│   └── external/            # 外部服务集成  
├── web/                     # Web层  
│   ├── controller/          # 控制器  
│   ├── dto/                 # 数据传输对象  
│   └── advice/              # 全局异常处理  
└── config/                  # 配置
```

### 4.2 代码实现

#### 4.2.1 数据模型

```
// 业务模型  
package com.example.demo.service.model;  
  
import lombok.Data;  
  
@Data  
public class Product {  
    private Long id;  
    private String name;  
    private double price;  
    private int stock;  
      
    // 业务逻辑直接在模型中  
    public boolean isAvailable() {  
        return stock > 0;  
    }  
      
    public void decreaseStock(int quantity) {  
        if (quantity > stock) {  
            throw new IllegalArgumentException("Not enough stock");  
        }  
        this.stock -= quantity;  
    }  
}
```

#### 4.2.2 集成层接口

```
// 数据库集成接口  
package com.example.demo.integration.database;  
  
import com.example.demo.service.model.Product;  
import java.util.List;  
import java.util.Optional;  
  
// 没有复杂的端口和适配器分离，直接定义操作接口  
public interface ProductRepository {  
    List<Product> findAll();  
    Optional<Product> findById(Long id);  
    Product save(Product product);  
    void deleteById(Long id);  
}
```

#### 4.2.3 业务服务

```
// 业务服务  
package com.example.demo.service.business;  
  
import com.example.demo.integration.database.ProductRepository;  
import com.example.demo.service.model.Product;  
import org.springframework.stereotype.Service;  
import org.springframework.transaction.annotation.Transactional;  
  
import java.util.List;  
import java.util.Optional;  
  
@Service  
public class ProductService {  
      
    private final ProductRepository productRepository;  
      
    // 直接注入所需依赖  
    public ProductService(ProductRepository productRepository) {  
        this.productRepository = productRepository;  
    }  
      
    public List<Product> getAllProducts() {  
        return productRepository.findAll();  
    }  
      
    public Optional<Product> getProductById(Long id) {  
        return productRepository.findById(id);  
    }  
      
    public Product createProduct(Product product) {  
        return productRepository.save(product);  
    }  
      
    @Transactional  
    public void updateStock(Long productId, int quantity) {  
        Product product = productRepository.findById(productId)  
            .orElseThrow(() -> new RuntimeException("Product not found"));  
          
        product.decreaseStock(quantity);  
        productRepository.save(product);  
    }  
}
```

#### 4.2.4 控制器

```
// REST控制器  
package com.example.demo.web.controller;  
  
import com.example.demo.service.business.ProductService;  
import com.example.demo.service.model.Product;  
import com.example.demo.web.dto.ProductDTO;  
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind.annotation.*;  
  
import java.util.List;  
import java.util.stream.Collectors;  
  
@RestController  
@RequestMapping("/api/products")  
public class ProductController {  
      
    private final ProductService productService;  
      
    public ProductController(ProductService productService) {  
        this.productService = productService;  
    }  
      
    @GetMapping  
    public List<ProductDTO> getAllProducts() {  
        return productService.getAllProducts().stream()  
            .map(this::toDto)  
            .collect(Collectors.toList());  
    }  
      
    @GetMapping("/{id}")  
    public ResponseEntity<ProductDTO> getProductById(@PathVariable Long id) {  
        return productService.getProductById(id)  
            .map(this::toDto)  
            .map(ResponseEntity::ok)  
            .orElse(ResponseEntity.notFound().build());  
    }  
      
    @PostMapping  
    public ProductDTO createProduct(@RequestBody ProductDTO productDto) {  
        Product product = toEntity(productDto);  
        return toDto(productService.createProduct(product));  
    }  
      
    @PutMapping("/{id}/stock")  
    public ResponseEntity<Void> updateStock(  
            @PathVariable Long id,  
            @RequestParam int quantity) {  
        productService.updateStock(id, quantity);  
        return ResponseEntity.ok().build();  
    }  
      
    private ProductDTO toDto(Product product) {  
        ProductDTO dto = new ProductDTO();  
        dto.setId(product.getId());  
        dto.setName(product.getName());  
        dto.setPrice(product.getPrice());  
        dto.setStock(product.getStock());  
        return dto;  
    }  
      
    private Product toEntity(ProductDTO dto) {  
        Product product = new Product();  
        product.setId(dto.getId());  
        product.setName(dto.getName());  
        product.setPrice(dto.getPrice());  
        product.setStock(dto.getStock());  
        return product;  
    }  
}
```

#### 4.2.5 数据库实现

```
// JPA实体 - 与业务模型类似  
package com.example.demo.integration.database.entity;  
  
import lombok.Data;  
import javax.persistence.*;  
  
@Entity  
@Table(name = "products")  
@Data  
public class ProductEntity {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
    private String name;  
    private double price;  
    private int stock;  
}  
  
// JPA仓库  
package com.example.demo.integration.database.repository;  
  
import com.example.demo.integration.database.entity.ProductEntity;  
import org.springframework.data.jpa.repository.JpaRepository;  
  
public interface JpaProductRepository extends JpaRepository<ProductEntity, Long> {  
}  
  
// 仓库实现  
package com.example.demo.integration.database;  
  
import com.example.demo.integration.database.entity.ProductEntity;  
import com.example.demo.integration.database.repository.JpaProductRepository;  
import com.example.demo.service.model.Product;  
import org.springframework.stereotype.Repository;  
  
import java.util.List;  
import java.util.Optional;  
import java.util.stream.Collectors;  
  
@Repository  
public class ProductRepositoryImpl implements ProductRepository {  
      
    private final JpaProductRepository jpaRepository;  
      
    public ProductRepositoryImpl(JpaProductRepository jpaRepository) {  
        this.jpaRepository = jpaRepository;  
    }  
      
    @Override  
    public List<Product> findAll() {  
        return jpaRepository.findAll().stream()  
            .map(this::toModel)  
            .collect(Collectors.toList());  
    }  
      
    @Override  
    public Optional<Product> findById(Long id) {  
        return jpaRepository.findById(id)  
            .map(this::toModel);  
    }  
      
    @Override  
    public Product save(Product product) {  
        ProductEntity entity = toEntity(product);  
        return toModel(jpaRepository.save(entity));  
    }  
      
    @Override  
    public void deleteById(Long id) {  
        jpaRepository.deleteById(id);  
    }  
      
    private Product toModel(ProductEntity entity) {  
        Product product = new Product();  
        product.setId(entity.getId());  
        product.setName(entity.getName());  
        product.setPrice(entity.getPrice());  
        product.setStock(entity.getStock());  
        return product;  
    }  
      
    private ProductEntity toEntity(Product product) {  
        ProductEntity entity = new ProductEntity();  
        entity.setId(product.getId());  
        entity.setName(product.getName());  
        entity.setPrice(product.getPrice());  
        entity.setStock(product.getStock());  
        return entity;  
    }  
}
```

### 4.3 优缺点分析

**优点：**

* • 结构简单，学习曲线平缓
* • 减少了接口和层次数量，代码量更少
* • 遵循Spring框架惯例，对Spring开发者友好
* • 开发效率高，适合快速迭代
* • 仍然保持了业务逻辑和外部依赖的基本分离

**缺点：**

* • 分离不如经典六边形架构严格
* • 业务逻辑可能会混入非核心关注点
* • 领域模型不够丰富
* • 对复杂业务场景支持有限

### 4.4 适用场景

* • 中小型应用，业务逻辑相对简单
* • 需要快速开发和迭代的项目
* • 原型或MVP开发
* • 启动阶段的项目，后期可能演进到更严格的架构

# 五、总结

六边形架构的核心价值在于将业务逻辑与技术细节分离，提高系统的可维护性、可测试性和灵活性。

无论选择哪种实现方式，都应该坚持这一核心原则，保持领域模型的纯粹性和边界的清晰性。

需要特别说明的是，架构应该服务于业务，而非相反。选择合适的架构方式，应以提高开发效率、系统质量和业务适应性为目标。