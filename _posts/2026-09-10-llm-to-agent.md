---
title: "从 LLM 到 Agent：LangChain、LlamaIndex、Agent 与 Harness"
tags:
  - ai
date: 2026-09-10 19:59:00 +0800 # 可选：覆盖文件名里的日期
---

### 从 LLM 到 Agent：LangChain、LlamaIndex、Agent 与 Harness

你是否接触过这些概念：**LLM、Tool、Agent、Agent Loop、Harness、LangChain、LangGraph、LlamaIndex**

繁多的概念令我产生一个疑问：

> 既然已经有 Claude Code、Codex 这样的成熟 Agent，所谓“开发 Agent”到底是在开发什么？难道是重新开发一套 Harness？

经过梳理之后，我终于有了些眉目，现在让我们从一个最基本的 Agent 执行过程开始理解。

#### 什么是 Agent？

##### 1. 从 LLM 到 Agent

以下是最基本的 LLM 调用范式：

```text
User
 ↓
LLM
 ↓
Answer
```

用户提出问题，LLM 根据上下文生成答案，结束。

Agent 与 LLM 的关键区别在于：**Agent 不只是生成最终答案，还可以决定下一步应该采取什么行动。**

例如用户要求：

> 分析某个目标的基本 Web 服务情况，并生成报告。

Agent 的执行过程可能如下：

```text
User
 ↓
LLM
 ↓
决定调用 DNS Tool
 ↓
DNS Result
 ↓
LLM
 ↓
决定调用 HTTP Tool
 ↓
HTTP Result
 ↓
LLM
 ↓
决定调用其他分析 Tool
 ↓
Result
 ↓
LLM
 ↓
Final Answer
```

因此可以暂时把 Agent 理解为：

> **一个能够根据任务和反馈，反复决定“下一步做什么”，直到得出最终结果的 LLM 系统。**

##### 2. Tool：“说”到“做”

LLM 本身不具备执行程序的能力，例如模型生成：

```json
{
  "tool": "dns_lookup",
  "arguments": {
    "domain": "example.com"
  }
}
```

这并不意味着模型真的进行了 DNS 查询，这只是 token 序列的解析结果。

它表达了一种意图：

> 我希望调用 `dns_lookup()`。

如果想要真正调用 `dns_lookup()`，则需要依靠外部工具。

因此：

```text
LLM
 │
 │ Tool Call
 ▼
Harness / Runtime
 │
 ▼
Tool
 │
 ▼
Real World
```

这样就实现了 LLM 与现实世界的交互。

Tool Call 可以是很多东西比如：

```text
read_file()
write_file()
run_shell()
search_web()
query_database()
search_document()
call_api()
```

于是 Claude Code、Codex 等 Agent 不只是“聊天机器人”，它们拥有可以实际操作开发环境的工具，真正参与进了执行流程。

##### 3. Agent Loop：永不停止...直到

如果 LLM 调用一次 Tool 就结束，那与想要喝水但是只走了一步的人没有区别。

为了让 LLM 达到目标，我们必须让他思考当前的调用结果，除非目标达成，否则进行下一的轮思考与调用。

类似：

```text
Think
 ↓
Act
 ↓
Observe
 ↓
Think
 ↓
Act
 ↓
Observe
 ↓
...
 ↓
Finish
```

对应到程序逻辑，可以极度简化为：

```python
while True:

    response = llm(messages)

    if response.tool_call:
        result = execute_tool(response.tool_call)
        messages.append(result)

    else:
        return response
```

例如：

```text
用户：修复这个 Bug
        ↓
LLM：先读取文件
        ↓
read_file()
        ↓
LLM：发现问题，修改代码
        ↓
write_file()
        ↓
LLM：运行测试
        ↓
run_test()
        ↓
测试失败
        ↓
LLM：重新分析
        ↓
write_file()
        ↓
run_test()
        ↓
测试通过
        ↓
Final Answer
```

这个不断进行：

> **思考 → 行动 → 获取结果 → 再思考**

的机制，就是最基本的 **Agent Loop**。

##### 4. Harness：让 Agent 真正运行起来的环境

这时出现一个问题：

谁负责：

- 调用 LLM？
- 保存上下文？
- 执行 Tool？
- 保存 Tool Result？
- 再次调用 LLM？
- 控制 Agent Loop？
- 限制危险操作？
- 管理 Token？
- 保存状态？
- 记录日志？
- 处理失败和重试？

这些都不能单纯依赖 LLM。

将 LLM 作为整个任务的思考中枢，为其提供工具调用、上下文管理、对话记忆等等环境的外部运行系统，我们将其称之为 **Harness**：

```text
┌──────────────────────────────┐
│            Harness           │
│                              │
│  Model API                   │
│  Context Management          │
│  Agent Loop                  │
│  Tool Execution              │
│  State / Memory              │
│  Permission                  │
│  Sandbox                     │
│  Error Handling              │
│  Logging / Observability     │
│                              │
└──────────────┬───────────────┘
               │
               ▼
              LLM
```

看一个简单易懂的类比：

| 概念       | 类比                             |
| ---------- | -------------------------------- |
| LLM        | 大脑                             |
| Tool       | 手里的工具                       |
| Agent Loop | “想 → 做 → 看 → 再想”的循环      |
| Harness    | 身体 + 工作环境 + 运行系统       |
| Agent      | 利用以上能力完成任务的整体智能体 |

#### Agent 开发？

##### 1. “开发 Agent”不等于“开发 Harness”

所谓 Agent Development 通常并不是从头开发一个 Claude Code 出来，更多时候是在设计：

```text
Agent 的目标
+
System Prompt
+
Tools
+
Workflow
+
State
+
Knowledge
+
Evaluation
```

例如开发一个 Research Agent：

```text
User
 ↓
Research Agent
 ↓
Search
 ↓
Read
 ↓
Analyze
 ↓
发现信息不足
 ↓
Search Again
 ↓
Synthesize
 ↓
Report
```

或者开发一个 Data Agent：

```text
User Question
 ↓
Agent
 ↓
Generate SQL
 ↓
Execute SQL
 ↓
Observe Result
 ↓
Fix SQL
 ↓
Analyze Data
 ↓
Answer
```

这些都属于 Agent 开发，他们关注的是 **LLM 下一步的决策方向**和**工具调用的颗粒度**，而不是重新实现一整套底层 Harness。

##### 2. 那为什么已经有 Claude Code / Codex，还需要开发 Agent？

著名的 Claude Code、Codex，他们都是**Coding Agent**，换言之，他们关注的是让 LLM 在软件工程环境中完成 Coding Task，所以模型在设计上是为了满足编写代码的要求。如果你只是需要让 AI 写代码，那么无论是 CC 还是 Codex，我相信都能满足你的愿望，无需自扰开发一整个 Agent。

如果现在要制定旅游攻略呢？如果现在要根据特定知识完成一个复杂研究呢？尽管上述 Agent 也有很强的通用能力，他们的流程设计上毕竟不是为了完成我们特殊需求而定制的，结果难免差强人意，于是我们就要针对任务开发具备独特流程、工具、数据能力的 Agent。

注意了，现在讨论的都是 Agent 开发，只有当现有 Runtime / Framework 无法满足需求，或者需要非常强的底层控制时，才有必要进一步开发自己的 Harness。

#### LangChain, LangGraph, LlamaIndex

##### 1. LangChain 是什么？

LangChain 可以理解为：

> **帮助开发者构建 LLM / Agent 应用的框架。**

如果完全自己实现 Agent，需要处理：

```text
调用 Model
 ↓
解析 Tool Call
 ↓
执行 Tool
 ↓
保存 Result
 ↓
更新 State
 ↓
再次调用 Model
 ↓
判断是否结束
```

LangChain 将其中很多常见模式进行了封装。

因此开发者可以把更多注意力放在：

```text
Agent 要解决什么问题？
 ↓
需要哪些 Tools？
 ↓
如何组织 Workflow？
 ↓
需要什么 Context？
 ↓
怎样评价结果？
```

而不是每次重新实现基础设施。

##### 2. LangGraph：当 Agent Loop 变复杂

简单 Agent 可以是：

```text
LLM → Tool → LLM → Tool → LLM
```

但复杂 Agent 可能出现：

```text
                 ┌→ Search
                 │
User → Planner ──┼→ Database
                 │
                 └→ Knowledge Base
                        ↓
                     Verify
                        ↓
                ┌───────┴───────┐
              Retry           Answer
```

这时候需要更加明确地管理：

- State
- Node
- Routing
- Retry
- Persistence
- Human-in-the-loop
- Long-running workflow

LangGraph 更接近这一层：

> **Stateful Agent Runtime / Workflow Orchestration**

##### 3. LlamaIndex：让 Agent 使用自己的数据

> 如果 Agent 需要查询几千份 PDF、Wiki、数据库和内部文档怎么办？

这就是 LlamaIndex 擅长的领域之一。

基本流程：

```text
Documents
 ↓
Parse / Chunk
 ↓
Embedding
 ↓
Index
 ↓
Retrieval
 ↓
Relevant Context
 ↓
LLM
```

也就是常见的 RAG：

> Retrieval-Augmented Generation

##### 4.粗略区分记忆：

```text
LangChain
→ 更偏 Agent / Tool / Application Orchestration

LangGraph
→ 更偏 Stateful Workflow / Agent Runtime

LlamaIndex
→ 更偏 Data / Retrieval / RAG
```

但三者的实际功能存在明显重叠，不能把这张分类表理解成严格边界。

#### 结语

到此为止，你应该了解了 Agent 的整个流程，知道了什么是 Agent 以及 Agent 开发的概念和基本框架，并且明白了为何我们要在已经存在 CC、Codex 此类 Agent 的情况下，依然选择开发一套 Agent，真正实现了从 LLM 到 Agent 的阶段式跨越。