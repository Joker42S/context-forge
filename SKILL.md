---
name: context-forge
description: 从 AI 对话、文档或聊天记录中精炼出可复用的 Prompt、Skill、Knowledge，去重后按规范写入 context-forge 仓库的 prompts/、skills/<slug>/SKILL.md、knowledge/，更新 INDEX.md 并提交到对应项目的 git 分支。适用于“精炼这段对话”“沉淀资产”“把方法固化成 prompt/知识”“复盘并把方法存下来”等请求。
---

# Context Forge — AI 资产精炼师

把给定上下文中的可复用内容精炼为 **Prompt / Skill / Knowledge**，按本仓库规范落盘。产出必须是可直接复用的资产，**不是对话总结**。

## 何时使用

- 用户要求精炼、沉淀一段对话、文档或复盘记录
- 需要把临时方法固化成 prompt 或 skill
- 需要沉淀项目术语、业务规则、踩坑经验、决策记录

不适用：单纯润色、单纯总结、搬运通用公开教程。

## 硬约束（先读）

1. **独立会话**：不得在承载原始业务任务的会话里直接精炼。加载本 skill 和重读上下文会挤占并污染业务上下文；应先开一个新会话，把待精炼内容以文件或粘贴方式带入。详见 `README.md` 的「注意事项」。
2. **main 只放文档**：main 上只允许文档类改动，即 `SKILL.md`、`README.md`、`reference/`、`templates/`。精炼出的资产一律写入 `ctx/<project>` 分支，绝不提交到 main。
3. **一项目一分支**：资产按项目隔离，禁止在一个分支里混多个项目。
4. **只基于上下文**：不编造外部事实；合理归纳标注「归纳」，不确定标注「待确认」。
5. **宁缺毋滥**：上下文不足就不产出，某类为空写「无」，不要凑数。
6. **提交需确认、push 需明确要求**：提交到本地项目分支前先展示清单并征得确认；未经用户明确要求不得 push。

## 工作流

### 1. 定位仓库与项目

- 以包含本 `SKILL.md` 的仓库根目录作为工作目录，而不是当前业务项目的根目录。先执行 `git status`；若有未提交改动，停止并询问用户，禁止丢弃或覆盖。
- 确定项目标识 `<project>`：优先使用用户给出的项目名或仓库名，转换为小写 kebab-case（仅 `[a-z0-9-]`）。无法判断时询问用户，不要猜。

### 2. 切到项目分支

有远端时先同步：

```bash
git fetch origin
git switch main
git pull --ff-only
```

然后切到项目分支。先确认本地和远端是否已有该分支，不要用 `||` 兜底，避免在切分支失败（例如工作区不干净）时误建新分支：

```bash
git branch -a --list "*ctx/<project>*"
git switch ctx/<project>                                      # 本地已有
git switch -c ctx/<project> --track origin/ctx/<project>      # 远端已有、本地没有
git switch -c ctx/<project> main                              # 远端和本地都没有，从 main 新建
```

- 分支已存在：直接使用，先读 `INDEX.md` 与目录清单，后续以更新、合并为主。
- 新建分支：从 main 创建，并准备写入 `INDEX.md`。
- 不要 rebase 或 force-push 项目分支；需要同步 main 的文档更新时用 `git merge main`。

### 3. 精炼

按 `reference/forge-guide.md` 执行分类、原子化、来源与置信度标注、质量检查。先在内部产出精炼报告（不落盘，除非用户要求存档到 `reports/`）。

分类判定：

| 内容形态 | 归类 |
| --- | --- |
| 单轮或单任务可复用的指令模板 | Prompt |
| 多步骤、可执行、带判断标准的方法 | Skill |
| 组织特有的事实、规则、术语、踩坑、决策 | Knowledge |
| 通用公开教程、空泛口号、零散话术 | 不收录，列入「仅索引」 |

同一内容可复用时，优先固化为 Prompt 或 Skill；纯背景信息才归 Knowledge。

### 4. 查重

写入前检查 `INDEX.md`、`prompts/`、`skills/`、`knowledge/` 中的已有资产：

- 语义重复：更新原文件，在 INDEX 中标注更新日期，不新建。
- 部分重叠：拆分或合并，保证一条资产只表达一个可复用单元。

### 5. 落盘

使用 `templates/` 下的模板，按类型放置：

| 类型 | 路径 | 文件名 |
| --- | --- | --- |
| Prompt | `prompts/<slug>.md` | 小写 kebab-case |
| Skill | `skills/<slug>/SKILL.md` | 小写 kebab-case，`name` 与目录名一致 |
| Knowledge | `knowledge/<slug>.md` | 小写 kebab-case |

命名约定：

- `name` 字段用人读的 `场景-动作-对象`，例如「代码评审-生成-风险清单」。
- `slug` 用英文或拼音的小写 kebab-case，例如 `code-review-risk-list`。Skill 目录必须为 ASCII，才能被 agent 作为 skill 加载。
- 禁止 `最终版`、`新建`、日期后缀等无意义命名。

写入要求：

- frontmatter 必填字段按模板填写；`project` 与当前分支一致，`updated` 用当天日期。
- 资产必须自包含，不写「见上文」「如前所述」，不引用会话中的临时 ID。
- 每条注明 `source`（原文摘要或位置）与 `confidence`（high / medium / low）。
- Knowledge 含敏感信息时在 `access` 中写明，并做脱敏；无法脱敏则不落盘，列入「待确认」。

### 6. 更新索引

维护项目分支根目录的 `INDEX.md`：项目信息、统计、全部资产表格。首次使用时按 `templates/index.md` 创建；新增、更新、跳过都要保持一致，链接必须可点击。

### 7. 自检

- [ ] 分类正确，无重复、无空泛、无编造
- [ ] 每条资产有来源与置信度，且只有一个可复用单元
- [ ] 路径、命名、frontmatter 符合规范
- [ ] INDEX 链接有效，计数与文件一致
- [ ] 无敏感信息，无 main 分支数据
- [ ] `git status` 中只有本分支应提交的内容

### 8. 汇报与提交

向用户展示：精炼统计、新增/更新文件、INDEX 变更、被排除内容及原因、待确认问题。

用户确认后，按显式路径暂存并提交：

```bash
git add prompts skills knowledge INDEX.md
git commit -m "ctx(<project>): add 3 prompts, 1 skill, 2 knowledge"
```

不要使用 `git add -A` 或 `git add .`。未经明确要求不要 push。

## 汇报格式

面向用户的汇报参考 `reference/forge-guide.md` 的「资产精炼报告」结构，重点保留：

- 精炼说明与统计
- 新增 / 更新 / 跳过清单（含路径）
- 「仅索引 / 不建议收录」及原因
- 待确认问题
- 质量检查结论
- 最终仓库落位

## 参考

- `reference/forge-guide.md` — 分类定义、精炼规则、质量检查、报告模板
- `templates/prompt.md`、`templates/skill.md`、`templates/knowledge.md`、`templates/index.md` — 落盘模板
- `README.md` — 分支模型、安装方式、注意事项
