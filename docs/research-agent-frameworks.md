# 生产级 Agent 框架核心模块研究报告

## 结论

**目前市面上用于生产的 Agent 框架，主要基于「图（Graph）」模式**，核心是有向状态图（StateGraph）执行引擎。

在 Spring AI Alibaba 中，这一角色由 **`spring-ai-alibaba-graph-core`** 模块承担。

---

## 1. Spring AI Alibaba 模块架构

```
spring-ai-alibaba-graph-core       ← 核心运行时引擎（基础层）
        ↓ 依赖
spring-ai-alibaba-agent-framework  ← 高层 Agent API（能力层）
        ↓ 依赖
应用层（chatbot、deepresearch 等）  ← 业务应用（应用层）
```

官方 README 明确说明：

> "Spring AI Alibaba Graph serves as the underlying runtime of the Agent Framework, providing essential capabilities such as **persistence, workflow orchestration, and streaming** required for long-running stateful agents."

### graph-core 提供的核心能力

| 核心类 | 功能 |
|---|---|
| `StateGraph` | 图定义（节点 + 边） |
| `CompiledGraph` | 编译后的可执行图（执行引擎） |
| `OverAllState` | 跨节点共享状态容器 |
| `RunnableConfig` | 执行配置（线程数、超时等） |
| `Node / Action` | 执行原语抽象 |
| Persistence 后端 | 支持 PostgreSQL、MySQL、Oracle、MongoDB、Redis、File |
| Reactive 流式 | 基于 `Flux` 的响应式流输出 |

### 所有生产 Agent 都基于 graph-core

`agent-framework` 中所有 Agent 最终都会被编译成 `StateGraph` 并通过 `CompiledGraph` 执行：

- `ReactAgent` — ReAct 推理-行动循环 Agent
- `SequentialAgent` — 顺序执行多个子 Agent
- `ParallelAgent` — 并行执行多个子 Agent（扇出/聚合）
- `LoopAgent` — 迭代执行（按次数、条件、数组）
- `LlmRoutingAgent` — 基于 LLM 的动态路由 Agent

---

## 2. 市场主流框架对比

| 框架 | 语言 | 底层核心模式 | 核心模块 |
|---|---|---|---|
| **LangGraph** | Python / JS | 有向状态图（StateGraph） | `langgraph-core` |
| **AutoGen** | Python | Actor / 消息传递 | `autogen-core` |
| **CrewAI** | Python | 角色制 + 顺序图 | `crewai` flow |
| **Semantic Kernel** | C# / Python | Kernel / Plugin Process | Process Framework |
| **Dify** | Python | DAG 工作流 | workflow engine |
| **Spring AI Alibaba** | Java | 有向状态图（StateGraph） | `spring-ai-alibaba-graph-core` |

### 主流趋势：Graph-based StateGraph 模式

LangGraph 在 2024 年将 **StateGraph** 模式推向生产，此后几乎所有主流框架都向图模式收敛。原因如下：

1. **有状态执行**：状态在节点间传递，支持长时间运行的复杂任务
2. **条件路由**：边可以是条件化的，支持动态分支决策
3. **断点续跑（Checkpointing）**：执行可以在任意节点暂停和恢复，支持 Human-in-the-Loop
4. **并行分支**：图结构天然支持并行执行（fan-out/fan-in）
5. **可观测性**：图结构可直接可视化和调试（Studio/Admin）
6. **持久化**：状态可以持久化到各种存储后端

---

## 3. Spring AI Alibaba 对市场模式的增强

相比原生的 LangGraph 模式，Spring AI Alibaba 在 graph-core 基础上增加了以下生产增强：

| 特性 | 说明 |
|---|---|
| **Context Engineering** | 内置 PII 检测、上下文压缩、消息管理策略 |
| **Human-in-the-Loop** | 原生支持，基于 graph checkpoint 暂停/恢复 |
| **Interceptor 链** | 模型调用和工具调用的双层拦截（重试、降级、限流） |
| **Hook 系统** | 30+ 生命周期钩子，支持细粒度控制 |
| **A2A 通信** | 基于 Nacos 的 Agent-to-Agent 服务发现和通信 |
| **多持久化后端** | PostgreSQL、MySQL、Oracle、MongoDB、Redis、File |
| **响应式流** | 基于 Flux 的实时流式输出 |

---

## 4. 参考文件

- `spring-ai-alibaba-agent-framework/pom.xml` — 声明 graph-core 为第一依赖
- `spring-ai-alibaba-graph-core/pom.xml` — graph-core 模块定义
- `spring-ai-alibaba-agent-framework/src/main/java/.../agent/ReactAgent.java` — 使用 StateGraph 构建 ReAct Agent
- `README.md` — 官方架构说明

---

*生成日期：2026-03-21*
