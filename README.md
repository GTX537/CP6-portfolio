# CP6 — ERP / MES / WMS 一体化系统

CP6 是基于 **.NET 8/10 + Vue 3** 的现代化 ERP / MES / WMS 一体化系统，源于一个日本大型纸器包装制造企业的基幹系统重构项目（mcframe7 平台 Add-on → 独立 Web 系统）。

## 技术栈

| 层 | 技术 |
|---|---|
| 后端 | ASP.NET Core 10 / EF Core 10 + Dapper / SQL Server 2022 |
| 前端 | Vue 3.5 + TypeScript + Element Plus + Pinia + vue-i18n（5 国语） |
| 实时通信 | SignalR |
| 消息队列 | RabbitMQ + Kafka |
| 容器 | Docker Compose / Kubernetes |
| 测试 | xUnit + Moq（**282 用例**） |

## 子系统

- **ERP（PA 系列 販売管理）**：取引先 / 見積 / 御見積 / 製品マスタ / 受注 / FSC / シート単価 / 版型木型
- **MES（ME 系列 製造執行）**：生産計画 / 製造指図 / 製造実績 / 品質検査 / 不良管理 / 設備 / OEE
- **WMS（MSBBWM 系列 倉庫管理）**：仓库/库存/入出库/棚卸/补充/Kit/Pallet/QC/Lot トレース 等 28 个模块

## 跨模块闭环（Bridge Hook）

CP6 通过 4 个对称的 Bridge Hook 接口实现 ERP↔MES↔WMS 自动联动，遵循 **Best-Effort + 冪等 + appsettings 可禁用 + IntegrationEvent 持久化** 设计原则：

| 接口 | 触发 → 动作 |
|---|---|
| `IMesBridgeHook` | ERP 受注作成 → MES 製造指図 自动展开 |
| `IWmsBridgeHook` | MES 指図発行 → WMS 材料出庫指示 / 受注作成 → 出荷指示 / 全工程完了 → 完成品入庫 |
| `IErpBridgeHook` | WMS 出荷確定 → ERP 受注 出荷実績回写 / **WMS RMA 確定 → ERP CreditNote** (Phase 10a) |
| `IOrderCancelBridgeHook` (**Phase 6**) | ERP 受注取消 → 反向級联 MES WO / WMS Outbound 取消 |

详见 [`docs/PROJECT_STRUCTURE.md`](docs/PROJECT_STRUCTURE.md) §2.3。

## Phase 6-10 改进（最近迭代）

| Phase | 内容 | 文档 |
|---|---|---|
| Phase 6 | 受注取消反向級联 + IntegrationEvent 持久化 + Retry Worker + DeadLetter 告警 | [`docs/PHASE6_SPEC.md`](docs/PHASE6_SPEC.md) |
| Phase 7 | Stock QC 状态管理（FAILED/HOLD 自动阻止出货）+ QualityInspection NG 自动联动 | — |
| Phase 8 | 受注済未出荷 Dashboard widget + CSV 导出 | — |
| Phase 9 | 材料欠品反流（OutboundService 检测短缺 → T_MaterialShortage + SignalR 告警） | — |
| Phase 10a | RMA → ERP CreditNote 自动回写（OrderDetail.ReturnedQty 累计） | — |
| Phase 10b | Bridge Hook Health Monitor（24h 成功率 + DLQ Dashboard + 手动补偿） | — |

## 项目文档

- [`docs/PROJECT_STRUCTURE.md`](docs/PROJECT_STRUCTURE.md) — 代码架构 + 业务流 + 模块清单 + ER 图
- [`docs/PROJECT_IMPROVEMENT_PLAN.md`](docs/PROJECT_IMPROVEMENT_PLAN.md) — 4 维度 × 11 gap 的改进路线
- [`docs/PHASE6_SPEC.md`](docs/PHASE6_SPEC.md) — Phase 6 完整可执行规格
- [`DEVELOPMENT-GUIDE.md`](DEVELOPMENT-GUIDE.md) — 从零搭建开发环境

## 快速开始

### 前置
- .NET 10 SDK
- Node.js 20.19+
- Docker Desktop (含 Compose)

### 启动

```bash
# 1. 配置环境变量
cp .env.example .env
# 编辑 .env，设置 MSSQL_SA_PASSWORD / RABBITMQ_PASSWORD / JWT_SECRET

# 2. 启动全部服务
docker-compose up -d
```

访问：
- 前端：http://localhost:8080
- API + Swagger：http://localhost:9991/swagger
- RabbitMQ 管理台：http://localhost:15672

### 测试

```bash
dotnet test          # 后端单元测试（282 用例）
cd cp6.web && npm run e2e   # 前端 Playwright e2e
```

## 项目结构

```
CP6/
├── CP6.Entity/      # 实体层 — DomainModels + DTOs
├── CP6.Core/        # 核心层 — Services + BridgeHooks + EFDbContext + Migrations
├── CP6.WebApi/      # API 层 — Controllers + SignalR + BackgroundServices + Filters
├── CP6.Tests/       # 测试 — xUnit + Moq（282 用例）
├── cp6.web/         # 前端 — Vue 3 + TS + Element Plus
├── docs/            # 文档 — 架构 / 规格 / i18n 种子 SQL
├── k8s/             # Kubernetes 清单
└── docker-compose.yml
```

## License

MIT
