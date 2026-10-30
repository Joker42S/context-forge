# Context Forge

> 把一次性对话，锻造成可复用资产。

## 这是什么

AI 对话里最有价值的部分，往往在会话结束时被一起丢掉：一条反复打磨的 Prompt、一套验证过的排查步骤、一条只有团队才知道的业务规则。下次遇到同类问题，又得从头讲一遍。

Context Forge 是一个**可被 AI agent 直接加载的 skill，同时也是一个资产仓库**。给它一段上下文，它按固定规范精炼 **Prompt / Skill / Knowledge**，原子化沉淀，并用 git 分支按项目管理。

它解决三个问题：

1. **精炼有标准**：按分类定义和质检规则精炼，不做泛泛总结。
2. **存放有规范**：每条资产自包含、可检索、可直接复用。
3. **文档与数据分离**：`main` 始终保持为可分发的 skill，项目资产只存在于项目分支。

## 三类资产

| 类型 | 回答的问题 | 主判据 | 典型内容 | 落位 |
| --- | --- | --- | --- | --- |
| Prompt | 「怎么说」 | 一次性投喂：全文作为单次输入使用，变量替换即可 | 指令模板、输入输出约束 | `prompts/<slug>.md` |
| Skill | 「怎么做」 | 渐进式披露：description 供触发匹配，正文按需加载 | SOP、检查流程、异常分支、工具调用方法 | `skills/<slug>/SKILL.md` |
| Knowledge | 「依据什么」 | 被引用的事实依据，不直接执行、只写事实 | 术语、规则、踩坑、决策记录、内部口径 | `knowledge/<slug>.md` |

不要放错：Prompt 不放需要按需加载的方法和大段背景；Skill 不放组织事实库和术语大全；Knowledge 不放操作步骤和模板指令。不收录：无内部适配点的通用公开教程原文、空泛口号、零散话术、没有来源的总结。某类没有内容就写「无」，不硬凑。

## 快速开始

### 1. 获取

把仓库放到 agent 能读取的位置。若你的 agent 支持 skills 目录，直接克隆进去即可：

```bash
git clone <repo-url> ~/.claude/skills/context-forge
```

路径以你的 agent 约定为准（例如 Claude Code 使用 `~/.claude/skills/`；其他 agent 放到其 skill 或 plugin 目录）。核心是：agent 能读到仓库根目录的 `SKILL.md`。

不做安装也可以，在会话中让 agent 读取 `SKILL.md` 和 `reference/forge-guide.md` 即可。

### 2. 在独立会话中精炼

**不要在业务对话里直接精炼**，原因见「注意事项」。

新开一个会话，把待精炼内容以文件或粘贴方式带入：

```text
读取 ~/.claude/skills/context-forge/SKILL.md，按它精炼下面的上下文。
项目：acme-checkout
<粘贴对话，或给出文件路径>
```

### 3. 审阅并提交

agent 会先给出精炼报告、新增/更新清单、文件路径、INDEX 变更和 `git status`。确认后，它会把资产提交到 `ctx/<project>` 分支。

push 需要你明确要求；agent 不应自行推送。

## 分支模型

- **`main`**（主分支；仓库若用 `master` 等名称，以实际为准，下文以 `main` 指代）：只放文档——`SKILL.md`、`README.md`、`reference/`、`templates/`，以及三个数据目录的占位文件。**不提交任何精炼出的资产。**
- **`ctx/<project>`**：一个项目一条分支，从 `main` 创建，存放该项目全部 Prompt / Skill / Knowledge 和 `INDEX.md`。

这样做的收益：

- 不同项目的资产互不污染，切换分支即切换知识库。
- `main` 永远是干净、可分发的 skill 版本，文档改动不会混进业务数据。
- 每个项目的历史独立，便于回溯和沉淀。

```text
main               ●────●──────────●──────────●     仅文档
                    \             \          \
ctx/acme-checkout    ●────●────────●──────────●     项目资产
ctx/billing          ●────●
```

常用命令：

```bash
# 新建项目分支
git switch main && git pull --ff-only
git switch -c ctx/acme-checkout

# 切回已有项目
git switch ctx/acme-checkout

# 首次推送项目分支（需确认后执行）
git push -u origin ctx/acme-checkout
```

需要同时处理多个项目时，用 worktree 避免来回切分支：

```bash
git worktree add ../cf-acme ctx/acme-checkout
```

### 分支同步

`main` 上的文档更新（例如改进 `SKILL.md` 或模板）不会自动出现在项目分支。定期合入：

```bash
git switch ctx/acme-checkout
git merge main
```

不要对项目分支做 rebase 或 force-push。

## 目录结构

`main`：

```text
context-forge/
├── SKILL.md                    # skill 入口：工作流与硬约束
├── README.md
├── reference/
│   └── forge-guide.md          # 分类定义、精炼规则、质检、报告模板
├── templates/
│   ├── prompt.md
│   ├── skill.md
│   ├── knowledge.md
│   └── index.md
├── prompts/                    # 占位，main 上不存资产
├── skills/                     # 同上
└── knowledge/                  # 同上
```

`ctx/<project>` 分支在此基础上增加：

```text
INDEX.md
prompts/<slug>.md
skills/<slug>/SKILL.md
knowledge/<slug>.md
reports/                      # 可选：需要留存精炼报告时
```

注意：`skills/` 下是精炼出来的资产，不是安装用的 skill；被 agent 加载的是仓库根目录的 `SKILL.md`。

## 资产规范

- **原子化**：一个文件只表达一个可复用单元；重复内容合并。
- **命名**：`name` 用人读的 `场景-动作-对象`（如「代码评审-生成-风险清单」）；`slug` 与文件名用小写 kebab-case（如 `code-review-risk-list`）。Skill 目录必须为 ASCII，才能被 agent 识别为 skill。禁止 `最终版`、`新建` 等命名。
- **自包含**：不写「见上文」「如前所述」，不引用会话中的临时 ID。
- **可追溯**：每条资产标注 `source`（原文摘要或位置）和 `confidence`（high / medium / low）。
- **时效**：填写 `validity`；未知就写「未知」。
- **更新优先**：同一资产已存在时更新原文件，不新建重复条目。
- **索引一致**：`INDEX.md` 的统计与链接必须和实际文件一致。
- **脱敏**：Knowledge 填 `access`，敏感信息必须脱敏；无法脱敏则不落盘，列入「待确认」。

frontmatter 示例：

```yaml
---
type: knowledge
name: 结算-账期-计算口径
slug: settlement-billing-cycle
project: acme-checkout
kind: 数据口径
source: 2026-10-28 与财务的对话，第 3 段
confidence: high
validity: 2026 Q4
access: 内部
tags: [billing]
related: []
updated: 2026-10-30
---
```

## 注意事项

### 1. 为什么必须在独立会话中精炼

加载 skill 会注入完整的精炼流程，并要求模型重新通读、分类、改写上下文。在承载业务任务的会话里做这件事，会带来两个问题：

- **挤占上下文**：原始讨论、代码和决策细节被精炼流程覆盖或压缩，后续业务对话失去精度。
- **污染判断**：模型一边要继续当前任务，一边要扮演「资产精炼师」，容易把进行中的假设当成结论写进资产。

所以建议开一个**独立会话**（相当于给这次精炼开一条「上下文分支」），只带入需要沉淀的材料；业务会话保持原样。数据侧再用独立的 git 分支隔离。

### 2. `main` 不提交数据

`main` 只接受文档类改动。精炼出的 Prompt、Skill、Knowledge、索引、报告都属于项目数据，只能提交到 `ctx/<project>` 分支。向 `main` 提 PR 时，只应包含 `SKILL.md`、`README.md`、`reference/`、`templates/` 的改动。

### 3. 敏感信息

对话中可能包含密钥、客户数据、内部链接。精炼前先判断：能公开吗？需要脱敏吗？无法确认的，不落盘，放进「待确认」。提交前检查 `git diff`，不要使用 `git add -A`。

### 4. 不编造，不为凑数而精炼

只基于给定上下文。合理推断标「归纳」，不确定标「待确认」。上下文很短时，允许产出 0 条资产。

### 5. 不动用户未提交的改动

agent 切分支前必须检查 `git status`。有未提交改动时停止并询问，禁止用 `checkout`、`reset`、`clean` 清理。

### 6. 提交与推送

提交前展示清单并征得确认；只暂存明确路径；push 必须由用户明确要求。项目分支不要 force-push。

## 设计原则

- **资产是给下一次用的，不是给这一次看的。** 判断标准是「换一个会话、换一个人，能否直接复用」。
- **文档与数据同仓不同分支。** 分发的是 skill，沉淀的是分支。
