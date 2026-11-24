# Viora — 产品愿景、需求与实施蓝图

> 版本：1.0 · 整理日期：2026-11-24 · 状态：已确认的产品方向 + 待实现的工程计划
>
> 项目：Viora · Tagline: **Your Media. Your Way.** · GitHub: `suki827/viora`

## 1. 背景与愿景

互联网上的视频资源分布在直播频道、影视网站、开放媒体库、YouTube 官方频道、教育平台以及个人 NAS 等不同位置。用户需要在多个 App / 网站之间切换、分别搜索和管理收藏，体验碎片化。

**Viora 的目标**：建设个人视频资源聚合平台，把不同合法来源的直播、电影、电视剧、纪录片、英语学习、动漫以及个人视频统一发现、分类、搜索、收藏和观看。**能在本平台合规直接播放则直接播放；无法内播才使用官方嵌入或跳转。**

核心体验：**打开一个入口 → 按自己设定的目录浏览或统一搜索 → 点击观看**。不要求用户了解 M3U、CMS、HLS、DASH、适配器等实现细节。

### 已确定的产品决策

1. 名称 **Viora**（不再使用 MediaHub 作为项目名）。
2. **内容分类和来源分类同时存在**，且目录结构由管理员自定义。
3. 支持一级、二级、三级目录；一个内容可以归属多个目录。
4. 采用 **Stremio 风格海报墙**与统一详情页，但不依赖 Stremio 默认 UI/分类。
5. **Web 优先**，兼容电脑、手机以及电视浏览器，电视端必须支持遥控器方向键。
6. **尽可能内部直接播放，不支持则跳转**；不强行破解 DRM。
7. 使用 **TypeScript + React + NestJS + PostgreSQL + Prisma**。
8. 优先复用成熟播放器/插件思想；先做模块化单体，避免过早微服务。
9. 可以在 Raspberry Pi 5 (4GB) 上运行轻量聚合服务；视频尽量客户端直连来源，不由 Pi 大规模转码。
10. 先实现可用 MVP，再扩展多协议、多用户、智能推荐及原生 Android TV。

## 2. 产品范围

### 2.1 核心内容类型

- **直播**：新闻、国际频道、纪录片、儿童与教育直播；M3U/HLS、EPG。
- **点播**：电影、电视剧、动漫、纪录片、教育课程与单集视频。
- **开放媒体与订阅**：官方 RSS/Podcast 视频附件、公开 API、支持嵌入的平台内容。
- **个人媒体库**：后续通过 Jellyfin 或适配器接入。
- **不可内播平台**：展示合法可获得的内容信息与官方打开入口。

分类不等同于播放来源：例如 BBC Learning English 同时可以出现在“内容 / 英语学习 / 听力 / 中级”和“来源 / 教育媒体 / BBC Learning English”。

### 2.2 MVP 必须实现（P0）

- 初始管理员、受保护的管理后台与基础鉴权。
- **两棵分类树**：`content` 和 `source`；每棵最多三级，可创建、重命名、排序、隐藏、删除（有关联内容时限制或迁移）。
- 媒体库：媒体条目、海报、详情、类型、国家/语言/年份、电视剧季集及频道。
- 媒体与分类多对多关联；来源原始条目映射、基本去重。
- 首页 / 分类页 / 搜索页 / 详情页 / 播放器 / 管理后台；海报墙与分页或无限滚动。
- 资源适配器框架 + M3U 导入 + 至少一种已获授权的点播示例。
- 播放方案决策：`direct` / `embed` / `external` / `unavailable`；友好的失败信息，备用源可切换。
- Web 播放 MP4/HLS 的浏览器能力验证；字幕和播放错误的基础处理。
- 电视浏览器方向键导航、确认与返回、可见焦点、返回时保留焦点位置。
- 收藏、观看进度与继续观看的基础持久化。
- Docker Compose 本地/ARM64 可部署、数据库备份文档、健康检查与基本测试。

### 2.3 后续功能（P1/P2）

- Stremio Addon Protocol（catalog/meta/stream）输入与可选输出兼容层。
- CMS API、RSS、Jellyfin、其他官方 API 的来源适配器。
- EPG 展示与频道节目时间表；资源健康定期检测、自动重试。
- 批量来源更新、智能去重候选、人工审核、自定义自动分类规则。
- 首页可编排行、跨设备观看历史、多个观看配置档、海报与字幕增强。
- 若业务压力需要，再引入 Redis/BullMQ 和独立 worker。
- 根据电视浏览器实际体验决定是否开发原生 Android TV APK (Kotlin/Media3)。

### 2.4 明确非目标

- 不是重新开发音视频编解码器。
- 不保证所有网站的视频都可以直接播放。
- 不绕过 DRM、付费墙、登录访问控制或平台嵌入限制；不收集未授权的影视内容。
- V1 不做视频下载站、P2P 服务、海量视频转码、公开大规模分发或复杂推荐算法。
- 不把视频源配置等同于用户内容分类，不要求用户手工维护大量 M3U。

## 3. 统一目录与 UX 规范

### 3.1 内容分类（示例，可以在后台修改）

| 一级 | 二级 | 三级示例 |
|---|---|---|
| 直播电视 | 按内容 | 新闻、纪录片、儿童 |
| 直播电视 | 按地区 | 新西兰、英国、美国、国际 |
| 电影 | 按地区 | 欧美、中国、日韩 |
| 电影 | 按类型 | 动作、科幻、剧情、喜剧 |
| 电视剧 | 按地区 | 美剧、英剧、国产剧、韩剧 |
| 纪录片 | 自然/科学/人文 | 野生动物、宇宙、历史 |
| 英语学习 | 听力/口语/课程 | 初级、中级、高级、自然拼读 |
| 动漫与儿童 | 动画/学习 | 儿童动画、日本动漫、数学启蒙 |

不强制每个一级类都拥有三级。地区、年份、语言、画质等可作为独立筛选字段，不要把所有组合展开成大量目录。

### 3.2 来源分类（示例）

| 一级 | 二级 | 三级示例 |
|---|---|---|
| 电视台 | 新西兰 | TVNZ、RNZ |
| 电视台 | 国际新闻 | BBC、DW、France 24 |
| 视频网站 | YouTube | 官方频道 / 主题 |
| 教育媒体 | 英语学习 | BBC Learning English、VOA |
| 影视接口 | 自定义 API | 来源 A、来源 B |
| 个人媒体 | 本地媒体库 | 电影、电视剧 |

**重要**：可见的来源分类节点不等于程序 `connector`，它们是不同实体；一个平台可有多个连接器，一个连接器可提供多个内容类别。

### 3.3 用户路径与电视交互

- 首页：继续观看、收藏、直播快捷入口、自定义内容行。
- 侧边栏：内容目录 + “按来源浏览” + 收藏。
- 进入一级类后，用横向二级标签和三级筛选而非强迫逐级打开 3 个页面。
- 统一详情页：海报/简介/年份/剧集/字幕/播放来源/播放按钮。
- 播放：优先授权、兼容、可靠的直连；其次官方嵌入；最后外部官方入口。
- TV：上下切内容行，左右移海报，Enter/OK 选择，Back/Esc 返回；始终显示焦点，记住上一位置。
- TV 模式可手动打开；不单凭 viewport 宽度推断设备。
- 常见内容争取 2–3 次用户操作内开始播放。

## 4. 技术选型与架构

| 层 | 技术 | 理由 |
|---|---|---|
| Monorepo | pnpm workspace | 前后端共享类型；轻量 |
| Web | React + TypeScript + Vite | 自定义界面，跨浏览器 |
| UI | Tailwind CSS | 海报墙和响应式样式 |
| 状态/数据 | TanStack Query + Zustand | 远端请求缓存 + 本地交互状态 |
| TV 导航 | Norigin Spatial Navigation（验证兼容后采用） | 方向键焦点管理 |
| Web 播放 | Shaka Player（验证 HLS/DASH），HTML5 video | 复用成熟播放内核 |
| API | NestJS (REST) | 模块化单体、DTO/验证、调度 |
| DB | PostgreSQL | 结构化媒体与多对多关系 |
| ORM | Prisma | Schema、Migration、类型安全 |
| 搜索 | PostgreSQL FTS + trigram（按需） | V1 无独立搜索引擎 |
| 任务 | NestJS Scheduler | V1 周期同步，无需 Redis |
| 部署 | Docker Compose + Nginx | 开发与 Pi ARM64 部署 |

**设计原则**：前端只访问 Viora API；适配器隔离外部源；应用核心不依赖 Stremio、Kodi 或 TVBox 的数据库结构。媒体内容尽量由浏览器直接从授权来源获取，以节省服务器带宽。

### 4.1 推荐项目结构

```text
viora/
├── apps/
│   ├── web/src/{app,pages,components,features,styles}
│   └── api/src/{modules,common}
├── packages/
│   ├── shared/
│   ├── adapter-sdk/
│   └── adapters/{m3u,cms,stremio,rss,jellyfin}
├── prisma/{schema.prisma,migrations,seed.ts}
├── infra/{docker,nginx}
├── docs/
├── tests/{integration,e2e}
├── docker-compose.yml
└── pnpm-workspace.yaml
```

只建立需要的子目录，不为尚未开发的模块堆积无用占位文件。

### 4.2 NestJS 模块

- `AuthModule`: 管理员与登录/会话。
- `CategoriesModule`: 两棵分类树与内容映射。
- `MediaModule`: 作品、季集、频道与元数据。
- `SourcesModule`: 来源平台名称、类型与状态。
- `ConnectorsModule`: 具体连接器实例和受保护配置。
- `IngestionModule`: 拉取、规范化、增量同步和去重。
- `SearchModule`: 全文与筛选查询。
- `PlaybackModule`: 播放方案选择、可用性与兜底。
- `LibraryModule`: 收藏、观看历史、观看进度。
- `HomepageModule`: 首页板块自定义。
- `JobsModule`: 计划任务、同步状态、错误记录。

## 5. 数据模型

### 5.1 关键表

| 表 | 核心字段与关系 |
|---|---|
| `users` | id, login, password_hash, role |
| `categories` | id, namespace(content/source), parent_id, name, slug, sort_order, visible |
| `media` | id, media_type(movie/series/episode/film/live/video), title, original_title, year, language, region, poster_url, summary |
| `media_categories` | media_id, category_id, assignment_type(manual/rule), unique pair |
| `seasons` | id, media_id, season_number |
| `episodes` | id, media_id, season_id, episode_number, title |
| `sources` | id, name, kind, status, homepage_url |
| `connectors` | id, source_id, adapter_type, config (secrets protected), enabled, last_sync_at |
| `external_items` | id, connector_id, external_id, media_id, source_metadata; unique(connector_id, external_id) |
| `playback_options` | id, external_item_id, episode_id?, mode, protocol, resolver_config, expires_at, health |
| `favorites` | user_id, media_id; unique(user_id, media_id) |
| `watch_progress` | user_id, media_id, episode_id?, position_sec, duration_sec, updated_at |
| `watch_history` | id, user_id, media_id, episode_id?, last_played_at |
| `home_sections` | id, section_type, title, config, sort_order |
| `sync_jobs` | id, connector_id, status, started_at, finished_at, error |
| `source_health_checks` | id, playback_option_id, status, checked_at, response_ms |
| `classification_rules` | id, conditions, target_category_id, priority, enabled |

以上是**逻辑模型**，不是已实现的 Prisma schema。代码阶段统一枚举、外键及生命周期状态；特别注意不同媒体类型共享字段的空值、分集媒体主键和播放进度唯一性。

### 5.2 必须维护的数据不变量

- `categories.parent_id` 同 namespace，且最大深度 3，无循环。
- `media_categories` 多对多；删除分类前处理子节点和媒体关联。
- `external_items` 在一个连接器内部用 external_id 唯一；跨来源媒体匹配不能只靠标题。
- 收藏与观看进度绑定 **Viora 内部 ID**，不能绑定会过期的外部 URL。
- 自动归类不能覆盖人工编辑。
- 不在普通数据库字段或日志中明文暴露第三方 Token/密码。
- 播放 URL 尽量按需解析，并记录过期时间；不要假设流地址永久有效。

## 6. 适配器架构

```ts
type Capability = 'catalog' | 'search' | 'detail' | 'playback' | 'live' | 'epg';

interface MediaAdapter {
  readonly id: string;
  readonly capabilities: Capability[];
  list?(query: CatalogQuery): Promise<MediaItem[]>;
  search?(keyword: string): Promise<MediaItem[]>;
  getDetail?(externalId: string): Promise<MediaDetail>;
  getPlayback?(externalId: string, episodeId?: string): Promise<PlaybackOption[]>;
  getChannels?(): Promise<LiveChannel[]>;
  getEPG?(channelId: string): Promise<EPGEntry[]>;
}
```

这些类型为**概念接口**，实现时应在 `packages/adapter-sdk` 中定义完整类型、分页、错误类型、能力检查和取消/超时机制。

最早接入：M3U（可使用已有 `live_config.m3u` / `english_hd.m3u` 的合法频道）及一套授权公开点播测试源。后续扩展 CMS、Stremio Addon、RSS、Jellyfin、官方平台 API。

安全约束：仅运行可信适配器；不执行外部资源下发的任意脚本；远程 URL 必须做协议、DNS/IP、重定向、超时、体积和内网访问限制，防 SSRF。鉴权凭据仅后端持有。

## 7. 播放流程

1. 用户在统一媒体库点击“播放”。
2. `PlaybackModule` 查询该媒体/分集所有候选 `playback_options`，必要时通过适配器实时解析。
3. 基于授权状态、协议/浏览器能力、来源健康、字幕语言、清晰度、过期时间排序。
4. 返回 `direct`（MP4/HLS/DASH）、`embed`（官方嵌入）、`external`（官方跳转）或 `unavailable`（明确理由）。
5. 播放错误时可选择下一个兼容且有权使用的来源，不能循环无止境重试。
6. 定期更新观看进度；直播通常不以点播时长保存观看位置。

要求特别验证：CORS、HTTPS mixed content、浏览器媒体能力、DRM、字幕、电视全屏和返回键。不能将 HTTP 200 等同于可播，也不能绕过平台访问控制。

## 8. API 初稿

```text
GET    /api/health
GET    /api/categories?namespace=content|source
POST   /api/admin/categories
PATCH  /api/admin/categories/:id
GET    /api/media?page=1&pageSize=24&categoryId=...
GET    /api/media/:id
GET    /api/search?q=...
GET    /api/media/:id/playback?episodeId=...
GET    /api/home
GET    /api/me/favorites
PUT    /api/me/watch-progress
GET    /api/admin/sources
POST   /api/admin/connectors
POST   /api/admin/connectors/:id/sync
GET    /api/admin/jobs
```

所有管理端点需要鉴权和权限检查；入参用 DTO 校验；分页与排序明确上限；API 错误采用一致结构。Stremio 兼容接口为后续适配层，不作为核心业务 API。

## 9. 开发里程碑与验收

### Phase 0 — 工程骨架（第一项 Codex 任务）

- pnpm Monorepo，`apps/web` React/Vite，`apps/api` NestJS，`packages/shared`、`packages/adapter-sdk`。
- PostgreSQL + Prisma 基础 schema 和迁移；Docker Compose + Nginx；环境示例、忽略规则。
- 基础 UI 布局 / 页面占位、`/api/health`，首次 API 联通、种子数据。
- ESLint / Prettier / 测试、README、架构说明与 ARM64 注意事项。
- **验收**：构建与启动命令可运行；迁移成功；Web 能读取 API；无秘钥入库；测试结果如实记录。

### Phase 1 — 自定义目录与媒体库

- 后台分类 CRUD、排序、隐藏、三级约束；内容分类和来源分类。
- 海报墙、媒体详情、季集数据、过滤和 PostgreSQL 搜索。
- **验收**：能新建 3 级目录；相同媒体归入两棵树的不同节点；手工调整后保持稳定。

### Phase 2 — 聚合和播放

- SDK、M3U 导入、首个授权点播连接器、数据标准化/去重。
- 视频流决策，Shaka/HTML5 播放、失败兜底、备选来源。
- **验收**：直播与点播均可通过同一界面选择观看；无法内播有清晰跳转路径。

### Phase 3 — 日常使用与 TV

- 收藏、观看历史、继续观看、自定义首页、同步调度/健康检测。
- 电视遥控器焦点管理、全屏、返回行为、性能优化。
- **验收**：真实安卓电视浏览器可使用遥控器完成浏览、选择、播放与返回。

### Phase 4 — 增量扩展

- Stremio / RSS / Jellyfin 等插件，自动分类，复杂去重、队列/缓存（有需要再加）。
- **验收**：新增来源无需修改分类和播放器核心；故障的单个适配器不影响整体。

## 10. 部署、运维与数据保护

- 生产目标：Raspberry Pi 5 ARM64、Docker Compose，Web(Nginx) + API + PostgreSQL；后续可迁移到更强的机器。
- 不在 4GB Pi 上做批量转码，不默认经服务器代理所有视频。
- 仅通过受控反向代理暴露 Web；PostgreSQL 不暴露公网；远程运维可选 Tailscale。
- 测试/生产环境分离；版本锁定；迁移与回滚策略、日志限额、定期 pg_dump 和恢复演练。
- 数据库放在健康磁盘，不在不可靠的旧盘上保存唯一副本。
- 对上传和外部海报 URL 做限制；凭据按需加密；第三方平台规则和区域授权应明确遵守。

## 11. 项目成功指标

- 用户可在一个界面完成主要来源的浏览和搜索。
- 高优先级的合法测试视频可直接播放，失败后展示可操作的备用或跳转方案。
- 目录添加、调整和来源变化不影响收藏/历史。
- 普通电视遥控器无需鼠标就能完成主要流程。
- 单个来源断开不影响全局页面与其他来源。
- 真实设备上完成数据库备份恢复与重启持久化验证。

## 12. Codex 协作规则

- **不要直接把整个路线图一次性实现**；从 Phase 0 做起。
- 先阅读此文件，再看 `docs/CODEX_PHASE0.md`。
- 尽量使用已有成熟依赖；安装兼容且稳定的版本并在 README 说明。
- 先列出设计和实施步骤，再提交代码，跑格式检查、类型检查、构建和测试；记录实际未运行的环节。
- 重要架构变化需更新本文或添加 ADR，避免悄悄改变既定产品方向。
- 不收集真实账号凭据，不引入未经授权的媒体播放源。
