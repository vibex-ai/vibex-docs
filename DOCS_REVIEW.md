# Vibex 文档站评估报告

> 评估对象：`vibex-docs`（Mintlify 站点，简体中文 / 繁体中文 / 英文三语，每语 24 页，共 72 页；线上另有 9 个未入库的 changelog 页面）
> 对照基准：`vibex-ai/vibex` 仓库 **`v0.1.0-rc.3`**（当前最新发布，2026-09-12）与线上站点 <https://vibex.peatboy.com/docs>
> 评估日期：2026-09-15

---

## 0. 结论摘要

你的直觉是对的：**这个文档站目前是"架构说明"而不是"产品文档"。** 它读起来像是给已经了解 Vibex 的人做的一份设计约束备忘，而不是给新用户的上手材料。

具体表现为四个层次的问题：

| 优先级 | 问题类别 | 数量 | 典型例子 |
| --- | --- | --- | --- |
| **P0** | **内容与真实产品不符** | 7 项 | Config Center 10 个分区里 7 个打不开；安装页让用户去装 Rust；开发者指南引用了 4 个不存在的命令 |
| **P0** | **核心能力完全没有文档** | 9 项 | 无头服务端 `vibex-server`、托管 Agent 安装、跨会话搜索 |
| **P1** | **可用性：看不懂也做不了** | 6 项 | 全站 0 张截图；没有字段级参考；术语与 UI 不一致 |
| **P2** | **结构、元数据、工程流程** | 8 项 | 本地仓库落后于线上；无 SEO 元数据；无参照类章节 |

最关键的两条：

1. **文档的主论点已经错了。** 首页和开发者指南反复强调"桌面端是唯一的权威运行时"，但 rc.3 已经交付了 `vibex-server`——一个与桌面端共用 `DesktopRuntime` 内核的无头权威运行时，桌面端可以降级成它的远程客户端。这句话现在会直接误导用户的部署决策。
2. **文档把"代码里存在"当成了"用户能用"。** Config Center 的 10 个分区里有 7 个在 rc.3 的桌面端**没有任何入口**（`Scheduled`、`Automation`、`Prompts & Hooks`、`Advanced`、`Relay`、`Recovery`、以及作为独立标签页的 `Model Providers`）。`scheduled-automation.mdx` 整页、`mcp-skills-prompts-hooks.mdx` 的两节、`troubleshooting.mdx` 的恢复步骤，用户都照做不了。

第三条同样关键：**本地 git 仓库已经落后于线上站点。** 线上有 `/changelog`、`/changelog/v0.1.0-rc.2`、`/changelog/v0.1.0-rc.3`（三语共 9 个页面），本地仓库里**不存在**。说明有人在 Mintlify 网页编辑器里直接改，没有回灌仓库。

---

## 1. 评估方法

1. 通读 `vibex-docs` 全部 51 个 MDX 页面（17 用户页 + 7 开发者页，× 3 语言）。
2. 以 **`v0.1.0-rc.3` tag** 为基准（而非本地 `main`——本地 `main` 比 rc.3 多 32 个未发布提交，且工作区有未提交改动），逐条核对文档中的命令、UI 标签、枚举值、协议字段。
3. 用 `git grep v0.1.0-rc.3` 提取真实枚举与用户可见字符串；用 `apps/desktop/src/locale.rs`（rc.3 版，1686 行，含 en/zh-CN/zh-TW 三语字符串表）作为 UI 文案的权威来源。
4. 用 `gh release view` 与解包真实 `.deb` 产物核对发布事实。
5. 抓取线上 `sitemap.xml` 与页面文本，和本地仓库做差异比对。

**版本基线说明（重要）**：本报告以 **`v0.1.0-rc.3`**（当前 GitHub Release 的最新版本，也是用户实际能下载到的版本）为核对基准。本地工作树在 `main` 上，比 rc.3 多 32 个提交且含未提交改动，其中至少有一处会改变结论（`Remote Runtime` 设置分区在 rc.3 存在、在 rc.3 之后被移除）。凡涉及这种版本差异的地方，报告里都单独标注。

---

## 2. P0：内容与真实产品不符

### 2.1 ❌ "桌面端是唯一的权威运行时" —— 已过时，且会误导部署决策

**现状**：`index.mdx:10`、`en/index.mdx:6`、`zh-TW/index.mdx:8`、`developer/index.mdx:6`、`developer/architecture.mdx:6` 都在说桌面端是唯一权威运行时。

**真相**：rc.3 的发布说明第一条亮点就是"远程权威端已承接整个工作台"。仓库里有 `apps/server`（`vibex-server`）和 `deploy/server/`（README + systemd unit + Caddyfile + compose），其自述是：

> "the desktop runtime and the cloud server are the same `DesktopRuntime` core behind different frontends — the desktop adds a GPUI window, the server adds a daemon lifecycle. Both expose the identical Remote v2 gateway, so a paired mobile client cannot tell them apart."
> —— `deploy/server/README.md`

**影响**：用户读到"桌面端是唯一权威"就不会想到可以自建服务器；反过来，已经自建服务器的用户会以为文档写错了。

**改法**：把措辞从"唯一权威"改为"权威运行时"（authority），并明确存在**两种权威端形态**：
- 内嵌在桌面端里的本地运行时（默认）
- 独立的无头 `vibex-server`（自托管 / 局域网 / 公网）

桌面端和移动端都是**客户端**，可以连接到其中任意一个。这句话是整站的地基，必须最先改。

---

### 2.2 ❌ 安装页把"想用产品的人"当成了"想改代码的人"

**现状**：`install.mdx` 全篇 71 行，从第 6 行开始就是 `git clone` + `pnpm install --frozen-lockfile` + `pnpm dev:desktop`。第 27 行的 Note 说"正式安装包由项目发布流程生成……请查看仓库 Release 页面"，但**没有给任何下载地址、任何平台安装步骤**。

**真相**（已用 `gh release view` 与解包 `.deb` 核实）：rc.3 的 GitHub Release 有 **26 个产物**，桌面端覆盖 6 个平台/架构组合：

| 平台 | 产物格式 | 备注 |
| --- | --- | --- |
| Linux x86_64 | `.deb` + `.AppImage` | 依赖 `libvulkan1`、`libxcb*`、`libxkbcommon*`、`libdbus-1-3`、`libfontconfig1`、appindicator |
| Linux aarch64 | `.deb` | 无 AppImage（linuxdeploy 无法交叉构建） |
| macOS aarch64 / x86_64 | `.dmg` | **未签名、未公证**，首次启动会被 Gatekeeper 拦截 |
| Windows x86_64 / aarch64 | NSIS `.exe` | **未签名**，SmartScreen 会警告；per-user 安装，无需管理员 |
| Android arm64 | `.apk` + `.aab` | APK 可侧载 |

**普通用户完全不需要 Rust / Node / pnpm / Git。** 安装包是自包含的（约 200 MB）。Node 22 只在安装托管 Agent 时按需从 nodejs.org 拉取。

**同时还漏掉了一整套"发布通道"概念**（这是用户必然会踩的坑）：

- 通道由构建时决定，不可运行时切换：`preview` / `rc` / `stable`
- 三个通道有**各自独立的 productName、app id 和 SQLite 数据库目录**（`desktop-preview` / `desktop-rc` / `desktop-stable`），所以可以并存安装、互不干扰
- rc.3 是 RC 通道。`releases/latest` 现在指向 rc —— 营销站那个"Download Vibex"按钮把所有人导向 RC
- macOS/Windows 未签名；**Android RC 的 APK 每次发布用临时 CI 密钥签名，因此不同 RC 之间无法覆盖升级，必须先卸载**

**改法**：把 `install.mdx` 拆成两页，或者重写成"用户安装"为主、"从源码构建"折叠到页面底部或移到开发者指南。新增：

- `install.mdx` —— 面向用户：按平台给下载链接、安装命令、首次启动放行说明（Gatekeeper / SmartScreen）、依赖说明、卸载方法
- `install/from-source.mdx` —— 面向贡献者：现有的 clone + pnpm 流程
- `updates.mdx` —— 更新机制、通道概念、数据目录隔离、如何切换通道

---

### 2.3 ❌ 开发者指南引用了 4 个不存在的命令

这是最容易被验证、也最伤信任的一类错误。

| 文档中写的命令 | 出现位置 | 实际情况 |
| --- | --- | --- |
| `pnpm check:all` | `developer/build-and-test.mdx:14` | **不存在** |
| `pnpm check:acp` | `build-and-test.mdx`, `contributing.mdx`, `en/`, `zh-TW/` | **不存在** |
| `pnpm check:code-workbench` | 同上 | **不存在** |
| `pnpm check:terminal` | 同上 | **不存在** |

已核对的 rc.3 `package.json` 实际脚本共 38 个。真实对应关系：

| 文档想表达的 | 应该写的真实命令 |
| --- | --- |
| ACP / Agent 检查 | `pnpm smoke:agents`（= `smoke:acp:bridge-contract` + `smoke:codex` + `smoke:claude`） |
| 代码工作台检查 | `pnpm check:ui-visual` + `pnpm smoke:files` + `pnpm smoke:git` |
| 终端检查 | `pnpm smoke:pty` |
| "更广的检查" | `pnpm check`（含 rust / frontend / licenses / ui-widget-layering / ui-component-ratchet / ui-visual） |

**还漏掉了一大批真实存在的门禁**：`check:release`、`check:module-ownership`、`check:source-size`、`e2e:regression`、`smoke:db`、`smoke:backup`、`smoke:diagnostics`、`baseline:performance`、`check:native-content-package`、`check:move-only`、`check:ui-fixture-sandbox`。

**改法**：`build-and-test.mdx` 应该改成一张**从 `package.json` 生成的完整命令表**，并加一条 CI 校验（脚本名变化时文档不同步就报错）。

---

### 2.4  Config Center 的 10 个分区里，有 7 个在桌面端根本打不开

**现状**：`config-center.mdx:11-23` 列了一张 10 行的分区表——Agents、Model Providers、MCP、Skills、Prompts & Hooks、Advanced、Scheduled、Automation、Relay、Recovery。

**真相**（已在 rc.3 tag 上逐行核对 `apps/desktop/src/management.rs`）：Config Center 的顶部导航只构建 **3 个标签页**：

```rust
// rc.3 apps/desktop/src/management.rs:8616 render_nav
let items = [
    (ManagementSection::Agents, copy.agents, IconName::Bot),
    (ManagementSection::Mcp,    copy.mcp,    IconName::Network),
    (ManagementSection::Skills, copy.skills, IconName::BookOpen),
];
```

其余分区被显式归一化掉了：

```rust
// rc.3 apps/desktop/src/management.rs:16639
fn management_primary_section(section: ManagementSection) -> ManagementSection {
    match section {
        Agents | ModelProviders => Agents,
        Mcp => Mcp,
        Skills => Skills,
        PromptsHooks | Advanced | Scheduled | Automation | Relay | Recovery => Advanced,
    }
}
```

而 `Advanced` **没有对应标签页**。`navigation.active` 在整个 rc.3 里只被赋值为 `Agents`（`:1659`、`:1670`），`select_section` 只由那 3 个标签页触发。`app.rs` 里对 `Advanced / Scheduled / Automation / Relay / Recovery` 的引用为 **0**。

**也就是说 rc.3 的用户实际只能打开：Agent、MCP、Skills。**

由此产生的具体错误：

| 文档写的 | 实际情况 |
| --- | --- |
| `config-center.mdx` 10 行分区表 | 只有 3 个可达；其余 7 个是死代码路径 |
| `quickstart.mdx:41` "在 Config Center 的 Agents 或 **Model Providers** 中添加" | **没有 Model Providers 标签页**。供应商配置在 Agent 详情页里的 **Model provider configuration** 卡片中 |
| `agents-and-providers.mdx:21` "打开 **Model Providers**，添加 Codex、Claude 或 ACP 类型的供应商配置" | 同上，用户照做会找不到入口 |
| `scheduled-automation.mdx` 整页（51 行） | **Scheduled 与 Automation 在桌面端不可达**，是 API-only（只能经 Remote v2 访问） |
| `mcp-skills-prompts-hooks.mdx` 的 Prompts 与 Hooks 两节 | 位于不可达的 Advanced 页内 |
| `privacy-and-recovery.mdx:24`、`troubleshooting.mdx` 多处 "打开 **Recovery**" | Recovery 不可达 |
| `scheduled-automation.mdx:53` 运行状态 | 真实枚举是 `Queued / Running / Succeeded / Failed / **Skipped** / Canceled`，文档写的 `waiting` 和 `recovered` 不存在，且漏了 `skipped` |
| `scheduled-automation.mdx:41-47` 6 种节点 | 真实 `AutomationNodeKind` 只有 5 种（AgentPrompt / ApprovalGate / FileCheck / GitCheck / TerminalCheck）；Manual/Scheduled 是 **trigger** 不是节点 |

**更麻烦的是**：即使打开 Advanced 页，Scheduled 任务的 "Run" 按钮**并不会真的运行任务**——它只把 `next_run_at_ms` 清空并插入一条 `Running` 的运行记录，真正的执行器 `ScheduledTaskRunner` 从未被 app 或 server 启动过。自动化图里的 `FileCheck` / `GitCheck` / `TerminalCheck` 三种节点在执行时**必定失败**（`automation/unsupported_node_kind`）。

**这不是文档写错了，而是文档把"代码里存在"当成了"用户能用"。** AGENTS.md 里"不要承诺只存在于 spike、测试或未发布分支的功能"这条规则，在这里被违反了。

**改法（二选一，建议先内部确认）**：
- **若 rc.3 就是当前发布基线**：把 `config-center.mdx` 的分区表改成只有 Agent / MCP / Skills 三行，并明确写出"供应商配置在 Agent 详情页中"；`scheduled-automation.mdx` 要么下线、要么标注为"暂未在桌面端开放"；Prompts/Hooks 同理。
- **若这些标签页即将开放**：等它们真正可达后再发布对应文档，并在此之前用 `<Warning>` 明确标注当前版本不可用。

> 补充：本节结论先在 rc.3 tag 上验证（`git show v0.1.0-rc.3:apps/desktop/src/management.rs`）。本地 `main` 已经把这个文件拆成了 `app/ui/domain/management/*`，但结论一致——导航同样只构建 3 个标签页。

---

### 2.5 ❌ 远程章节：少了一种配对方式、少了一种连接串，还提到了不存在的开关

`remote-mobile.mdx` 与 `troubleshooting.mdx` 的远程部分**只覆盖了桌面→手机这一条路径**，而 rc.3 实际有两条完全不同的配对路径：

| 路径 | 连接串 | 有效期 | 谁发起 |
| --- | --- | --- | --- |
| 桌面端生成、手机扫码（SAS 校验码） | `vibex://open/<transport>#/pair/<offer>` | 60–120 秒 | 桌面端 |
| 无头服务器生成、桌面端或手机填入 | `vibex://pair#/code/<payload>` | 默认 **5 分钟**，最长 30 分钟 | `vibex-server` |

文档只写了第一种（`developer/remote-protocol.mdx:12`、`remote-mobile.mdx:20-23`），第二种完全没提——包括它携带 DER 证书、做证书指纹固定（`sha256:…`）、因此局域网自签场景不需要公共 CA 这个关键点。

其他具体偏差：

| 文档写的 | 实际 |
| --- | --- |
| 三种连接方式：Direct / Tailnet / Self-hosted Relay | 还有第 4 种 **Local network pairing（局域网配对）**，零配置发现，主机名 `vibex-lan.local` |
| `troubleshooting.mdx:35`「确认桌面端 **Remote gateway** 已启用」以及 `remote-mobile.mdx` 多处「Remote gateway」 | **`Remote gateway` 不是任何用户可见字符串**（`rg -i "remote gateway" apps/desktop/src` → 0 命中），用户按文档找不到这个东西 |
| `remote-mobile.mdx:32` 「full_control：可发送消息、写入文件、执行终端输入和 Git 写操作」 | ✅ 权限模型正确（`read_only` / `approve_only` / `full_control`；13 类操作；级别由桌面端在创建 offer/code 时决定，客户端无法自行提权） |
| `remote-mobile.mdx:38` 「文件编辑和 Git 操作的真实结果始终来自桌面端」 | 表述**低估了移动端**：移动端**可以**编辑并保存文件、执行终端输入、做 Git 写操作（stage/revert/commit/fetch/push/worktree create），只是权威状态在宿主端 |
| `usage.mdx:47` 「移动端只接收经过权限过滤的投影」 | 移动端用量页**只显示会话数**，并提示 "Usage details are available on the desktop host." |
| 未提 | 移动端**没有 Config Center**，无法管理供应商配置；Settings 只有 Connection / Session timeline / Appearance / Notifications / About |

**关于 "Remote Runtime" 设置分区的版本说明（衔接 2.6）**：这个分区在 **rc.3 存在**，但在 rc.3 之后的提交 `3e72051`（2026-09-13）里被移除了——"Settings → Remote Runtime is gone: the manager is the single place runtimes are mutated"。在下一个版本里，配对与权威端切换统一走标题栏的 **Runtime manager**（`cmd-shift-o`，内含 "Pairing code" / "Connection string" 两个标签、`Switch to this runtime`、证书指纹展示）。

**这恰好暴露了文档站缺的一样东西：没有声明自己描述的是哪个版本。** 建议在站点上明确"本文档对应 vX.Y.Z"，否则每次发版都会出现这种"代码改了，但没人知道文档该不该跟着改"的漂移。

---

### 2.6 ❌ Settings 页面漏了一整个分区

**现状**：`settings.mdx:15` 的分区表列了 General / Appearance / Workbench / Session / Terminal / Shortcuts / Data & Diagnostics / About。

**真相**：rc.3 的 `SettingsSection` 枚举是——

```rust
enum SettingsSection {
    General, Appearance, RemoteRuntime, Workbench,
    Session, Terminal, Shortcuts, Data, About,
}
```

**漏掉的是 `RemoteRuntime`（远程运行时）。** 这是一个完整的设置分区，用来：

- 输入服务器地址 + 一次性配对码来 **Pair**
- 粘贴 `vibex://pair#/code/…` 连接串（自带证书，无需系统 CA）
- 把桌面端从"本地权威端"切换为"远程客户端模式"
- **Forget** 清除凭据并切回本地运行时
- 显示固定的服务器证书指纹 `sha256:…` 供人工比对

这也是 2.1 那条错误的直接后果——因为文档认定桌面端是唯一权威，就自然没有"切换权威端"这个页面。

---

### 2.7 ❌ 本地仓库落后于线上站点

**线上 `sitemap.xml` 有 81 个 URL，本地仓库只对应 72 个。** 差的 9 个是三语版的 `/changelog`、`/changelog/v0.1.0-rc.2`、`/changelog/v0.1.0-rc.3`。

线上 changelog 页面文本已抓取确认存在，且导航里已经加了"版本记录"标签页。这 9 个页面的 `lastmod` 是 2026-09-12（rc.3 发布当天），但**从未提交到 git**。

**影响**：仓库不再是唯一事实源；下一次有人从仓库发布会**删掉线上已有的 changelog**。

**改法**：
1. 立刻从线上导出这 9 个页面回灌仓库（Mintlify 支持导出，或直接从线上 HTML 还原）
2. 确定单一发布流程：要么只从 git 部署，要么接受网页编辑器，但**必须定期回灌**
3. 短期可以加一个 CI 检查：线上 sitemap 与仓库页面集合不一致就告警

---

## 3. P0：核心能力完全没有文档

以下能力在 rc.3 已实现、且是用户会主动寻找的，但文档站**一个字都没有**：

### 3.1 无头运行时 `vibex-server`（最大的空白）
`apps/server`、`deploy/server/`。文档站全文 grep `vibex-server`、`ghcr.io` 均为 **0 命中**。缺失内容包括：
- 容器镜像 `ghcr.io/vibex-ai/vibex-server`（多架构 amd64/arm64，tag 规则 `rc` / `latest` / `edge`）
- 一次性数字配对码（`NNN-NNN-NNN`，默认 5 分钟 TTL，只存 SHA-256，单次使用）
- `vibex-server pairing-code` / `status` / `revoke DEVICE_ID` 子命令
- 三种部署形态：loopback、局域网（自签证书 + 指纹固定）、公网 HTTPS
- 完整环境变量表（`VIBEX_HOME`、`VIBEX_DB_PATH`、`VIBEX_WORKSPACE_ROOTS`、`VIBEX_BIND_ADDR`、`VIBEX_DEPLOYMENT_MODE`、`VIBEX_TLS_MODE`、`VIBEX_ALLOWED_HOSTS`、`VIBEX_ALLOWED_ORIGINS`…… 约 24 个）
- `VIBEX_PROVIDER_SECRET_STORE=keychain|file` 的取舍（无头主机没有可用 keychain，默认写 `provider-secrets.json`，仅属主可读）
- systemd 部署（`deploy/server/systemd.md`）

### 3.2 托管 Agent 安装（Vibex 帮你装 Agent）
文档 `agents-and-providers.mdx` 只说"你也可以注册任意兼容 ACP 的可执行程序：填写命令、参数、环境变量"。**完全没提 Vibex 可以自己下载、安装、检查更新和卸载 Agent 运行时**（`install_managed_agent` / `check_managed_agent_update` / `uninstall_managed_agent`）。这是新手最需要的功能——不然用户得自己先去装 Claude Code 或 Codex。

### 3.3 桌面端连接服务器（权威端切换）
见 2.4。搜索框里的 "Using / Waiting to switch to / Preparing / Still using / Use current" 就是切换权威端时的状态文案，文档没有任何说明。

### 3.4 跨会话搜索
`session_search_placeholder: "Search sessions and messages"`、`session_search_open: "Search sessions"`、`session_search_loading: "Indexing session messages..."`。这是一个**跨全部会话和消息正文的搜索索引**（rc.3 发布说明里的 "session search index"）。文档 `sessions.mdx` 只写了"在当前会话中按 cmd-f 搜索时间线"。

### 3.5 子 Agent / 委派时间线
`sessions.mdx:38` 只有一行 "delegation | 子 Agent 或委派任务的状态和结果"。实际 rc.3 支持**加载子 Agent 独立时间线**，还有无头二进制内置的 agent-delegation MCP sidecar（`--agent-delegation-mcp`）。

### 3.6 自动更新与发布通道
`crates/app-update` 是一个完整的签名更新器：Ed25519 验签、GitHub Releases Atom feed 发现、AppImage 真正自我替换 / deb+NSIS 拉起系统安装器、启动 30 秒后首次检查、rc 通道每 2 小时一次。UI 在 **Settings → About**（"Check now" / "Download" / "Install update" / "No newer signed release is available"）。文档只有 `settings.mdx:74` 的一句"提供检查已签名发行版更新的入口"。

### 3.7 移动端到底怎么装
- Android：可以从 GitHub Release 侧载 `arm64-v8a` APK（`applicationId = ai.vibex.mobile`）
- **iOS：只有未签名的模拟器包（`ios-simulator.app.zip`）和 XCFramework。没有真机包、没有 TestFlight、没有 App Store。** 也就是说普通 iPhone 用户**装不了**移动端。

文档 `remote-mobile.mdx` 通篇假设"配对后的移动端可以……"，却从没说移动端怎么获取。这会浪费用户大量时间。

### 3.8 变更日志 / Release Notes
`docs/operations/release-notes-v0.1.0-rc.{2,3}.md` 有完整的中英文发布说明（rc.3 = 91 个提交）。线上站点已经有了（见 2.5），但**本地仓库没有**，而且文档站没有从任何地方链接到它。建议正式建一个 `changelog` 标签页并入库。

### 3.9 设备管理与审计
`RemoteDevicePermissionLevel`（`read_only` / `approve_only` / `full_control`）、设备列表、撤销、**设备审计记录**（rc.3 新增"读取设备审计记录"）。文档 `remote-mobile.mdx` 只有权限级别的三行说明，没有设备管理操作路径。

---

## 4. P1：可用性——新人看不懂，也照做不了

### 4.1 全站 0 张截图

**51 个页面，零个 `![...]()`。** 对一个 GUI 应用来说这是致命的。而素材其实是现成的：

- `vibex/docs/assets/showcase/desktop-en.png` / `desktop-zh.png`（README 头图）
- `vibex-site/public/assets/workbench.png`、`git-diff.png`、`providers.png`、`mobile-session.png`
- `vibex/docs/assets/agents/*.svg`（16 个 Agent logo）

Mintlify 支持 `<Frame>` 组件。至少这些页面必须有图：首页、快速开始、代码工作台、Git、终端、Config Center、Settings、用量统计、移动端配对。

### 4.2 全是概念描述，没有字段级参考

随便举个典型句子（`agents-and-providers.mdx:22`）：

> "2. 填写显示名称、base URL、模型和 reasoning effort（如果供应商支持）。"

用户读完仍然不知道：显示名称有没有字符限制？base URL 要不要带 `/v1`？模型列表从哪来（手填还是探测）？reasoning effort 的可选值是什么？

**同样的模式出现在每一页。** 缺少的是 Mintlify 的 `<ParamField>` / `<ResponseField>` / 表格形式的字段参考。

### 4.3 没有任何任务导向的"怎么做"

现有页面全部是"功能说明书"式的。缺的是这种：

- 5 分钟接入 Claude Code
- 5 分钟接入 Codex（含登录流程）
- 配一个 OpenAI 兼容 / Anthropic / Google Vertex 供应商
- 用 worktree 并行跑两个 Agent 再对比
- 把手机配对到桌面端
- 自建一台服务器，让笔记本和手机都连上去
- 从 Claude Code 迁移历史会话
- 写第一个 SKILL.md
- 配一个每天早上跑一次的定时任务

### 4.4 术语与真实 UI 对不上

这是我逐条比对 `locale.rs`（rc.3）得到的。**用户按文档找 UI，是找不到的**：

| 文档用词 | 真实 UI 文案（zh-CN） | 位置 |
| --- | --- | --- |
| 推理深度 | **思考深度** | `reasoning_depth` |
| conversation mode（未翻译） | **对话模式** | `conversation_mode` |
| 供应商配置 | **模型供应商** | `model_provider_label` |
| 工作区 / 打开工作区 | **项目** / 侧栏 `Projects`、`新建项目` | `sidebar_projects`、`sidebar_new_project` |
| Agent 会话 / 新建会话 | **新建会话**（EN 是 `New chat`） | `sidebar_new_session` |
| 远程配对 | **配对移动设备** | `pair_mobile` |
| Provider、Provider matrix | 文档混用英文 Provider 和中文"供应商" | — |

### 4.4.1 更严重的是：文档漏掉了 **项目 → 工作区** 的层级模型

这不是措辞问题，是**概念模型缺失**。真实的数据结构是：

```rust
// crates/core/src/workspace.rs
pub enum WorkspaceMode { CurrentCheckout, VibexWorktree }

pub struct ProjectRecord   { id, name, root_path, created_at_ms, updated_at_ms }

pub struct WorkspaceRecord { id, project_id, root_path, mode, ... }   // ← 挂在 project 下

pub struct ProjectWorkspaceSummary {
    project: ProjectRecord,
    workspace: WorkspaceRecord,
    aggregate_status: WorkspaceAggregateStatus,
    git_branch: Option<String>,
    git_dirty: bool,
}
```

也就是说真实模型是 **项目（1）→ 工作区（N）**：

- 「**项目**」是用户创建的、持久的、带名字的记录（所以侧栏有 `新建项目` / `删除项目` / `删除项目？`）
- 「**工作区**」属于某个项目，带一个 `mode`
- 同一个项目下**可以同时存在多个工作区**——这正是 `CurrentCheckout` 和 `VibexWorktree` 的由来：worktree 不是"另一种模式"，而是**同一个项目下的第二个工作区**
- `workspaces.mdx:11` 把两种模式写成二选一的"两种工作区模式"，实际上它们是同一个项目里可以并存的两个工作区

文档 `workspaces.mdx` 全篇把"工作区"当成用户选择的最顶层对象，从不提"项目"，导致：

1. 用户看到侧栏的"项目"和文档里的"工作区"对不上
2. 用户不理解为什么可以同时有一个 CurrentCheckout 和一个 worktree
3. `usage.mdx` 里 `Project` 作为独立筛选维度显得莫名其妙

**改法**：新写一个「项目与工作区」概念页，用一张图讲清楚 `项目 → 工作区（CurrentCheckout | VibexWorktree）→ 会话` 的层级，并同步修正 `workspaces.mdx`、`git.mdx`、`usage.mdx` 的表述。

### 4.5 缺一个概念页 / 术语表

新人搞不清这些词的关系：工作区、项目、会话、时间线、运行、回合、Agent、供应商配置、模型、通道、权威端、Runtime、Remote v2、Relay、worktree、Skill、Prompt、Hook、MCP、委派、elicitation。文档默认读者已经懂，实际没有。

### 4.6 故障排查没有可执行内容

`troubleshooting.mdx` 全是"确认……检查……查看……"，没有任何**具体错误码**。而架构文档自己说"权限不足必须返回结构化错误"。实际代码里有大量结构化错误码，例如：

- `remote_agent_workspace_root_missing`
- `remote_agent_workspace_mode_mismatch`
- `remote_pairing_code_invalid`
- `remote_pairing_code_expired`
- `release_channel_override_rejected`
- `provider_native_import_source_missing`
- `device_revoked`、`resync_required`、`server_shutdown`

应该做一张**错误码 → 含义 → 处理**的表格。这是排查页最有价值的部分，现在完全没有。

---

## 5. P2：结构与内容质量

### 5.1 缺少"参照类"内容（reference）
整个站没有一节是查得动的：没有 CLI/环境变量参考、没有配置字段参考、没有错误码表、没有完整快捷键表、没有协议字段表。Mintlify 的 API 参考能力（OpenAPI、`<ParamField>`）完全没用。

### 5.2 前端元数据几乎为空
51 个页面的 frontmatter 只有 `title` 和 `description`。没有 `keywords`、没有 `og:image`、没有 `sidebarTitle`、没有 `icon`。对 SEO 和分享卡片都不利。（`docs.json` 里有 favicon 和 logo，但页面级 OG 图没有。）

### 5.3 架构页的 crate 覆盖不全
`developer/architecture.mdx` 只列了 8 个 crate，实际 workspace 有 **23 个 crate + 4 个 app**。缺的包括 `agent`、`agent-acp`、`agent-claude`、`agent-codex`、`db`、`fs`、`git`、`terminal`、`content`、`remote`、`relay`、`backup`、`diagnostics`、`app-update`、`config-switch`、`desktop-runtime`、`vibex-markdown`、`vibex-terminal-ui`，以及 **`apps/server`**。

### 5.4 自动化图的节点描述与代码不符
（该页本身在桌面端不可达，见 2.4；即使用 Remote v2 访问，描述也与实现不符。）

`scheduled-automation.mdx:41-47` 把 "Manual trigger / Scheduled trigger" 列为节点类型。实际代码里触发器和节点是两套东西：

- `AutomationNodeKind` = `AgentPrompt` / `ApprovalGate` / `FileCheck` / `GitCheck` / `TerminalCheck`（**5 种，不是 6 种**）
- `AutomationGraphTrigger` = `Manual` / `ScheduledTask`（触发器，不是节点）
- 边条件是第三个枚举：`Always` / `OnSuccess` / `OnFailure` / `OnApproval`，文档完全没提
- 节点有 `position: {x, y}`，且 `FileCheck` / `GitCheck` / `TerminalCheck` 在执行时必定失败（`automation/unsupported_node_kind`）

### 5.5 快捷键表不完整
可重绑定的快捷键共 **14 个**（`FOUNDATION_SHORTCUTS`，rc.3），文档只列了 9 个，漏了：

| 操作 | 默认键 | 分组 |
| --- | --- | --- |
| Open runtimes（打开运行时） | `cmd-shift-o` | Runtime |
| Go to line in editor（跳转到行） | `ctrl-g` | Editor |
| Undo image edit（撤销图片编辑） | `cmd-z` / `ctrl-z` | Image editor |
| Redo image edit（重做图片编辑） | `cmd-shift-z` / `ctrl-shift-z` | Image editor |

而且文档没有体现快捷键的**分组**（Workbench / Composer / Navigation / Runtime / Editor / Image editor），也没有说明快捷键**可以重绑定**（Settings → Shortcuts，带冲突校验和 Reset all）。另外 `code-workbench.mdx` 和 `settings.mdx` 各有一张部分重叠的快捷键表，两张都不完整——应该合并成一张完整的参考页。

### 5.6 其他具体的事实错误（逐条可验证）

| 文档位置 | 文档写的 | 实际 |
| --- | --- | --- |
| `git.mdx:26` | "按文件或**块**执行 Stage / Unstage" | 只有整文件暂存，**没有 hunk 级暂存**（`GitMutationKind` 只有 `Stage`/`Unstage`） |
| `git.mdx` | 未提 | 没有 stash、没有 tag、diff 只有 unified 一种（无并排视图） |
| `code-workbench.mdx:41-45` | "打开图片后进入编辑模式，完成裁剪或其他支持的编辑" | 图片编辑器**只挂在 Composer 附件上**（发送前标注图片），不是工作区图片文件的通用编辑器 |
| `code-workbench.mdx:36` | "Office 文档：预览受支持的 Office 文件内容" | 只有 `docx`/`xlsx`/`ods`/`pptx` 能渲染；`doc`/`xls`/`ppt` 会被分类识别但**报 `office_legacy_format_unsupported`**；`odt`/`odp` 完全不识别 |
| `terminal-and-previews.mdx` / `settings.mdx:36` | 暗示滚动缓冲区可配置 | 终端 scrollback **硬编码 10000 行，不可配置** |
| `settings.mdx:36` | "Terminal 设置默认 shell 和新建终端的初始目录" | 正确；但文档未提"默认 shell 可从检测到的 shell 中选，也可自定义路径" |
| `mcp-skills-prompts-hooks.mdx:14` | MCP 作用域 "global、user、project 或 workspace" | 编辑器里**只提供 User / Workspace 两个选项**（Global / Project 存在于枚举但 UI 不提供） |
| `mcp-skills-prompts-hooks.mdx:26-29` | Skills "设置作用域，并在 Agent/Provider 关联矩阵中选择使用者" | Skill 编辑器**没有作用域选择器**，也没有路径/SKILL.md 字段 |
| `usage.mdx:41` | Coverage "完整上报、推导、部分上报还是未知" | ✅ **正确**，对应 `Complete / Derived / Partial / Unknown` |
| `sessions.mdx:47-51` | 14 个导入来源 | ✅ **正确**，与 `LocalHistorySource::ALL` 完全一致 |
| `usage.mdx` 维度 | Time / Agent / Project / Model provider / Model + Session 筛选 | ✅ **正确** |

### 5.7 `install.mdx` 泄漏了文档站自身的构建约束
`install.mdx:10`：

> "| Node.js | 22。Node 26 不受当前 Mintlify 和部分构建脚本支持。 |"

"Mintlify" 是文档站自己的技术栈，和用户安装 Vibex 毫无关系。这类内部约束应移到贡献者指南或 `AGENTS.md`。

### 5.8 内链太少，信息层级偏平
全站只有 10 个不重复的内部链接（`/developer/remote-protocol` 出现 3 次，其余各 1 次）。页面之间基本不互相引用，读者很容易走到死胡同。首页的导航卡片也只指了 4 个页面。

### 5.9 Mintlify 能力没用起来
只用了 8 种组件（Warning 8、Card 8、Step 6、Info 4、Steps 2、Note 2、Columns 2、Tip 1）。没用 `<Tabs>`（多平台命令极适合）、`<Accordion>`（FAQ/错误码）、`<Frame>`（截图）、`<CodeGroup>`、`<ParamField>`、`<Update>`（changelog）、`<Tooltip>`。

---

## 6. i18n 问题

三语的**标题层级是对齐的**（每页 heading 数量一致），这一点做得不错。但仍有问题：

1. **繁体中文用「專案」15 次**，而 AGENTS.md 明确要求繁体统一用「工作區」、简体用「工作区」。`zh-TW/privacy-and-recovery.mdx:6`「保存專案、工作區、會話……」把两者并列，读起来像是两个不同的东西。
2. **英文页是中文直译**，不是地道的英文技术写作。例如 `en/index.mdx:6` 的 "The desktop is Vibex's only authoritative runtime"（同时也是事实错误）。
3. **没有共享术语表**，三语各自漂移，无法靠人工长期维持。
4. **AGENTS.md 要求"无法确认的实现细节用 `TODO` 标注"，但全站 0 个 TODO**——而实际上有多处无法核实的表述（比如那句 Node 26）。要么是没人遵守，要么是这条规则没有落地手段。
5. 术语大小写/形式不统一：文档里 `Provider` 和「供应商」混用，`Remote gateway` 和「远程网关」混用。
6. **产品自身有未本地化的界面**，文档不应承诺这些界面有中文：`pdf_surface.rs` 与 `office_surface.rs` 里**没有任何 `locale::text` 调用**——`PDF Preview`、`Open PDF`、`Fit width`、`PAGES`、`Page N`、`Rendering visible pages…`、`Bounded read-only Office preview`、`Open in system`、`Loading Office document…` 在三语下都保持英文。另外运行时枚举（定时任务类型、运行/审计状态、能力探测状态）大量用 Rust `{:?}` 直接渲染，任何语言下都显示英文标识符。
7. 设置页的分组标题 `WORKSPACE` 在简体中文下被译成「**工作流**」（应为「工作区」），这是产品侧的翻译错误，文档描述分区时不要跟着用。

---

## 7. 建议的目标信息架构

现有 IA 的问题是：**按"功能模块"切分，而不是按"用户要完成什么"切分**，而且缺整整一层"参照"。

```
📘 开始使用
   ├─ Vibex 是什么（含与 CLI Agent / 其他工作台的区别）
   ├─ 下载与安装（按平台 + 首次启动放行）
   ├─ 5 分钟快速开始（含截图）
   └─ 核心概念与术语表          ← 新增

🛠 日常使用
   ├─ 项目与工作区
   ├─ 会话与时间线
   ├─ Composer 输入（/ @ $ 附件）
   ├─ 代码工作台（文件树 / 编辑器 / 预览）
   ├─ Git 与 Worktree
   ├─ 终端
   ├─ 跨会话搜索               ← 新增
   ├─ 用量统计
   └─ 通知与注意力管理          ← 新增

🔌 扩展与自动化
   ├─ Agent：安装、登录、切换    ← 重写（含托管安装）
   ├─ 模型供应商配置            ← 重写（字段级 + 各供应商示例）
   ├─ MCP / Skills / Prompts / Hooks
   ├─ 定时任务
   └─ 自动化图

🌐 远程与自托管                 ← 整个 Tab 重写
   ├─ 远程总览：三种拓扑（本地 / 自托管服务器 / Relay）
   ├─ 自建无头运行时 vibex-server  ← 新增（含完整环境变量参考）
   ├─ 桌面端连接服务器            ← 新增
   ├─ 移动端                     ← 新增（含 iOS 现状说明）
   ├─ Relay 自托管
   └─ 设备与权限管理              ← 新增

⚙️ 设置与恢复
   ├─ Settings 全分区（含 Remote Runtime）
   ├─ 数据、备份与恢复
   ├─ 更新与发布通道              ← 新增
   └─ 隐私与安全

📋 参考                        ← 整个新章节
   ├─ 快捷键完整表
   ├─ 命令行与环境变量
   ├─ 配置文件字段
   ├─ 错误码表
   └─ 支持的 Agent 矩阵

❓ 帮助
   ├─ 常见问题 FAQ
   ├─ 故障排查（按错误码）
   └─ 更新日志 / Release Notes   ← 已存在于线上，需入库

👩‍💻 开发者指南（现有 7 页 + 扩充）
   ├─ 架构与 crate 地图（补全 23 crates）
   ├─ 本地开发环境
   ├─ 构建、测试与质量门禁（命令表由 package.json 生成）
   ├─ 发布流程与打包矩阵          ← 新增
   ├─ 平台支持与证据要求
   ├─ Remote v2 协议参考（补齐帧格式）
   └─ 贡献指南
```

---

## 8. 逐页修改清单

| 页面 | 问题 | 优先级 |
| --- | --- | --- |
| `index.mdx` / `en` / `zh-TW` | "唯一权威运行时"错误；"以源码仓库为中心维护"已过时；无截图 | **P0** |
| `quickstart.mdx` | 全程 clone+编译；"Model Providers" 入口不存在；应改为下载安装 + 首次会话；无截图 | **P0** |
| `install.mdx` | 应重写为面向用户的安装页；拆分 from-source；补签名/Gatekeeper/SmartScreen/通道说明 | **P0** |
| `settings.mdx` | 缺 Remote Runtime（rc.3）/ Runtime manager（后续版本）；快捷键表 9/14 且漏分组与可重绑定说明；缺 General 的启动与通知项 | **P0** |
| `agents-and-providers.mdx` | "Model Providers" 不是标签页（在 Agent 详情内）；缺托管 Agent 安装/更新/卸载；缺字段级说明；Health/Capability check 在不可达页面 | **P0** |
| `remote-mobile.mdx` | 缺移动端获取方式（含 iOS 仅模拟器）；缺配对码与 `vibex://pair#/code/…`；缺第 4 种局域网配对；提到不存在的 "Remote gateway"；低估移动端能力；缺"移动端无 Config Center" | **P0** |
| `config-center.mdx` | **分区表 10 行里 7 行不可达**；需改为 Agent / MCP / Skills 三个标签页并说明供应商配置位置 | **P0** |
| `scheduled-automation.mdx` | **整页在桌面端不可达**；节点类型 6→5；触发器不是节点；运行状态枚举错误；应下线或标注未开放 | **P0** |
| `troubleshooting.mdx` | 必须加错误码表；"Remote gateway" 不存在；"Recovery" 不可达；全站最需要重写的一页 | **P0** |
| `developer/build-and-test.mdx` | **4 个不存在的命令**；缺大量真实门禁 | **P0** |
| `developer/contributing.mdx` | 同样引用不存在的命令 | **P0** |
| `developer/index.mdx` | "桌面端是唯一权威"错误 | **P0** |
| `sessions.mdx` | 缺跨会话搜索；delegation 只有一行；导入章节准确但缺截图 | P1 |
| `code-workbench.mdx` | 无截图；快捷键表与 settings 重复且不全；图片编辑器作用域写错；Office 格式需具体化；外部更改检测被夸大 | P1 |
| `git.mdx` | "按块 Stage" 不存在（只有整文件）；缺 worktree 重命名与冲突处理步骤；无截图 | P1 |
| `terminal-and-previews.mdx` | 无截图；Office 格式需具体化；滚动缓冲区不可配置；PDF 能力未说明 | P1 |
| `mcp-skills-prompts-hooks.mdx` | Prompts/Hooks 两节不可达；MCP 作用域只有 User/Workspace；Skill 编辑器无作用域选择；无 JSON 示例；无 SKILL.md 编写指南 | P1 |
| `usage.mdx` | 指标与覆盖度**准确**；缺截图；"Project" 与"工作区"的关系需说明 | P2 |
| `privacy-and-recovery.mdx` | "Recovery" 不可达；应补 `docs/operations/recovery-matrix.md` 的持久化矩阵 | P1 |
| `developer/architecture.mdx` | crate 只列 8/23；缺 `apps/server`；架构图缺无头运行时；缺 Project→Workspace 层级 | P1 |
| `developer/platforms.mdx` | 缺真实产物格式（deb/dmg/exe/apk/aab）；缺通道矩阵与签名现状 | P1 |
| `developer/remote-protocol.mdx` | 缺 `vibex://pair#/code/…`、配对码、自动更新机制、帧格式；比仓库 `docs/remote/protocol-v2.md` 薄一半 | P1 |
| `developer/setup.mdx` | 基本正确 | P2 |
| 全站 | 0 截图、0 keywords、0 TODO、无版本声明、组件单调、内链稀疏 | P1 |

---

## 9. 文档站做对的地方（不要改坏）

客观地说，现有文档在几个方面质量是好的，重构时应保留：

1. **术语克制、不吹牛。** 全站没有营销话术，明确写出"Usage Statistics 不是账单页面"、"停止 Agent 不会撤销文件改动"、"Relay 不保存业务数据"这类边界，这比大多数产品文档诚实。
2. **隐私与脱敏要求贯穿全站。** 每页都提醒不要外泄 token、Prompt、文件内容、终端字节，且给出了可操作的脱敏清单。
3. **`usage.mdx` 的指标与维度描述准确。** Coverage 的 `Complete / Derived / Partial / Unknown`、5 个维度、3 种视图都与 rc.3 代码一致。
4. **`sessions.mdx` 的本地导入章节准确。** 14 个来源、扫描只读元数据、`Imported / Already imported / Not found / Failed` 四态、不导入凭据——都能与代码对上。
5. **`mcp-skills-prompts-hooks.mdx` 的 Hook 事件名正确**（`terminal_activity` / `session_start` / `session_stop` / `permission_request`），MCP 传输类型也正确（`stdio` / `http` / `sse`）。
6. **三语的标题层级严格对齐**（每页 heading 数量一致），说明有翻译流程约束。
7. **`developer/architecture.mdx` 的"变更评审清单"** 和 `contributing.mdx` 的工程原则，对贡献者是有实际约束力的。

问题不在"写得好不好"，而在**校准（calibration）**：把代码里存在的东西当成了用户能用的东西，并且没有跟着版本更新。

---

## 10. 建议的实施顺序

**第 1 步：修事实错误（1–2 天，不改结构）**
先解决 2.1–2.7。这七条是"文档在骗人"级别的，改动量小但收益最大。优先级依次是：

1. 把"唯一权威运行时"这个地基改掉（2.1）
2. 把 Config Center / Scheduled / Automation / Prompts / Hooks / Recovery 的"不可达"问题处理掉（2.4）——**建议先内部确认这些功能是否会在下个版本开放，再决定是"下线文档"还是"标注未开放"**
3. 修 4 个不存在的命令（2.3）
4. 安装页改为面向用户（2.2）
5. 远程章节补配对码与连接串（2.5）
6. Settings 补远程分区/运行时管理器（2.6）

**第 2 步：把线上 changelog 回灌仓库，并定一条发布流程**
避免下一次部署把线上内容删掉。

**第 3 步：重写 install / quickstart / settings，补齐远程章节**
这四块覆盖了"新用户第一步"和"最大的功能空白（无头服务端）"。同时在站点上**声明文档对应的版本号**。

**第 4 步：加截图 + 建概念页 + 建参考章节**
这一步是"从能看变成好用"的分水岭。

**第 5 步：加自动化校验**
- 从 `package.json` 生成命令表，脚本名对不上就 CI 失败
- 从 `locale.rs` 抽取 UI 字符串做术语一致性检查（防止"工作区/项目"再次漂移）
- 线上 sitemap vs 仓库页面集合比对告警
- 枚举值（节点类型、权限级别、通道）从代码生成
- "文档声明的 UI 入口是否真实可达"检查（防止再次出现 2.4 这类问题）

**第 6 步：内容补齐**
按第 8 节清单逐页推进。

---

## 附录：本次核查用到的命令

```bash
# 以 rc.3 为基准核对
git show v0.1.0-rc.3:package.json
git show v0.1.0-rc.3:apps/desktop/src/locale.rs
git show v0.1.0-rc.3:apps/desktop/src/app.rs | grep -n 'FOUNDATION_SHORTCUTS' -A 20
git grep -rn 'enum ManagementSection' -A 25 v0.1.0-rc.3
# 核对 Config Center 实际可达的标签页（3 个）
git show v0.1.0-rc.3:apps/desktop/src/management.rs | grep -n 'fn render_nav' -A 12
git show v0.1.0-rc.3:apps/desktop/src/management.rs | grep -n 'fn management_primary_section' -A 12
# 核对 Settings 分区
git show v0.1.0-rc.3:apps/desktop/src/app.rs | grep -n 'enum SettingsSection' -A 12

# 核对发布产物
gh release view v0.1.0-rc.3 -R vibex-ai/vibex
gh release list -R vibex-ai/vibex

# 核对线上站点
curl -sS -L https://vibex.peatboy.com/docs/sitemap.xml
```

---

*本报告基于 `vibex-ai/vibex` 的 `v0.1.0-rc.3` 标签与 2026-09-15 的线上站点状态。*
