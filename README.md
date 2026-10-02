# DeepSeek Harness 0.2.0-rc.2：新特性全景与 Harness 演化趋势

> **一句话结论：0.2.0-rc.2 不是一次功能迭代，而是一次"物种扩张"——dsh 从"插件化的 Agent 框架"进化成了一个可安装、可组合、可运营的 Agent 操作系统，并由此暴露出 Harness Engineering 正在形成的九条结构性演化趋势。**
>
> 研究对象：DeepSeek Harness（`dsh`）源码发布 `deepseek-harness-master`，根清单 `@deepseek-ai/dsh-root` **0.2.0-rc.2**（快照 2026-09-29）
> 对比基线：`0.1.0-rc.5`（2026-08-14 索引）与前序研究（2026-08-23 两份专题报告 + 五大创新）
> 配套文档：`FILE_INDEX.dsh-0.2.0-rc.2.md` / `.html`
> 日期：2026-10-02

---

## 一、版本定位：从"能跑的框架"到"可交付的产品系统"

上一版（0.1.0-rc.5）已经建立了 dsh 的骨架：事件溯源会话（`Model-visible ⟺ Logged`）、能力接缝、异构子代理、跨平台沙箱、机器强制的可靠性门禁。**0.2.0-rc.2 把这套骨架变成了一个完整的、可以安装到终端用户机器上的产品**。

| 维度 | 0.1.0-rc.5（基线） | 0.2.0-rc.2（本次） | 变化性质 |
|---|---|---|---|
| 包组 / 包数 | 54 组 / ~219 包 | **55 组 / 316 包** | +44% 模块数 |
| 会话格式写入器 | v0 | **v4**（已发布格式 v3） | 4 次结构性跃迁 |
| 应用形态 | CLI + Web | **CLI + Web + Electron Desktop + Desktop Host** | 新增一整条桌面跑道 |
| 发行机制 | 直接运行插件树 | **profile + bundle + patch 分层** | 组合方式产品化 |
| 前端包 | 少量 | **client/ 63 个包** | Web 变成真正的工作台 |
| 长期决策记录 | ~680 条 | **≈ 526 EN（≈1051 含中文）实现态 Note** | 决策资产持续膨胀 |
| 文档 | 多篇 | **359 篇 md / 64 个子系统页** | 文档即基础设施 |
| 测试规格 | — | **≈ 1,501 个 `*.spec.ts`** | 质量门禁持续加码 |

**三句话读懂这一版：**

1. **横轴变宽**：Agent 的"手"从 shell/文件扩展到浏览器、桌面、Office、SSH、LSP、持久终端、程序化编排——每一个都是可替换的能力接缝。
2. **纵轴变深**：会话日志从"可重放"深化为"可迁移（v0→v4 相邻迁移链）、可投影缓存、可跨进程加锁、可跨版本读取"。
3. **外层变厚**：桌面端、profile/bundle、插件市场页、账号与配额、崩溃诊断、有界遥测——从"研发工具"补齐为"可运营产品"。

---

## 二、新特性全景图谱

按能力域归纳 0.2.0-rc.2 相对基线的净新增/重大改造。证据为源码路径或 Agent Note 路径。

### 2.1 桌面应用（Desktop）——本次最大净新增面

| 项目 | 内容 |
|---|---|
| 形态 | Electron 外壳，内置一份精确签名的 dsh 生产运行时，独占 `$DSH_HOME/profiles/desktop`，默认端口 **19387** |
| 关键路径 | `apps/desktop/`（外壳 + NSIS 安装器 + 更新/托盘/欢迎页）、`apps/desktop-host/src/{office,platform-session,quit-inspection,update-tasks}.ts` |
| 机制 | 立即加载打包 Web 资源 → 私有 Host 以 Electron Node 模式启动 → 通过 Node IPC + `dsh-app://app/` 连接；强制更新、崩溃报告、内置 Python/Node/pnpm"主运行时"（`$DSH_HOME/dsh-runtimes/dsh-primary-runtime`） |
| 决策 | `architecture/2026-08-25-electron-desktop-packaging-and-updates.md`、`2026-09-11-desktop-electron-node-runtime.md`、`2026-09-14-desktop-primary-runtime.md`、`2026-09-22-fatal-diagnostics-and-crash-reports.md`、`2026-09-23-desktop-close-to-background-and-quit-confirmation.md`、`2026-09-27-desktop-cli-runtime.md` |

**为什么重要**：它把 dsh 从"要装 Node/pnpm 才能跑的源码"变成"双击即用的本地应用"。这是 Harness 走向"操作系统类比"的关键一步——操作系统必须先能被安装。

### 2.2 Agent Teams（实验性）——从"委托"到"有状态团队"

| 项目 | 内容 |
|---|---|
| 形态 | `ctx.agentTeams` 协作域：持久花名册 + 任务 DAG + 邮箱，叠加在 continuable subagent 上 |
| 关键路径 | `packages/experimental/{agent-team,agent-team-profile,tool-agent-team,client-ui-agent-team}`、`docs/subsystems/agent-team.md` |
| 机制 | `TeamId` = 根 SessionId 的品牌化；`TeamTaskId` 带 compare-and-set `revision`、`blockedBy` DAG、advisory `writeScopes`；`agentTeam` 投影从根日志重放 |
| 决策 | `architecture/2026-09-18-agent-teams-single-bundle.md`（一个开关取代 Host/Web 双 bundle） |

**为什么重要**：dsh 不再只有"一次性委托"，而是有了**持久化、可恢复、可审计的多智能体协作**。这正是从"Agent"到"Agent 组织"的门槛。

### 2.3 Workflow + PTC——程序化编排

| 项目 | 内容 |
|---|---|
| 形态 | 模型编写编排脚本（workflow）扇出 subagent；PTC（programmatic tool calling）在沙箱 Node 子进程中运行 JS 程序调用宿主绑定 |
| 关键路径 | `packages/workflow/{workflow,workflow-ptc,tool-workflow,tool-ralph}`、`packages/ptc-runtime/{ptc-runtime,ptc-runtime-node}`、`packages/experimental/ptc-runtime-python` |
| 机制 | `ctx.workflowEngine.start()` → `WorkflowRun`；`run_in_background` 注册为 `kind:'workflow'` job；`PtcRunFailure` 是与异常正交的分类法（超时/中止/worker 退出/沙箱不可用…） |
| 决策 | `architecture/2026-09-11-sandboxed-node-ptc-runtime.md`、`2026-09-13-workflow-ptc-sandbox-reuse.md`、`feature/2026-09-01-workflow-run-in-background.md` |

**为什么重要**：编排不再是"模型一步步 tool call"，而是**模型写程序、程序编排 Agent**——Agent 系统获得了递归与规模化能力。

### 2.4 后台作业与调度（Jobs / Schedule）

- **Jobs**（`packages/jobs/*`, `ctx.jobs`）：owner-fenced 的后台作业注册表，每个作业一个有界输出环，消费/观察双游标，`list/get/read/kill/wait/remove`，作业完成默认唤醒空闲 owner（`2026-09-22-unbounded-completion-wakes-by-default.md`）。
- **Schedule**（`packages/schedule/schedule`, `ctx.schedule`）：Host 持有的一次性/周期提醒，`after|at|every|daily|weekly|cron`（Vixie 5 段）、显式 IANA 时区、DST 规则、有界投递历史，投递回原 Session 并可冷恢复。

**为什么重要**：长时程任务不再阻塞一个 turn；"时间"成为 Agent 的第一类输入。

### 2.5 Goal / Plan / Guard——目标、计划与循环卫生

- **Goal**（`ctx.goals`）：同一 Session 的持久完成目标，revisioned 状态 + 自动续做轮次（`goal-round-driver`）。
- **Plan**（`ctx.planMode`）：可回放的 plan 模式协作状态 + `exit_plan_mode` 工具。
- **Guard**：`repeat-tool-reminder`（重复调用提醒）+ `timeout-policy`（工具调用截止时间）。

**为什么重要**：Harness 开始显式建模"用户要什么（Goal）""Agent 怎么规划（Plan）""循环如何不空转（Guard）"——这是**agentic 长任务可靠性**的三个支点。

### 2.6 浏览器与桌面操作（Browser-use / Computer-use）

- `ctx.browserUse` / `ctx.computerUse` 是"注册唯一 provider"的接缝；provider 包括 Playwright MCP、Chrome DevTools MCP、Stagehand 原生、Cua Driver MCP/原生。
- **Sidebar Browser**（`client/ui-sidebar-browser`）：在右侧栏打开沙箱化的 HTTP(S) 页面（含 loopback）；桌面端用 Electron webview 承载。

**为什么重要**：Agent 的"手"从进程内延伸到浏览器与 GUI，且仍保持 provider 注册 + 每 Session 所有权的显式约束。

### 2.7 Office / 文档

- Host 侧 `ctx.officeToPdf`（`packages/document/office-to-pdf`）：DOC/XLS/PPT → PDF，独立发布的原生引擎或 WASM 回退，有界准入 + 内容哈希缓存 + 缺失字体诊断。
- 内置 Office 技能（`skill/skill-office`）+ 浏览器内电子表格预览（FortuneSheet + 打过补丁的 ExcelJS/SheetJS）。

**为什么重要**：真实文档工作流（写作→预览→交付）成为一等能力，而不是"让模型想办法装库"。

### 2.8 Profile / Bundle / Preset / 插件生态——发行工程

| 能力 | 说明 | 路径 |
|---|---|---|
| **Profile** | 命名组合（`web`/`headless`/`sdk`/`sdk-minimal`/`acp`），列出 bundle 栈与 patch | `docs/architecture.md#profiles-and-bundles` |
| **Bundle** | 发行格式：`package.json` 的 `dsh.profile`/`dsh.bundle` 声明，可被上层 patch | `packages/bundle/*` |
| **Agent Preset** | 用普通 Cordis YAML 声明子插件，注册表保留运行中的历史 revision | `packages/preset/*`、`2026-09-18-declarative-agent-presets.md` |
| **插件管理页** | 引导式安装、持久化 creator 管理、配置编辑、插件清单 | `boot/plugin-manager`、`host/plugin-inventory`、`client/ui-plugin-manager`、`ui-settings-plugins` |
| **实验能力可选包** | Agent Teams、语音输入、Auto review、Schedule 作为可选 bundle（默认关） | `2026-09-21-experimental-capabilities-as-optional-bundles.md` |

**为什么重要**：dsh 把"如何组合"从代码变成了配置与发行物——这是平台化的必经之路，也是"一切皆插件"从口号落地的证据。

### 2.9 MCP / Hooks / Skills——互操作与能力复用

- **MCP**：从"只有工具"扩展到 **resources + 作用域化 server instructions + SDK 协议协商**（`packages/mcp/mcp-resources`）。
- **Hooks**：复用 Claude Code / Codex 的 shell hook 配置（`packages/hooks/*`），通过 `hook/invoked`、`hook/result` 落日志。
- **Skills**：注册表 + 目录/加载工具，新增 Office 技能与工作区依赖技能，创作者技能"渐进式披露"。

**为什么重要**：dsh 选择**吸收而非对抗**既有 Agent 生态（Claude Code、Codex、MCP），同时保持这些能力可审计。

### 2.10 上下文 / 压缩 / 会话格式 v3→v4 / 检索

- **格式 v3**：规范化事件信封，message/tool-result 必须带 `surfaceOp`；log-only 事件仅 `type/seq/time/data/ignorable`（`2026-09-06-v3-canonical-session-envelopes.md`）。
- **格式 v4**：一等 `role:'tool'` 消息、developer 角色 tool add/remove + `deferLoading`、producer-owned 消息源、`turn/end.reason: forked`（`2026-09-15-first-class-tool-role-messages.md`）。
- **相邻迁移链**：`session-format-v0-to-v1` … `v3-to-v4`，只加版本化后继，**绝不移动/覆盖/删除已提交的文件**（`2026-08-31-released-session-format-migrations.md`）。
- **投影缓存**：`session-projection-cache` 持久化 `(sessionId,key,ver,seq,val)`，节流写回 + 关键点强制 checkpoint + 跨版本读兼容。
- **压缩演进**：image offload、tool-result pruner、checkpoint policy。

### 2.11 工具管线：动态化、三阶段、系统提示入 surface

- **动态工具更新**：`request/header.tools` + `developer/message` 的 `tool-registry` 变更 + `Session.toolHistory()` + `projectToolUpdates`；DeepSeek 侧发送 tool-changes beta 头。
- **三阶段工具调用**（客户端）：`preparing`（参数生成中的实时增量）→ `start`（`tool/call`）→ `result`（`tool/result`），准备阶段只作展示、绝不持久化/进入模型。
- **系统提示成为 surface 节点**：`system/message` 成为普通 surface 事件，提示变更 = 节点替换 + 开启新 `series`，头部受压缩保护。

**为什么重要**：这三者共同指向一个事实——**Harness 正在与模型 API 共同演进**（缓存感知的工具目录、稳定的系统提示节点），而不是把模型当成无状态函数。

### 2.12 治理、安全与运营

- **沙箱**：bwrap/Landlock/Seatbelt/Windows ACL；`sandbox-same-mode`；Windows 强制完整性级（`2026-09-19-windows-acl-mandatory-integrity-confinement.md`）。
- **原生原语**：`native/system` 新增 POSIX `flock`，支撑**跨进程 Session 写租约**。
- **Auto Review（实验）**：`auto` 权限预设下，每次原生/PTC 内层调用前做同模型风险评估，可回退到人工审批。
- **运营可观测**：fatal diagnostics + 崩溃报告、有界 session-log 上传、OTel 字节上限、产品遥测默认受反馈门控。
- **账号/凭据**：DeepSeek 账号登录、配额充值、登出；凭据引用与授权流作为显式 seam。

### 2.13 Web 客户端革命——从聊天框到工作台

右侧栏停靠引擎（`ui-dockkit` + `ui-sidebar-right`）取代旧 Detail 面板；资源模型；文件树、文档预览、浏览器、终端、Jobs、Schedule、Plan、Goal、Subagent、Trajectory、Deliverables 等页面；会话置顶/归档、归档停止运行中工作、默认工作区；统一模型输入控件、引用预览、语音输入（实验）。

### 2.14 被移除 / 被简化（减法的信号）

| 简化项 | 证据 |
|---|---|
| 移除 E2B 沙箱/文件/子进程 provider，改为 POSIX SSH | `simplification/2026-09-11-remove-e2b-providers.md` |
| 会话持久化改为 **JSONL 唯一**（删除 SQLite 权威存储） | `simplification/2026-08-30-jsonl-only-session-persistence.md` |
| DeepSeek 官方线路改为 **Messages-only**（去掉 Chat Completions 协议选择器） | `simplification/2026-09-19-deepseek-messages-only.md` |
| `ralph` 工具默认关闭 | `simplification/2026-09-12-ralph-off-in-shipped-defaults.md` |
| 移除不必要的 `./invariant` 伴生包 | `simplification/2026-08-28-omit-unneeded-invariant-companions.md` |

**洞察**：这次 release 同时展示了"加法"（新能力域）与"减法"（收敛抽象、删除冗余后端）。**主动做减法，是工程成熟度的反直觉标志。**

---

## 三、十项深入剖析

### 3.1 桌面端：Harness 必须先能被"安装"

操作系统的第一性属性不是功能多，而是**能被安装、能自我更新、能自我诊断**。dsh 0.2.0 的桌面端把这三件事补齐：内置精确版本的运行时（消除"环境漂移"）、强制/自动更新（版本可控）、崩溃报告与致命诊断（可运营）。这一点常被忽视，却决定了 Agent 能否真正进入普通用户的桌面，而不只是工程师的终端。

### 3.2 能力接缝的"经济学"

`docs/capability-seams.md` 显示接缝已经覆盖：`llm`、`fs`、`subprocess`、`sandbox`、`ssh`、`shell`、`terminals`、`lsp`、`ptcRuntime`、`workflowEngine`、`jobs`、`schedule`、`sessionPersistence`、`sessionQuery`、`sessionProjections`、`subagents`、`skills`、`spillStore`、`web`、`mcpResources`、`browserUse`、`computerUse`、`officeToPdf`、`storage`、`credentials`……每一个都被声明为 **Service Definition / Provider / Consumer** 三元组。

**深层规律**：接缝数量随能力域线性增长，但**消费者数量随接缝超线性增长**（一个 provider 替换会带动 Bash、PTY、LSP、沙箱一起移动）。这带来了巨大的可组合性，也带来了成本——**增加一个能力，必须设计完整的三角色，而不是塞一个函数**。这解释了为何包数从 219 涨到 316。

### 3.3 会话日志：从"可重放"到"可迁移 + 可缓存"

0.1 时代的最强不变量是 `Model-visible ⟺ Logged`。0.2 把它推进为四层保证：

1. **可重放**：`deriveEventMessage()` 单一投影规则。
2. **可迁移**：相邻版本迁移链，已提交的 generation 永不改名/覆盖/删除。
3. **可缓存**：投影缓存持久化，热路径读缓存、冷路径重放。
4. **可并发**：原生 `flock` 跨进程写租约，避免多进程写坏同一个 Session。

这是本次 release **最深、最不显眼、也最重要的变化**。它决定了 dsh 能否支撑长期运行、多进程、多版本并存的真实生产环境。

### 3.4 动态工具 + 三阶段工具调用 + 系统提示入 surface

三件事共同回答一个问题：**当模型 API 开始原生支持 prompt caching 与工具目录变更时，Harness 该如何配合？**

- 动态工具更新让工具目录成为日志中的一个"可 fold 的历史"，使缓存失效点可精确界定。
- 三阶段工具调用让 UI 在模型"还在生成参数"时就能展示活动，而不必等到 `tool/call`。
- 系统提示成为 surface 节点 0，让提示变更像消息替换一样可重放、可压缩保护。

**这是 Harness 与模型 API"协同设计"的标志**：过去 Harness 是模型外面的壳，现在两者在协议层互相迁就。

### 3.5 Agent Teams：为什么"团队"比"委托"难一个数量级

一次性委托（subagent）只需保证"结果能回来"。团队需要**持久身份、共享任务状态、冲突检测（writeScopes）、消息去重、权限与取消**。dsh 把这些都落在**根 Session 日志**上（`TeamId = 根 SessionId`），因此团队状态**天然可重放、可审计**——这是它相较"另起一个房间记状态"的关键设计选择。

### 3.6 Workflow + PTC：把"编排"从模型推理中解放

逐步 tool call 编排的多轮次成本高、上下文易漂移。让模型**写一段程序**，由沙箱程序去扇出 Agent，本质是用**确定性代码**替代**概率性多轮对话**。这与"把可靠性当工程纪律"一脉相承：能写成代码的，就不要指望模型每次都"想对"。

### 3.7 Jobs + Schedule：长时程执行的两种时间尺度

- Jobs 解决"**同一时间**内的长任务"（未来完成、后台运行、完成唤醒）。
- Schedule 解决"**跨时间**的触发"（未来某个时刻/周期）。

两者都把"等待"从 Agent 的占用中解耦出来——这是从"请求-响应"到"持续运行系统"的转变。

### 3.8 治理运营化：从"沙箱存在"到"治理可运营"

0.1 已有沙箱与 fail-closed。0.2 把它推进为**可运营的治理体系**：Windows ACL 强制完整性、Auto Review 同模型评审、fatal diagnostics/crash reports、有界遥测与上传、账号/配额/凭据治理、插件清单与运行时不变式。区别在于：**治理不再是"开关"，而是"有指标、有上限、有审计、有人工回退"的持续运行能力**。

### 3.9 发行工程：Profile / Bundle / Preset 是"平台化"的骨架

"一切皆插件"只有在**能组合、能分发、能覆盖、能回滚**时才真正成立。profile（命名组合）→ bundle（发行物）→ patch（分层覆盖）→ preset（Agent 级组合）→ 可选实验包，构成了一条完整的"组合-分发"链。这是 dsh 从"框架"变成"平台"的机制基础。

### 3.10 工程纪律作为护城河

359 篇文档、≈526 条实现态决策记录、生成的 tool/config/persistence catalog、双语强制配对、文档字节预算、约 1500 个 spec。**这些"非功能性"资产恰恰是最难被复制的**：竞争对手可以模仿 API，很难模仿一套持续运转、机器强制的工程纪律。

---

## 四、Harness 演化趋势：九条结构性趋势

> 每条：**现象 → 证据 → 洞察 → 启示**。

### T1 · 从"框架"到"可发布的发行版"（Distribution Engineering）

- 现象：profile/bundle/preset/可选实验包/桌面安装器。
- 证据：`packages/bundle/*`、`docs/architecture.md#profiles-and-bundles`、`architecture/2026-08-05-profile-plugin-bundles.md`。
- 洞察：Agent 框架的竞争焦点，正从"运行时能力"转向"如何把能力打包成用户能安装、能升级、能自定义的发行物"。**一切皆插件"的终点是"一切皆可发行"**。
- 启示：评估 Harness 时，除了问"支持多少能力"，更要问"这些能力如何组合、如何分发、如何隔离与回滚"。

### T2 · 能力接缝经济学（Seam Economy）

- 现象：接缝数量爆炸，每个都是 Definition/Provider/Consumer 三元组；消费者可跨接缝联动（换 fs 提供者 = 换 Bash/PTY/LSP/沙箱）。
- 证据：`docs/capability-seams.md`（数十个 `ctx.*` 接缝）。
- 洞察：**抽象不是免费的**。接缝让系统可替换、可测试、可组合，但每个接缝都要设计完整三角色、维护契约、编写测试。成熟 Harness 的标志是**知道在哪里加接缝、在哪里用根因法收敛**（参见同期做减法）。
- 启示：设计 Agent 系统时，把"可替换性"当资产来经营，同时用"接缝预算"约束复杂度。

### T3 · 日志即脊梁（Log as Backbone）

- 现象：会话格式 v0→v4、相邻迁移链、投影缓存、跨进程写租约、`Model-visible ⟺ Logged` 成为运行时不变式。
- 证据：`packages/session/*`、`docs/architecture.md#session-log`。
- 洞察：Harness 的正确性根基不在 prompt，而在**可重建的事实日志**。可重放（replay）→ 可迁移（migrate）→ 可缓存（projection cache）→ 可并发（lease），是同一根脊梁的四次加固。
- 启示：如果系统无法从日志重建任意一次模型请求，它就不具备生产级的可审计性；日志格式的版本化与迁移纪律，是长期演进的前提。

### T4 · 从单体 Agent 到"有状态的多智能体组织"

- 现象：Agent Teams（花名册/任务 DAG/邮箱）、Workflow（程序化扇出）、Goal（目标续做）、Schedule（定时唤醒）、continuable subagent（冷恢复）。
- 证据：`experimental/agent-team*`、`workflow/*`、`goal/*`、`schedule/*`、`subagent/subagent`。
- 洞察：多智能体的难点不是"能 spawn"，而是**持久身份 + 共享状态 + 冲突与权限**。dsh 的策略是"团队状态落在根会话日志上"，从而复用事件溯源的全部好处。
- 启示：设计多 Agent 协作时，先确定"共享状态的唯一事实源"，再谈通信协议。

### T5 · 执行世界的可替换性（One Seam, Many Worlds）

- 现象：`subprocess`/`fs`/`sandbox` 既有 local、sandbox，也有 SSH 提供者；PTC 与 Workflow 复用同一沙箱；持久终端只是一个 shell 后端。
- 证据：`packages/ssh/*`、`packages/sandbox/*`、`architecture/2026-09-11-posix-ssh-runtime.md`。
- 洞察：同一份策略，可以在本地、沙箱、远端三种"执行世界"中一致执行——治理因此可以**跟随执行位置移动，而不是绑定在某台机器上**。
- 启示：把"执行环境"当作可注入的 provider，而不是把路径/命令硬编码进业务逻辑。

### T6 · 与模型 API 协同进化（Cache-aware, Protocol-modern）

- 现象：动态工具更新 + tool-changes beta 头、系统提示入 surface、一等 tool-role 消息、Anthropic Messages 协议收敛。
- 证据：`architecture/2026-09-20-dynamic-tool-updates.md`、`2026-09-02-system-prompt-as-surface-node.md`、`2026-09-15-first-class-tool-role-messages.md`。
- 洞察：Harness 不再是被动的"外层壳"，而是与模型协议层**互相迁就、共同设计**。缓存命中率、提示稳定性、工具目录变更的可控性，正成为 Harness 的核心指标。
- 启示：关注模型侧新特性（prompt caching、tool 变更、role 语义）时，要同步设计 Harness 的日志与投影语义。

### T7 · 从"能用"到"可运营"（Operationalization）

- 现象：崩溃报告/致命诊断、有界遥测与日志上传、插件清单与运行时不变式、账号/配额/凭据治理、Auto Review。
- 证据：`architecture/2026-09-22-fatal-diagnostics-and-crash-reports.md`、`2026-09-24-bounded-session-log-upload.md`、`packages/credentials/*`。
- 洞察：能演示 ≠ 能上生产。运营能力（可诊断、可限流、可审计、可人工介入）是产品与玩具的分水岭。
- 启示：给 Agent 系统排优先级时，把"故障可诊断"和"资源有上限"放在与新功能同等重要的位置。

### T8 · 本地优先 + 自主可控（Local-first & Sovereign）

- 现象：Electron 内置完整运行时、原生 Landlock/flock 原语、桌面/CLI 共享数据但独立锁文件、双 SDK（TS/Python）+ 单文件运行时、MIT 开源。
- 证据：`apps/desktop/*`、`native/system/*`、`python/*`。
- 洞察：在"数据不外流、环境不漂移、供应商不锁定"的企业诉求下，**本地内置运行时 + 开源核心**是一条差异化路线，与云端闭源 Agent 形成对照。
- 启示：技术自主性是可采用性的前提；发行形态（本地/云）本身就是一个战略选择。

### T9 · 工程纪律即护城河（Discipline as Moat）

- 现象：文档字节预算、双语强制配对、生成的 catalog、约 1500 spec、约 526 条决策记录、fail-loud、机器强制门禁。
- 证据：`docs/AGENTS.md`（预算）、`packages/README.md`、`.agents/notes/implemented/*`。
- 洞察：功能会被复制，**持续运转的工程纪律很难被复制**。这正是 dsh "可靠性 = 工程化，而非 prompt 调优"论点的组织级体现。
- 启示：投资于可执行的规范（lint/gate/budget/decision log），其回报是长期的可维护性，而非短期功能数。

---

## 五、对既有研究的更新

### 5.1 上下文管理与记忆（对比 `dsh-上下文管理记忆-架构研究报告-2026-08-23`）

原报告的核心公理（日志为唯一事实源、surface 三事件、压缩=缩小投影而非删除记忆）**依然成立且被加固**。本次新增：

| 方向 | 2026-08 基线 | 0.2.0-rc.2 新增 |
|---|---|---|
| 日志格式 | v0 结构 | v3 规范信封 + v4 一等 tool-role 消息 |
| 系统提示 | `request/header` 字段 | 成为 surface 节点 0，可替换、可压缩保护 |
| 工具目录 | 静态 header | 动态更新（`request/header.tools` + `developer/message`） |
| 投影 | 每步折叠 | `session-projection-cache` 持久化 + 跨版本读 |
| 压缩 | text summary + tool-result prune | 增加 image offload、checkpoint policy |
| 跨会话 | session-reference | 客户端会话引用 + 有界 model budget/spill 复用 |

**更新后的关键认知**：`Model-visible ⟺ Logged` 已从"不变量"升级为"可迁移 + 可缓存 + 可并发的工程体系"；上下文管理的重心正从"如何压缩"转向"如何让投影可缓存、迁移可验证"。

### 5.2 上下文路由与 SubAgent 分治（对比 `dsh-上下文路由与SubAgent分治-对比Codex-2026-08-23`）

原来的"异构能力接缝 vs 同构线程"对比依然有效，但 dsh 一方发生了**结构化升级**：

| 维度 | 2026-08 | 0.2.0-rc.2 |
|---|---|---|
| 子代理形态 | one-shot + continuable | 增加 **Agent Teams**（花名册/任务 DAG/邮箱） |
| 生命周期 | continuable 冷恢复 | 增加 **activation capacity**、人工 inbox 控制（Queue/Steer/Edit/Remove） |
| 路由 | provider 声明 | 增加模型选择路由 + 用户授权路由 |
| 编排 | 模型逐次调用 | 增加 **Workflow 脚本 + PTC** 程序化扇出 |
| 审计 | descriptor 事件 | 团队状态全部落入根 Session 日志，可重放 |

**更新后的对比结论**：dsh 用"**能力接缝 + 事件溯源**"承载异构与审计，Codex 用"**同构线程 + 工具族**"承载一致性与简洁。0.2 的 dsh 在保持异构优势的同时，补上了"团队、编排、调度"这些原本属于 Codex 强项的长时程协作能力，两者的差距从"哲学不同"收敛为"取舍不同"。

### 5.3 五大创新的演化（对比 `新发布的DeepSeek Harness的五大创新及其启示`）

| 原"创新" | 0.2.0-rc.2 的演进 |
|---|---|
| ① 会改写自己的运行时 | 保留并收敛（`extensions/*` + `dynamicCordisRunner`），同时新增 **declarative presets** 作为更安全的常规组合方式；`tool-cordis` 与 inspector 提供跨 realm 检视 |
| ② 事件溯源 + 可重放会话 | 升级为 v3/v4 + 迁移链 + 投影缓存 + 跨进程写租约（**本章 T3**） |
| ③ 异构子代理协议 | 扩展为 **Agent Teams + Workflow/PTC + continuable activation**（**T4**） |
| ④ 可移植沙箱 + fail-closed | 扩展为**可运营治理**（Windows ACL、Auto Review、崩溃诊断、有界遥测）（**T7**） |
| ⑤ 可靠性当作工程纪律 | 演化为**发行工程 + 工程纪律双轮**（profile/bundle + doc budgets + decision log）（**T1/T9**） |

---

## 六、与 Claude Code / Codex 的对比（2026-10 更新）

| 维度 | DeepSeek Harness 0.2.0-rc.2 | Claude Code | Codex |
|---|---|---|---|
| 架构定位 | 可组合 Agent 操作系统（profile/bundle） | CLI Agent | CLI Agent |
| 交付形态 | CLI / Web / **Desktop(Electron)** / ACP / JSON-RPC SDK / Python SDK | Web IDE | Web IDE |
| 会话可观测 | 事件日志 + v0→v4 迁移 + 投影缓存 + FTS5 检索 | 黑盒 | 黑盒 |
| 多智能体 | 异构子代理 + **Agent Teams** + Workflow/PTC | 有限 | 同构线程工具族 |
| 长时程 | **Jobs + Schedule + Goal + Plan** | 有限 | 有限 |
| 执行世界 | local / sandbox / **SSH** / PTC，同一接缝 | 本地为主 | 本地为主 |
| 跨平台沙箱 | bwrap/Landlock/Seatbelt/**Win-ACL** | 仅 Linux | 仅 Linux |
| 浏览器/桌面操作 | Browser-use + Computer-use 接缝 | 有 | 有 |
| 治理运营 | 崩溃报告 + 有界遥测 + 账号/配额 + Auto Review | 内部 | 内部 |
| 开源 | MIT + 完整决策记录 | 闭源 | 闭源 |

**关键差异（更新版）**：dsh 的护城河不再是"某个新功能"，而是**"发行 × 接缝 × 事件溯源 × 工程纪律"四者叠加形成的系统可演进性**。

---

## 七、风险与张力（批判性视角）

1. **复杂性预算**：55 组 / 316 包 / 数十接缝 / 21 个实验包。可组合性以认知与维护成本为代价。长期看，"如何让新贡献者不被淹没"是最大挑战。
2. **版本不稳定**：官方明确"会有兼容性破坏"，格式已跳到 v4，API 预稳定。企业采用需承担跟进成本。
3. **可选包与实验的边界**：实验能力（Teams/语音/Auto review/Schedule）默认关闭、以可选 bundle 发布——这是好设计，但也意味着"发布态"与"能力态"之间存在落差，评估时需明确启用组合。
4. **迁移纪律的成本**：相邻迁移链 + 永不删除提交代，是极强的正确性保证，但要求每个版本变更都配套迁移包与验证。这是纪律的代价，也是纪律的价值。
5. **与模型的耦合**：动态工具、system prompt surface、tool-changes beta 等特性与 DeepSeek 模型侧深度耦合。换用其它模型时，这些优化的收益可能下降。

---

## 八、给企业 / 团队的启示

1. **问"能跑"之前，先问"能装、能升、能诊"**：发行形态（本地/云/桌面）与升级、诊断能力，决定 Agent 能否进入生产。
2. **把"共享状态的唯一事实源"作为多 Agent 设计的第一决策**：dsh 选择会话日志；你的系统选什么？
3. **抽象要经营，不要堆砌**：能力接缝带来可替换性，但每个接缝都要完整三角色与测试；用"接缝预算"约束复杂度。
4. **能写成代码的，不要让模型每次都去"想"**：Workflow/PTC 用确定性程序替代概率性多轮推理，是可靠性工程的方向。
5. **治理要可运营，而非只是开关**：可诊断、有上限、可审计、可人工回退，才算治理闭环。
6. **投资可执行的工程纪律**：文档预算、决策记录、生成式 catalog、双语配对——短期无趣，长期是护城河。

---

## 九、结语

0.1 时代的 dsh 回答了"Agent 系统应当长什么样"：事件溯源、能力接缝、可重放、fail-closed。

0.2.0-rc.2 回答了一个更难的问题：**"这样的系统如何变成一个真实存在、可安装、可组合、可运营、可持续演进的产品？"** 它的答案不是某个单点功能，而是一整套结构性机制——发行工程（profile/bundle）、能力接缝经济、日志脊梁的四次加固、有状态的多智能体组织、以及把工程纪律本身当作产品竞争力。

由此看到的九条趋势，正在把 **Harness Engineering** 从"工程师 + 好的 prompt"推进为一门真正的系统工程学科：

> **框架比的是能力，操作系统比的是秩序。dsh 0.2.0-rc.2 的价值，不在它多了什么功能，而在它把"秩序"变成了可安装、可组合、可审计的工程事实。**

---

*配套文件：`FILE_INDEX.dsh-0.2.0-rc.2.md`（源码索引）· 本文档对应的 HTML 版本。分析基于 `deepseek-harness-master`（0.2.0-rc.2，2026-09-29 快照）源码与文档，路径均可回溯。*
