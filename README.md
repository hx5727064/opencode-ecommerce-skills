# OpenCode E-commerce Operations Skill Collection

A comprehensive e-commerce operations skill collection for OpenCode AI assistants, covering the entire workflow from product selection to operations.

[中文版](#中文版) | [English](#english-version)

---

## English Version

### Overview

This repository contains 5 specialized AI agent skills for e-commerce operations, designed to work with OpenCode AI assistants. Each skill provides expert-level guidance for specific aspects of e-commerce operations.

### Skills Overview

| Skill | Description | Trigger Keywords |
|-------|-------------|------------------|
| `ecommerce-product-selection` | E-commerce product selection expert | product selection, market analysis, competitor analysis, trend forecasting |
| `ecommerce-ops-setup` | E-commerce team & system design expert | team building, role definition, process standardization |
| `ai-agent-design` | AI Agent architecture design expert | agent design, multi-agent collaboration, self-organizing architecture |
| `ecommerce-deployment` | E-commerce system deployment expert | Docker, deployment, operations, monitoring, alerting |
| `agent-collaboration` | AI Agent collaboration workflow expert | agent collaboration, message flow, task assignment, state machine |

### Installation

#### Method 1: Direct Installation (Recommended)

```bash
# Clone the repository
git clone https://github.com/hx5727064/opencode-ecommerce-skills.git

# Navigate to directory
cd opencode-ecommerce-skills

# Install to OpenCode
opencode skills install .
```

#### Method 2: Individual Skill Installation

```bash
# Install individual skills
opencode skills install ./ecommerce-product-selection
opencode skills install ./ecommerce-ops-setup
opencode skills install ./ai-agent-design
opencode skills install ./ecommerce-deployment
opencode skills install ./agent-collaboration
```

### Usage Examples

After installation, trigger skills using natural language in OpenCode:

```
# Product Selection
> Help me analyze the smart home product market in the US using the Six Forces Model

# Team Building
> Design a 5-person e-commerce team organizational structure

# Agent Architecture
> Design a self-organizing agent team for daily operations tasks

# System Deployment
> Deploy the AI operations system to a Docker environment

# Agent Collaboration
> Design a complete workflow from product selection → procurement → listing → promotion
```

### Skill Details

#### 1. E-commerce Product Selection (`ecommerce-product-selection`)

**Core Capabilities**:
- Six Forces Model Assessment (Market Size, Competition Level, Profit Margin, Supply Chain Difficulty, Trend Heat, Risk Level)
- Three Explosive Models (Explosive Product, Explosive Style, Explosive Point) Design
- Market Opportunity Scorecard
- Deep Competitor Analysis
- Risk Assessment Matrix
- Data-driven Decision Support

**Trigger Keywords**:
- product selection, product decision, product analysis
- market analysis, market research, market opportunity
- competitor analysis, competitor research
- trend forecasting, trend analysis
- product evaluation, product screening

#### 2. E-commerce Team & System Design (`ecommerce-ops-setup`)

**Core Capabilities**:
- Organizational Structure Design (Flat/Matrix/Agile Teams)
- Role & Responsibility Definition (From CEO to Operations Specialist)
- Core Role KPI Design
- Standard Operating Procedures (SOP) Design
- KPI System Design
- Team Training Plan

**Trigger Keywords**:
- team building, team design, organizational structure
- role setting, job description, role definition
- process standardization, SOP design
- KPI design, performance evaluation
- team training, talent development

#### 3. AI Agent Architecture Design (`ai-agent-design`)

**Core Capabilities**:
- Five-Component Agent Architecture (LLM + Memory + Planning + Tools + Action)
- Same-API Role Isolation Solutions
- Multi-Agent Collaboration Architecture
- Self-Organizing Agent Teams (Meta-Agent Pattern)
- State Machine Design
- Exception Handling Mechanism
- Cost Optimization Strategy

**Trigger Keywords**:
- agent design, agent architecture
- multi-agent collaboration, multi-agent system
- self-organizing architecture, meta-agent
- role isolation, agent isolation
- state machine, workflow design

#### 4. E-commerce System Deployment (`ecommerce-deployment`)

**Core Capabilities**:
- Docker Container Deployment
- Docker Compose Service Orchestration
- Environment Variable Configuration
- Monitoring & Alerting (Prometheus + Grafana)
- Backup & Recovery Strategy
- Security Configuration
- Performance Optimization

**Trigger Keywords**:
- deployment, Docker, Docker Compose
- operations, environment configuration, server setup
- monitoring, alerting, Prometheus, Grafana
- backup, recovery, data persistence
- security configuration, SSL certificates

#### 5. Agent Collaboration Workflow (`agent-collaboration`)

**Core Capabilities**:
- Message Flow Specification Design
- Task State Machine Design
- Parallel Execution Engine
- Exception Escalation Mechanism
- Collaboration Monitoring & Audit
- Cross-Agent Communication Protocol

**Trigger Keywords**:
- agent collaboration, agent communication
- message flow, message passing
- task assignment, task scheduling
- state machine, state management
- parallel execution, asynchronous execution
- exception escalation, exception handling

### System Requirements

- OpenCode v0.1.0+
- Python 3.8+
- Node.js 16+

### Related Projects

- [AI Operations System Architecture](https://github.com/hx5727064/ai-ecommerce-ops) - Complete AI operations system implementation
- [E-commerce Operations Documentation](https://github.com/hx5727064/ecommerce-operations-docs) - Complete e-commerce operations documentation set

### Contributing

Contributions are welcome! Please create SKILL.md files following this format:

```yaml
---
name: your-skill-name
description: Skill description with trigger keywords
---

# Skill Name

## Core Capabilities
1. Capability 1
2. Capability 2
3. Capability 3

## Trigger Keywords
- keyword1
- keyword2
- keyword3

## Output Format
### Format 1
[Template]

### Format 2
[Template]

## Best Practices
- Practice 1
- Practice 2

## Important Notes
- Note 1
- Note 2
```

### License

MIT License

### Author

AI Operations System Team

---

## 中文版

### 概述

本仓库包含5个专门用于电商运营的AI Agent技能，专为OpenCode AI助手设计，涵盖从选品到运营的全流程。

### 技能列表

| 技能 | 描述 | 触发关键词 |
|------|------|------------|
| `ecommerce-product-selection` | 电商选品决策专家 | 选品、市场分析、竞品分析、趋势预测 |
| `ecommerce-ops-setup` | 电商团队与系统设计专家 | 团队组建、岗位设置、流程标准化 |
| `ai-agent-design` | AI Agent架构设计专家 | Agent设计、多Agent协作、自组织架构 |
| `ecommerce-deployment` | 电商系统部署运维专家 | Docker、部署、运维、监控、告警 |
| `agent-collaboration` | AI Agent协作流程专家 | Agent协作、消息流转、任务分配、状态机 |

### 安装方法

#### 方法1：直接安装（推荐）

```bash
# 克隆仓库
git clone https://github.com/hx5727064/opencode-ecommerce-skills.git

# 进入目录
cd opencode-ecommerce-skills

# 安装到OpenCode
opencode skills install .
```

#### 方法2：单个技能安装

```bash
# 安装单个技能
opencode skills install ./ecommerce-product-selection
opencode skills install ./ecommerce-ops-setup
opencode skills install ./ai-agent-design
opencode skills install ./ecommerce-deployment
opencode skills install ./agent-collaboration
```

### 使用方法

安装后，在OpenCode中直接使用自然语言触发技能：

```
# 选品决策
> 帮我分析美国市场的智能家居产品，用六力模型评估

# 团队组建
> 我要搭建一个5人电商团队，请设计组织架构

# Agent架构设计
> 设计一个自组织的Agent团队来处理日常运营任务

# 系统部署
> 帮我部署AI运营系统到服务器

# Agent协作
> 设计选品→采购→上架→推广的完整协作流程
```

### 技能详情

#### 1. 电商选品决策 (`ecommerce-product-selection`)

**核心能力**：
- 六力模型评估（市场容量、竞争程度、利润空间、供应链难度、趋势热度、风险等级）
- 三爆模型（爆品、爆款、爆点）设计
- 市场机会评分卡
- 竞品深度分析
- 风险评估矩阵
- 数据驱动决策支持

**触发关键词**：
- 选品、选品决策、选品分析
- 市场分析、市场调研、市场机会
- 竞品分析、竞品调研
- 趋势预测、趋势分析
- 产品评估、产品筛选

#### 2. 电商团队与系统设计 (`ecommerce-ops-setup`)

**核心能力**：
- 组织架构设计（扁平化/矩阵式/敏捷小组）
- 岗位职责定义（从CEO到运营专员）
- 核心岗位核心指标设计
- 标准化流程设计（SOP）
- KPI体系设计
- 团队培训计划

**触发关键词**：
- 团队组建、团队设计、组织架构
- 岗位设置、岗位职责、岗位说明
- 流程标准化、SOP设计
- KPI设计、绩效考核
- 团队培训、人才培养

#### 3. AI Agent架构设计 (`ai-agent-design`)

**核心能力**：
- 五组件Agent架构设计（LLM+记忆+规划+工具+行动）
- 同一API下的角色隔离方案
- 多Agent协作架构
- 自组织Agent团队（Meta-Agent模式）
- 状态机设计
- 异常处理机制
- 成本优化策略

**触发关键词**：
- Agent设计、Agent架构
- 多Agent协作、多Agent系统
- 自组织架构、Meta-Agent
- 角色隔离、Agent隔离
- 状态机、工作流设计

#### 4. 电商系统部署 (`ecommerce-deployment`)

**核心能力**：
- Docker容器化部署
- Docker Compose服务编排
- 环境变量配置
- 监控告警配置（Prometheus+Grafana）
- 备份恢复策略
- 安全配置
- 性能优化

**触发关键词**：
- 部署、Docker、Docker Compose
- 运维、环境配置、服务器配置
- 监控、告警、Prometheus、Grafana
- 备份、恢复、数据持久化
- 安全配置、SSL证书

#### 5. Agent协作流程 (`agent-collaboration`)

**核心能力**：
- 消息流转规范设计
- 任务状态机设计
- 并行执行引擎
- 异常升级机制
- 协作监控审计
- 跨Agent通信协议

**触发关键词**：
- Agent协作、Agent通信
- 消息流转、消息传递
- 任务分配、任务调度
- 状态机、状态管理
- 并行执行、异步执行
- 异常升级、异常处理

### 系统要求

- OpenCode v0.1.0+
- Python 3.8+
- Node.js 16+

### 相关项目

- [AI运营系统架构](https://github.com/hx5727064/ai-ecommerce-ops) - 完整的AI运营系统实现
- [电商运营文档](https://github.com/hx5727064/ecommerce-operations-docs) - 电商运营完整文档集

### 贡献指南

欢迎贡献新的技能！请按照以下格式创建SKILL.md文件：

```yaml
---
name: your-skill-name
description: 技能描述，包含触发关键词
---

# 技能名称

## 核心能力
1. 能力1
2. 能力2
3. 能力3

## 触发关键词
- 关键词1
- 关键词2
- 关键词3

## 输出格式
### 格式1
[模板]

### 格式2
[模板]

## 最佳实践
- 实践1
- 实践2

## 注意事项
- 注意1
- 注意2
```

### 许可证

MIT License

### 作者

AI运营系统团队
