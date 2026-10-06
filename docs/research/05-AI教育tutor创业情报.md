# 05 · AI 教育 Tutor 创业情报

> 覆盖方向 8：竞品与模式、个性化 tutor 技术路线、市场机会。直接服务用户的「个人 AI 教育 tutor」创业项目。

## TL;DR

1. 所有 AI tutor 的理论原点是 **Bloom 2 Sigma 问题**（1984：一对一辅导的学生成绩超过 98% 的传统课堂学生）——AI 是史上第一次让 1:1 辅导的边际成本趋近于零；世界银行在尼日利亚的 GPT-4 课后辅导实验已报告大幅学习增益，商业与实证双重验证需求成立。
2. 海外格局三分：**护栏派**（Khanmigo：苏格拉底式、不直接给答案、绑定自家课程内容、免费教师工具）、**学科专精派**（Duolingo Max 语言、Amira 阅读）、**通用聊天机器人**（无护栏但灵活）。Khanmigo 的教训：设计与安全最强，但受限于英语/美国市场与 Khan 课程体系。
3. 国内大厂靠"渠道 + 内容 + 硬件"挤压纯软件 tutor（松鼠AI 智适应大模型、作业帮/豆包、学习机品类），政策端"AI+教育"被普遍描述为近万亿市场、千亿级赛道——留给个人的缝隙是**极窄人群 × 深度陪伴 × 垂直数据**。
4. 技术路线已经收敛为：**RAG（内容限定）+ 知识追踪（BKT/DKT 学习者建模）+ 苏格拉底策略层（引导而非给答案）**，开源界已有完整参考实现（Bayesian knowledge tracing + Socratic policy scaffold 的个人 tutor capstone）。
5. 最大风险提示：Khanmigo 被指"无技能门槛的辅导"（tutors without skill gates）效果存疑——**没有学习者建模与难度适配的"陪聊式 tutor"不构成产品**；用户的切入点应避免与豆包式免费通用问答正面竞争。

## 一、赛道底层逻辑

- **Bloom 2 Sigma**：一对一辅导让学生的成绩分布比传统课堂高 2 个标准差（超过 98% 的同侪）。所有 AI tutor 的商业叙事都建立在这一个数字上。（来源：https://www.edugenius.app/blog/khanmigo-vs-other-ai-tutors-an-honest-comparison）
- **世界银行尼日利亚实验**：课后项目使用 GPT-4 聊天机器人做 tutor，报告了大幅学习增益——"令人鼓舞但仍早期"的实证。（同上来源引用 World Bank, 2025）
- **现实渗透率仍低**：RAND 调查显示 2023–24 学年只有少数美国教师在教学中用过 AI——供给端创新远快于采用端，教育渠道是慢变量。（同上）
- **Duolingo 的商业化验证**：2025 年因 AI 工具提升用户参与度而上调全年收入指引，财报后股价单日涨近 30%（路透：https://www.investing.com/news/stock-market-news/duolingo-raises-2025-revenue-forecast-as-ai-tools-boost-user-engagement-4174336 ；中文报道：https://www.chinastarmarket.cn/detail/2109423）。

## 二、竞品表格

### 2.1 海外主要玩家

| 产品 | 模式/定位 | 技术与设计要点 | 定价（公开资料） | 强项 / 弱项 | 来源 |
| --- | --- | --- | --- | --- | --- |
| Khanmigo (Khan Academy) | K-12 苏格拉底式 tutor + 教师工具 | GPT-4；"先问后说"不直接给答案；强护栏；绑定 Khan 课程内容 | 学生版约 $4/月（区域差异）；教师版免费 | 安全与教学设计标杆 / 英语为主、课程绑定、效果证据仍在积累 | https://www.edugenius.app/blog/khanmigo-vs-other-ai-tutors-an-honest-comparison |
| Duolingo Max | 语言学习（订阅档） | AI 视频通话对练、解释答案；AI 参与度带动收入指引上调 | 订阅制（Max 为最高档） | 游戏化留存行业第一 / 学科单一 | https://www.investing.com/news/stock-market-news/duolingo-raises-2025-revenue-forecast-as-ai-tools-boost-user-engagement-4174336 |
| Speak | AI 口语陪练 | 语音优先、场景化对话 | 订阅制 | 垂直做深"开口说" / 同类竞争激烈 | https://www.edugenius.app/blog/khanmigo-vs-other-ai-tutors-an-honest-comparison （分类归属） |
| Amira Learning | 阅读专精（儿童朗读评估） | 语音测评 + 分级阅读 | B2B/学区 | 学科闭环 / 市场窄 | 同上 |
| SchoolAI / MagicStudent | 教师管控型平台 | 教师设定边界与观察学生使用 | 面向学校 | 采购友好 / 学生端体验一般 | 同上 |
| ChatGPT / Gemini（通用） | 万能但无护栏 | — | 免费+订阅 | 灵活 / 会直接给答案，教育场景失控 | 同上 |

补充：有论文直接研究 Khanmigo 的"无技能门槛"问题（Khanmigo Tutors Without Skill Gates，https://qu3ry.net/articles/llm-skill-gating-khan-academy.pdf）——提示词里没有难度适配时，tutor 无法针对学生水平教学。

### 2.2 国内主要玩家

| 产品 | 模式 | 关键事实 | 来源 |
| --- | --- | --- | --- |
| 松鼠AI | 智适应学习 + 全学科多模态教育大模型 | "因材施教"叙事；智适应大模型应用成果发布；智能老师赋能基础教育 | https://finance.sina.com.cn/jjxw/2025-06-26/doc-infckxsm3528362.shtml 、https://baike.baidu.com/item/全学科多模态智适应教育大模型/67341526 、https://www.bjnews.com.cn/detail/1741784125168975.html |
| 豆包（字节）教育场景 | 通用大模型免费问答 + 学习机硬件 | 以免费通用能力挤压纯软件 tutor 的付费空间 | https://www.cls.cn/detail/2258400 （赛道综述） |
| 作业帮等 K12 工具系 | 拍照搜题 → AI 讲解 | 渠道与题库资产深 | 同上 |
| 幻课+机器人（教育 MCN 创业） | AI 课程 + 机器人硬件方案 | 首月付费用户破万（36氪首发） | https://www.c114.net.cn/industry/48715.html |
| 某新东方系"AI 学校" | AI 替代课堂教学 | 创始人让儿子辍学用 AI 上课；负债 9 亿不垮、拟开到美国 | https://www.infoq.cn/article/duddbjgyyaj1cliv9kme |
| 学术界动向 | 智能体式 AI 导师 | CCF 报告《面向大规模个性化教育的智能体式 AI 导师》 | https://ccf.org.cn/web/html7/JBdetail.html?channelId=f86b56e...CmsId=458a8793c195497db94046af36da6d52 |

市场判断来源：21 世纪经济报道《"AI+教育"全面推进，有望引领近万亿市场》（http://www.21jingji.com/article/20260413/herald/9082ed161fceec7b35600a8946b613b1.html）；财联社《技术平权引爆"AI+教育"，科技巨头、教育龙头竞逐千亿赛道》（https://www.cls.cn/detail/2258400）；电商报《AI教育赛道 2026 变局：三大模式分化背后的商业逻辑与盈利路径》（https://imgs-b2b.100ec.cn/detail--6662541.html）。

## 三、个性化 tutor 技术路线（可动手层）

### 3.1 参考架构（综合开源实现与论文）

```
学习者模型          内容层            策略层             评估层
 │                  │                │                 │
知识追踪            课程/题库 RAG     苏格拉底提问策略     掌握度仪表盘
(BKT/DKT/IRT)  →    (引用定位+权限) → (不直接给答案)   →   (家长/自学者视图)
                                        ↓
                                  难度适配(skill gating)
```

- **知识追踪**：第二代教育代理的学生模型是概率图/神经网络，输出"掌握某知识点的概率"，代表模型 IRT 与贝叶斯知识追踪 BKT（中文综述：https://www.hxedu.com.cn/hxeduRes/simplecharacter/1781586427753.pdf）。
- **完整开源参考**：`ai-engineering-from-scratch` 的 Personal AI Tutor capstone——"Bayesian knowledge tracing + Socratic policy scaffold"（https://github.com/stabuev/ai-engineering-from-scratch/blob/main/phases/19-capstone-projects/17-personal-ai-tutor/code/main.py，配套 skill 文档 https://raw.githubusercontent.com/rohitg00/ai-engineering-from-scratch/main/phases/19-capstone-projects/17-personal-ai-tutor/outputs/skill-ai-tutor.md）。
- **多 Agent LMS**：Lumina——BKT + DKT + 多 Agent 自适应学习管理系统（IJCA 论文 https://www.ijcaonline.org/archives/volume187/number122/srinivas-ijca-2026-a4901066e72b.pdf）。
- **强化学习个性化**：有研究用目标驱动 RL 奖励函数做个性化导学（https://svedbergopen.com/index.php/ijaiml/article/download/119/88 经 Google Scholar 转引）。
- **内容边界与引用**：与 04 篇墨刀 S1–S4 法同构——只依据当前课程资料回答、无依据时明说、引用可定位。这是教育产品可信度的生命线（也是用户 tutor 项目可以直接复用的 PRD 条款）。

### 3.2 工程要点（前 1% 实践者的迁移）

- 护栏写在系统层而非提示词层（Khanmigo 的"restraint"是产品设计原则，不是一句 system prompt）。
- 记忆文件化：学习者画像/错题本/掌握度都落成 Markdown/JSON，交给 agent 维护（01 篇 Anthropic agentic memory 模式）。
- 效果可度量：每个学习会话结束输出掌握度变化，家长/自学者可见——把"聊天记录"升级为"学习资产"。

## 四、机会点（针对用户的定位建议）

1. **缝隙 1：极窄人群。** 大厂做"全学科"，个人做"某一类人的某一件事"。候选：游戏开发者学英语/学 Unity/学发行——用户自己的转型路径就是垂直内容壁垒（对应 02 篇"绑定你独有的数据/场景/信任"）。
2. **缝隙 2：技能门槛缺失。** Khanmigo 被论文指出无 skill gating；个人 tutor 应从第一天就带 BKT 式掌握度建模，哪怕只是"每个知识点一个 0–1 掌握概率 + 错题驱动复习"的最小实现。
3. **缝隙 3：成人自学市场。** 国内大厂火力集中在 K12 与学习机；成人职业学习（科创、转行、考证）的深度陪伴 tutor 供给稀疏，且成人付费决策快、无政策风险。
4. **缝隙 4：内容资产化。** 把用户的学习过程本身变成产品的一部分（"我如何用 AI 学游戏发行"公开课 + tutor 工具），复用 02 篇一人公司飞轮：内容获客 → 工具变现 → 数据反哺。
5. **风险清单**：豆包等免费通用问答的价格压制（应对：护栏 + 学习者建模 + 垂直内容，这些通用聊天做不到）；教育行业政策敏感性（成人职业学习风险最低）；效果承诺的合规边界（只报告掌握度数据，不承诺提分）。

## 五、未尽事项（如实说明）

- Speak、Amira 的最新收入/用户数据未取得一手来源（仅获分类与定位信息），第二期补充。
- 国内"AI tutor"创业公司的融资明细（除幻课外）公开一手材料少，本篇以大厂与模式分析为主。
