矿山安全知识库智能对话平台（MineSafetyAI）是一个面向矿山行业的智能问答系统，采用检索增强生成（RAG）技术，为矿山安全生产提供智能化知识检索与问答服务。

核心技术栈包括 ElasticSearch、Kafka、WebSocket、Spring Security、Docker、MySQL 和 Redis。

平台旨在帮助矿山企业构建安全知识库，通过自然语言交互快速获取安全规程、应急预案、操作规范等信息，提升矿山安全管理效率和应急处置能力。

系统主要功能：

- 矿山安全文档的批量导入与智能解析
- 安全规程、应急预案等文档的语义索引
- 基于自然语言的安全知识问答
- 多租户架构支持不同矿区独立管理

后端的整体项目结构：

```bash
src/main/java/com/yizhaoqi/minercap/
├── MinerCapApplication.java      # 主应用程序入口
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
