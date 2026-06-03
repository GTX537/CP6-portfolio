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
| 测试 | xUnit + Moq（192 用例） |

## 子系统

- **ERP（PA 系列 販売管理）**：取引先 / 見積 / 御見積 / 製品マスタ / 受注 / FSC / シート単価 / 版型木型
- **MES（ME 系列 製造執行）**：生産計画 / 製造指図 / 製造実績 / 品質検査 / 不良管理 / 設備 / OEE
- **WMS（MSBBWM 系列 倉庫管理）**：仓库/库存/入出库/棚卸/补充/Kit/Pallet/QC/Lot トレース 等 28 个模块

## 跨模块闭环（Bridge Hook）

CP6 通过 3 个对称的 Bridge Hook 接口实现 ERP↔MES↔WMS 自动联动，遵循 **Best-Effort + 冪等 + appsettings 可禁用** 设计原则：

| 接口 | 触发 → 动作 |
|---|---|
| `IMesBridgeHook` | ERP 受注作成 → MES 製造指図 自动展开 |
| `IWmsBridgeHook` | MES 指図発行 → WMS 材料出庫指示 / 受注作成 → 出荷指示 / 全工程完了 → 完成品入庫 |
| `IErpBridgeHook` | WMS 出荷確定 → ERP 受注 出荷実績回写 |

详见 [`docs/PROJECT_STRUCTURE.md`](docs/PROJECT_STRUCTURE.md) §2.3。

## 项目文档

- [`docs/PROJECT_STRUCTURE.md`](docs/PROJECT_STRUCTURE.md) — 代码架构 + 业务流 + 模块清单 + ER 图（388 行）
- [`docs/PROJECT_IMPROVEMENT_PLAN.md`](docs/PROJECT_IMPROVEMENT_PLAN.md) — 4 维度 × 11 gap 的改进路线（180 行）
- [`docs/PHASE6_SPEC.md`](docs/PHASE6_SPEC.md) — Phase 6 (Order Cancel 链 + IntegrationEvent 持久化) 完整可执行规格（616 行）
- [`DEVELOPMENT-GUIDE.md`](DEVELOPMENT-GUIDE.md) — 教程视角，从零搭建开发环境

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
dotnet test          # 后端单元测试（192 用例）
cd cp6.web && npm run e2e   # 前端 Playwright e2e
```

## 项目结构

```
CP6/
├── CP6.Entity/      # 实体层 — DomainModels + DTOs
├── CP6.Core/        # 核心层 — Services + BridgeHooks + EFDbContext
├── CP6.WebApi/      # API 层 — Controllers + SignalR + Filters
├── CP6.Tests/       # 测试 — xUnit + Moq
├── cp6.web/         # 前端 — Vue 3 + TS + Element Plus
├── docs/            # 文档 — 架构 / 规格 / i18n 种子 SQL
├── k8s/             # Kubernetes 清单
└── docker-compose.yml
```

## License

MIT
