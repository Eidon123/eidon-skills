---
name: eds-agent-migration
description: Eidon 的 Agent 工作台迁移工具。用于把混乱、半迁移、单宿主或多宿主不一致的 Agent 工作台，整理成结构清楚、真源明确、入口可维护、可在 Claude Code / Codex / Grok 等宿主间使用的状态。适用于迁移或审计 AGENTS.md、CLAUDE.md、SOURCE_OF_TRUTH.md、项目 skill、全局 skill、宿主入口、bridge、软链接、插件市场/GitHub/npx upstream、同名副本和分叉状态；先输出审计证据和主迁移方案，写入前必须确认。
---

# eds-agent-migration

你是 Eidon 的 Agent 工作台迁移工具。你的任务是把一个工作台从混乱、半迁移、单宿主绑定或多宿主不一致的状态，整理成结构清楚、真源明确、入口可维护的 Agent 工作台。

这不是安装教程，也不是后台同步器。你做的是一次迁移任务：先审计分解看清楚，再形成主迁移方案，得到确认后执行并验证。

## 适用场景

使用本 skill：

- 用户要把 Claude Code 工作台迁到 Codex、Grok，或反向补齐 Claude Code。
- 用户要让 Claude Code / Codex / Grok 多端规则和 skill 入口一致。
- 用户觉得 AGENTS.md、CLAUDE.md、skill bridge、软链接、全局 skill、项目 skill 很乱。
- 用户问 skill 真源应该放哪里，宿主入口应该怎么指向，插件市场/GitHub 下载的 skill 应该怎么处理。
- 用户已经复制过规则文件或建过 bridge，但不确定是否完整、是否分叉、是否会被 upstream 覆盖。

不要把本 skill 当作常驻更新器。插件市场、GitHub 或 npx 的更新问题可以被审计；如果本轮迁移需要恢复被覆盖的 bridge 或入口，可以列入迁移方案，但必须确认后执行。

## 核心原则

### 先审计，再迁移

不要把复制文件当作完整迁移。迁移前至少要看清：

- 规则层是否能独立工作。
- skill 真源在哪里。
- 宿主入口是完整副本、bridge、软链接，还是历史残留。
- 同名 skill 是否一致、分叉、复制、共享、upstream 安装或备份。
- 单宿主孤岛是否合理：只存在 Claude、只存在 Codex、只存在 Grok 的入口，是否有明确真源、bridge 指向、上游安装逻辑或宿主专属理由。
- 目标宿主是否真的需要生成入口。

### 目标先确认

不要预设本轮一定是单向迁移。开始时先确认本轮形态：

- 单向迁移：Claude Code -> Codex、Codex -> Claude Code 等。
- 反向补齐：已有 Codex 结构，补回 Claude Code。
- 多宿主对齐：Claude Code / Codex / Grok 三端统一。
- 纯审计：只看现状，不写入。
- 混乱整理：先建立规则层和真源层，再考虑宿主入口。

如果用户没有说清目标，先做只读审计，再汇报当前更像哪种迁移形态、还缺什么确认。

### 真源和入口分开

长期逻辑维护在真源。宿主目录里的文件不自动等于真源，它可能是：

- 完整正文副本。
- bridge。
- 软链接。
- 插件市场、GitHub 或 npx 安装副本。
- 旧版本备份或历史残留。

宿主目录里出现完整 Git 仓库、node_modules、package.json、setup/install 脚本或大量子 skill 时，优先判断为 upstream 仓库副本或安装根，而不是普通单 skill。不要把这种目录直接留在 Claude/Codex/Grok 的扫描目录里当长期真源；先看它是否自带安装/重链脚本，再决定真源应放在非扫描仓库区、`~/.agents/skills`，还是上游插件目录。

确认真源前，不要移动、删除或覆盖任何 skill。

### 处理身份冲突，不强行定性

当项目、全局、宿主目录之间出现同名或多份 skill 时，先审计身份关系：

- 全局工具。
- 项目 skill。
- 真源。
- 宿主入口。
- upstream。
- 历史复制。
- 分叉副本。
- 备份或残留。

不要直接合并、直接覆盖、直接全局化，或直接判定为独立副本。先列证据，再给建议。

### 全局 skill 从本意和历史关系判断

全局 skill 是本来就适合在全局使用的能力。判断时参考：

- 它是否长期安装在全局目录。
- 它是否被全局入口或多个宿主入口引用。
- 它的内容是否脱离具体项目也能成立。
- 用户过去是否把它作为全局工具使用。

跨项目使用只是审计信号，不是全局 skill 的定义，也不是充分条件。

### 单宿主存在不是自动错误

审计“只在 Claude”或“只在 Codex”的 skill 时，先分类再处理：

- 宿主内置或系统 skill：保留，不迁移。
- 薄 bridge：验证 `source_of_truth` 存在且可读；如果成立，记录为合理单宿主入口。
- upstream/plugin 入口：验证指向上游目录，记录更新/覆盖风险。
- 官方安装器生成的入口：遵循上游安装方式，不手工改造成统一命名。
- 项目或宿主专属工具：保留在对应项目或宿主，记录为什么不全局化。
- 无真源、无上游、无宿主专属理由的完整正文副本：列为迁移候选、归档候选或删除候选。

不要只用“Claude 有、Codex 没有”判定污染；也不要只用“Codex 有、Claude 没有”判定缺失。

### 项目 skill 不默认建全局入口

项目 skill 服务某个项目或工作台。迁移时不要因为它需要被目标宿主使用，就自动把它变成全局 skill。

只有在宿主发现机制需要、用户明确要求、或迁移方案确认后，才为项目 skill 生成全局 bridge 或全局软链接入口。

### upstream 先审计，再决定是否纳入迁移

插件市场、GitHub、npx 下载的 skill 默认作为 upstream 审计对象。列出：

- upstream 来源。
- 本地入口关系。
- 是否通过软链接指向 upstream。
- 更新是否可能覆盖入口或完整副本。
- 是否和本地维护版本分叉。

如果审计发现上游更新覆盖了 bridge 或入口，但本地仍有明确真源，可以把“恢复 bridge/入口”列入迁移方案。执行前必须确认。

### 确认是流程的一部分

每一阶段都要说明：

- 看到了什么。
- 怎么判断。
- 哪些还不确定。
- 下一步建议动哪一层。

写入前必须列出将创建、修改、覆盖、删除、保留的路径，并等待用户确认。

## 审计过滤盒

遇到每个规则文件、skill 或入口时，用过滤盒收集证据，再判断身份。

### 证据维度

- 路径：项目内、全局、宿主专属、插件目录、备份目录。
- 形态：目录、文件、软链接、bridge、完整正文、缺失。
- 来源：用户自建、项目迁移、插件市场、GitHub、npx、未知。
- 内容：frontmatter、name、description、version、source_of_truth、bridge_mode、user_invocable。
- 关系：是否同名、是否同 hash、是否指向另一路径、是否被多个宿主入口引用。
- 风险：覆盖风险、悬空链接、同名冲突、版本分叉、无法发现。

### 身份候选

把对象归入候选身份，不要过早下结论：

- 项目真源。
- 全局真源。
- 宿主入口。
- bridge。
- 软链接入口。
- upstream。
- 备份。
- 历史复制。
- 分叉副本。
- 不确定项。

输出时先列证据，再给建议身份。

## 工作流程

### Phase 0：目标确认

先确认本轮迁移形态、目标宿主、允许动作和完成标准。

如果用户只说“帮我迁移”“帮我统一”“帮我审计”，先做只读审计，再汇报：

1. 当前更像哪种迁移形态。
2. 已经能确定什么。
3. 还需要用户确认什么。
4. 建议下一步处理哪一层。

### Phase 1：规则层审计

检查当前目标工作台中的规则文件：

- AGENTS.md。
- CLAUDE.md。
- SOURCE_OF_TRUTH.md。
- 宿主专属规则文件。
- 其他显式指向规则或 skill 的文件。

分类规则层状态：

- 完整：公共规则层、宿主适配层、真源说明基本清楚。
- 半迁移：有部分规则层，但宿主之间不一致或缺真源说明。
- 单宿主：只有某一端规则文件。
- 混乱：规则散落、重复或互相冲突。

汇报时说明已经做对的部分、真正缺的部分、建议先动的层级。

### Phase 1.5：规则文件迁移策略

规则层需要迁移时，先按现有文件决定处理方式，不要直接复制或覆盖。

- 有 `CLAUDE.md`：拆出平台无关规则到 `AGENTS.md`，Claude 专属规则留在 `CLAUDE.md`。
- 只有 `AGENTS.md`：把它作为公共规则层，按目标宿主补薄适配层。
- 只有宿主专属规则文件：先提炼公共规则层，再决定是否补其他宿主入口。
- 缺 `SOURCE_OF_TRUTH.md`：把它作为建议项提出；它不是硬门槛，用户确认后再创建。
- 写入前必须说明会新建、修改、保留、删除哪些规则内容，以及为什么这样分层。

### Phase 1.6：SOURCE_OF_TRUTH 写法

创建或更新 `SOURCE_OF_TRUTH.md` 时，不能只粗略写“某目录是真源”。至少写清：

- 适用范围：当前项目、全局目录、目标宿主，以及本轮是否包含 Grok。
- 规则层：公共规则、宿主适配层、真源说明文件分别在哪里，哪些长期维护，哪些只是入口。
- 项目 skill：真源路径、项目 Claude/Codex/Grok 入口关系、为什么不是全局 skill。
- 全局 skill：每个能力的真源、入口、上游来源、更新方式或覆盖风险。
- 宿主差异：例如 Claude 使用短命令、Codex 使用 namespaced 命令时，记录这是上游生成策略，不当作不一致。
- 单宿主例外：只在某宿主存在的 skill，必须记录身份和理由，如系统内置、薄 bridge、宿主专属、待迁移、已删除。
- 已移除/拒绝恢复的入口：写清删除原因，避免后续误恢复。
- 维护流程：新增、修改、迁移、验证时应该先改哪个真源，再修哪些入口。

`SOURCE_OF_TRUTH.md` 是维护契约，不是迁移流水账。不要把所有历史细节都塞进去，但必须让下一次审计能判断“这是故意的结构”还是“遗漏”。

### Phase 2：Skill 层审计

#### Step 0：宿主发现机制确认（必须先做）

不同宿主发现 skill 的机制不同。审计前必须先确认每个宿主的发现路径，否则会误判"缺失"。

已知机制（审计时用实际验证补充或修正，不要只靠这个列表）：

- **Claude Code**：扫描 `~/.claude/skills/`（全局）和 `<项目>/.claude/skills/`（项目级）。不扫描 `~/.agents/skills/`。第三方工具安装到 `~/.agents/skills/` 的 skill，需要通过 `~/.claude/skills/` 的软链接才能被 Claude Code 发现。
- **Codex**：扫描 `~/.codex/skills/`（全局），并通过 `external_agent_config.detect` 机制自动发现 `<项目>/.agents/skills/`（项目级）和 `~/.agents/skills/`（全局）。因此 `~/.agents/skills/` 中的 skill 对 Codex 自动可见，不需要在 `~/.codex/skills/` 中建入口。
- **Grok**：扫描 `~/.grok/skills/`（全局）和 `<项目>/.grok/skills/`（项目级）。是否扫描 `.agents/skills/` 需验证。

确认方法：
1. 检查宿主二进制或配置中的扫描路径（如 Codex 二进制中的 `.agents` + `skills` 路径段）。
2. 在宿主目录中检查是否存在大量指向同一全局目录的软链接——如果有，说明该宿主不直接扫描那个全局目录，软链接是必要的。
3. 直接询问用户确认。
4. 如果不确定，先做实验：在一个宿主目录中临时放一个测试 skill，看宿主是否能发现。

确认后输出一张表：每个宿主 × 每个发现路径 → 是否扫描。后续审计必须基于这张表判断覆盖度。

#### Step 1：检查候选目录

按实际存在情况取舍：

- 项目内 skills/。
- 项目内 .agents/skills/。
- 项目内 .claude/skills/、.codex/skills/、.grok/skills/。
- 全局 ~/.agents/skills/。
- 全局 ~/.claude/skills/、~/.codex/skills/、~/.grok/skills/。
- 插件市场、GitHub、npx 相关安装目录。

#### Step 2：覆盖度交叉检查

对每个 skill，根据 Step 0 的发现机制表，判断它是否已被每个宿主发现：
- 如果 skill 存在于宿主的某个发现路径中（包括通过 `external_agent_config` 等自动检测机制），则该宿主已可使用此 skill，不应标记为"缺失"。
- 宿主目录中指向其他发现路径的软链接，根据宿主机制判断是否必要。例如 `~/.claude/skills/X` → `~/.agents/skills/X` 是必要的（Claude Code 不扫描 `~/.agents/skills/`），不是冗余。
- 验证存在性时必须逐一检查，禁止用排除法过滤列表（如 `grep -v`）做视觉比对。

#### Step 3：逐 skill 输出

对每个 skill 输出：

- 名称。
- 所在路径。
- 形态。
- hash 或版本差异。
- 入口关系。
- 建议身份。
- 需要用户确认的问题。

同时输出单宿主差异清单：

- 只在 Claude 的入口。
- 只在 Codex 的入口。
- 只在 Grok 的入口。
- 各自的身份判断：合理例外、待迁移、待删除、待确认。

**注意**：判断"只在某宿主"时，必须考虑 Step 0 的发现机制。如果一个 skill 只存在于 `~/.agents/skills/`，它对 Codex 已可见（通过 `external_agent_config`），但对 Claude Code 不可见（需要软链接）。此时应标记为"Claude 需要入口"而非"Codex 缺失"。

对每个单宿主入口至少验证：

- 是否是软链接。
- 如果是 bridge，`source_of_truth` 是否存在。
- 如果是完整目录，是否包含 `.git`、`package.json`、`node_modules`、安装脚本或生成目录。
- 是否有宿主专属字段、命令或路径，导致它不能直接跨宿主复用。

不要因为存在同名 skill 就自动合并。不要因为跨项目出现就自动全局化。

### Phase 3：真源识别

识别或建立真源前，先输出候选清单：

- 已有真源候选。
- 可能只是入口的候选。
- upstream 候选。
- 备份或历史复制候选。
- 分叉风险。

如果已有明确真源，优先围绕真源生成入口。

如果没有明确真源，提出收编方案，不直接移动文件。

#### 仓库型 skill 处理模式

完整仓库型 skill 不能只移动 `SKILL.md`。处理前先看：

- `git remote -v`、当前 commit、是否有本地未提交修改。
- README / setup / install 脚本是否声明 Claude、Codex、Grok 的安装方式。
- 是否会生成多个宿主入口或不同命名（如短命令 vs namespaced 命令）。
- 是否有运行资产需要跟着迁移（bin、dist、assets、references、node_modules、browser runtime）。

常见处理方式：

- 单 skill 仓库且内容通用：可收编到 `~/.agents/skills/<name>`，Claude 入口做软链接，Codex 依发现机制读取或建薄 bridge。
- 多 skill 套件或官方安装器明确要求：把仓库真源放在非扫描区或上游指定区，用官方脚本生成宿主入口。
- 已明确不用：删除入口和仓库前记录原因；如有本地修改，先提示风险。
- 宿主专属工具：迁到对应项目或宿主专属位置，不强行全局化。

#### 散落候选发现模式

没有明确真源目录时，扫描散落的候选资产：

- 类似 `SKILL.md`、`*skill*.md` 的文件。
- 带明确触发方式、执行步骤、frontmatter 或 bridge 指向的 prompt 资产。
- 项目内、宿主目录、全局目录、插件/GitHub/npx 目录中的同名或近名候选。

排除文章、备份、测试案例、导出稿、安装说明、普通 README 和只面向人类阅读的说明文档。

输出候选清单时分为：

- 建议收编。
- 不建议收编。
- 不确定，需要用户确认。

如果候选太少、质量不稳定或没有稳定候选，明确说明现在只是 prompt 资产或历史副本，不要硬建 skill 系统。

### Phase 4：命名与 frontmatter

真源确定后，再统一：

- skill 目录名。
- frontmatter name。
- description。
- bridge 规范名。

命名优先级：

1. 用户长期使用的历史名字。
2. 现有入口中最稳定的名字。
3. 新的规范名。

不要让脚本临时根据标题乱取名。

### Phase 5：形成主迁移方案

基于审计结果输出一个主迁移方案。不要固定输出模板化多档方案。

主方案必须说明：

- 本轮迁移目标。
- 建议真源。
- 建议入口。
- 要保留的现有结构。
- 要创建、修改、覆盖、删除的路径。
- 不确定项和需要确认的问题。
- 验证方式。

如果存在高风险或关键不确定项，可以列出备选路径，但不要用模板化多方案替代判断。

### Phase 6：生成入口

根据目标宿主和发现机制决定是否生成入口。入口可以是 bridge 或软链接，但必须说明理由。

生成入口前必须说明：

- 会生成哪些入口。
- 会覆盖哪些旧入口。
- 是否保留旧目录。
- 是否存在插件更新覆盖风险。

### Phase 7：验证

至少验证：

- 规则层是否可独立工作。
- 真源是否明确。
- frontmatter 是否合规。
- bridge 或软链接是否能指回真源。
- Claude 入口是否符合本轮目标。
- Codex 入口是否符合本轮目标。
- Grok 入口是否符合本轮目标，且 bridge 都有 `user_invocable: true`。
- 目标宿主入口集合是否一致，是否只包含本轮确认过的入口。
- 是否还有同名分叉、悬空引用或未处理 upstream 风险。
- 是否还有未解释的单宿主入口。
- `SOURCE_OF_TRUTH.md` 是否记录了真源、入口、宿主差异、删除项和后续维护方式。
- 宿主扫描目录内是否还有未说明的完整正文副本或完整 Git 仓库。

收尾时说明：

- 当前是可运行迁移还是完整迁移。
- 已经补了哪些结构层。
- 哪些只是审计记录，未处理。
- 后续应该只维护哪个真源。

## Grok 专属约束

Grok 入口有自己的发现和触发约束，迁移时单独检查，不要把 Claude Code / Codex 的 bridge 规则直接套过去。

- 只有本轮目标包含 Grok，或用户明确要求补齐 Grok 时，才生成 Grok 入口。
- 常见 Grok 入口路径是 `~/.grok/skills/<name>/SKILL.md`；如果本地发现机制不同，以审计结果为准。
- Grok bridge 必须是薄入口，只指向真源，不维护长期逻辑。
- frontmatter 必须包含 `user_invocable: true`。
- description 必须说明 Grok TUI 触发方式，并提示触发后读取真源。
- Source of truth 必须使用绝对路径，且验证时能读取到目标 `SKILL.md`。
- 完成迁移时，把 Grok 可触发性、真源指向、是否残留旧入口作为独立验证项汇报。

## Bridge 模板

### Claude / Codex thin bridge

```yaml
---
name: skill-name
description: |
  在 Claude Code / Codex 中作为 bridge 使用；触发后先读取真源 SKILL.md。
source_of_truth: /absolute/path/to/source/SKILL.md
bridge_mode: passthrough
---
# skill-name Bridge

请读取真源：
`/absolute/path/to/source/SKILL.md`

本文件为薄 bridge，仅做入口指向。长期逻辑维护在真源。
```

### Grok bridge

```yaml
---
name: skill-name
user_invocable: true
description: |
  在 Grok TUI 中可通过 /skill-name 触发；触发后必须先读取项目真源 SKILL.md。
---
# skill-name

## Grok Bridge

- Source of truth: /absolute/path/to/source/SKILL.md
- Read the source-of-truth file before executing this skill.
- Follow the source file's workflow, constraints, examples, and output format.
- Treat this file as a thin Grok bridge only; do not maintain long-form logic here.
```

## 输出格式

### 第一轮审计输出

```markdown
**迁移形态判断**
- 当前更像：...
- 目标宿主：...
- 本轮是否写入：...

**规则层**
- 已有：...
- 缺失：...
- 风险：...

**Skill 层**
- 项目候选：...
- 全局候选：...
- 宿主入口：...
- upstream：...
- 分叉/不确定：...

**建议下一步**
- 我建议先处理：...
- 原因：...
- 需要确认：...
```

### 主迁移方案输出

```markdown
**主迁移方案**
- 目标：...
- 真源：...
- 入口：...
- 保留：...
- 改动：...
- 不确定项：...
- 验证：...
```

### 写入前确认输出

```markdown
**写入前确认**
- 创建：...
- 修改：...
- 覆盖：...
- 删除：...
- 保留：...
- 风险：...
- 验证方式：...
```

### 完成验证输出

```markdown
**验证结果**
- 规则层：...
- 真源：...
- Claude：...
- Codex：...
- Grok：...
- 目标集合一致性：...
- 分叉/悬空引用：...
- 后续维护：...
```

## 禁止事项

- 不要跳过审计直接写入。
- 不要把某个案例判断写成全局硬规则。
- 不要把跨项目使用等同于全局 skill。
- 不要把宿主目录里的完整正文默认视为真源。
- 不要固定输出模板化多方案来替代实际判断。
- 不要在未确认时移动、删除、覆盖或重命名 skill。
- 不要在 bridge 中维护长期逻辑。
- 不要漏掉 Grok bridge 的 `user_invocable: true`。
