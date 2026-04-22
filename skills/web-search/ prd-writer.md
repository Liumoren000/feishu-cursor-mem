---
name: prd-writer
description: Structures raw product ideas into executable PRDs using a staged workflow. Use when users mention product ideas, PRD writing, requirement documents, feature definition, requirement review, or incremental PRD updates.
---

# PRD Writer

## Purpose

Turn early product ideas into clear, implementable PRD content for non-PM users.

Use a staged process:
- First align direction
- Then expand implementation details
- Always validate from user value, business viability, and technical feasibility

Default response language: Chinese.

## Entry Rule (Step 0)

Before producing any PRD, ask:

`在开始之前，能先跟我说说你这个产品/功能的核心想法是什么吗？哪怕一两句话就行——你想解决什么问题，大概想用什么方式解决？`

Then classify:
- **Proceed**: User can explain "who has what problem and how to solve it"
- **Guide first**: Idea is vague but has direction
- **Pause**: No problem definition at all; suggest clarifying target user and pain point first

## Mode Detection

Select one mode:
- **Mode A (from zero)**: User has idea but no PRD
- **Mode B (review existing doc)**: User asks to evaluate/improve a PRD
- **Mode C (incremental update)**: User asks to add/modify requirements in an existing PRD

## Mode A - From Zero (Two Versions)

### 1) Three-Lens Diagnosis (mandatory)

Ask 1-2 questions at a time, conversationally. Do not dump a full questionnaire.

Cover three lenses:

- **User lens**
  - Current workaround?
  - Why current options are insufficient?
  - Any real user validation?

- **Business lens**
  - Revenue/business value?
  - Competitive alternatives and differentiation?
  - Rough target user size?

- **Engineering lens**
  - Core capability in-house or third-party dependent?
  - Hardest implementation risk?
  - MVP must-have features?

If product form is unclear, recommend one with reasons:
- Web App
- WeChat Mini Program
- Native App
- AI Skill / Plugin

Before continuing, summarize your understanding and ask for confirmation.

### 2) Output Version 1: Product Concept Doc

Goal: align direction only, keep concise.

Use this template:

```markdown
# 【产品名称】（暂定）

## 一句话定位
> 这是一个给【目标用户】用的【产品形态】，帮他们【解决什么问题】。
> 与现有方案相比，核心差异是【差异化优势】。

## 产品形态
- 当前选型：【网页 Web App / 微信小程序 / 原生 App / AI Skill】
- 选择理由：
- 阶段策略（如有）：

## 目标用户
- 核心用户画像（1-2 类）
- 他们的核心痛点
- 为什么会选择这个产品

## 产品价值
- 用户获得的价值
- 商业价值/变现逻辑

## 核心功能方向（只列方向，不展开细节）
- 功能方向 1
- 功能方向 2
- 功能方向 3

## 不做什么（边界）
- 超出范围的需求及原因

## 待确认问题
- 还需要用户回答的问题
```

After output, ask:
`这份概念版符合你的预期吗？产品定位、目标人群、核心方向——有没有哪里感觉不对？没问题了我们再展开细节。`

Only proceed to Version 2 after explicit user confirmation.

### 3) Output Version 2: Implementation PRD

If information is missing, mark `[待补充]` instead of guessing.

Required structure:

1. 产品概述（inherit from Version 1）
2. 目标用户与使用场景（2-3 scenarios）
3. 核心用户动线（Mermaid + 1-2 exception branches）
4. 功能清单（tree + priority: 🔴/🟡/⚪）
4.1 关键页面布局线框图（ASCII）
5. 功能详细描述（for each 🔴 core feature）
6. 文案规范（style + user-facing copy）
7. 非功能性需求（performance/security/compatibility/storage/permissions）
8. 待确认问题

For each core feature in section 5, include:
- 功能描述 + 触发条件
- 交互细节（反馈/危险确认/空状态/失败引导）
- 状态清单（默认/加载中/成功/失败/禁用/空状态）
- 边界条件（空/超长/网络/无权限/并发/格式异常）
- 数据规范（字段/类型/长度/必填/默认值/校验规则）

## Mode B - Evaluate Existing PRD

Score out of 10 and assess with ✅ / ⚠️ / ❌:

- **Direction layer**: user/pain point, one-line positioning, product form rationale, business value, boundaries
- **Structure layer**: user flow, prioritized feature list, key page wireframe, per-core-feature detail
- **Detail layer**: state coverage, edge cases, interaction detail, data schema, separate user-facing copy spec

Output order:
1. Overall score
2. Problems grouped by layer
3. Ask how user wants to revise next

## Mode C - Incremental Merge Update

Follow merge-not-overwrite principle:
1. Get current PRD
2. Clarify requested additions/changes
3. Identify impacted sections
4. Apply merged edits and mark each change with `【本次更新】`
5. Recheck consistency (flow/features/data/edge cases)

## Universal Rules

1. Never skip stages; no deep detail before concept alignment
2. Always pass all three lenses (user/business/engineering)
3. Proactively fill novice blind spots (interaction/data/copy)
4. Keep dual-audience writing separate (dev/AI vs end users)
5. Merge updates without erasing existing valid content
6. Prefer `[待补充]` over fabrication
7. Prefer diagrams/tables over long pure text where helpful


