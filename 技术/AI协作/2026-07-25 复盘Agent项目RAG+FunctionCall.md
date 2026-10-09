# 复盘Agent项目（RAG+FunctionCall）（材料笔记）

> B站「闪行星」复盘电信CRM智能客服POC。Java+Python解耦架构，手写RAG + 自定义模型包装器接入LangChain Agent。

## 基本信息

- 来源：B站视频 "复盘我的第一个Agent项目（RAG+FunctionCall）| 面试前的系统梳理"
- 作者：闪行星
- 日期：2025-08-27
- 链接：https://www.bilibili.com/video/BV1ETvTzsEgP/

## 架构：Java + Python 混合

### 数据流

```
用户 → Java（Web层，非阻塞WebClient）→ HTTP POST
→ Python（FastAPI）→ Hybrid Router（意图分类）
→ RAG问答链 / Function Call执行链
→ 统一JSON响应 → Java → 前端
```

### Hybrid Router（意图路由）

- 不用传统关键词匹配或分类模型，直接让大模型判断：FAQ / Flow / UNKNOWN
- Prompt限制输出三种之一，温度调零，输出后strip+lower做后处理

## RAG问答链（手写，未用LangChain Retriever）

| 组件 | 选择 | 理由 |
|------|------|------|
| 向量模型 | BGE中文（本地，512维） | 轻量，中文表现好 |
| 向量数据库 | Milvus（IVF + L2距离） | ES高维支持弱，Faiss缺持久化 |
| 框架 | 完全手写（SentenceTransformers + Milvus SDK） | LangChain Retriever太黑盒，调试困难 |

### 不用LangChain Retriever的原因

1. 用sentence transformers更简单，适配LangChain需要写自定义embedding类
2. 希望精细控制Milvus检索参数（top k、相似度阈值）
3. 自己搭流程方便插日志加断点，每一步可定位

## Function Call执行链

### 自定义DeepSeek Chat Model

- 继承LangChain `BaseChatModel`，重写`generate`方法
- 把DeepSeek接口包装成LangChain能识别的标准模型接口
- 手动判断模型输出是否触发function_call
- 从映射表找函数 → 执行 → 封装`AIMessage`返回

### 工具注册

- `@tool`装饰器注册本地函数
- 工具定义转换器：LangChain工具对象 → DeepSeek规范的JSON Schema
- 手动接管执行而非交给LangChain自动调度：方便加日志、兜底策略、错误处理

### 为什么还要用LangChain Agent

- 当前场景一次function call可以不用agent
- 但为**多轮推理、工具链调用、记忆管理**预留了扩展口
- "既保留了agent的组织能力，又保证了对DeepSeek定制接口的完全可控性"

## 架构决策总结

| 挑战 | 方案 |
|------|------|
| Java做不了灵活AI组件调度 | Java只负责业务框架，智能部分全交给Python |
| LangChain不兼容DeepSeek | 写模型包装器，重写generate |
| 可控性vs框架封装 | RAG完全手写，Function Call手动接管执行 |
| 高延迟LLM API调用 | WebClient（非阻塞）而非RestTemplate |

## 概念关联

- [问题分解](<../../教育/学习方法/问题分解.md>) — 工程架构中的分层拆解（Java业务层 / Python智能层）
- [实践](<../../哲学/社会实践/实践.md>) — 从"让LLM跑起来"到"打通工程化整条链路"的实践过程
