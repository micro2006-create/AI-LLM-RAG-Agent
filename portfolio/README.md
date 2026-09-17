# 作品集：基于 ReAct Agent 的扫地机器人智能客服系统

## 项目信息

| 项目 | 内容 |
|------|------|
| **项目名称** | 基于 ReAct Agent 的扫地机器人智能客服系统 |
| **GitHub 仓库** | https://github.com/micro2006-create/AI-LLM-RAG-Agent |
| **技术栈** | Python / LangChain / LangGraph / Chroma / DashScope / PyYAML / Streamlit |
| **项目类型** | 个人项目 |

## 项目简介

面向扫地机器人品牌的智能客服系统，采用 LangChain ReAct Agent 架构，集成 6 个业务工具（向量知识库检索、天气查询、用户位置查询、用户ID查询、当前月份查询、外部CSV数据查询），通过 3 层中间件链实现工具调用监控、模型调用日志记录与动态提示词切换。系统根据用户提问自动路由到对应工具链完成多步推理，支持流式输出与多轮会话，并能基于外部数据动态生成用户月度效率报告。

## 核心功能

1. **ReAct Agent**：基于 `create_agent` 构建 ReAct 推理框架，集成 6 个 `@tool` 工具，LLM 自主决策工具调用链路
2. **3 层中间件链**：`monitor_tool`（工具监控）+ `log_before_model`（模型日志）+ `report_prompt_switch`（动态 Prompt 切换）
3. **RAG 检索增强**：Chroma 向量库存储扫地机器人知识库（100问/故障排除/维护保养/选购指南），k=3 检索 top-3 相似片段
4. **MD5 文件级去重**：避免重复文档入库
5. **YAML 配置驱动**：模型参数、分块策略、Prompt 路径均通过配置文件管理
6. **流式输出**：`stream_mode="values"` 实现 Agent 推理过程实时渲染

## 运行截图

### 1. 初始页面

系统启动后的智能客服界面：

![初始页面](https://cdn.jsdelivr.net/gh/micro2006-create/AI-LLM-RAG-Agent@main/portfolio/images/01_initial.png)

### 2. 测试问题一：故障排除（RAG 检索）

**用户提问**：扫地机器人无法开机怎么办？

**系统行为**：Agent 自主调用 `rag_summarize` 工具，从 Chroma 向量库中检索 top-3 相关知识片段（k=3），基于检索结果生成故障排除步骤。

![故障排除问答](https://cdn.jsdelivr.net/gh/micro2006-create/AI-LLM-RAG-Agent@main/portfolio/images/02_question_troubleshoot.png)

### 3. 测试问题二：天气查询（工具调用）

**用户提问**：帮我查一下北京今天的天气

**系统行为**：Agent 自主调用 `get_weather` 工具查询北京天气，返回天气信息并组织自然语言回复。

![天气查询问答](https://cdn.jsdelivr.net/gh/micro2006-create/AI-LLM-RAG-Agent@main/portfolio/images/03_question_weather.png)

### 4. 测试问题三：月度效率报告（多步工具链 + 动态 Prompt 切换）

**用户提问**：帮我生成用户U001的月度效率报告

**系统行为**：Agent 链式调用 `get_user_id` → `get_current_month` → `fetch_external_data`，中间件 `report_prompt_switch` 检测到报告生成场景，自动切换到报告生成 Prompt，最终输出结构化月度效率报告。

![月度效率报告问答](https://cdn.jsdelivr.net/gh/micro2006-create/AI-LLM-RAG-Agent@main/portfolio/images/04_question_report.png)

## 技术架构

```
用户提问 → Streamlit 前端
    ↓
LangChain ReAct Agent (create_agent)
    ↓
3 层中间件链:
  ├── monitor_tool (工具入参与结果监控)
  ├── log_before_model (模型调用日志)
  └── report_prompt_switch (动态 Prompt 切换)
    ↓
6 个 @tool 工具:
  ├── rag_summarize (向量知识库检索, k=3)
  ├── get_weather (天气查询)
  ├── get_location (用户位置查询)
  ├── get_user_id (用户ID查询)
  ├── get_current_month (当前月份查询)
  └── fetch_external_data (外部CSV数据查询)
    ↓
stream_mode="values" 流式输出 → Streamlit 前端
```

## 知识库数据

| 文件 | 内容 |
|------|------|
| `data/扫地机器人100问.pdf` | 扫地机器人常见问题100问 |
| `data/扫地机器人100问.txt` | 上述PDF的文本版本 |
| `data/故障排除.txt` | 常见故障及排除方法 |
| `data/维护保养.txt` | 日常维护保养指南 |
| `data/选购指南.txt` | 扫地机器人选购建议 |
| `data/external/records.csv` | 外部用户使用数据（用于报告生成） |
