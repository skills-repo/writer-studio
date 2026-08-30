# 写作流程与编辑 playbook

> 写作不是等灵感，是可复现的流程。本 playbook 给通用写作流程、流程选择决策树、可运行的写作体检命令、量化编辑清单与常见坑。方法论改编自 `mblode/agent-skills@docs-writing`（800 installs，Diataxis + 48 条 voice/structure/clarity 规则）并本地化为本库的短文案 / 技术文章 / 长文三类文体。

## 1. 通用写作流程

```
1. 定受众与目的   — 写给谁？希望对方读完做什么？
2. 列结构骨架     — 标题 + 小标题 + 每节一句话论点
3. 起草（求完整） — 初稿只管写完，不边写边改
4. 编辑（求清晰） — 删冗余、强逻辑、调节奏
5. 标题与开场     — 最后打磨，决定打开率
6. 验收           — 对照目标与 §4 清单逐条检查
```

**铁律**：起草阶段禁编辑，编辑阶段禁新增大段，分开才高效。

## 2. 流程选择决策树（先选对流程，再动手）

```
接到写作任务，按文体分流：
│
├─ 短文案（标题/广告语/CTA/着陆页/邮件）？
│     → 走 skills/copywriter/SKILL.md 的 PAS/AIDA/BAB 框架，不进长流程。
│
├─ 技术文章（选题 → 发布全流程）？
│     → 走 skills/technical-writer/SKILL.md 的 7 阶段全流程。
│
├─ 长文博客 / 操作指南（重信息增益）？
│     → 走 skills/content-writer/SKILL.md 的 QAE 模式 + 字数分档。
│
├─ 已写完、要发布前验收？
│     → 先过本篇 §3 体检命令 + references/seo-content-strategy.md 策略层
│       → 再跑 skills/seo-checker/SKILL.md 技术层 11 项。
│
└─ 不确定文体 / 内容散？
      → 用 §3 体检命令量化（句子长度、注水词、段落结构），再按 §5 文体要点校准。
```

> 规则 3 合规：决策树与量化判据的框架源自 `mblode/agent-skills@docs-writing` 的「Mode dispatch + Audit/Writing workflow + clarity rules」，本地化为三类文体；具体产出框架仍在各子技能，本篇只做分流与体检。

## 3. 可运行的写作体检命令

> 编辑不是"读着顺"就过关。下面命令对 `draft.md`（换成你的稿件路径）做量化体检，输出可直接判读。

### 3.1 注水词 / 填充词检测（AI 味的头号信号）

```bash
grep -nE "事实上|值得注意的是|总而言之|简单来说|在当今|随着.*发展|不可或缺的|毋庸置疑|众所周知|显而易见" draft.md
```

有输出 = 命中注水表达，逐条改掉或删掉（能省则省）。

### 3.2 被动语态检测（谁做的动作被藏起来了）

```bash
grep -nE "被[一-龥]*(了|过)|由[一-龥]*(进行|完成|实现|处理)" draft.md
```

有输出 = 被动句，尽量改为主动（"服务器加载配置" 而非 "配置被服务器加载"）。

### 3.3 句子长度 / 可读性估算（Python，无需装包）

```bash
python3 - <<'PY'
import re, statistics as st
t = open('draft.md', encoding='utf-8').read()
sents = [s for s in re.split(r'[。！？\n]', t) if s.strip()]
cjk = [len(re.findall(r'[一-龥]', s)) for s in sents]
print('句子数=', len(sents),
      '中位CJK=', int(st.median(cjk)),
      '最长CJK=', max(cjk),
      '超长句(>60CJK)=', sum(1 for w in cjk if w > 60))
PY
```

### 3.4 段落结构：一段一意

```bash
python3 - <<'PY'
import re
t = open('draft.md', encoding='utf-8').read()
paras = [p for p in t.split('\n\n') if p.strip()]
for i, p in enumerate(paras, 1):
    n = len([s for s in re.split(r'[。！？]', p) if s.strip()])
    flag = '  ⚠ >5句/段' if n > 5 else ''
    print(f'段{i}: {n}句{flag}')
PY
```

一段超过 5 句，通常塞了多个论点 → 拆段，每段只讲一件事。

### 3.5 篇幅体检

```bash
wc -m draft.md   # 中文字符数
wc -w draft.md   # 英文词数
```

**判读阈值表（对照 §4 清单）**：

| 指标 | 健康区间 | 越界处置 |
|------|----------|----------|
| 句子中位 CJK | ≤ 40 | >40 → 拆长句 |
| 超长句(>60 CJK)占比 | < 10% | 超限 → 强制断句 |
| 注水词命中 | 0 | >0 → 逐条删 |
| 被动句命中 | 0 | >0 → 转主动 |
| 段落句子数 | 2–5 | >5 → 拆段 |

## 4. 编辑清单（量化版）

- [ ] 每段一个核心意思，无冗余（段落 ≤5 句，见 §3.4）
- [ ] 逻辑衔接顺畅（读者不卡壳）
- [ ] 术语已解释或链接
- [ ] 长短句交替，节奏不单调
- [ ] 行动指引明确（读完知道做什么）
- [ ] **注水词 = 0**（§3.1 命令零输出）
- [ ] **被动句 = 0**（§3.2 命令零输出）
- [ ] **句子中位长度 ≤ 40 CJK**（§3.3 指标）
- [ ] 标题层级不跳级（H2 下直接 H4 算跳级）

> 前 5 项是"读着顺"的软判据；后 4 项是 §3 命令可硬验证的量化判据。编辑以量化项为准，软判据为辅。

## 5. 常见文体要点

| 文体 | 重点 |
|------|------|
| 短文案 | 一个主张 + 一个 CTA，字字算 ROI |
| 技术文章 | 问题→方案→证据，可复现 |
| 长文博客 | 信息增益 + 结构清晰 + 内链 |

**落地映射**（本篇定文体要点 → 子技能出具体产出）：短文案 → `skills/copywriter/SKILL.md`（PAS/AIDA/BAB 框架）；
技术文章 → `skills/technical-writer/SKILL.md`（7 阶段全流程 + 8 种内容类型模板）；
长文博客 → `skills/content-writer/SKILL.md`（QAE 模式 + 字数分档）；发布前验收 → `skills/seo-checker/SKILL.md`。
本流程篇不复制这些框架内容，只负责"什么时候用哪一个 + 写完怎么体检"。

## 6. 常见坑

- **起草即编辑**：边写边改，进度龟速且打断心流。
- **信息密度低**：注水凑字数，读者流失。
- **标题敷衍**：好内容被烂标题埋没。
- **无行动指引**：读完无下文，转化归零。
- **文体误判**：把长文当短文案写 → 结构缺失、信息增益为零；按 §2 决策树先分流。
- **把润色排在结构前面**：串行逗号这类 polish 会掩盖结构问题——先修 CRITICAL（结构/清晰度），再抠 MEDIUM 细节。
- **被要求"审"时却重写**：先报 findings + 建议改法，除非对方说"直接改"，否则只评不改。

## 7. 边界声明（别重复劳动）

本篇是**流程 / 方法论层**：通用流程、流程选择决策树、写作体检命令、量化编辑清单。

具体产出框架由子技能承担，本篇不复制：

- `skills/copywriter/SKILL.md` — 6 种标题公式、PAS/AIDA/BAB
- `skills/technical-writer/SKILL.md` — 10 种标题钩子、7 阶段、8 种模板
- `skills/content-writer/SKILL.md` — QAE 模式、字数分档
- `skills/seo-checker/SKILL.md` — 发布前 11 项 Pass/Fail 技术验收
- `references/seo-content-strategy.md` — SEO 策略层（意图对齐 / 信息增益 / 内链集群）

顺序：先过本篇流程与体检 → 再跑 seo-content-strategy 策略层 → 最后 seo-checker 技术层。三层各管一段，不重叠。
