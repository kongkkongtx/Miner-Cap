矿山安全知识库智能对话平台（MineSafetyAI）是一个面向矿山行业的智能问答系统，采用检索增强生成（RAG）技术，为矿山安全生产提供智能化知识检索与问答服务。

核心技术栈包括 ElasticSearch、Kafka、WebSocket、Spring Security、Docker、MySQL 和 Redis。

平台旨在帮助矿山企业构建安全知识库，通过自然语言交互快速获取安全规程、应急预案、操作规范等信息，提升矿山安全管理效率和应急处置能力。

系统主要功能：

- 矿山安全文档的批量导入与智能解析
- 安全规程、应急预案等文档的语义索引
- 基于自然语言的安全知识问答
- 多租户架构支持不同矿区独立管理

用到的技术栈包括，先说后端的：

+ 框架 : Spring Boot 3.4.2 (Java 17)
+ 数据库 : MySQL 8.0
+ ORM : Spring Data JPA
+ 缓存 : Redis
+ 搜索引擎 : Elasticsearch 8.10.0
+ 消息队列 : Apache Kafka
+ 文件存储 : MinIO
+ 文档解析 : Apache Tika
+ 安全认证 : Spring Security + JWT
+ AI集成 : DeepSeek API/本地 Ollama+豆包 Embedding
+ 实时通信 : WebSocket
+ 依赖管理 : Maven
+ 响应式编程 : WebFlux

后端的整体项目结构：

```bash
src/main/java/com/yizhaoqi/minercap/
├── SmartPaiApplication.java      # 主应用程序入口
├── client/                       # 外部API客户端
├── config/                       # 配置类
├── consumer/                     # Kafka消费者
├── controller/                   # REST API端点
├── entity/                       # 数据实体
├── exception/                    # 自定义异常
├── handler/                      # WebSocket处理器
├── model/                        # 领域模型
├── repository/                   # 数据访问层
├── service/                      # 业务逻辑
└── utils/                        # 工具类
```

再说前端的，包括：

+ 框架 : Vue 3 + TypeScript
+ 构建工具 : Vite
+ UI组件 : Naive UI
+ 状态管理 : Pinia
+ 路由 : Vue Router
+ 样式 : UnoCSS + SCSS
+ 图标 : Iconify
+ 包管理 : pnpm

前端的整体项目结构：

```bash
frontend/
├── packages/           # 可重用模块
├── public/             # 静态资源
├── src/                # 主应用程序代码
│   ├── assets/         # SVG图标，图片
│   ├── components/     # Vue组件
│   ├── layouts/        # 页面布局
│   ├── router/         # 路由配置
│   ├── service/        # API集成
│   ├── store/          # 状态管理
│   ├── views/          # 页面组件
│   └── ...            # 其他工具和配置
└── ...               # 构建配置文件
```

## 核心功能

### 安全知识库管理

平台支持矿山安全相关文档的批量上传与智能解析，包括安全操作规程、应急预案、安全培训资料等。支持文档分类管理与标签组织，便于快速检索。

### AI驱动的RAG问答

平台核心是 RAG 检索增强生成技术：

- 将安全文档进行语义分块处理
- 调用 Embedding 模型为文本块生成向量
- 将向量存储到 ElasticSearch 支持语义搜索
- 根据用户问题检索相关安全知识
- 结合大语言模型生成准确的回答

### 多租户架构

平台支持多矿区、多部门独立管理，每个租户可拥有独立的知识库和文档管理权限，确保数据安全与权限隔离。

### 实时对话

通过 WebSocket 实现实时对话交互，支持流式响应，用户可快速获取安全知识问答结果。

## 前置环境

在开始之前，请确保已安装以下软件：

- Java 17
- Maven 3.8.6 或更高版本
- Node.js 18.20.0 或更高版本
- pnpm 8.7.0 或更高版本
- MySQL 8.0
- Elasticsearch 8.10.0
- MinIO 8.5.12
- Kafka 3.2.1
- Redis 7.0.11
- Docker（可选，用于运行 Redis、MinIO、Elasticsearch 和 Kafka 等服务）

## 架构设计

平台采用现代化的分层架构设计，具有清晰的关注点分离、可扩展的组件设计。模块化架构便于后续功能扩展和技术升级。

控制层用于处理 HTTP 请求，验证输入，管理请求/响应格式化，并将业务逻辑委托给服务层。控制器按领域功能组织，遵循 RESTful 设计原则。

```java
@RestController
@RequestMapping("/api/v1/documents")
public class DocumentController {
    @Autowired
    private DocumentService documentService;
    
    @DeleteMapping("/{fileMd5}")
    public ResponseEntity<?> deleteDocument(
            @PathVariable String fileMd5,
            @RequestAttribute("userId") String userId,
            @RequestAttribute("role") String role) {
        // 参数验证和委托给服务
        documentService.deleteDocument(fileMd5);
        // 响应处理
    }
}
```

服务层主要用来处理应用的业务逻辑，具有事务感知能力，能够处理跨越多个数据源的操作。

```java
@Service
public class DocumentService {
    @Autowired
    private FileUploadRepository fileUploadRepository;
    
    @Autowired
    private MinioClient minioClient;
    
    @Autowired
    private ElasticsearchService elasticsearchService;
    
    @Transactional
    public void deleteDocument(String fileMd5) {
        // 文档删除的业务逻辑
        // 协调多个仓储和系统
    }
}
```

数据访问层使用 Spring Data JPA 进行数据库操作，提供了对 MySQL 的 CRUD 操作。

```java
@Repository
public interface FileUploadRepository extends JpaRepository<FileUpload, Long> {
    Optional<FileUpload> findByFileMd5(String fileMd5);
    
    @Query("SELECT f FROM FileUpload f WHERE f.userId = :userId OR f.isPublic = true OR (f.orgTag IN :orgTagList AND f.isPublic = false)")
    List<FileUpload> findAccessibleFilesWithTags(@Param("userId") String userId, @Param("orgTagList") List<String> orgTagList);
}
```

实体层由映射到数据库表的 JPA 实体以及用于 API 请求和响应的 DTO（数据传输对象）组成。

```java
@Entity
public class FileUpload {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String fileMd5;
    private String fileName;
    private String userId;
    private boolean isPublic;
    private String orgTag;
    // 其他字段和方法
}
```

## 前端启动

```bash
# 进入前端项目目录
cd frontend

# 安装依赖
pnpm install

# 启动项目
pnpm run dev
```

## 后端启动

```bash
# 确保已安装 Java 17 和 Maven 3.8.6+

# 安装依赖并编译
mvn clean install

# 启动 Spring Boot 应用
mvn spring-boot:run
```

## 注意事项

1. 启动前请确保 MySQL、Redis、Elasticsearch、Kafka、MinIO 等中间件已正确配置并运行
2. 根据实际环境修改 `application.yml` 中的配置信息
3. 前端默认端口为 5173，后端默认端口为 8080