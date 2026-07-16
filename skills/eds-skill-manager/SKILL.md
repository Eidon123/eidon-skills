---
name: eds-skill-manager
description: |-
  Eidon Skills (EDS) 的结构管理器。用于盘点全局与当前项目的 skill，规范化目录和 skill 发现入口，创建或维护唯一的全局 SOURCE_OF_TRUTH.md，以及安装、更新、移除、修复 skill 或多 skill 套件；也可在完整 skill 审计后按用户选择维护可选的 workspace 路径注册。用户提到 skill 清单、skill 放在哪里、全局/项目 skill、重复副本、跨宿主 skill 入口统一、软链接、bridge、lock、upstream、套件更新、入口损坏、SOT、workspace 路径注册或 EDS skill 管理时使用。无明确动作时默认只读盘点全局和当前项目；不评价 skill 的内容质量、安全性、使用频率或业务价值。
---

# EDS Skill Manager

管理 Agent skill 的结构关系。自己完成盘点、normalize、SOT、install、update、remove 和 repair；skill 生命周期始终是核心，workspace registry 只是显式选择后的可选结构能力。

## 边界

只处理结构事实：

- skill 的名称、路径、作用域和文件形态。
- 开发源、发布上游、安装副本、宿主入口之间的关系。
- suite 成员、安装管理器、lock、软链接、bridge 和发现路径。
- 全局目录、项目目录、外部托管目录、宿主专属目录和历史残留。
- 用户明确选择登记的 workspace 根路径、规则入口和生命周期状态。

不要处理：

- skill 是否有用、是否低频、是否值得保留。
- skill 正文的业务质量、安全审计、广告或恶意行为检测。
- 未经用户要求的 prompt 改写或功能重构。
- workspace 的业务内容、模块索引、跨系统同步逻辑或读写授权。

遇到这些需求时，说明本 skill 只能提供结构证据，把内容判断留给相应能力或用户。

## 核心不变量

### 唯一 SOT

唯一权威文件固定为：

`<用户确认的全局-skill-目录>/SOURCE_OF_TRUTH.md`

- 不猜测全局 skill 目录。
- 创建、迁移、修复 SOT 或处理 workspace registry 前，完整读取 `references/source-of-truth-contract.md`。
- 已有一个有效 SOT 时，先读取并验证其物理身份、schema、authority domain、所在目录与自述路径一致。
- 先用 `realpath`/inode 合并同一物理文件的逻辑别名。只有不同物理文件的作用域重叠且 `authority-domain` 相同时才构成冲突。
- 项目业务使用的同名 `SOURCE_OF_TRUTH.md` 属于其他 authority domain，不能当作全局 EDS SOT 冲突。
- 没有 SOT 时，先完成全局和当前项目的全量只读盘点，再请用户确认哪个目录作为全局 skill 目录。
- `SOURCE_OF_TRUTH.md` 由本 skill 独占写入。Migration 和其他 Agent 只能提交证据或已验证映射。

SOT 使用 `eds-sot/v1`，至少记录：

1. `Contract`：schema、scope、authority domain、canonical path、维护者和读取条件。
2. `Intended Structure`：用户批准的根目录、管理策略、身份关系、例外、tombstone 和可选 registry 政策。
3. `Verified State`：带 evidence/status/verified-at 的观察、未管理对象和漂移。

不要把现场观察静默写成意图。预期对象缺失时保留 Intended 并记录 drift；发现额外对象时记录 unmanaged，获得用户批准后才能进入 Intended。证据、状态与验收等级不变时不要刷新 `verified-at`。

完整成员清单、成员数量、全量 hash 和其他可从目录、lock 或 upstream 重建的 inventory 只在当前响应中生成，不持久化，也不创建 `CATALOG.md`。

### 默认项目目录

当前项目的默认真源是：

`<project>/skills/<skill-name>/SKILL.md`

- 从当前工作目录、项目规则或版本库边界确定 `<project>`。
- 已有明确且可用的其他项目真源时，先记录差异，不要静默搬迁。
- 无法确定当前项目时，只盘点全局，并明确说明项目范围未建立。

### 四层身份

不要用一个模糊的“真源”覆盖所有角色。分别记录：

1. `development-source`：可编辑、可测试、可提交的开发仓库或项目真源。
2. `publication-upstream`：远端仓库、发布源、版本或 commit。
3. `installed-copy`：供 Agent 使用的标准安装副本；可能受 lock 或安装器管理。
4. `host-entry`：宿主扫描到的软链接、bridge 或官方生成入口。

同一 skill 可以同时具有四层。不要直接修改 lock 管理的安装副本；不要让宿主默认直连未发布的开发工作树。

### 外部托管

官方仓库和官方 setup/upgrader 管理的套件标记为 `external-managed`：

- 保留在官方指定目录。
- 使用官方流程更新和生成入口。
- 不复制进标准全局 skill 目录。
- 不强求不同宿主使用相同入口名称。
- 单独统计，不混入标准安装成员数量。

## 默认行为

用户未指定动作，或只说“看看 skill”“管理 skill”时：

1. 只读盘点全局和当前项目。
2. 输出结构分类、异常和证据。
3. 不创建 SOT，不移动目录，不修链接，不更新任何包。
4. 给出下一步建议，但等待用户选择动作。

只有顶层完整审计闭环且结果不是 partial/failed 时，才根据 SOT 中的 registry 状态提供一次可选提示。普通 inventory、嵌套执行和失败结果不提示，也不扫描其他 workspace。

只读操作不需要确认。任何创建、修改、移动、覆盖、安装、更新、删除或修复都必须先预检并获得确认。

## 全阶段流式进度

所有阶段都向用户持续报告进度：

- 阶段开始时说明正在检查什么、是否只读、预计下一步是什么。
- 阶段结束时报告已检查数量、关键发现和下一阶段。
- 长命令或大目录扫描期间至少每 30 秒报告一次当前范围和累计数量。
- 执行批次时逐项报告 `开始 / 完成 / 失败 / 跳过`，不要等全部结束后一次性汇报。
- 不输出冗长命令回显；只输出用户可核对的结构事实。

## 统一工作流

### Phase 1：确定范围和动作

识别本轮是：

- `inventory`：只读盘点。
- `normalize`：统一结构、命名和入口关系。
- `sot`：建立、迁移或修订唯一 SOT。
- `workspace-registry`：在用户明确选择后发现、登记、验证或修复 workspace 路径。
- `install`：安装 skill 或 suite。
- `update`：更新已安装项或 suite。
- `remove`：移除指定结构层。
- `repair`：修复入口、lock、SOT 或安装关系。

同时确定当前项目、目标 skill/suite、涉及的宿主和允许的写入范围。用户未说明时使用 `inventory`。

### Phase 2：发现真实路径

从以下证据发现候选目录：

- 用户给出的路径和已有 SOT。
- 当前项目规则、环境变量、安装管理器元数据和 lock。
- 宿主配置、宿主官方说明、只读发现命令和现有入口的实际目标。
- 上游仓库、官方安装脚本和套件元数据。

不要硬编码未经验证的 Claude、Codex、Grok 或其他宿主路径。常见路径只能作为候选，必须用配置、文档、CLI 行为或已验证入口确认。路径存在本身不证明宿主会扫描它。

对目录和入口同时记录：

- 逻辑路径。
- `lstat` 形态。
- `readlink` 目标。
- 解析后的物理路径。
- 谁负责创建和更新它。

第一轮只读盘点不得通过临时创建测试 skill 验证发现机制。确有必要时，把实验路径、清理方式和风险放进写入预检。

### Phase 3：全量盘点

有 SOT 时，从 SOT 契约出发，反向检查实际文件系统和管理器状态。

无 SOT 时，完整盘点：

1. 所有候选全局 skill 目录及其物理目标。
2. 当前项目的 `skills/` 和已验证的项目宿主入口。
3. 已验证宿主的全局入口。
4. lock、安装器元数据、suite 上游和入口指向的外部仓库。
5. 同名目录、目录级软链接、逐项软链接、bridge、完整副本和悬空引用。

“全量”指沿已发现的目录、lock、入口和 upstream 关系把结构闭环，不指无边界扫描整个用户磁盘。

全量 skill 盘点不包含 workspace discovery。不得借 registry 功能扫描当前项目的同级目录、home、Obsidian、iCloud 或其他未经用户确认的根。

只读取完成结构判断所需的 frontmatter、指针和安装元数据；不要分析 skill 正文含义。

### Phase 3.5：可选 workspace registry

仅在用户选择“现在建立”或明确请求 registry 动作后执行：

1. 确认允许发现的有限候选根；没有确认根时不跨 workspace 扫描。
2. 只读检查候选根、规则入口、逻辑路径、realpath 和可复核用途证据。
3. 列出候选和不确定项，让用户逐项选择正式登记对象。
4. 把选中项写入精确预检；发现不等于登记，登记不等于读写授权。
5. 写入后反向验证根路径和规则入口；未回滚且仍保留的失败项标记为 degraded，全部回滚则恢复 not-configured，不伪造 active。

registry 的有效状态由两层推导，不在两处重复保存：Intended 的 `setup-decision` 只取 `undecided` 或 `declined-permanently`；若为永久拒绝，有效状态就是 `declined-permanently`，否则读取 Verified 的 operational status：

- `not-configured`：在下一次符合资格的完整审计后提示。
- `configuring`：精确 registry manifest 已批准并开始写入，但验证尚未完成。
- `active`：已登记项通过约定的结构验证。
- `degraded`：获批登记首次验证失败但仍保留，或已有 active 登记发生漂移；只提示修复，不重新询问是否建立。

选择“现在建立”只进入有限发现和精确预检；registry manifest 获批后的第一次 SOT 写入才在 Verified 持久化 `configuring`，全部选中项验证通过后转为 `active`，失败时按回滚后的事实记录 `not-configured` 或 `degraded`。选择“以后再说”保持 `setup-decision: undecided` 和 `not-configured`。永久拒绝只能在获批写入可用 SOT 后把 Intended 改为 `setup-decision: declined-permanently`，不依赖模型记忆或另建偏好文件。显式重置把该决策改回 `undecided`，有效状态恢复为 Verified 的 operational status。

SOT 不存在不是一种 registry 状态，也不能为了记住选择而跳过 SOT 创建流程。嵌套 Manager 不直接提示，只把 offer 状态返回顶层调用者；顶层调用者只有在自己的整体任务也满足顶层完整 skill 审计资格时才能展示三项选择，否则只报告状态和能力。普通 inventory 不后台巡检登记项；只有显式 registry 验证或相关 workspace 实际使用发现漂移时，才把 `active` 转为 `degraded`。

### Phase 4：结构分类

每个对象分别标记以下独立维度；一个对象可以在不同维度各有一个或多个标签。

**作用域**：

- `global-standard`
- `project-local`
- `external-managed`
- `host-system`
- `host-specific`
- `unknown`

**生命周期角色**：

- `development-source`
- `publication-upstream`
- `installed-copy`
- `host-entry`
- `backup-or-residue`

**入口形态**：

- `direct-directory`
- `directory-symlink`
- `item-symlink`
- `bridge`
- `official-generated`
- `no-entry`

**管理方式**：

- `lock-managed`
- `manual-managed`
- `official-managed`
- `unmanaged`

**健康状态**：

- `healthy`
- `identical-duplicate`
- `diverged-copy`
- `broken-link`
- `orphan-lock`
- `unlocked-install`
- `name-mismatch`
- `unverified`

分类必须附路径、链接目标、hash/version、lock/upstream 和判断证据。开发源领先已发布版本时标记为 `unpublished-change`，不要误判为应立即覆盖的分叉。

### Phase 5：形成预检

任何写入前输出一份可执行预检：

- 目标状态和选择它的理由。
- 创建、修改、移动、覆盖、删除、保留的精确路径。
- 将运行的安装器或官方命令及其作用域。
- suite 的当前成员、上游正式成员、新增项和退出项。
- lock、SOT 和每个宿主入口的预期变化。
- SOT 当前 hash、拟修改 section、保留的未知字段和写前冲突检查方式。
- 备份、回滚方式、验证方式和不可逆风险。
- 本批次中的普通项与拆出的复杂项。

对所有可能写入的 Git 仓库读取分支、HEAD、remote 和 `git status`，并检查拟修改文件是否已有未提交改动。对非 Git 路径记录类型、权限、软链接目标和可复核 hash。已有改动必须保留；与本轮重叠且无法安全合并时停止并询问。

预检期间如果证据不足，继续只读调查；无法闭环时明确询问，不猜测。

### Phase 6：确认与执行

对预检完成的普通更新项组成一个批次，只请求一次确认。确认后连续执行，不逐项重复询问。

被 `eds-agent-migration` 调用时使用嵌套模式：只向 Migration 返回精确预检 manifest，由 Migration 统一向用户确认；收到完全相同的已批准 manifest 后直接执行，不重复询问。范围、风险或外部副作用变化时停止，把新 manifest 交回 Migration 重新确认。

把以下复杂项拆成独立批次，分别预检和确认：

- 有未提交修改、内容分叉或来源冲突。
- 涉及删除、降级、重命名、suite 成员退出或开发仓库。
- 需要外部官方安装器、额外权限、认证或无法回滚的操作。
- 宿主发现路径未验证，或执行中出现预检外的新范围。
- 更新管理器自身，且可能影响当前执行入口。

其他动作也遵循“完整预检后一次确认”。只有范围变化、新的破坏性风险或授权要求出现时才暂停并再次确认。

执行前备份所有将修改、移动、覆盖或删除的路径，并保存恢复清单。软链接备份链接本身及其解析目标说明，不递归复制目标；备份放在宿主扫描目录之外并验证可读。备份失败立即停止。使用对应安装管理器维护其 lock；不要手工伪造或修改 lock。操作外部套件时只使用已验证的官方流程。

写入 SOT 前重新读取并比较预检 hash。内容发生变化时废弃原预检并重新审计，不覆盖其他 Agent 的并发修改。

### Phase 7：验证并更新 SOT

从最终文件系统反向验证：

- 每个目录含可读 `SKILL.md`，目录名与 frontmatter `name` 一致。
- 所有软链接和 bridge 都解析到预期物理目标。
- 每个目标宿主能通过已验证机制发现应暴露的 skill。
- lock、安装副本、upstream、suite 成员和管理器状态一致。
- 项目 skill 仍属于项目，外部托管项没有被收进标准目录。
- 没有新增同名分叉、悬空入口、孤儿 lock 或未说明的完整副本。

除 workspace registry 的两阶段状态事务外，结构验证通过后再按 contract 更新唯一 SOT：用户批准的变化进入 Intended，现场证据进入 Verified。registry 按已批准 manifest 先原子写入 Intended 登记与 Verified `configuring`，重新读取确认并记录新的 transaction hash 后再验证目标；第二次写入前必须以该新 hash 做 CAS 检查，匹配后才原子写入 `active`、`degraded` 或回滚后的 `not-configured`，不匹配则停止并重审。任何阶段都不能把 `configuring` 报告成成功。写入后重新读取 SOT，并逐项对照最终路径、入口和管理器状态；使用相同证据重跑时不得产生 diff。反向验证通过后才能报告成功。部分失败时优先安全回滚；无法回滚时记录未解决漂移，绝不写成成功。

验收分为三级：目标宿主完成真实发现冒烟测试才是 `完整完成`；仅验证发现机制和静态入口时是 `结构可运行，宿主待实测`；仍有未知发现机制时只能报告 `仅审计/部分完成`。不得把后两级写成完整完成。

## 各动作规则

### Normalize

- 先确定每项的作用域和四层身份，再移动或重链。
- 全局标准安装副本进入已确认的全局 skill 目录。
- 项目 skill 默认归入 `<project>/skills`。
- 只有当一个专用入口目录中的所有成员都应共享同一真源时，才使用整目录软链接。
- 宿主目录需要与系统项、外部套件或宿主专属项共存时，使用逐项入口或官方生成入口。
- 保留正文内容和可追溯来源；不要用 normalize 顺便重写业务逻辑。

### SOT

- 无 SOT：全量盘点 -> 提议全局目录 -> 用户确认 -> 创建 SOT -> 反向验证。
- 有 legacy SOT：记录源 hash，从现场重算证据，迁移明确政策与例外；遇到无法分类的章节时停止询问，不静默丢弃或保留为权威快照。
- 有有效 SOT：先查漂移，再按 Intended/Verified 规则更新；相同证据必须幂等。
- 有同名文件：先按物理身份和 authority domain 分类，只有真实同域冲突才要求用户选择。
- SOT 只记录不可安全重建的结构意图、必要映射、例外和带证据状态，不复制正文或派生 inventory。

### Workspace Registry

- registry 不属于普通 skill 审计的成功标准；未配置时可以正常完成 skill 管理。
- 提示选项为“现在建立 / 以后再说 / 永久不提示”，一次符合资格的完整审计最多提示一次。
- registry policy 的用户决策只写 Intended，运行与漂移状态只写 Verified；有效状态按上述优先级推导，不复制成第三个字段。
- workspace 的 Intended 字段为稳定 ID、名称/别名、用户确认的绝对逻辑根、一句话用途、规则入口和生命周期意图；解析后的 realpath、status、evidence 与 verified-at 只写入 Verified。
- 进入已登记 workspace 后先读取其本地规则。SOT 不覆盖业务真源、项目规则、宿主优先级或权限边界。
- 若跨宿主规则尚未迁移，只把 `eds-agent-migration` 作为明确的可选下一步；不自动启动，也不影响本轮 skill 管理结果。

### Install

- 验证来源、版本、正式成员和安装方式。
- 有可信安装管理器时使用它写入标准安装副本和 lock。
- suite 默认安装全部正式成员；只有用户明确要求时才维护成员子集，并在 SOT 记录例外。
- 为目标宿主创建入口前先验证发现机制。
- 不把开发仓库或外部托管仓库伪装成普通安装副本。

### Update

- 同时比较开发源、发布上游、安装副本、lock 和宿主入口。
- 对 suite 重新获取正式成员集合；不要假设“更新现有成员”会自动安装新增成员或清理退出成员。
- 普通项完成一次批量预检和一次确认。
- 将本地修改、来源冲突、成员退出、自更新和外部官方套件拆出。
- 更新后同步验证 lock、入口、suite 清单和 SOT。

### Remove

- 先区分要移除的是宿主入口、安装副本、lock 记录、项目真源、开发仓库还是外部仓库。
- 用户只说“删除 skill”时，默认提议移除安装副本及其生成入口和管理记录；保留开发源、发布上游和外部仓库，除非用户明确包含它们。
- 检查其他入口、项目或 suite 是否仍引用目标。
- 所有删除都单独列入预检并确认，不用排除法或批量模糊匹配删除。

### Repair

- SOT 只能恢复路径、身份、入口和管理方式，不能恢复缺失的 skill 正文。正文只能来自 lock 对应安装器、已验证 upstream、development source 或可读备份；没有权威内容来源时停止并询问。
- 修复悬空链接、缺失入口、name/目录不一致、孤儿 lock、未跟踪安装副本和 SOT 漂移。
- lock 管理项优先重装或使用管理器修复，不直接修补安装副本。
- 修复不得顺便升级版本，除非预检明确包含更新。

## 输出要求

只读盘点至少输出：

- 已确认的全局目录、当前项目和宿主发现路径。
- 按作用域分组的 skill/suite 数量。
- 每个异常的结构分类、证据、影响和建议动作。
- SOT 状态及下一步需要用户决定的事项。
- registry 状态、是否满足提示资格，以及是否仅具备能力但尚未激活。

写入完成至少输出：

- 实际完成、失败、跳过的项目。
- 写入和删除的路径。
- 验证结果与剩余漂移。
- 唯一 SOT 的路径和最近验证状态。
- 本次更新的 Intended/Verified section、保留的 drift 和 registry 激活状态。

不要把“发现数量相同”当作验证成功。验证的是身份关系、解析目标、管理状态和宿主可发现性。

## 禁止事项

- 不经全量盘点就创建 SOT。
- 不因文件同名而跳过物理身份和 authority-domain 判断。
- 不硬编码未经验证的跨宿主发现路径。
- 不把逻辑入口路径误称为物理全局目录。
- 不直接编辑 lock 管理的安装副本或手工伪造 lock。
- 不把外部托管套件复制进标准全局目录。
- 不用相同名称强行统一不同宿主的官方入口。
- 不在未确认时创建、移动、覆盖或删除任何结构。
- 不把内容评价包装成结构分类。
- 不在 SOT 中记录未经验证的成功状态。
- 不把派生 inventory、完整成员清单或计数写入 SOT，也不创建持久化 Catalog。
- 不在未获选择时扫描或登记其他 workspace。
- 不把 workspace 登记解释为业务同步、读写授权或全局规则覆盖。
- 不把本机个人路径、workspace 名称、凭据或实例数据写入可发布的 skill、reference 或 README。
