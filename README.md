```markdown
# LLM 自动化评估 Agent

## 项目简介
本项目实现了一个**自动化多模型评估 Agent**，用于客观对比不同大语言模型在代码生成、调试、解释任务上的表现。Agent 能够自动调度多个模型 API，执行标准化测试，并生成对比报告。

## 核心痛点
- **手动评估耗时**：人工逐个模型跑测试，一天仅能完成 10–20 个用例，样本量不足。
- **结果主观**：不同人的评判标准不一，缺乏可复现的量化指标。
- **难以覆盖长尾场景**：简单的“写个快排”无法体现模型在复杂代码理解上的差异。

## 核心逻辑流（长链推理 + 多 Agent 协作）

### 长链推理流程
Agent 执行以下 5 步推理链：
1. **任务解析**：从测试集中读取一个代码任务（如“实现带超时的 LRU 缓存”）。
2. **并行调用**：由 Executor Agent 同时向 GPT-4、Claude 3.5 Sonnet、DeepSeek-V3 发送相同 Prompt。
3. **输出收集**：等待所有模型返回代码片段。
4. **自动评分**：由 Judge Agent 运行单元测试（pytest）及静态分析（圈复杂度、注释率）。
5. **报告生成**：由 Reporter Agent 汇总通过率、平均耗时、Token 消耗并输出 Markdown 对比表。

### 多 Agent 协作
- **Executor Agent**：负责调用模型 API 并收集原始输出。
- **Judge Agent**：对代码输出进行自动化评分。
- **Reporter Agent**：生成可视化报告和趋势图。

### Agent 工作流图
```mermaid
graph TD
    A[测试任务集] --> B[Executor Agent]
    B --> C[调用 GPT-4 API]
    B --> D[调用 Claude 3.5 API]
    B --> E[调用 DeepSeek-V3 API]
    C --> F[Judge Agent]
    D --> F
    E --> F
    F --> G[运行单元测试]
    G --> H[计算通过率 & Token消耗]
    H --> I[Reporter Agent]
    I --> J[生成对比报告]
成果与 Token 消耗
下图展示了本 Agent 在开发测试阶段使用 GPT-4 API 的日消耗 Token 趋势（2026年4月28日-5月16日），总消耗约 180 万 Token：

https://token_usage.png

完整运行日志参见 run_agent.log。初步结果显示：GPT-4 在复杂逻辑理解上通过率较高，Claude 3.5 在代码注释生成上更详细，DeepSeek-V3 在边缘用例处理上存在不足。

后续计划
申请到小米 MiMo 百万亿 Token 后，将把 MiMo-V2.5 加入评估池，扩展测试集到 200+ 用例，并开源完整报告。
