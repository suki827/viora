# Codex 任务：Viora Phase 0 — 可运行的基础工程

> 先阅读 [Viora 产品与架构蓝图](VIORA_PROJECT_BLUEPRINT.md)。本文件是第一阶段的具体实施指令，不代表后续全部功能同时实现。

## 目标

在当前 `suki827/viora` 仓库中构建**可运行、可测试、支持 ARM64 部署**的全栈 Monorepo 骨架。

## 必须完成

1. `pnpm-workspace.yaml` 和根部 workspace scripts；TypeScript、格式化和 lint 配置。
2. `apps/web`：React、Vite、TypeScript、Tailwind；包括首页/目录/搜索/媒体详情/播放器/管理后台占位路由；深色响应式布局，保证可键盘操作。
3. `apps/api`：NestJS 模块化单体；`GET /api/health`；CORS 与输入校验基础设置。
4. `packages/shared`：复用 DTO、枚举和共用类型（先少量必要定义）。
5. `packages/adapter-sdk`：能力声明和基础适配器类型（不接真实外部来源）。
6. Prisma + PostgreSQL：基础 Category、Media、MediaCategory、Source、Connector 逻辑模型；初始 migration、少量确定性的 seed 数据；分类约束的后续设计说明。
7. Web API 客户端读取真正的分类树数据（不是所有数据都 hardcode）；提供清晰的 loading/error/empty 状态。
8. Docker Compose：Web、API、Postgres；Nginx 同源反向代理；.env.example；Dockerfile ARM64 兼容。
9. README：环境要求、安装/启动、迁移/seed、测试命令、Docker 运行、备份方法与常见问题。
10. 最低限度测试：NestJS health、分类列表、Prisma schema/migration、Web 构建；可加 CI。

## 不要在 Phase 0 做

- 不要实现复杂抓取、下载、转码、DRM 或正式视频内容接入。
- 不要加微服务、Redis、BullMQ、Kubernetes。
- 不要在不知来源授权/稳定性的情况下预置真实播放地址。
- 不要把目录体系绑定到某个 Stremio 插件或 M3U 的字段。
- 不要凭空声称命令已经跑通；无法运行时报告原因。

## 验收条件

- `pnpm install` 成功，根脚本可运行类型检查、lint、build、test（以实际脚本为准）。
- 开发模式下 Web 正常访问，API health 返回成功。
- Docker Compose 能创建 3 项服务（Web、API、DB）；数据库迁移可以重复执行。
- 分类 API 实际从 PostgreSQL 返回 seed 分类。
- 没有敏感信息被提交；基础安全设置已落实。
- README 清楚列出从空仓库启动的命令、预期端口与已验证平台。
- 输出简明交付总结：新增文件、设计选择、实际执行命令和结果、已知限制与下一阶段建议。

## 实施方式

优先在独立工作分支完成并提交 PR；审阅后合并到 `main`。若仓库只有 main，可先创建 `develop`，再建立功能分支。任何架构取舍应以可维护、最小可运行为目标。
