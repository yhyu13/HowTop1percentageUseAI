# AI_Tutor（狸爱学 PBL）如何用"前 1%"AI 技巧：从产品到技术

> 分析对象：`C:\Git-repo-AI\AI_Turtor`（主工程 `pbl/`，生产站 https://www.ailiaixue.com）
> 分析基准：本仓库 `docs/research/00–05` 前 1% AI 用户调研（第一期）
> 写作日期：2026-10-09

## TL;DR

1. **你其实已经站在 top-1% 线上。** `AGENTS.md` + `memory/` + `docs/agent-architecture.md`（7 角色流水线 + L1–L4 分级验证 + G1–G6 fail-closed 门禁）正是调研里「顶级用户把 AI 当系统经营」的工业级形态。真正的差距不在"用不用 AI"，而在**产品侧的上下文工程与学习者建模**。
2. **三处最大的技术缺口**：① AI 对话每轮加载**全部历史**，只有一句"请简短"的伪压缩；② 号称"AI 评估"的 `evidence-analyzer.ts` 其实是**正则关键词**；③ `knowledgeMastery` 是 ±0.03 的规则加减，**不是知识追踪**。这三处恰是前 1% 用法与普通用法的分水岭。
3. **最大的产品机会**：你的"苏格拉底式、不直接给答案"恰好命中 Anthropic 实证里**唯一能让人用 AI 还变强**的模式（概念提问型，平均 86 分）。把它变成**可量化卖点（答案泄露率）**，就是区别于豆包/通用聊天与 Khanmigo 的护城河。
4. **对标 Khanmigo 的教训**：没有 skill gating 的 tutor 不构成产品。你已经埋了 `KnowledgePoint.prerequisiteIds` 与 `knowledgeMastery`，只差把它们接成「前置达标才推荐/解锁」的门控 + 可视化掌握图谱。
5. **上下文纪律 = 成本纪律**：Karpathy 说 90% 的 AI 账单花在不需要的上下文上。你的 `lib/ai/prompts.ts` 单文件 **66 KB**、对话每轮全量历史，是成本与质量的双重风险——要压到"最小高信号 token 集"。
6. **今天就能做的最高性价比动作**：把 `docs/agent-architecture.md` 自己写的 §8「最小可用版本」落地（R1 Router 常驻 / R5 独立分级验证 / 打 tag 前 `bun run build` 门禁）——这三点覆盖了本项目历史上最贵的三次翻车（rc5 / #106 / #109）。

---

## 一、项目现状快照（我实际读到的）

| 维度 | 现状 | 关键文件 |
| --- | --- | --- |
| 产品 | AI狸爱学·个人项目化学习系统；小学 3–5 年级 PBL，教师定主题 → AI 为每个学生定制项目 → AI 导师 6 阶段引导 → 过程评估 → 成长报告 | `prd.md`、`docs/prd.md`、`docs/ai-roadmap.md` |
| 技术 | Next.js 16 App Router 单体 + Prisma 6 + PostgreSQL 18 + Bun；AI 走 DeepSeek（OpenAI 兼容）经 Vercel AI SDK；TOS 上传；PM2 + Caddy；`schoolId` 租户隔离 | `AGENTS.md`、`package.json`、`prisma/schema.prisma` |
| AI 资产 | 学生导师对话 / 项目生成 / 策略层（SIPA）/ 证据分析 / 上下文构建 | `app/api/ai/chat/route.ts`、`lib/ai/project-generator.ts`、`lib/ai/sipa.ts`、`lib/ai/evidence-analyzer.ts`、`lib/ai/{driving,mentor}-context.ts` |
| 数据 | `KnowledgePoint`（含 `prerequisiteIds` / `misconceptions`）、`StudentLearningProfile.knowledgeMastery`（知识点→0–1）、`LearningEvidence`、`Conversation`/`ChatMessage` | `prisma/schema.prisma:317-413` |
| 工程规范 | 宪法 + 记忆 + 多 Agent 架构 + 分级验证 + 门禁 | `AGENTS.md`、`CLAUDE.md`、`memory/core/*`、`docs/agent-architecture.md` |
| 商业 | 7 年财务模型：B2C 订阅 + B2B 学校 + 教师工具包，国际为主；目标 2033 IPO | `7-Year-Financial-Model.md`、`docs/price.md` |

---

## 二、关键判断：你已经领先的地方（别推倒重来）

对照调研结论，下面这几件事你**已经做对了**，应保留并在其上继续叠加：

1. **规则文件即宪法**：`AGENTS.md` 把路由、不可破坏约束、验证基线、服务器升级 SOP 全部钉死；`CLAUDE.md` 只做单点跳转——正是 Anthropic/Karpathy 倡导的"规则文件内核"。
2. **记忆文件化**：`memory/core/invariants.md`（不可漂移约束）+ `symptom-routing.md`（症状→文件，上百次 bug 沉淀）+ `workflows.md`——这是 Anthropic "agentic memory / structured note-taking" 的真实工程化。
3. **多 Agent 分工与门禁**：`docs/agent-architecture.md` 的 R1–R8 角色契约 + G1–G6 门禁 + fail-closed，质量高于绝大多数商业团队。它已明确指出"翻车从不是能力不够，而是缺一道机械门禁或一次真实环境验证"。
4. **确定性优先**：`sipa.ts` 用规则做问题类型/思考深度/情绪判定，`knowledgeMastery` 用确定性 delta——在 LLM 不稳定处用规则兜底，是正确架构取向（不是缺陷）。
5. **产品理念正确**：苏格拉底式引导、按需学习、过程性评价、教师每周 <10 分钟——产品方向与调研里"AI 教育必须带护栏 + 学习者建模"一致。

**结论：你要的不是"引进 top-1% 技巧"，而是"把已知的正确原则在 AI 运行时补齐"。**

---

## 三、产品侧：把前 1% 技巧落到 tutor 上

### 3.1 上下文工程就是产品力（不只是技术）

调研结论：顶级应用的竞争力是"用最小的高信号 token 集合换取最大概率的正确行为"（Anthropic）。对你的产品，这意味着**导师质量 = 喂给模型的那段上下文的质量**。

现状问题（我在 `app/api/ai/chat/route.ts` 读到）：
- 每轮 `prisma.chatMessage.findMany(...)` **把整段对话历史全量塞回**，对话越长越慢越贵越"糊"；
- `system: contextPrompt` 里拼接了巨大的 `prompts.ts`（66 KB 级别的阶段提示词）+ 上下文；
- 所谓"长对话压缩"只是在中点插一句"请保持回复简短"——这是**伪压缩**，不减 token，也不保留关键信息。

**改法（直接对应 Anthropic 三件套）：**
- **Compaction（真压缩）**：对话超过阈值时，把历史交给便宜模型总结成"已确认目标 / 已解决卡点 / 未解决卡点 / 关键决定"，用摘要替换旧历史，只保留最近 N 轮原文。
- **结构化笔记（Agentic Memory）**：给每个学生-项目维护一份**学习笔记块**（见 4.2），每轮只注入这份精简状态，而不是原始历史。
- **子 Agent**：把"生成 / 导师 / 评估 / 摘要"拆成独立上下文（见 4.5）。

### 3.2 学习者建模：从"标签"升级到"知识追踪 + 门控"

调研结论（方向 8）：技术路线已收敛为 **RAG（内容限定）+ 知识追踪（BKT/DKT）+ 苏格拉底策略 + skill gating**；Khanmigo 被论文批评"无技能门槛"。

现状：`knowledgeMastery` 是 `Record<knowledgePointId, number>`，由 `evidence-analyzer` 的规则给出 ±0.03/0.04 的 delta——**是热度表，不是掌握概率**。

**改法：**
- 升级为**贝叶斯知识追踪（BKT）**：`p(掌握)` 用 `learn/guess/slip` 三参数递推，而不是线性加减；有 `LearningEvidence` 作为观测序列，数据天然够用。
- **接上 `KnowledgePoint.prerequisiteIds` 做 skill gating**：前置知识点未达标 → 不推送当前知识点，而是先补齐前置（这正是"诊断→选择模式→学习→验证→应用"闭环里的"诊断"）。
- **产品化**：把掌握度做成**学生可见的"掌握图谱"**（绿/黄/红 + 前置路径），教师看板输出"全班知识点热力图"——这是通用聊天机器人永远给不了的东西，也是卖给学校的核心交付物。

### 3.3 反退化设计 = 差异化卖点（可量化）

调研结论（方向 7）：把任务整包交给 AI 的开发者**技能退化最重（调试能力）**；而"概念提问型"（只问概念、自己动手）平均 86 分且速度不慢。**AI 当解释器，不当答案生成器。**

你的产品理念已经对了，缺的是**把它变成可测量、可展示的证据**：

- 新增运行时指标 **「答案泄露率」**：AI 回复中是否直接给出了本应由学生产出的答案/成果（用轻量分类器或规则+LLM 判断）。
- 新增 **「认知参与度」**：学生的追问密度、独立产出占比、卡点自解率。
- 把这两个指标放进**成长报告与学校汇报**——"我们不让孩子把思考外包给 AI"，这是面对家长/学校最有力的合规与价值叙事，也是与"免费豆包"的正面区隔。

### 3.4 项目生成与评估的"去黑箱"

- 项目生成（`lib/ai/project-generator.ts`）是产品核心（"为每个学生定制"），应纳入**提示词资产化 + 回归测试**（见 4.6），否则质量不可控、无法 A/B。
- 评估（`lib/ai/evidence-analyzer.ts`）目前是**纯正则**，与"AI 评估"的产品叙事不符，且无法识别真正的误解（`KnowledgePoint.misconceptions` 字段没被利用）。改法见 4.3。

### 3.5 分发与信任：一人公司的真相

调研结论（方向 2）：模型不是护城河，**分发、信任、垂直数据才是**；红利窗口以月计（seeu.food 三个月被卷死）。

对你的 B2B2C 学校生意：
- 真正稀缺的是**教师信任与转介绍**，不是功能数量。把"教师每周 <10 分钟 + 一键出报告"做成可传播的样板（公开课、示范校、教师社群）。
- 你的**作品广场（`/plaza`、`/showcase`）本身就是增长回路**——学生作品可分享、可互评，是天然的获客与留存资产，应在 roadmap 里提升优先级（研究里金浩斌靠平台免费曝光零买量）。
- 金融模型假设了陡峭增长曲线（Year1 2400 学生 → Year7 200 万），但调研反复证明**增长瓶颈在渠道不在产品**；建议财务模型对"学校采购周期 6–12 个月"给更保守的 bear case 权重。

---

## 四、技术侧：把 top-1% 做法映射到具体文件（对着你的 roadmap）

下表每行都可直接变成 issue 的验收口径。

| # | 现状 | 前 1% 做法 | 触碰文件 | 验收口径 |
| --- | --- | --- | --- | --- |
| T1 | 每轮全量加载对话历史；伪压缩（插一句"请简短"） | 真 Compaction：摘要旧轮 + 保留最近 N 轮 | `app/api/ai/chat/route.ts`、`Conversation`（加 `summary` 字段） | 单对话 P95 输入 token 降 ≥50%，且不丢"已解决卡点/未解决卡点" |
| T2 | 上下文靠长字符串拼接 | 结构化上下文构建 + token 预算 | `lib/ai/mentor-context.ts`、新增 `lib/ai/context-builder.ts` | 每段上下文有独立 token 上限；超预算按优先级裁剪 |
| T3 | 掌握度 = 规则 ±0.03 加减 | 贝叶斯知识追踪（BKT），用 `LearningEvidence` 递推 | `lib/actions/learning-evidence.ts`、新增 `lib/ai/knowledge-tracing.ts` | 掌握度有可解释参数（learn/guess/slip）；单测覆盖收敛 |
| T4 | 无 skill gating；`prerequisiteIds` 未用 | 前置知识点达标才推荐/解锁；掌握图谱可视化 | `lib/curriculum.ts`（或新模块）、学生端知识点页、教师看板 | 未达标前置时不出当前知识点；图谱可读 |
| T5 | 评估是正则关键词 | 轻量 LLM 结构化评估（误解/证据/情绪），规则兜底 | `lib/ai/evidence-analyzer.ts`、`lib/ai/prompts*.ts` | 输出稳定 JSON schema；规则层作为 fallback 与门禁 |
| T6 | 单一大 chat agent | 拆 生成/导师/评估/摘要 四类子 Agent，各自干净上下文 | `lib/ai/*`、`app/api/ai/*` | 每个 Agent 有独立 system prompt 与输入契约；互不污染 |
| T7 | `prompts.ts` 66 KB + `prompts_0.ts` + `.backup` | 提示词版本化 + 黄金用例回归 + A/B | `lib/ai/prompts*/` | 有 `prompts/vX` 结构；CI 跑提示词回归 |
| T8 | 单一 DeepSeek 模型 | 模型分层：分类/摘要走便宜档，导师走强档；开缓存 | `.env.example`、AI client | 成本面板按学生/项目可见；单价有目标 |
| T9 | Dev 门禁成熟，但"AI 质量"无门禁 | 加 G7：答案泄露率 / 幻觉率 / 输出格式 门禁 | `docs/agent-architecture.md`、`test/` | 提示词变更必须过 G7 才合并 |

### 4.1 真 Compaction（T1）——性价比最高的一处

现在每轮把全部 `ChatMessage` 塞回模型。改法：
- `Conversation` 增加 `summary String?` 与 `summarizedUpTo`（记录已压缩到的消息序号）。
- 触发条件：历史 token 超阈值（比如 > N 轮或 > X token）。
- 压缩提示词产出**固定四段**：`已确认目标 / 已解决卡点 / 未解决卡点 / 关键决定`（对应 Anthropic 保留"架构决策 / 未解决 bug / 实现细节"，丢弃冗余工具输出）。
- 注入时 = `summary` + 最近 N 轮原文。**不要**再插"请简短"。
- 收尾（`sipa.converge`）那一轮仍走全量，因为要写完整成果摘要。

### 4.2 结构化记忆：给每个学生-项目一份"学习笔记"（T2）

参照 Anthropic "structured note-taking"（NOTES.md 模式）：每轮只注入一份精简状态，而不是原始历史。建议形态（存在 `Conversation.summary` 或新增 `ProjectLearningNote`）：

```
[学习笔记]
- 当前里程碑：第3阶段 · 设计问卷
- 已掌握（≥0.7）：变量控制(0.82)、数据记录(0.74)
- 薄弱（<0.4）：抽样方法(0.31) ← 前置"总体与样本"未达标，先补
- 兴趣/偏好：恐龙、喜欢画图
- 未解决卡点：不知道怎么让问题"可测量"
- 最近决定：问卷先做 5 题小样本
```

这份笔记既是**给模型的上下文**，也是**给教师看板的摘要**——一份数据两处复用。

### 4.3 评估去正则化（T5）

`evidence-analyzer.ts` 现在是关键词匹配（`/不会|不懂|卡住/` 等）。保留它作为**确定性门禁与兜底**，但主路径换成**便宜模型的结构化输出**：
- 输入：学生消息 + AI 回复 + 当前知识点（含 `misconceptions`）。
- 输出（强 schema）：`{ sentiment, blockerLevel, misconceptionIds[], masteryEvidence[], abilityTags[], suggestedLearningMode }`。
- 只有 LLM 失败/超时才回落到正则。
- 好处：能真正利用 `KnowledgePoint.misconceptions`，把"错误诊断"做实（roadmap 第四阶段想要的能力）。

### 4.4 BKT 与 skill gating（T3/T4）

- 新建 `lib/ai/knowledge-tracing.ts`：对每个 `knowledgePointId` 维护 `{ pKnown, learn, guess, slip }`，用 `LearningEvidence` 序列递推；初始 `pKnown=0.5` 与现在默认一致，可灰度对比。
- gating 规则：只有 `prerequisiteIds` 全部 `pKnown ≥ 阈值`（如 0.6）才把该知识点纳入推荐与练习；否则先补前置。
- 教师看板按 `subject × gradeLevel` 聚合出热力图（`KnowledgePoint` 已有 `@@index([schoolId, subject])`）。

### 4.5 子 Agent 拆分（T6）

对齐 Anthropic「主 Agent 只做计划与综合，子 Agent 用独立上下文深挖后回传摘要」：

```
项目生成 Agent ──▶ 产出 planSnapshot（已有 project-generator.ts）
导师 Agent     ──▶ 对话（已有 chat/route.ts，需瘦身）
评估 Agent     ──▶ 结构化证据（改造 evidence-analyzer.ts）
摘要 Agent     ──▶ 对话压缩 + 阶段总结（新增）
报告 Agent     ──▶ 成长叙事（已有 report-generator.ts）
```

关键纪律：**每个 Agent 有独立 system prompt 与输入契约，不共享一段巨型 prompt**。这也是解决 `prompts.ts` 66 KB 膨胀的解法。

### 4.6 提示词资产化 + 回归测试（T7）

- 拆分为 `lib/ai/prompts/<agent>/<name>.v1.ts`（或 JSON），带版本号；删除 `prompts_0.ts` 与 `.backup`。
- 建**黄金用例集**（`test/ai-golden/`）：典型学生输入 → 期望行为（不泄露答案、语气适配年级、识别指定误解）。提示词变更必须跑回归。
- 这与你的 `AGENTS.md`「写测试 → 实现 → CI 绿」循环天然契合，只是把测试对象从代码扩到提示词。

### 4.7 模型分层与成本（T8）

- 分类/摘要/情绪 → 便宜档（Haiku 级）；导师对话 → 强档（Sonnet 级）。
- 开启 prompt caching（Claude）复用稳定前缀（阶段提示词 + 项目上下文）。
- `usage-tracking` 落到每学生/每项目面板（roadmap 第六阶段已有），并设日上限。
- **成本纪律来自上下文纪律**：先做 T1/T2，成本自然下来。

### 4.8 把 Dev 门禁延伸到"AI 质量门禁"（T9）

你已有 G1–G6（代码层）。新增 **G7**：
- 提示词变更 → 必须过黄金用例；
- 上线前抽检**答案泄露率**（AI 是否越界直接给答案）；
- 输出格式合规（JSON schema 失败率）。

这样"AI 用对没有"也变成机械门禁，而不是靠人感觉——完全符合 `docs/agent-architecture.md` 的哲学：**"翻车从不是能力不够，而是缺一道机械门禁"**。

---

## 五、30 / 60 / 90 天行动

**第 1–2 周（低成本、高确定性）**
1. 落地 `docs/agent-architecture.md` §8 最小版：R1 Router 常驻、R5 独立分级验证、打 tag 前 `bun run build`（G3）。
2. T1 真 Compaction（先上线，作为对话质量的立竿见影改进）。
3. 建 `test/ai-golden/` 骨架 + 一个"不泄露答案"用例。

**第 3–6 周**
4. T2 结构化上下文 + 学习笔记块（同时在教师看板复用）。
5. T5 评估去正则化（LLM 结构化 + 规则兜底）。
6. T7 提示词拆分与版本化，删除历史备份文件。

**第 7–12 周**
7. T3 BKT + T4 skill gating + 掌握图谱（学生端 + 教师热力图）。
8. T6 子 Agent 拆分（摘要 Agent 先落地）。
9. T9 G7 AI 质量门禁 + 答案泄露率指标进成长报告。

---

## 六、风险与红线

- **别让"提升 AI"破坏确定性**：`sipa.ts` 的规则层是资产，不是债务。LLM 只加在它之上，不替换它。
- **隐私与合规**：学生画像、对话日志属敏感数据；引入 LLM 评估时保持 `schoolId` 租户隔离与脱敏（roadmap 第六阶段已列）。
- **成本失控**：任何"多 Agent / 长上下文"改动都必须先有 token 预算与上限，否则正面撞上 Karpathy 说的"90% 浪费"。
- **K12 红线**：产品是儿童场景，AI 输出需内容审核；苏格拉底式护栏既是教育理念也是安全边界，不要为了"体验流畅"而放松。
- **验证分级**：凡涉及真实时间/网络/设备的改动（TTS、上传、外部域名），必须到 L4 真实环境验证——这是你 #116、#109 两次翻车的直接教训。

---

## 七、来源

**项目内文件**：`AGENTS.md`、`CLAUDE.md`、`AI_PLAYBOOK.md`、`prd.md`、`docs/ai-roadmap.md`、`docs/agent-architecture.md`、`docs/status-summary.md`、`7-Year-Financial-Model.md`、`app/api/ai/chat/route.ts`、`lib/ai/{mentor-context,evidence-analyzer,project-generator,sipa}.ts`、`prisma/schema.prisma`

**调研依据**（本仓库）：
- `docs/research/00-总览与方法论.md`（十点发现 + 来源清单）
- `docs/research/01-顶级用户习惯与工作流.md`（上下文工程 / 规则文件 / 子 Agent / 反退化六模式）
- `docs/research/05-AI教育tutor创业情报.md`（Khanmigo skill gating 教训 / BKT 技术路线 / 市场机会）
- `docs/workflow/首批可落地行动清单.md`

**外部一手来源（关键几条）**：Anthropic《Effective context engineering for AI agents》、Anthropic《AI assistance and coding skills》、Karpathy 四准则、墨刀 AI PRD 教程、Khanmigo 横评、Duolingo AI 上调指引。完整 URL 见 `docs/research/00-总览与方法论.md` 来源清单。
