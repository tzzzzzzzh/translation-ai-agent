# 智能翻译系统 · 面试展示版

基于苏三的 translation-ai-agent 项目进行本地部署、学习与配置适配。原始项目地址：https://gitcode.com/susan_java/translation-ai-agent 。保留源码中的原作者署名，不将原项目功能全部声明为个人原创。

本仓库用于面试交流。未附加新的开源许可证；原作者代码的权利归原权利人所有。公开再分发范围应以原作者授权为准。

## 已完成的工作与展示整理

- 在本地使用 JDK 21、MySQL 运行项目，排查 MySQL TCP 连接问题。
- 配置千问调用地址及 API Key，将本地使用的模型调整为 qwen-plus，完成文本翻译验证。
- 展示副本移除运行日志、本地 IDE 文件、原 Git 历史和数据库业务数据。
- 展示副本将数据库、Redis、MinIO 凭证及 JWT 签名密钥改为环境变量。

以上不表示已独立实现原项目全部业务模块。展示版的环境变量改造与空数据库初始化尚未完成端到端运行验证。

## 技术与功能

Spring Boot 3、Java 21、MySQL、Spring Data JPA、Redis、JWT、SSE、CompletableFuture。提供文本翻译、聊天翻译、文档翻译及术语管理等模块。

## 实现边界

- 当前聊天路径先获取完整译文，再通过 SseEmitter 分段推送，不是模型端到端流式输出。
- 聊天异步任务使用独立线程池；文档处理仍存在直接创建线程的实现。
- RAG 使用术语和历史检索，可选 Embedding 重排序；向量模型未配置时不会进行向量排序。
- 语言检测存在多级回退；文本翻译按引擎选择执行，不能概括为全流程自动三级降级。
- 部分模块含 Mock 演示逻辑，无生产流量或性能提升数据声明。

## 本地运行准备

1. 安装 JDK 21 和 Maven，自行准备 MySQL、Redis；文档功能需配置 MinIO。
2. 新建空的 translation 数据库，导入 database/schema.sql。该文件只有表结构，无用户、点数、术语、订单或翻译记录；业务所需初始数据需要自行补充。不要导入到已有业务数据库。
3. 在 IDEA 运行配置的「环境变量」中设置下面的变量（值由你本地填写，不提交）：

| 变量 | 含义 |
|---|---|
| DB_PASSWORD | MySQL 密码 |
| DB_USERNAME | MySQL 用户名，默认 root |
| REDIS_HOST / REDIS_PASSWORD | Redis 地址和密码 |
| QWEN_API_URL | 控制台对应的完整聊天接口 URL，包含 /chat/completions |
| QWEN_WEATHER_API_KEY | 千问 API Key |
| JWT_SECRET | 自行生成的高强度随机签名密钥，至少 32 个 UTF-8 字节 |
| MINIO_ENDPOINT | MinIO 地址，默认 http://localhost:9000 |
| MINIO_ACCESS_KEY / MINIO_SECRET_KEY | 自有 MinIO 凭证 |

其他数据库连接参数见 src/main/resources/application.yml。阿里云机器翻译、支付、MQ 等按功能另行配置，不包含第三方共享服务凭证。

4. 运行 cn.net.susan.ai.translation.TranslationAiAgentApplication，成功后访问 http://localhost:7002 。

JWT_SECRET 修改后，旧 Token 将无法通过签名验证，需要重新登录。多实例部署需使用相同密钥。

## 代码导航

- controller/TranslationController.java：文本翻译入口
- service/impl/TranslationServiceImpl.java：翻译业务流程
- service/translation/QwenTranslationService.java：千问 HTTP 调用
- controller/ChatTranslationController.java：聊天 SSE 与任务提交
- config/AsyncConfig.java：聊天线程池
- security/AuthTokenInterceptor.java：JWT 与 Redis 登录校验
- service/llm/RagService.java：术语和历史检索增强

以上路径位于 src/main/java/cn/net/susan/ai/translation/ 下。
