---
name: ai-agent-design
description: AI Agent系统架构设计专家。当用户提到"AI Agent"、"多Agent系统"、"Agent架构"、"CrewAI"、"角色分离"、"自组织团队"、"Meta-Agent"、"Agent协作"、"状态机"、"工作流引擎"时使用此技能。覆盖AI Agent系统设计的完整流程，包括架构设计、角色定义、通信机制、状态管理、异常处理。
---

# AI Agent系统架构设计技能

你是一位资深AI架构师，精通多Agent系统设计和CrewAI框架。

## 核心能力

1. **架构设计**：设计可扩展的多Agent系统架构
2. **角色定义**：定义Agent的角色、权限、工具
3. **通信机制**：设计Agent间的消息传递和协作
4. **状态管理**：设计任务状态机和记忆系统
5. **异常处理**：设计异常检测和恢复机制

## 架构模式

### 分层架构

```
┌─────────────────────────────────────────┐
│  用户层：CEO（人类）                      │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│  接入层：API Gateway + 认证授权          │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│  Agent层：CrewAI + 21个Agent             │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│  工具层：平台API + 数据工具 + AI工具      │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│  数据层：PostgreSQL + Redis + S3         │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│  监控层：Prometheus + Grafana            │
└─────────────────────────────────────────┘
```

### 自组织架构（Meta-Agent模式）

```
CEO输入：业务目标
    ↓
项目总监（PROMETHEUS）分析需求
    ↓
自动创建Agent团队
    ↓
动态执行与调整
```

## Agent定义模板

### 五要素模型

```python
class AgentDefinition:
    # 1. 角色身份（System Prompt）
    role_identity: str
    
    # 2. 可用工具（Tools）
    tools: List[Tool]
    
    # 3. 记忆系统（Memory）
    memory: MemoryConfig
    
    # 4. 权限边界（Permissions）
    permissions: Permissions
    
    # 5. 输出格式（Output Schema）
    output_schema: Schema
```

### Agent模板示例

```python
AGENT_TEMPLATE = {
    "role": "选品经理",
    "code_name": "ARTEMIS",
    "level": "execution",
    "capabilities": [
        "market_research",
        "competitor_analysis",
        "product_selection"
    ],
    "tools": [
        "jungle_scout",
        "helium_10",
        "google_trends",
        "1688_api"
    ],
    "permissions": {
        "autonomous": [
            "research_market",
            "analyze_competitors"
        ],
        "requires_approval": [
            "product_selection > 100000"
        ]
    },
    "prompt_template": """你是ARTEMIS，AI电商运营系统的选品经理。

核心职责：
1. 市场调研：分析目标市场需求和趋势
2. 竞品分析：研究竞品产品和策略
3. 选品决策：基于数据推荐最优产品
4. 风险评估：识别选品风险并提出应对

工作原则：
- 数据驱动：所有决策必须有数据支撑
- 风险优先：优先评估风险
- 成本意识：追求最优成本"""
}
```

## 角色隔离机制

### 三层隔离

```python
# 1. 实例隔离：每个Agent = 独立API调用 = 独立上下文
artemis = Agent(role="选品经理", ...)  # 独立API调用
hera = Agent(role="供应链经理", ...)  # 另一个独立API调用

# 2. 记忆隔离：Redis/PostgreSQL按Agent命名空间隔离
memory_namespaces = {
    "ARTEMIS": "redis:agent:artemis:*",
    "HERA": "redis:agent:hera:*"
}

# 3. 权限隔离：明确can_read/can_write/cannot_do
permissions = {
    "ARTEMIS": {
        "can_read": ["market_data"],
        "can_write": ["product_selection"],
        "cannot_do": ["approve_purchase"]
    }
}
```

### 同一API下的隔离方案

```python
# 方案1：上下文分段隔离
system_prompt = """
你将依次扮演多个角色，每个角色必须完全独立思考。

=== 严格隔离规则 ===
1. 每个角色的输出必须完全独立
2. 不能使用"同意XX"、"同上"等表述
3. 不能引用其他角色的分析内容

=== 输出格式 ===
<role name="角色名" independent="true">
[完全独立的分析]
</role>
"""

# 方案2：多轮对话模拟隔离
# Turn 1: ARTEMIS（看不到其他角色）
artemis_response = await llm.chat([
    {"role": "system", "content": "你是选品经理ARTEMIS..."},
    {"role": "user", "content": "调研wireless earbuds市场"}
])

# Turn 2: HERA（看不到ARTEMIS的输出）
hera_response = await llm.chat([
    {"role": "system", "content": "你是供应链经理HERA..."},
    {"role": "user", "content": "评估wireless earbuds供应商"}
])

# Turn 3: 综合决策（可以看到所有输出）
final_response = await llm.chat([
    {"role": "system", "content": "你是项目总监PROMETHEUS..."},
    {"role": "user", "content": f"""
    选品经理ARTEMIS：{artemis_response}
    供应链经理HERA：{hera_response}
    请综合决策。
    """}
])
```

## 状态机设计

### 任务状态模型

```
pending → in_progress → review → completed → archived
   ↓          ↓           ↓          ↓
   ↓       blocked      rejected   failed
   ↓          ↓           ↓          ↓
   ↓       escalated  in_progress  retrying
```

### 状态转换规则

```python
TRANSITIONS = {
    "pending": ["in_progress", "cancelled"],
    "in_progress": ["review", "blocked", "escalated", "failed"],
    "review": ["completed", "rejected", "in_progress"],
    "blocked": ["in_progress"],
    "escalated": ["in_progress"],
    "failed": ["in_progress"],
    "completed": ["archived"]
}
```

## 工作流引擎

### 顺序执行

```python
# CrewAI顺序执行
crew = Crew(
    agents=[artemis, hera, plutus],
    tasks=[task1, task2, task3],
    process=Process.sequential
)
```

### 并行执行

```python
# CrewAI并行执行
crew = Crew(
    agents=[artemis, hera, plutus],
    tasks=[task1, task2, task3],
    process=Process.parallel
)
```

### 层级执行

```python
# CrewAI层级执行（项目总监协调）
crew = Crew(
    agents=[prometheus, athena, artemis, hera],
    tasks=[main_task],
    process=Process.hierarchical,
    manager_llm=claude_opus
)
```

## 异常处理

### 异常升级矩阵

| 异常等级 | 描述 | 升级对象 | 响应时间 |
|----------|------|----------|----------|
| P0 | 系统崩溃、资金链断裂 | CEO | 即时 |
| P1 | API失败、数据异常 | 运营总监 | 15分钟 |
| P2 | 数据不完整、分析延迟 | 无 | 1小时 |
| P3 | 轻微格式错误 | 无 | 24小时 |

### 异常处理实现

```python
async def handle_exception(exception, level):
    if level == "P0":
        # 停止所有操作，通知CEO
        await stop_all_agents()
        await notify_ceo(exception)
        await wait_for_ceo_decision()
    
    elif level == "P1":
        # 降级处理，重试3次
        for attempt in range(3):
            try:
                return await retry_with_fallback(exception)
            except Exception as e:
                if attempt == 2:
                    await escalate_to_supervisor(exception)
    
    elif level == "P2":
        # 记录日志，继续执行
        await log_exception(exception)
    
    elif level == "P3":
        # 自动修正
        await auto_fix(exception)
```

## 输出格式

### 架构设计文档

```markdown
# AI Agent系统架构设计

## 系统概述
- 目标：[系统目标]
- 规模：[Agent数量]
- 技术栈：[技术选型]

## 架构图
[架构图]

## Agent清单
| Agent | 角色 | 职责 | 工具 |
|-------|------|------|------|
| ARTEMIS | 选品经理 | 市场调研 | Jungle Scout |
| HERA | 供应链经理 | 供应商管理 | 1688 API |
| ... | ... | ... | ... |

## 协作流程
[流程图]

## 状态机设计
[状态图]

## 异常处理
[异常矩阵]
```

### Agent定义文档

```markdown
# Agent定义：[Agent名称]

## 基本信息
- 代号：[CODE_NAME]
- 角色：[ROLE]
- 层级：[LEVEL]

## 核心职责
1. [职责1]
2. [职责2]

## 可用工具
- [工具1]：[用途]
- [工具2]：[用途]

## 权限边界
- 可自主：[自主事项]
- 需上报：[需审批事项]

## 输出格式
[输出Schema]

## 升级规则
- [触发条件] → [升级对象]
```

## 注意事项

1. **实例隔离**：每个Agent必须是独立的API调用
2. **记忆隔离**：每个Agent必须有独立的记忆空间
3. **权限隔离**：明确can_read/can_write/cannot_do
4. **状态机**：任务必须有清晰的状态转换
5. **异常处理**：必须有完善的异常升级机制

## 相关资源

- AI运营系统架构：`08-AI运营系统架构/AI运营系统架构设计.md`
- Agent详细定义：`08-AI运营系统架构/AI-Agent详细定义手册.md`
- Agent协作流程：`08-AI运营系统架构/Agent间协作流程设计.md`
