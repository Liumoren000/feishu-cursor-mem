# 中医药全链路追溯系统 PRD (v2.0 深度细化版)

## 1. 修订记录
| 版本 | 日期 | 描述 | 作者 |
| :--- | :--- | :--- | :--- |
| v1.0 | 2026-04-22 | 初稿：涵盖基础业务流程 | 需求文档机器人 |
| v2.0 | 2026-04-22 | 深度细化：补充数据建模、接口定义、防伪算法及业务细节 | 需求文档机器人 |

## 2. 系统架构设计
### 2.1 技术栈建议
- **前端**：Next.js + Tailwind CSS + Lucide Icons (响应式 H5 + B 端管理后台)
- **后端**：Node.js (Next.js API Routes) + Prisma ORM
- **数据库**：PostgreSQL (核心业务) + Redis (扫码频率控制与缓存)
- **存储**：阿里云 OSS / AWS S3 (用于确权文件、质检报告、视频存证)

### 2.2 核心实体关系 (ERD)
- **Company (企业)**: `id, name, license_no, address, status, created_at`
- **Product (产品)**: `id, company_id, name, specification, unit, gmp_certified (bool)`
- **Batch (批次)**: `id, product_id, batch_no, origin_location (JSON: lat/lng/desc), harvest_date, processing_method, test_report_url`
- **TraceCode (溯源码)**: `id, batch_id, sn (流水号), security_code (加密防伪码), status (active/inactive), scan_count, first_scan_time, last_scan_location`

## 3. 功能需求细化

### 3.1 B 端：赋码与供应链管理

#### 3.1.1 赋码生成算法逻辑
- **溯源码组成**：`{Domain}/{Type}/{ProductID}/{BatchID}/{SN}{VerificationCode}`
- **VerificationCode 算法**：基于 `BatchID + SN + SecretKey` 的 HMAC-SHA256 截断值，确保无法通过 SN 推测防伪码。
- **状态管理**：
  - **待激活**：码已生成但未关联生产数据。
  - **已激活**：码已绑定批次并正式出库。
  - **已作废**：因退货或损坏人工标记。

#### 3.1.2 生产环节数据采集
- **输入要求**：
  - 种植环节：肥料/农药使用记录（可选）、气象数据（接入 API）。
  - 加工环节：炮制时长、温度记录、操作工 ID。
  - 质检环节：必须上传第三方检测机构盖章的 PDF。

#### 3.1.3 物流节点挂载
- **功能**：通过 App/小程序扫码，将“入库”、“出库”、“转运”节点实时挂载到 TraceCode。
- **字段**：`node_name, operator, timestamp, gps_info`。

### 3.2 C 端：H5 消费者溯源体验

#### 3.2.1 扫码状态机逻辑
- **场景 A：首次扫描**
  - 显示“正品”标识。
  - 显示扫描时间为当前时间。
  - 记录 `first_scan_time`。
- **场景 B：重复扫描**
  - 醒目黄色警告：“该码已被扫描过！”。
  - 显示首次扫描的时间和大概城市（基于 IP）。
  - 提示：“若非本人操作，请谨防假冒”。
- **场景 C：码不存在/已作废**
  - 红色警报：“无效码”或“码已作废”。

#### 3.2.2 溯源信息展示层级
1. **基础层**：产品名称、批次、保质期。
2. **证据层**：点击查看《中药材质量检验报告》原件、种植地全景图。
3. **流程层**：时间轴形式展示从“育苗”到“上架”的所有节点。

### 3.3 监管与大数据预警

#### 3.3.1 异常扫描模型
- **异地预警**：5 分钟内在两个地理位置相距 > 500km 的地方扫描同一个码。
- **高频预警**：单个码在 1 小时内被扫描 > 10 次。
- **黑名单机制**：自动标记可疑 IP 段。

## 4. 核心 API 接口定义

### 4.1 溯源查询接口
- **URL**: `GET /api/trace/:full_code`
- **Response**:
```json
{
  "code": 200,
  "data": {
    "status": "genuine", // genuine, suspect, invalid
    "product": { "name": "当归", "spec": "250g/袋" },
    "batch": { "no": "B2026042201", "report_url": "..." },
    "trace_logs": [
      { "time": "2026-04-20", "desc": "产地采收", "location": "甘肃岷县" },
      { "time": "2026-04-21", "desc": "入库质检", "location": "西安中心仓" }
    ],
    "scan_info": { "count": 1, "first_time": "..." }
  }
}
```

### 4.2 赋码生成接口 (Internal Only)
- **URL**: `POST /api/batch/generate-codes`
- **Request**: `{ batch_id: string, count: number }`
- **Auth**: API-Key based internal auth.

## 5. 验收标准 (AC)
- **AC1**：系统支持一次性导出 10 万个不重复的溯源码，生成耗时 < 30s。
- **AC2**：H5 页面在 3G 网络环境下，首次加载首屏时间不得超过 3s。
- **AC3**：数据库中 `verification_code` 必须加盐存储，防止数据库泄露后伪造码被批量生成。
- **AC4**：后台管理界面需提供“一键拉黑可疑批次”功能，操作后对应所有码立即失效。
