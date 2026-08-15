# DeepSeek Harness 0.1.0-rc.5 — Source Code Index

> **DeepSeek Harness (`dsh`)** is an open-source agent harness developed by [DeepSeek AI](https://deepseek.com). It uses an architecture where **everything is a plugin**, powered by [Cordis](https://github.com/cordiverse/cordis) (described in [_A Programming Paradigm for Spatiotemporal Composability_](https://github.com/cordiverse/paper)). Currently in **developer preview** (compatibility-breaking changes expected).

This index documents the full source tree of the `deepseek-harness-package/` release: the `@deepseek-ai/dsh-*` workspace packages (npm), the `dsh` CLI and web frontend, the Python SDK and bundled runtime, the native Landlock sandbox addon, the vendored Cordis framework source, documentation, repository gates, and agent workflows.

---

## Directory Structure Overview

| Directory | Role | Description |
|---|---|---|
| `packages/` | Source code | 54 npm groups (~219 modules) of `@deepseek-ai/dsh-*` workspaces, organized as `packages/<group>/<pkg>/` |
| `apps/` | Applications | Product entry points: `cli` (the `dsh` launcher) and `web` (browser frontend build) |
| `examples/` | Examples | Runnable `cordis.yml` demo leaves (ACP, headless, JSON-RPC, MCP, self-modification, schedule) |
| `vendor/` | Vendored framework | Pinned source copies of Cordis + foundation libraries, rescoped to `@deepseek-ai` |
| `native/` | Native addon | `landlock-run`: C11 Landlock self-restrict-then-exec launcher (`@deepseek-ai/node-addon-landlock-run`) |
| `python/` | Python SDK | `deepseek-harness-sdk` + `deepseek-harness-runtime-bin` (JSON-RPC over stdio) |
| `scripts/` | Repo gates | Build, test, verification gates, and documentation/catalog generators |
| `docs/` | Documentation | Architecture, glossary, generated catalogs, cookbook, subsystems, user guides, postmortems |
| `.agents/` | Agent workflows | Agent Notes (`notes/`) decision records + reusable `skills/` |
| `.github/` | CI / issue mgmt | GitHub Actions workflows + issue-management policy |
| `website/` | Docs site | VitePress projection of selected bilingual `docs/` sources |
| `patches/` | Patches | pnpm patches (currently `node-pty@1.1.0`) |
| `assets/` | Assets | Community-facing images |

---

## 1. Top-level Files

| File | Description |
|---|---|
| `AGENTS.md` | Root agent instructions: repository layout, commands, conventions, defensive patterns, vendoring policy |
| `CLAUDE.md` | Symlink to `AGENTS.md` |
| `README.md` / `README.zh.md` / `README.i18n.yaml` | Project readme (bilingual triplet) |
| `CONTRIBUTING.md` / `CONTRIBUTING.zh.md` / `CONTRIBUTING.i18n.yaml` | Contribution guide |
| `BENCHMARK.md` | Benchmark-running instructions (Python SDK + `jsonrpc-agent`) |
| `LICENSE` | MIT license |
| `THIRD_PARTY_NOTICES.md` | Third-party dependency license disclosures (generated) |
| `package.json` | Root workspace manifest `@deepseek-ai/dsh-root` — all `pnpm` scripts (build/test/lint/hygiene/doc-sync/release) |
| `pnpm-workspace.yaml` | Workspace globs, overrides, `allowBuilds`, `patchedDependencies` |
| `pnpm-lock.yaml` | Lockfile (680 KB) |
| `tsconfig.json` / `tsconfig.base.json` / `tsconfig.base.client.json` | TypeScript project references (host/client faces) |
| `tsconfig.host.json` / `tsconfig.client.json` | Host and client TS build faces |
| `tsdown.config.ts` | Bundler config (tsdown) for runtime output |
| `vitest.config.ts` / `vitest.shared.ts` | Unit test config |
| `vitest.e2e.config.ts` / `vitest.snapshot.config.ts` / `vitest.web.config.ts` / `vitest.web.perf.config.ts` / `vitest.web-stress.config.ts` | e2e / snapshot / web test configs |
| `knip.json` | Dead-code/exports analysis config |
| `.oxlintrc.json` / `.oxlintrc.staged.json` | Linter (oxlint) config |
| `lefthook.yml` | Git hooks (lefthook) |
| `.jscpd.json` | Copy-paste (clone) detection config |
| `.editorconfig` / `.gitattributes` / `.gitignore` / `.rgignore` | Editor / git / ripgrep config |
| `.gitlab-ci.yml` | GitLab CI config |

---

## 2. `packages/` — Source Code (54 groups, ~219 modules)

The product API spine, capability seams, and application halves. Organized as `packages/<group>/<pkg>/`, npm names `@deepseek-ai/dsh-<pkg>`. See `packages/README.md` for the full group table; `packages/AGENTS.md` for package conventions. (Additional groups not yet documented in the README table: `attachment`, `mcp`, `runtime-diagnostics`.)

### 2.1 `core/` — Product API Spine

| Package | Path | Description |
|---|---|---|
| `dsh-scope` | `core/scope` | Scoped-context registration primitive (tagged Cordis context, parent scope chains) |
| `dsh-session` | `core/session` | Event-sourced session log + in-memory `SessionStore` (`ctx.sessions`) |
| `dsh-system-prompt` | `core/system-prompt` | System-prompt assembly registry (`ctx.systemPrompt`) — ordered sections, tool schemas, persona |
| `dsh-tools` | `core/tools` | Tool registry + execution pipeline (`ctx.tools`): pre/execute/post-execute events, Code Mode, presentation |
| `dsh-agent` | `core/agent` | Agent interface, registry, process-local initiator scope, `agent/*` event vocabulary (`ctx.agents`) |
| `dsh-agent-default-model` | `core/agent-default-model` | Deployment default model selection (`ctx.agentDefaultModel`) |
| `dsh-agent-loop` | `core/agent-loop` | The concrete default agent loop driver (`ctx.agentLoop`) — the only concrete loop logic |
| `dsh-agent-tool-presentation` | `core/agent-tool-presentation` | Per-preset row declaring which tool form the model sees (native/code/both) |

### 2.2 `api/` — Remote API Layers

| Package | Path | Description |
|---|---|---|
| `dsh-api-remotes` | `api/remotes` | Host Agent/Session lookup policy + Client Remote contribution assembly |
| `dsh-api-gateway` | `api/gateway` | Two-sided Typert RPC endpoint: Host `ctx.typertGateway` + Client `ctx.remote` |

### 2.3 `typert/` — Source Analysis / Runtime Reflection

| Package | Path | Description |
|---|---|---|
| `dsh-typert-registry` | `typert/registry` | Runtime registry for generated reflection + Zod schemas (`ctx.typert`) |
| `dsh-typert-loader` | `typert/loader` | Discovers Loader entries and registers their generated `./typert` artifacts |
| `dsh-typert-generator` | `typert/generator` | TypeScript analyzer + model-driven generator for runtime artifacts (build-time) |
| `dsh-typert-protocol` | `typert/protocol` | Compiler-independent wire protocol: `TypertRemoteService`, `@Remote`/`@RemoteScope`, descriptors, codecs |

### 2.4 `goal/` — Persisted Same-session Goals

| Package | Path | Description |
|---|---|---|
| `dsh-goal` | `goal/goal` | Goal state + lifecycle (`ctx.goals`); event-sourced completion objective |
| `dsh-goal-round-driver` | `goal/goal-round-driver` | Same-session goal continuation (sequential goal rounds) |
| `dsh-tool-goal` | `goal/tool-goal` | Model-facing `get_goal`/`create_goal`/`update_goal` tools |
| `dsh-command-goal` | `goal/command-goal` | Human-facing `/goal` command |

### 2.5 `schedule/` — Session-local Reminders

| Package | Path | Description |
|---|---|---|
| `dsh-schedule` | `schedule/schedule` | Versioned Schedule events/fold, model-facing create/list/delete tools, live root-Agent timer owner |

### 2.6 `feedback/` — Recorded Human Feedback

| Package | Path | Description |
|---|---|---|
| `dsh-command-feedback` | `feedback/command-feedback` | Log-only `feedback/record` event + human `/feedback` command |
| `dsh-message-feedback` | `feedback/message-feedback` | Per-message rating/note sidecar (`ctx.messageFeedback`) + Host Remote contract |

### 2.7 `identity/` — Shared Identity

| Package | Path | Description |
|---|---|---|
| `dsh-anonymous-user-id` | `identity/anonymous-user-id` | Persists one anonymous harness-home correlation id (telemetry/feedback/DeepSeek requests) |

### 2.8 `llm/` — LLM Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-llm` | `llm/llm` | LLM service + shared streaming vocabulary (`ctx.llm`); adapter registry, `StreamChunk` protocol |
| `dsh-token-meter` | `llm/token-meter` | Replay-aware token measurement (`ctx.tokenMeter`) |
| `dsh-llm-retry` | `llm/llm-retry` | Provider-scoped retry policy via `agent/request-error` waterfall |
| `dsh-llm-deepseek` | `llm/llm-deepseek` | Direct DeepSeek chat-completions adapter (`deepseek-official` route) |
| `dsh-llm-pi-ai` | `llm/llm-pi-ai` | Generic multi-provider pi-ai adapter (OpenAI-compatible gateway, catalog-driven) |

### 2.9 `e2b/` — E2B Remote Runtime (POC)

| Package | Path | Description |
|---|---|---|
| `dsh-e2b` | `e2b/e2b` | Creates/owns one E2B sandbox lifecycle (`ctx.e2b`), shared SDK handle |
| `dsh-fs-e2b` | `e2b/fs-e2b` | Filesystem seam implemented over E2B Filesystem APIs (`ctx.fs`) |
| `dsh-subprocess-e2b` | `e2b/subprocess-e2b` | Subprocess seam over E2B Commands + PTY APIs (`ctx.subprocess`) |

### 2.10 `subprocess/` — Subprocess Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-subprocess` | `subprocess/subprocess` | Service Definition: executable lookup, managed spawns, terminal-process primitive, `DSH_*` env vocabulary (`ctx.subprocess`) |
| `dsh-subprocess-local` | `subprocess/subprocess-local` | Local Service Provider: detached process trees, bounded collection/spill, node-pty, tree signalling |

### 2.11 `shell/` — Bash Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-shell` | `shell/shell` | `ShellExecutor` contract (`ctx.shell`); Service Definition |
| `dsh-bash-local` | `shell/bash-local` | Local bash executor over `ctx.subprocess` |
| `dsh-bash-sandbox` | `shell/bash-sandbox` | Sandbox-confined bash executor (wraps argv through `ctx.sandbox`) |
| `dsh-pwsh-local` | `shell/pwsh-local` | Local PowerShell executor (Windows process behavior) |
| `dsh-pwsh-sandbox` | `shell/pwsh-sandbox` | Sandbox-confined PowerShell executor |
| `dsh-shell-env` | `shell/shell-env` | Managed `DSH_*` environment registry (`ctx.shellEnv`) |
| `dsh-tool-bash` | `shell/tool-bash` | Model-facing `bash` tool + background-job integration |
| `dsh-tool-bash-persistent` | `shell/tool-bash-persistent` | Model-facing `bash(command)` over one owner-scoped persistent `ctx.terminals` shell |
| `dsh-tool-pwsh` | `shell/tool-pwsh` | Model-facing `pwsh` tool (mirror of `tool-bash`) |

### 2.12 `terminal/` — Persistent PTY Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-terminal` | `terminal/terminal` | Backend registry, branded ids, exact-Agent ownership, session ops, awaited cleanup (`ctx.terminals`) |
| `dsh-terminal-bash` | `terminal/terminal-bash` | Shell backend over `ctx.subprocess.spawnTerminal`; readiness detection, sandbox policy |
| `dsh-tool-terminal` | `terminal/tool-terminal` | Six model-facing tools (open/send/read/signal/close/list) over `ctx.terminals` |

### 2.13 `code-runtime/` — Code-execution Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-code-runtime` | `code-runtime/code-runtime` | Service Definition: run one model-written program against host async bindings (`ctx.codeRuntime`) |
| `dsh-code-runtime-worker-thread` | `code-runtime/code-runtime-worker-thread` | Worker-thread backend (one fresh Worker per program) |

### 2.14 `sandbox/` — Process-confinement Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-sandbox` | `sandbox/sandbox` | Service Definition: `SandboxProvider`, mode vocabulary, fail-closed errors (`ctx.sandbox`) |
| `dsh-sandbox-local` | `sandbox/sandbox-local` | Local platform confinement backends (bwrap/Landlock/Seatbelt/Windows ACL) |
| `dsh-sandbox-policy` | `sandbox/sandbox-policy` | Resolves durable per-session sandbox policy (`ctx.sandboxPolicy`) |
| `dsh-sandbox-windows-acl` | `sandbox/sandbox-windows-acl` | Windows write-restriction ACL restricted-token runner (koffi FFI port) |

### 2.15 `fs/` — Filesystem Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-fs` | `fs/fs` | Service Definition: paths/URIs/containment, text IO, atomic mutation, `fs/*` policy events (`ctx.fs`) |
| `dsh-fs-local` | `fs/fs-local` | Local-filesystem implementation |
| `dsh-fs-sandbox` | `fs/fs-sandbox` | Sandbox-enforcing filesystem (mode fence on writes/edits) |
| `dsh-fs-observation-policy` | `fs/fs-observation-policy` | Policy gate plugin (observed-state + read-before-edit + version-guarded writes) |
| `dsh-tool-fs` | `fs/tool-fs` | Model-facing `read`/`read_image`/`write`/`edit` tools + executor |
| `dsh-tool-fs-search` | `fs/tool-fs-search` | Model-facing `glob`/`grep` discovery tools backed by packaged ripgrep |
| `dsh-tool-str-replace-editor` | `fs/tool-str-replace-editor` | Standalone model-facing `str_replace_editor` over `ctx.fs` |

### 2.16 `lsp/` — LSP Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-lsp` | `lsp/lsp` | Service Definition: provider registry by branded id + extension mapping, four semantic operations (`ctx.lsp`) |
| `dsh-lsp-stdio` | `lsp/lsp-stdio` | Generic multi-server stdio backend over `ctx.fs` + `ctx.subprocess` |
| `dsh-tool-lsp` | `lsp/tool-lsp` | Model-facing `lsp` tool (4 operations, one-based UTF-16 coordinates) |

### 2.17 `skill/` — Skill Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-skill` | `skill/skill` | Skill provider registration + lookup (`ctx.skills`) |
| `dsh-skill-badge` | `skill/skill-badge` | Bundled `dsh-badge` skill provider (markdown + PNG assets) |
| `dsh-skill-filesystem` | `skill/skill-filesystem` | Discovers skills from local filesystems (SKILL.md parsing) |
| `dsh-tool-skill` | `skill/tool-skill` | Publishes skill catalog + model-facing `skill` loader tool |

### 2.18 `compaction/` — Compaction Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-compaction` | `compaction/compaction` | Compaction seam + `compaction/*` event vocabulary (`ctx.compaction`) |
| `dsh-compaction-basic` | `compaction/compaction-basic` | Token-pressure + summarization backend (one-shot `llm.stream`) |
| `dsh-compaction-tool-result-pruner` | `compaction/compaction-tool-result-pruner` | Model-free tool-result pruning (`ctx.toolResultPruner`) |
| `dsh-command-compact` | `compaction/command-compact` | Human-facing `/compact` command |

### 2.19 `context/` — Request-context Extensions

| Package | Path | Description |
|---|---|---|
| `dsh-agent-instructions` | `context/agent-instructions` | Workspace-instruction context (AGENTS.md loading) |
| `dsh-session-reference` | `context/session-reference` | Bounded snapshots of other sessions (`ctx.sessionReferenceResolver`) |
| `dsh-time-context` | `context/time-context` | Current-time + elapsed-time context |
| `dsh-tmux-context` | `context/tmux-context` | tmux session/window/pane location context |

### 2.20 `subagent/` — Subagent Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-subagent` | `subagent/subagent` | Provider registration, delegation, continuation, durable descriptors (`ctx.subagents`) |
| `dsh-subagent-in-process-driver` | `subagent/subagent-in-process-driver` | Shared run driver for in-process providers |
| `dsh-subagent-spawn-in-process` | `subagent/subagent-spawn-in-process` | Fresh in-process child (empty session) |
| `dsh-subagent-fork-in-process` | `subagent/subagent-fork-in-process` | In-process child seeded from parent's completed history |
| `dsh-subagent-acp` | `subagent/subagent-acp` | Out-of-process child over ACP |
| `dsh-subagent-codex` | `subagent/subagent-codex` | Real Codex app-server child |
| `dsh-subagent-claude-code` | `subagent/subagent-claude-code` | Real Claude Code child via official SDK |
| `dsh-subagent-dsh-sdk` | `subagent/subagent-dsh-sdk` | Out-of-process Harness child via TypeScript SDK |
| `dsh-tool-subagent` | `subagent/tool-subagent` | Model-facing delegation tool |
| `dsh-tool-subagent-control` | `subagent/tool-subagent-control` | Model-facing child messaging + listing tools (`send_message`/`interrupt_agent`/`list_agents`) |
| `dsh-tool-subagent-report` | `subagent/tool-subagent-report` | Child-to-parent report channel |

### 2.21 `jobs/` — Background-job Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-jobs` | `jobs/jobs` | Job registry + lifecycle contract (`ctx.jobs`) |
| `dsh-jobs-local` | `jobs/jobs-local` | Process-local job registry |
| `dsh-tool-jobs` | `jobs/tool-jobs` | Model-facing job control + completion notices |

### 2.22 `workflow/` — Dynamic-workflow Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-workflow` | `workflow/workflow` | Workflow execution + lifecycle events (`ctx.workflowEngine`) |
| `dsh-workflow-worker-thread` | `workflow/workflow-worker-thread` | Runs workflow scripts in worker threads |
| `dsh-tool-workflow` | `workflow/tool-workflow` | General model-facing `workflow` tool |
| `dsh-tool-ralph` | `workflow/tool-ralph` | Fixed fresh-agent `ralph` workflow tool |

### 2.23 `web/` — Web Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-web` | `web/web` | Web provider registration, selection, shared errors (`ctx.web`) |
| `dsh-web-search-exa` | `web/web-search-exa` | Exa-backed search provider |
| `dsh-web-search-perplexity` | `web/web-search-perplexity` | Perplexity-backed search provider |
| `dsh-web-search-deepseek` | `web/web-search-deepseek` | Native DeepSeek (Anthropic-compatible) search provider |
| `dsh-web-fetch-http` | `web/web-fetch-http` | Anonymous HTTP(S) fetch provider |
| `dsh-tool-web` | `web/tool-web` | Model-facing `web_search`/`web_fetch` tools |

### 2.24 `attachment/` — Durable Attachment Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-attachment` | `attachment/attachment` | Immutable attachment refs, image limits, storage service (`ctx.attachments`) |
| `dsh-attachment-local` | `attachment/attachment-local` | Content-addressed private storage below `DSH_HOME` |

### 2.25 `spill/` — Tool-output Spill Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-spill` | `spill/spill` | Spill storage definition (`ctx.spillStore`) |
| `dsh-spill-local` | `spill/spill-local` | Session-scoped local file spill store |
| `dsh-spill-policy` | `spill/spill-policy` | Post-execution spill policy transformer |

### 2.26 `todo/` — Todo / Planning

| Package | Path | Description |
|---|---|---|
| `dsh-tool-todo` | `todo/tool-todo` | Model-facing `todo_write` tool (whole-list replacement) |

### 2.27 `plan/` — Plan Collaboration State

| Package | Path | Description |
|---|---|---|
| `dsh-plan-mode` | `plan/plan-mode` | Plan-mode state, guidance, commands, and review flow (`ctx.planMode`) |

### 2.28 `preset/` — Per-session Agent Composition

| Package | Path | Description |
|---|---|---|
| `dsh-agent-presets` | `preset/agent-presets` | Preset vocabulary, filesystem discovery, guarded per-agent mount (`ctx.agentPresets`) |
| `dsh-persona` | `preset/persona` | Agent persona as a composable row |

### 2.29 `guard/` — Loop-hygiene Guards

| Package | Path | Description |
|---|---|---|
| `dsh-repeat-tool-reminder` | `guard/repeat-tool-reminder` | Advisory reminders for repeated tool calls |
| `dsh-tool-call-timeout-policy` | `guard/timeout-policy` | Per-call tool deadline enforcer (`tools/execute` wrapper), zero-config |

### 2.30 `bundle/` — Profile Plugin Bundles

| Package | Path | Description |
|---|---|---|
| `dsh-base` | `bundle/base` | Shared dsh core as a patch-layer profile bundle (no runtime API) |
| `dsh-web-app` | `bundle/web-app` | Browser surface: web patch layer + `web-runtime` glue plugin |
| `dsh-headless` | `bundle/headless` | Direct one-shot task mode over base |

### 2.31 `extensions/` — Agent Runtime Self-modification

| Package | Path | Description |
|---|---|---|
| `dsh-tool-cordis` | `extensions/tool-cordis` | Model-facing runtime inspection + dynamic-package tools |
| `dsh-cordis-host-runner` | `extensions/cordis-host-runner` | Definition registry, `node:vm` sandbox, request-run round trip (`ctx.dynamicCordisRunner`) |
| `dsh-cordis-client-runner` | `extensions/cordis-client-runner` | Browser half: evaluates definitions into live browser plugins, answers run requests |
| `dsh-client-ui-cordis` | `extensions/ui-cordis` | Browser surfaces: frame-wide panel + read-only define card |

### 2.32 `hooks/` — Hook Bridges + Shared Protocol

| Package | Path | Description |
|---|---|---|
| `dsh-hook-protocol` | `hooks/hook-protocol` | Shared shell-hook wire protocol library (dialect-neutral; not a plugin) |
| `dsh-hooks-claude-code` | `hooks/hooks-claude-code` | Claude Code hook bridge plugin |
| `dsh-hooks-codex` | `hooks/hooks-codex` | Codex hook bridge plugin |

### 2.33 `session/` — Durable Session Data Plane

| Package | Path | Description |
|---|---|---|
| `dsh-session-persistence` | `session/session-persistence` | Persistence service definition + write coordination (`ctx.sessionPersistence`) |
| `dsh-session-checkpoint-policy` | `session/session-checkpoint-policy` | Semantic durability checkpoint policy |
| `dsh-session-persistence-jsonl` | `session/session-persistence-jsonl` | JSONL/zstd backend |
| `dsh-session-persistence-sqlite` | `session/session-persistence-sqlite` | SQLite backend (`node:sqlite`) |
| `dsh-session-projection` | `session/session-projection` | Projection seam: drive registry + whole-value serving (`ctx.sessionProjections`) |
| `dsh-session-projection-cache` | `session/session-projection-cache` | Persisted projection cache (`ctx.sessionProjectionCache`) |
| `dsh-session-stats` | `session/session-stats` | `sessionStats` projection unit (turn/step counts, wall times) |
| `dsh-session-telemetry` | `session/session-telemetry` | Telemetry Service Definition (sink contract + capture coordinator) |
| `dsh-session-telemetry-otel` | `session/session-telemetry-otel` | OpenTelemetry backend (live/replay/local modes) |
| `dsh-session-title` | `session/session-title` | Log-backed session titles + provider seam (`ctx.sessionTitle`) |
| `dsh-session-title-llm` | `session/session-title-llm` | Shared LLM-backed title generation library |
| `dsh-session-title-all-prompts-llm` | `session/session-title-all-prompts-llm` | Title provider summarizing every eligible prompt |
| `dsh-session-title-first-prompt-llm` | `session/session-title-first-prompt-llm` | Title provider summarizing the first eligible prompt |

### 2.34 `session-query/` — Session Retrieval Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-session-query` | `session-query/session-query` | Trusted reads, relationship queries, search (`ctx.sessionQuery`) |
| `dsh-session-query-sqlite` | `session-query/session-query-sqlite` | SQLite FTS5 query implementation |
| `dsh-session-log-export` | `session-query/session-log-export` | Web `/export` command + download state/modal (`ctx.sessionLogDownload`) |
| `dsh-tool-session-query` | `session-query/tool-session-query` | Workspace-authorized model query tools (search/trace/read) |

### 2.35 `settings/` — User-settings Capability Family

| Package | Path | Description |
|---|---|---|
| `dsh-settings` | `settings/settings` | Namespace registration, layered resolution, commits (`ctx.settings`) |
| `dsh-settings-file` | `settings/settings-file` | File-backed provider (YAML/JSON, hot external edit publishing) |

### 2.36 `credentials/` — Credential References

| Package | Path | Description |
|---|---|---|
| `dsh-credentials` | `credentials/credentials` | Credential-reference seam (`ctx.credentials`); refs not secrets |
| `dsh-credentials-local` | `credentials/credentials-local` | Environment + local-file provider |

### 2.37 `storage/` — Non-session Storage Family

| Package | Path | Description |
|---|---|---|
| `dsh-storage` | `storage/storage` | Backend registry + typed data forms (`ctx.storage`) |
| `dsh-storage-json` | `storage/storage-json` | JSON-file backend |
| `dsh-storage-sqlite` | `storage/storage-sqlite` | SQLite backend (`kv` facet) |
| `dsh-storage-domain` | `storage/storage-domain` | Validated domain-record storage (`ctx.storageDomain`) |

### 2.38 `workspace/` — Workspace Entity

| Package | Path | Description |
|---|---|---|
| `dsh-workspace` | `workspace/workspace` | Workspace registry + session membership (`ctx.workspaceRegistry`) |

### 2.39 `sdk/` — Out-of-process Runtime SDK

| Package | Path | Description |
|---|---|---|
| `dsh-sdk-protocol` | `sdk/protocol` | SDK runtime wire protocol (newline-delimited JSON-RPC + named types) |
| `dsh-sdk-client` | `sdk/client` | TypeScript client API driving a Harness runtime subprocess |
| `dsh-sdk-jsonrpc-server` | `sdk/server` | Serves out-of-process SDK clients over stdio JSON-RPC |

### 2.40 `acp/` — Agent Client Protocol Automation

| Package | Path | Description |
|---|---|---|
| `dsh-acp` | `acp/acp` | Automation-only ACP server over JSON-RPC stdio |

### 2.41 `interaction/` — Human-collaboration Plane

| Package | Path | Description |
|---|---|---|
| `dsh-commands` | `interaction/commands` | Registers/dispatches human commands (`ctx.commands`) |
| `dsh-user-approval` | `interaction/user-approval` | One-shot approval coordination (`ctx.approval`) |
| `dsh-permission-presets` | `interaction/permission-presets` | User-facing permission presets (`ctx.permissionPresets`) |
| `dsh-user-questions` | `interaction/user-questions` | Provider-neutral human Q/A seam (`ctx.userQuestions`) |
| `dsh-tool-ask-user` | `interaction/tool-ask-user` | Model-facing `ask_user_question` tool |

### 2.42 `boot/` — Shared App-bin Boot Glue

| Package | Path | Description |
|---|---|---|
| `dsh-app-boot` | `boot/app-boot` | Shared boot glue: `.env` loading, Loader guards, snapshot-aware config, settle-tree boot |
| `dsh-cmdline` | `boot/cmdline` | Launcher→app command-line handoff (`cmdlineArgs`, `appExit`) |

### 2.43 `host/` — Web-GUI Host Half

| Package | Path | Description |
|---|---|---|
| `dsh-host-apiproxy` | `host/apiproxy` | Shared host API gateway + wire contract (`ctx.apiProxy`) — typed API surface per domain |
| `dsh-host-webserver` | `host/webserver` | HTTP route carrier (`ctx.webServer`) |
| `dsh-host-frontend-static` | `host/frontend-static` | SPA dist server on the webserver fallback seat |
| `dsh-host-directory-picker` | `host/directory-picker` | Workspace-directory picking seam (`ctx.directoryPicker`) |
| `dsh-host-directory-picker-native` | `host/directory-picker-native` | Native OS-chooser backend + browser interaction |
| `dsh-host-directory-picker-browse` | `host/directory-picker-browse` | In-app directory-browser backend + interaction |
| `dsh-host-directory-picker-auto` | `host/directory-picker-auto` | Host-adaptive picker composition |
| `dsh-host-plugin-inventory` | `host/plugin-inventory` | Read-only projection of Loader entries (Remote `pluginInventory/list`) |

### 2.44 `client/` — Web-GUI Browser Half (`@deepseek-ai/dsh-client-*`)

| Package | Path | Description |
|---|---|---|
| `dsh-client-web` | `client/web` | Web shell kernel — two-stage boot (module face + plugin face) |
| `dsh-client-modules` | `client/modules` | Browser peer of Node's ESM loader (lazy CJS table) |
| `dsh-client-web-react` | `client/web-react` | Shell runtime → React rendering glue |
| `dsh-client-connection` | `client/connection` | Browser-host RPC communication + event delivery |
| `dsh-client-runtime` | `client/runtime` | Shared client services: sessions, workspaces, projection store, UI composition |
| `dsh-client-hmr` | `client/hmr` | Client plugin hot reload during development |
| `dsh-client-locale` | `client/locale` | Localization preferences + message dictionaries |
| `dsh-client-schema-form` | `client/schema-form` | Schema-backed draft handling for settings editors |
| `dsh-client-ui-slots` | `client/ui-slots` | Slot registry core + terminal design (React-free, cordis-free) |
| `dsh-client-ui-theme` | `client/ui-theme` | Applies the selected color theme |
| `dsh-client-ui-primitives` | `client/ui-primitives` | Shared React controls, icons, markdown renderers |
| `dsh-client-ui-attachment` | `client/ui-attachment` | Attachment display atoms (draft rail, gallery, lightbox, drop overlay) |
| `dsh-client-ui-layout` | `client/ui-layout` | Arranges main app regions (three-column AppFrame, panel geometry service) |
| `dsh-client-ui-sidebar` | `client/ui-sidebar` | Workspace/session navigation sidebar |
| `dsh-client-ui-workspace` | `client/ui-workspace` | Workspace selection + creation surfaces |
| `dsh-client-ui-conversation` | `client/ui-conversation` | Conversation domain: chat view, composer, details, controller |
| `dsh-client-ui-input-trigger` | `client/ui-input-trigger` | `/` and `@` input trigger pipeline |
| `dsh-client-ui-agent-preset` | `client/ui-agent-preset` | Agent-preset selection/management surfaces |
| `dsh-client-ui-commands` | `client/ui-commands` | Client command API + popupSelect surfaces |
| `dsh-client-ui-deliverables` | `client/ui-deliverables` | Produced-files + clickable-reference feature |
| `dsh-client-ui-goal` | `client/ui-goal` | GoalBar surface in the composer dock |
| `dsh-client-ui-jobs` | `client/ui-jobs` | Background-job list action in session header |
| `dsh-client-ui-message-feedback` | `client/ui-message-feedback` | Per-message Like/Dislike + note controls |
| `dsh-client-ui-model-selection` | `client/ui-model-selection` | Model selection directory + composer trigger |
| `dsh-client-ui-permission-presets` | `client/ui-permission-presets` | Permission preset settings row |
| `dsh-client-ui-plan` | `client/ui-plan` | Plan-mode status chip |
| `dsh-client-ui-settings` | `client/ui-settings` | Settings domain base layer (settingsScope, slot types) |
| `dsh-client-ui-settings-general` | `client/ui-settings-general` | Settings shell (chrome, navigation, General section) |
| `dsh-client-ui-settings-models` | `client/ui-settings-models` | Models settings page + onboarding dialogs |
| `dsh-client-ui-settings-plugin-inventory` | `client/ui-settings-plugin-inventory` | Read-only Plugin list tab |
| `dsh-client-ui-settings-plugins` | `client/ui-settings-plugins` | Plugins settings section + per-plugin config cards |
| `dsh-client-ui-skill` | `client/ui-skill` | Skill invocation source (input trigger) |
| `dsh-client-ui-subagent` | `client/ui-subagent` | Subagent catalog tree, composer replacements, `@` reference source |
| `dsh-client-ui-tool` | `client/ui-tool` | Tool presentation (root + atomic toolviews) |
| `dsh-client-ui-trajectory` | `client/ui-trajectory` | Turn-aware event ledger (timeline, inspector) |
| `dsh-client-ui-user-questions` | `client/ui-user-questions` | Question composer + plan-review panel |
| `dsh-client-ui-workflow-run` | `client/ui-workflow-run` | Reconstructs workflow runs as Chat nodes |
| `dsh-client-ui-directory-picker-browse` | `client/ui-directory-picker-browse` | In-app directory browsing dialog (browser half) |
| `dsh-client-ui-directory-picker-native` | `client/ui-directory-picker-native` | Renderless native chooser interaction (browser half) |

### 2.45 `examples/` — Demo Bundles

| Package | Path | Description |
|---|---|---|
| `dsh-agent-spine-demo` | `examples/agent-spine-demo` | Reusable executor-less, UI-less agent-spine bundle |
| `dsh-acp-demo` | `examples/acp-demo` | ACP automation application bundle |
| `dsh-sdk-jsonrpc-demo` | `examples/jsonrpc-demo` | External-config JSON-RPC runtime app |

### 2.46 `test-support/` — Development and Test Infrastructure

| Package | Path | Description |
|---|---|---|
| `dsh-acp-snapshot` | `test-support/acp-snapshot` | ACP snapshot-test toolkit |
| `dsh-agent-loop-testkit` | `test-support/agent-loop-testkit` | Mounts shared prerequisites for AgentLoop tests |
| `dsh-client-test-runtime` | `test-support/client-runtime` | jsdom slot test runtime for client feature specs |
| `dsh-llm-mock-server` | `test-support/llm-mock-server` | Deterministic OpenAI-compatible fault server |
| `dsh-llm-replay` | `test-support/llm-replay` | Replays recorded model responses (keyless) |
| `dsh-loader-smoke` | `test-support/loader-smoke` | Launches Loader-composed apps for smoke tests |

### 2.47 `util/` — Low-level Shared Utilities (zero-dependency)

| Package | Path | Description |
|---|---|---|
| `dsh-brand` | `util/brand` | Nominally branded types (`Branded<B>`) |
| `dsh-home-paths` | `util/home-paths` | Resolves Harness data root (`DSH_HOME`) and shared paths |
| `dsh-timeout` | `util/timeout` | Deadline and timeout-classification primitives |
| `dsh-output-retention` | `util/output-retention` | Bounds retained text and item collections |
| `dsh-atomic-write` | `util/atomic-write` | Replaces files atomically |
| `dsh-native-command` | `util/native-command` | Runs host-native commands without a shell (execFile) |
| `dsh-launch-environment` | `util/launch-environment` | Immutable launch-environment snapshot remembering source layers |

### 2.48 `mcp/` — Model Context Protocol

| Package | Path | Description |
|---|---|---|
| `dsh-mcp-client` | `mcp/mcp-client` | MCP client bridge registering external server tools on `ctx.tools` (`mcp__<server>__<name>`) |

### 2.49 `runtime-diagnostics/` — Development-time Invariant Checks

| Package | Path | Description |
|---|---|---|
| `dsh-invariants` | `runtime-diagnostics/invariants` | Configurable invariant registry (`ctx.invariants`); every package publishes a `./invariant` companion |

---

## 3. `apps/` — Applications

### 3.1 `apps/cli/` — `@deepseek-ai/dsh` (the `dsh` launcher)

| File | Description |
|---|---|
| `src/args.ts` | Commander adapter; launcher command grammar (`--profile`, `--patch`, `--dump-config`, `web`, `plugin`) |
| `src/bin.ts` | Executable entry (`node --import tsx/esm apps/cli/src/bin.ts`); reads version, dispatches mode |
| `src/profile-boot.ts` | Profile boot: resolves profile, stacks patch layers, mounts tree, signal handling, config-only HMR |
| `src/process-shutdown.ts` | Bounded/escalating process-exit controller (SIGINT=130, SIGTERM=0) |
| `src/plugin.ts` | `dsh plugin --profile <name>` — initializes profiles, reconciles `dsh.profile.bundles` |
| `src/dump-config.ts` | Boot-free `--dump-config`/`--dump-default-config` |
| `config/agent-presets/` | Shipped agent presets: `code/`, `cordis/`, `minimal/`, `standard/` (each `agent.cordis.yml` + `preset.yml`) |
| `reference/README.md` | Exact layer precedence, app-argument contract, plugin mgmt, deployment behavior |
| `composition.md` | Generated Mermaid graph of the dsh-base bundle plugin tree |
| `tests/` | args/process-shutdown/compat specs, snapshot, e2e (`built-bin.e2e.ts`, `web-agent-presets.e2e.ts`, `headless-shutdown.e2e.ts`) |

### 3.2 `apps/web/` — `@deepseek-ai/dsh-web-frontend`

| File | Description |
|---|---|
| `src/main.ts` | Thin bootstrap: finds `#root`, runs `new AppWebEntry(el).run()` from `@deepseek-ai/dsh-client-web` |
| `src/node-module-stub.ts` | Browser stand-in for `node:module` (`createRequire`) + type-only `LoadHookContext` |
| `vite.config.ts` | Build config: rejects standalone serve, vendor-chunk strategy, output layout, workspace aliases |
| `index.html` | SPA entry (zh-CN, `#root`, manifest + favicon) |
| `public/` | `favicon.svg`, `manifest.webmanifest` (PWA) |
| `stress-tests/` | `reasoning-chunks.stress.ts` — reasoning-chunk rendering stress scenario |
| `tests/` | ~90 browser e2e/snapshot/perf tests (`scaffold.ts`, `assembled-boot.ts`, `*.e2e.ts`) |

---

## 4. `examples/` — Runnable Demo Leaves

Umbrella workspace member for runnable demo compositions (dependency resolution only, not a build target). Each leaf keeps `cordis.yml` wiring + e2e/snapshot scenarios.

| Example | Description |
|---|---|
| `acp-agent/` | Automation-oriented ACP server over JSON-RPC stdio (`pnpm run demo:acp`); ~28 scenario overlays + fixtures |
| `headless-agent/` | One-shot headless coding agent (`dsh --profile headless "task"`) for replay + real-model tests |
| `jsonrpc-agent/` | Unattended coding agent for the Python SDK's bundled JSON-RPC runtime; `minimal.py` Python driver |
| `mcp-memory/` | Three default-off MCP memory reference overlays (memorix, mcp-reference-memory, engram) |
| `web-cordis/` | Self-referential demo: agent inspects/mounts its own Cordis plugins (`demo:cordis`) |
| `web-schedule/` | Session-local Schedule reminders overlay (`time-context` + `schedule`) |

---

## 5. `vendor/` — Vendored Cordis Source

Pinned source copies of Cordis + foundation libraries, renamed into the `@deepseek-ai` scope. Manifest + sync procedure in `vendor/README.md`. 18 logged local modifications. (Do not edit `vendor/*/src/` casually.)

| Directory | npm name | Role |
|---|---|---|
| `cordis/` | `@deepseek-ai/cordis` | Framework core: `Context`, `Service`, `Fiber`, typed events |
| `cosmokit/` | `@deepseek-ai/cosmokit` | Shared utilities (framework + Schemastery foundation) |
| `schemastery/` | `@deepseek-ai/schemastery` | Config schemas (`Schema`) behind every plugin `Config` |
| `loader/` | `@deepseek-ai/cordis-plugin-loader` | `cordis.yml` loading, plugin resolution, repository cache |
| `include/` | `@deepseek-ai/cordis-plugin-include` | Config includes and patch overlays (`applyEntryPatches`, `!!js` dialect) |
| `group/` | `@deepseek-ai/cordis-plugin-group` | Plugin group lifecycle |
| `timer/` | `@deepseek-ai/cordis-plugin-timer` | Timer plugin |
| `hmr/` | `@deepseek-ai/cordis-plugin-hmr` | Hot module reload / config watching |
| `logger-console/` | `@deepseek-ai/cordis-plugin-logger-console` | Console logger plugin |

---

## 6. `native/` — Native Landlock Sandbox Addon

`native/landlock-run/` — `@deepseek-ai/node-addon-landlock-run`: ~300 lines of C11 over raw Landlock kernel UAPI (statically linked against musl), installs a Landlock ruleset on itself then `exec`s the wrapped command. Fail-closed (exit 125).

| Path | Description |
|---|---|
| `packages/entry/src/main.c` | The entire C11 launcher (Landlock UAPI structs, raw syscalls 444/445/446, CLI parse, ABI negotiation, `restrict_self`) |
| `packages/entry/src/index.ts` | JS API: `launcherPath()`, `probe()`, `grantArgs()`, constants `LAUNCHER_BIN`/`LAUNCHER_FAILURE_EXIT` |
| `packages/linux-x64/` / `packages/linux-arm64/` | Platform npm packages with `prebuilds.json` + static-musl `bin/landlock-run` |
| `scripts/` | `build.ts`, `github-matrix.mjs`, `assemble-prebuilds.mjs`, `bump-release.mjs`, `pack-release.mjs`, `publish-release.mjs`, `verify-*.mjs` |
| `docs/` | `architecture.md`, `cli-contract.md`, `naming.md`, `packaging.md`, `release.md`, `support-matrix.md` |
| `test/` | `entry.test.js`, `launcher.test.js` (real-kernel proofs) |

---

## 7. `python/` — Python SDK & Bundled Runtime

Two packages driving the harness as a subprocess over newline-delimited JSON-RPC on stdio.

### 7.1 `python/sdk/` — `deepseek-harness-sdk` (module `deepseek_harness`)

| File | Description |
|---|---|
| `src/deepseek_harness/__init__.py` | Public exports: `DeepSeekHarness`, `HarnessClient`, `RunResult`, `Session`, error types |
| `src/deepseek_harness/api.py` | `DeepSeekHarness` (lazy runtime start, context manager), `Session.run()` inbox/idle-wait loop |
| `src/deepseek_harness/client.py` | `HarnessClient` (subprocess lifecycle, JSON-RPC request/notify/request-response, reader threads) |
| `src/deepseek_harness/errors.py` | `HarnessError`, `TransportClosedError`, `SdkProtocolError`, `JsonRpcError` |
| `src/deepseek_harness/models.py` | Type aliases + pydantic `ServerInfo`/`InitializeResponse` |
| `tests/` | `test_client.py`, `test_smoke_model.py`, `test_runtime_resolution.py`, `test_bundled_runtime.py`, `test_macos_deployment_target.py`, `test_release_version.py`, `manual_sdk_agent_smoke.py` |
| `pyproject.toml` / `uv.lock` | hatchling build; deps `pydantic>=2.12,<3` + `deepseek-harness-runtime-bin` |

### 7.2 `python/sdk-runtime/` — `deepseek-harness-runtime-bin` (module `deepseek_harness_runtime`)

| File | Description |
|---|---|
| `src/deepseek_harness_runtime/__init__.py` | Platform/arch tag maps; `bundled_runtime_path()`, `resolve_bundled_launch_args(mode)` |
| `src/deepseek_harness_runtime/runtime/cordis.yml` | Checked-in default agent config (`sdk-jsonrpc-server`, `agent-spine-demo`, `llm-deepseek`, JSONL persistence) |
| `src/deepseek_harness_runtime/deepseek-harness-runtime.json` | Package metadata sentinel |
| `hatch_build.py` | `RuntimeBuildHook` — wheel-only tags, validates runtime payload |
| `platforms.json` | linux-x64 / linux-arm64 / macos-arm64 platform tags |
| `package.json` | Private `dsh-jsonrpc-agent-pkg` deploy root (~110 plugin workspace deps = the compiled plugin set) |

---

## 8. `scripts/` — Repo Gates & Generators (~150 files)

Grouped by function (see `scripts/AGENTS.md`):

| Group | Representative scripts | Description |
|---|---|---|
| Gate runner | `run-gates.ts` | Orchestrates all local/CI quality gates with bounded in-process scheduling |
| Documentation gates | `verify-md-links.ts`, `verify-md-wrap.ts`, `verify-doc-refs.ts`, `verify-doc-budgets.ts`, `verify-mermaid.ts`, `verify-type-equiv.ts`, `verify-export-jsdoc.ts`, `doc-typecheck.ts` | Links, one-line paragraphs, budgets, Mermaid, drifted types, JSDoc, fenced-ts compile |
| Source/invariant gates | `verify-cordis-config.ts`, `verify-config-source-ownership.ts`, `verify-package-invariants.ts`, `verify-package-paths.ts`, `verify-runtime-closure.ts`, `verify-node-next-types.ts`, `check-workspace-constraints.ts` | Source-plane / runtime-invariant checks |
| Agent-notes gates | `verify-agent-note-format.ts`, `verify-archived-agent-notes.ts`, `verify-agent-note-classification.ts`, `verify-skill-invocation-metadata.ts` | Notes/skills gates |
| Translation gates | `translation-pairing.ts`, `translation-pairing-git.ts`, `translation-pairing-merge.ts`, `translation-prompt.ts`, `translation-brief.ts`, `verify-translation-pairing.ts`, `verify-translation-prompt.ts` | Bilingual pair consistency |
| Catalog generators | `gen-tool-catalog.ts`, `gen-config-catalog.ts`, `gen-persistence-catalog.ts`, `gen-cordis-catalog.ts`, `gen-cordis-api.ts`, `gen-module-graph.ts`, `gen-doc-graphs.ts`, `gen-scoped-events.ts`, `gen-client-catalog.ts`, `gen-third-party-notices.ts` | Emit `docs/*-catalog.md` + graphs |
| Release | `release/bump.ts`, `release/pack.ts`, `release/publish.ts`, `release/verify.ts`, `release/process.ts`, `release/families.ts`, `publish-npm-baseline.ts`, `build-exe-for-python-sdk.ts`, `build-python-release.py` | Version/pack/publish sequences |
| Build/repo helpers | `clean.ts`, `dev-web.ts`, `change-scope.ts`, `install-lefthook.mjs`, `rescope-vendor.ts`, `ts-project.ts`, `package-graph.ts`, `cordis-walk.ts`, `slot-walk.ts`, `jsdoc.ts`, `markdown.ts` | Shared helpers |
| Demos / CI shell | `demo-code-mode.mjs`, `demo-cordis.mjs`, `prepare-ci-bubblewrap.sh`, `wine-windows-gates.sh`, `check-macos-deployment-target.py`, `smoke-python-runtime.py` | Demos + platform helpers |
| Tests | ~44 `*.spec.ts` | Scripts tested like code (`run-gates.spec.ts`, `gen-cordis-catalog-partition.spec.ts`, …) |

---

## 9. `docs/` — Documentation

Bilingual triplet convention (`.md` + `.zh.md` + `.i18n.yaml`). Tier taxonomy + budgets in `docs/AGENTS.md`.

### 9.1 Top-level Reference / Guides

| File | Type | Description |
|---|---|---|
| `architecture.md` | Reference | Ordered map of the system; read before changing `packages/` |
| `glossary.md` | Reference | Canonical domain vocabulary (capability-seam, agent-scope, etc.) |
| `development.md` | Tutorial+Ref | New-contributor setup + contributor reference (repo layout, TS project layout, CI) |
| `testing.md` | Reference | Testing policy tier by tier (unit, coverage, e2e, snapshot, web) |
| `cordis-primer.md` | Tutorial | Cordis in five ideas (plugin/Service, context, inject, typed events, reversible effects) |
| `api-gateway.md` | Reference | Typert API Gateway (`@Remote`/`@RemoteScope`, Host/Client contract generation) |
| `defensive-patterns.md` | Reference | Bug-class rules for lifecycle/concurrency/subprocess/teardown work |
| `rescope.md` | Reference | Name mapping of vendored packages into the `@deepseek-ai` scope |
| `web-styling.md` | Reference | Web UI styling ownership (`ui-theme` owns `--dsw-*` tokens) |

### 9.2 Generated Reference Docs (auto-generated; freshness-gated by `doc-sync`)

| File | Description |
|---|---|
| `config-catalog.md` | Every `cordis.yml` config block per package (JSDoc declarations) |
| `tool-catalog.md` | Every model-facing tool schema |
| `persistence-catalog.md` | Session durable event log catalog |
| `module-graph.md` | Inter-package dependency Mermaid graph |
| `agent-lifecycle.md` | Agent turn/step lifecycle sequence diagram |
| `capability-seams.md` | Capability seams & core services flowchart |
| `event-producer-consumer.md` | Matrix of events (declared-in, dispatchers, listeners) |
| `tool-execution-pipeline.md` | Tool call pipeline graph |
| `graph-atlas.md` | Index of all generated relationship diagrams |

### 9.3 Subdirectories

| Directory | Description |
|---|---|
| `cookbook/` | Step-by-step how-tos (`adding-a-package`, `adding-a-tool`, `adding-a-conversation-node`, `adding-an-llm-adapter`, `adding-a-vendored-package`, `extension-cookbook`, …) |
| `subsystems/` | ~43 subsystem reference pages, one per subsystem (core, session, persistence, tools, sandbox, llm-streaming, typert, …) |
| `user/` | Product-facing guides published by the VitePress site (`guide/`, `develop/basic/`, `develop/framework/`, `develop/practice/`) |
| `cordis-tutorial/` | 7-chapter plugin-author walkthrough (`01-first-plugin` … `07-into-the-harness`) |
| `cordis-api/` | Generated Cordis core API reference (context, events, fiber, registry, service, inherited) |
| `postmortem/` | Incident write-ups (`0001-acp-default-export-drops-inject`, … `0004-…`) |
| `i18n/` | Bilingual documentation contract (pairing, terminology, translation rules/prompt, style samples) |

---

## 10. `.agents/` — Agent Workflows & Agent Notes

### 10.1 `notes/` — Agent Notes (decision records)

Encoded in path `{lifecycle}/{class}/yyyy-mm-dd-topic-title.md`; bilingual triplets. Navigation by folder path (deliberate no-index decision).

| Lifecycle | Description | Count |
|---|---|---|
| `implemented/` | Shipped decisions kept current (present tense) | ~505 notes (×2 zh) |
| `archived/` | Frozen historical snapshots (append-only) | ~142 notes |
| `proposed/` | Proposals awaiting implementation | ~25 notes |
| `rejected/` | Considered and declined | ~11 notes |

Class folders (closed set): `architecture`, `bug-fix`, `feature`, `process`, `simplification`, `testing`.

### 10.2 `skills/` — Reusable Workflows (11 skills)

| Skill | Description |
|---|---|
| `dsh-archive-agent-notes` | Classifies/supersedes/archives implemented notes |
| `dsh-code-review` | Orients PR review to repo standards (`change-scope` report) |
| `dsh-doc-site-sync` | Keeps the VitePress website a tested projection of repo Markdown |
| `dsh-doc-standards` | Applies `docs/AGENTS.md` placement/budgets/slop audit |
| `dsh-find-simplifications` | Turns "find simplifications" requests into evidence-backed proposed notes |
| `dsh-merging-stacked-prs` | Lands official GitHub PR stacks via `gh stack merge` |
| `dsh-pre-push-checks` | Selects smallest test/check set covering an outgoing diff |
| `dsh-prose-standard` | Editorial standard for required prose coverage |
| `dsh-translate-docs` | Extended bilingual-doc workflow (user-invocable) |
| `dsh-trim-cot-leakage` | Finds/fixes chain-of-thought leakage prose |
| `record-browser-gif` | Records GUI demos as deterministic GIFs |

---

## 11. `website/` — VitePress Documentation Site

`@deepseek-ai/website` — locally projected publication of selected bilingual `docs/` sources; holds only VitePress config, presentation assets, and the publication manifest.

| File | Description |
|---|---|
| `docs.ts` | Canonical publication manifest: locale/sidebar/page collections + builder helpers |
| `.vitepress/config.ts` | VitePress config: dual-locale theme, sidebars from `docs.ts`, search, Mermaid, dev-server watcher |
| `public/favicon.svg` / `public/wordmark.svg` | Site favicon + DeepSeek wordmark |

---

## 12. `.github/`, `patches/`, `assets/`

### 12.1 `.github/` — CI & Issue Management

| Path | Description |
|---|---|
| `workflows/ci.yml` | Main CI (push + PR) |
| `workflows/{e2e,e2b-e2e,pi-ai-provider-e2e}.yml` | Real-API / E2B / pi-ai e2e |
| `workflows/{release,release-vendor,python-release,build-exe-for-python-sdk}.yml` | Release workflows |
| `workflows/{landlock-run,landlock-run-release,sandbox}.yml` | Native addon + sandbox |
| `workflows/docs-pages.yml` | Deploy documentation |
| `workflows/{issue-lifecycle,issue-policy,expected-filenames}.yml` | Issue management |
| `issue-management/policy.mjs` | Single policy module (`lifecycle`/`pr` subcommands) driving ProjectV2 board |
| `issue-management/config.json` | Project board state machine config |
| `ISSUE_TEMPLATE/` | `bug`/`feature`/`idea`/`research`/`task` templates |
| `pull_request_template.md` | Requires linked issue + change/verification section |
| `dependabot.yml` | npm/uv/github-actions dependency updates |

### 12.2 `patches/` & `assets/`

| Path | Description |
|---|---|
| `patches/node-pty@1.1.0.patch` | pnpm patch for `node-pty` spawn-helper resolution (`DSH_NODE_PTY_SPAWN_HELPER`) |
| `assets/*.png` | Community images (wechat official account, wecom assistant, wecom survey) |

---

## 13. Code Statistics Summary

| Metric | Value |
|---|---|
| Total files (excl. node_modules/.git) | 7,404 |
| Total directories | 1,206 |
| TypeScript source (`*.ts`) | 2,319 |
| TypeScript/JSX components (`*.tsx`) | 259 |
| Markdown (`*.md`) | 2,348 |
| YAML (`*.yaml` + `*.yml`) | 1,236 |
| JSON (`*.json`) | 689 |
| JSONL fixtures (`*.jsonl`) | 264 |
| CSS | 111 |
| Python (`*.py`) | 19 |
| C source (`*.c`) | 2 (`native/landlock-run` launcher + entry) |
| npm package groups (`packages/`) | 54 |
| Package modules (`packages/<group>/<pkg>`) | ~219 |
| Vendored Cordis packages | 9 |
| Python packages | 2 |
| Native addon packages | 3 |
| Agent Notes (implemented/archived/proposed/rejected) | ~680 |
| Agent skills | 11 |
| Documentation subsystem pages | ~43 |

**Key facts:**

- **Package manager:** pnpm 11.7.0; **Runtime:** Node.js `^22.19.0 || >=24.0.0`; **Language:** TypeScript (`strict: true`), ESM-only (`"type": "module"`)
- **npm scope:** `@deepseek-ai/dsh-*`; framework peerDependency: `@deepseek-ai/cordis`
- **Test framework:** Vitest (unit, coverage, e2e, snapshot, web) + pytest (Python SDK)
- **Build:** `tsc` (types) + `tsdown` (runtime bundles) + Vite (web)
- **Architecture:** plugin-based agent harness on vendored Cordis — **everything is a plugin**
- **License:** MIT
