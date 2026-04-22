---
name: requirement-testcase-generator
description: 根据需求文档生成测试用例并输出 Markdown、JSON、Excel(.xlsx)。当用户提到“需求文档”“测试用例设计”“测试点覆盖”“正向/逆向/异常/并发场景”或要求导出 md/json/xlsx 时使用。
---

# Requirement Testcase Generator

## 适用场景

在以下情况使用本技能：
- 用户要求“根据需求文档生成测试用例”
- 用户要求覆盖正向、逆向、异常、并发场景
- 用户要求同时导出 `Markdown`、`JSON`、`Excel(.xlsx)`

## 目标

从需求文档中提取可测试需求，生成结构化测试用例，保证四类覆盖完整：
- 正向（Happy Path）
- 逆向（Invalid / Negative Input）
- 异常（系统异常、依赖异常、边界失败）
- 并发（竞态、重复提交、顺序冲突、幂等、超时与重试、最终一致性）

并输出三份等价结果：
1. `testcases.md`
2. `testcases.json`
3. `testcases.xlsx`

## 执行流程

按以下顺序执行，不要跳步：

1. 解析需求文档
   - 提取功能点、业务规则、数据约束、状态流转、外部依赖。
   - 标记不明确项并形成“待确认假设”。

2. 拆分测试对象
   - 每个功能点拆分为最小可验证行为（一个行为可映射多条用例）。
   - 为每个行为定义前置条件与可观测结果。

3. 设计四类场景
   - 正向：主流程成功、典型有效输入。
   - 逆向：非法输入、缺字段、越界、格式错误、权限不足。
   - 异常：依赖服务失败、网络抖动、数据库异常、事务回滚。
   - 并发：重复提交、并发更新、乱序到达、锁竞争、重试风暴。

4. 生成统一用例模型
   - 每条用例使用同一数据结构（见“JSON 模板”）。
   - `id` 必须唯一且稳定（如 `TC-模块-序号`）。
   - `priority` 使用 `P0/P1/P2`。
   - `type` 仅允许：`positive | negative | exception | concurrency`。

5. 输出三种格式
   - 先产出 `testcases.json`（作为单一事实源）。
   - 由 JSON 映射生成 `testcases.md`。
   - 由 JSON 生成 `testcases.xlsx`，列顺序与 Markdown 一致。

6. 自检
   - 检查每个需求点是否至少被 1 条用例覆盖。
   - 检查四类场景是否都存在，且并发场景不少于总数的 15%（需求涉及状态写入时）。
   - 检查三份输出条目数、`id` 集合完全一致。

## 覆盖规则（严格并发）

当需求涉及“写操作、状态变化、库存/额度、支付、审批、任务调度”任一项时，至少覆盖：
- 同一资源并发写（最后写入覆盖/版本冲突）
- 重复请求幂等（同一幂等键）
- 乱序请求处理（先后依赖被打乱）
- 超时后重试（客户端重试与服务端去重）
- 部分成功与最终一致性（异步补偿）

## 输出模板

### Markdown 模板（testcases.md）

使用以下列：

`ID | 需求点 | 场景类型 | 标题 | 前置条件 | 测试步骤 | 预期结果 | 优先级 | 标签`

示例：

```markdown
| ID | 需求点 | 场景类型 | 标题 | 前置条件 | 测试步骤 | 预期结果 | 优先级 | 标签 |
|---|---|---|---|---|---|---|---|---|
| TC-LOGIN-001 | 用户登录 | positive | 正确账号密码登录成功 | 账号已注册且启用 | 1. 输入正确账号密码 2. 点击登录 | 跳转首页并创建会话 | P0 | auth,smoke |
```

### JSON 模板（testcases.json）

```json
{
  "meta": {
    "source": "需求文档标题或链接",
    "generatedAt": "ISO-8601",
    "assumptions": []
  },
  "testcases": [
    {
      "id": "TC-LOGIN-001",
      "requirement": "用户登录",
      "type": "positive",
      "title": "正确账号密码登录成功",
      "preconditions": ["账号已注册", "账号状态=启用"],
      "steps": ["输入正确账号密码", "点击登录"],
      "expected": ["跳转首页", "创建有效会话"],
      "priority": "P0",
      "tags": ["auth", "smoke"]
    }
  ]
}
```

### Excel 模板（testcases.xlsx）

Sheet 名：`testcases`  
列顺序必须为：

`id, requirement, type, title, preconditions, steps, expected, priority, tags`

字段序列化规则：
- `preconditions`、`steps`、`expected`、`tags` 为数组时，写入单元格时使用 `\n` 连接。
- 第一行冻结；开启自动筛选；列宽自适应（最小 12，最大 60）。

## 质量门槛

生成结果必须满足：
- 无重复 `id`
- 无空 `expected`
- 每条用例至少 2 个步骤（冒烟类可例外）
- 并发用例必须明确并发主体、冲突资源、判定标准
- 三种输出总数一致

## 失败处理

若需求信息不足，不要停止。先输出：
1. 已确认需求点
2. 待确认问题清单
3. 基于当前假设的可执行用例（在 `meta.assumptions` 标明）

## 交付清单

默认交付以下文件（同一目录）：
- `testcases.md`
- `testcases.json`
- `testcases.xlsx`
