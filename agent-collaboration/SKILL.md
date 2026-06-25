---
name: agent-collaboration
description: AI Agent协作流程专家。当用户提到"Agent协作"、"消息流转"、"任务分配"、"状态机"、"并行执行"、"异常升级"、"工作流"、"任务调度"、"多Agent协调"、"跨Agent通信"时使用此技能。覆盖Agent间协作的完整流程，包括消息规范、状态管理、并行执行、异常处理、监控审计。
---

# Agent协作流程设计技能

你是一位资深系统架构师，精通分布式系统和多Agent协作。

## 核心能力

1. **消息规范**：设计Agent间的消息格式和流转
2. **状态管理**：设计任务状态机和生命周期
3. **并行执行**：设计并行任务识别和执行引擎
4. **异常处理**：设计异常检测和升级机制
5. **监控审计**：设计协作监控和审计日志

## 消息流转规范

### 消息格式

```json
{
  "message_id": "MSG-2026-001",
  "message_type": "task_assignment|task_update|task_complete|escalation|notification",
  "from_agent": "PROMETHEUS",
  "to_agent": "ARTEMIS",
  "task_id": "TASK-2026-001",
  "priority": "high|medium|low",
  "deadline": "2026-06-30T23:59:59Z",
  "payload": {
    "action": "execute_product_selection",
    "data": {
      "market": "US",
      "category": "Electronics"
    }
  },
  "metadata": {
    "created_at": "2026-06-25T10:00:00Z",
    "ttl": 3600,
    "retry_count": 0
  }
}
```

### 消息类型

| 类型 | 说明 | 发送方 | 接收方 |
|------|------|--------|--------|
| task_assignment | 任务分配 | 项目总监 | 执行Agent |
| task_update | 进度更新 | 执行Agent | 任务分配者 |
| task_complete | 任务完成 | 执行Agent | 任务分配者 |
| task_failed | 任务失败 | 执行Agent | 任务分配者 |
| escalation | 异常升级 | 执行Agent | 上级Agent |
| notification | 通知消息 | 任意Agent | 任意Agent |
| data_request | 数据请求 | 任意Agent | 数据分析师 |
| data_response | 数据响应 | 数据分析师 | 请求方 |

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

### 状态机实现

```python
from enum import Enum
from datetime import datetime

class TaskStatus(Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    REVIEW = "review"
    COMPLETED = "completed"
    ARCHIVED = "archived"
    BLOCKED = "blocked"
    ESCALATED = "escalated"
    FAILED = "failed"
    REJECTED = "rejected"
    CANCELLED = "cancelled"

class TaskStateMachine:
    def __init__(self):
        self.transitions = TRANSITIONS
        self.on_enter = {
            "in_progress": [self._log_start],
            "completed": [self._log_complete],
            "failed": [self._log_failure],
            "escalated": [self._notify_supervisor],
        }
    
    def can_transition(self, current, target):
        return target in self.transitions.get(current, [])
    
    def transition(self, task, new_status):
        if not self.can_transition(task.status, new_status):
            return False
        
        old_status = task.status
        task.status = new_status
        task.updated_at = datetime.now()
        
        for callback in self.on_enter.get(new_status, []):
            callback(task, old_status)
        
        return True
```

## 并行执行设计

### 并行任务识别

```python
def identify_parallel_tasks(tasks):
    """识别可并行执行的任务"""
    
    # 构建依赖图
    dependency_graph = build_dependency_graph(tasks)
    
    # 拓扑排序
    parallel_groups = []
    visited = set()
    
    def dfs(task, current_group):
        if task.task_id in visited:
            return
        
        dependencies = dependency_graph.get(task.task_id, [])
        if all(dep in visited for dep in dependencies):
            current_group.append(task)
        else:
            if current_group:
                parallel_groups.append(current_group)
            current_group = []
            for dep in dependencies:
                if dep not in visited:
                    dfs(get_task_by_id(dep, tasks), current_group)
            current_group.append(task)
        
        visited.add(task.task_id)
    
    for task in tasks:
        dfs(task, [])
    
    return parallel_groups
```

### 并行执行引擎

```python
import asyncio

class ParallelExecutor:
    def __init__(self, max_concurrent=10):
        self.max_concurrent = max_concurrent
        self.semaphore = asyncio.Semaphore(max_concurrent)
    
    async def execute_parallel(self, tasks):
        """并行执行任务组"""
        
        async def run_task(task):
            async with self.semaphore:
                try:
                    task.status = TaskStatus.RUNNING
                    task.result = await task.func(*task.args, **task.kwargs)
                    task.status = TaskStatus.COMPLETED
                except Exception as e:
                    task.error = e
                    task.status = TaskStatus.FAILED
        
        await asyncio.gather(*[run_task(task) for task in tasks])
        
        return [task.result if task.status == TaskStatus.COMPLETED else None 
                for task in tasks]
    
    async def execute_sequential_groups(self, groups):
        """按组顺序执行，组内并行"""
        
        all_results = []
        
        for group in groups:
            group_results = await self.execute_parallel(group)
            all_results.append(group_results)
        
        return all_results
```

## 异常处理流程

### 异常升级矩阵

| 异常等级 | 描述 | 升级对象 | 响应时间 |
|----------|------|----------|----------|
| P0 | 系统崩溃、资金链断裂、平台封号 | CEO | 即时 |
| P1 | API失败、数据异常、供应商违约 | 运营总监 | 15分钟 |
| P2 | 数据不完整、分析延迟 | 无 | 1小时 |
| P3 | 轻微格式错误、非关键数据缺失 | 无 | 24小时 |

### 异常处理流程

```python
class ExceptionHandler:
    def __init__(self):
        self.handlers = {
            "P0": self._handle_p0,
            "P1": self._handle_p1,
            "P2": self._handle_p2,
            "P3": self._handle_p3,
        }
    
    async def handle(self, exception, level):
        handler = self.handlers.get(level)
        if handler:
            return await handler(exception)
        return False
    
    async def _handle_p0(self, exception):
        """P0级：立即停止所有操作，通知CEO"""
        await self._stop_all_agents()
        await self._notify_ceo(exception)
        await self._wait_for_ceo_decision()
        return True
    
    async def _handle_p1(self, exception):
        """P1级：降级处理+重试"""
        for attempt in range(3):
            try:
                result = await self._retry_with_fallback(exception)
                if result:
                    return True
            except Exception as e:
                if attempt == 2:
                    await self._escalate_to_supervisor(exception)
        return False
    
    async def _handle_p2(self, exception):
        """P2级：记录日志，继续执行"""
        await self._log_exception(exception)
        return True
    
    async def _handle_p3(self, exception):
        """P3级：自动修正"""
        await self._auto_fix(exception)
        return True
```

## 协作监控

### 监控指标

| 指标 | 说明 | 目标值 |
|------|------|--------|
| 任务完成率 | 任务成功完成的比例 | >95% |
| 平均任务周期 | 从任务创建到完成的时间 | <24小时 |
| 并行执行率 | 并行执行的任务比例 | >60% |
| 异常处理及时率 | 异常在规定时间内处理 | >90% |
| 消息传递延迟 | 消息从发送到接收的时间 | <1秒 |

### 监控仪表盘

```python
dashboard_config = {
    "panels": [
        {
            "title": "任务状态分布",
            "type": "piechart",
            "targets": [{
                "expr": "sum by (status) (agent_tasks_total)"
            }]
        },
        {
            "title": "任务完成率趋势",
            "type": "graph",
            "targets": [{
                "expr": "sum(rate(agent_tasks_completed[5m])) / sum(rate(agent_tasks_total[5m])) * 100"
            }]
        },
        {
            "title": "Agent响应时间",
            "type": "graph",
            "targets": [{
                "expr": "histogram_quantile(0.95, rate(agent_response_time_bucket[5m]))"
            }]
        },
        {
            "title": "异常处理统计",
            "type": "barchart",
            "targets": [{
                "expr": "sum by (level) (agent_exceptions_total)"
            }]
        }
    ]
}
```

## 输出格式

### 协作流程文档

```markdown
# Agent协作流程设计

## 协作原则
- 任务驱动：所有协作以任务为单位
- 消息传递：Agent间通过消息队列通信
- 异步执行：任务执行异步化
- 状态跟踪：每个任务有完整的状态跟踪

## 消息规范
- 消息格式：[格式]
- 消息类型：[类型列表]
- 消息示例：[示例]

## 状态机设计
- 状态定义：[状态列表]
- 转换规则：[规则]
- 实现代码：[代码]

## 并行执行
- 并行识别：[方法]
- 执行引擎：[实现]
- 使用示例：[示例]

## 异常处理
- 升级矩阵：[矩阵]
- 处理流程：[流程]
- 实现代码：[代码]

## 监控指标
- 指标定义：[指标]
- 告警规则：[规则]
- 仪表盘：[配置]
```

## 协作流程示例

### 选品→采购→上架→推广→监控

```python
# 1. 项目总监发起任务
task_prometheus = {
    "task_id": "TASK-2026-010",
    "action": "new_product_launch",
    "assigned_to": "ATHENA",
    "deadline": "2026-07-15"
}

# 2. 运营总监分解任务
task_athena = [
    {"task_id": "TASK-2026-011", "action": "product_selection", "assigned_to": "ARTEMIS"},
    {"task_id": "TASK-2026-012", "action": "supply_chain_setup", "assigned_to": "HERA"},
    {"task_id": "TASK-2026-013", "action": "marketing_plan", "assigned_to": "ARES"},
]

# 3. 选品经理执行（并行调用数据分析师和供应链）
task_artemis = [
    {"task_id": "TASK-2026-014", "action": "market_analysis", "assigned_to": "APOLLO"},
    {"task_id": "TASK-2026-015", "action": "supplier_evaluation", "assigned_to": "HERA"},
    {"task_id": "TASK-2026-016", "action": "cost_analysis", "assigned_to": "PLUTUS"},
]

# 4. 供应链执行采购
task_hera = [
    {"task_id": "TASK-2026-017", "action": "place_order", "assigned_to": "HERA"},
    {"task_id": "TASK-2026-018", "action": "quality_check", "assigned_to": "HEPHAESTUS_QC"},
    {"task_id": "TASK-2026-019", "action": "warehouse_inventory", "assigned_to": "HERMES_WAR"},
]

# 5. 运营主管执行上架
task_atlas = [
    {"task_id": "TASK-2026-020", "action": "product_images", "assigned_to": "HEPHAESTUS"},
    {"task_id": "TASK-2026-021", "action": "product_copy", "assigned_to": "APOLLO_CONTENT"},
    {"task_id": "TASK-2026-022", "action": "list_product", "assigned_to": "ATLAS"},
]

# 6. 广告投手执行推广
task_ares = [
    {"task_id": "TASK-2026-023", "action": "ad_campaign", "assigned_to": "ARES"},
    {"task_id": "TASK-2026-024", "action": "video_content", "assigned_to": "MUSES"},
    {"task_id": "TASK-2026-025", "action": "live_stream", "assigned_to": "DIONYSUS"},
]

# 7. 数据分析师监控
task_apollo = [
    {"task_id": "TASK-2026-026", "action": "performance_monitor", "assigned_to": "APOLLO"},
    {"task_id": "TASK-2026-027", "action": "anomaly_detection", "assigned_to": "APOLLO"},
]
```

## 最佳实践

### 任务设计原则
1. **单一职责**：每个任务只做一件事
2. **明确输入输出**：任务的输入输出要清晰定义
3. **幂等性**：任务可以安全重试
4. **超时设置**：每个任务都要设置超时时间
5. **异常处理**：任务要处理所有可能的异常

### 消息设计原则
1. **自描述性**：消息要包含足够的上下文
2. **版本控制**：消息格式要支持版本演进
3. **幂等性**：消息处理要支持幂等
4. **最小化**：消息只包含必要信息
5. **可追溯**：消息要有完整的链路追踪

## 注意事项

1. **消息可靠**：确保消息不丢失
2. **状态一致**：确保状态与实际一致
3. **异常处理**：必须有完善的异常升级机制
4. **监控审计**：记录所有协作过程
5. **性能优化**：优化并行执行效率

## 相关资源

- Agent协作流程：`08-AI运营系统架构/Agent间协作流程设计.md`
- AI运营系统架构：`08-AI运营系统架构/AI运营系统架构设计.md`
- Agent详细定义：`08-AI运营系统架构/AI-Agent详细定义手册.md`
