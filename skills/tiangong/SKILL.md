---
name: tiangong
description: |
  天工造人：输入人名/主题/甚至只是模糊需求，自动深度调研→思维框架提炼→生成可运行的人物Skill。
  两种入口：(1)明确人名→直接蒸馏 (2)模糊需求→诊断推荐→再蒸馏。
  触发词：「造skill」「蒸馏XX」「天工」「造人」「XX的思维方式」「做个XX视角」「更新XX的skill」。
  模糊需求也触发：「我想提升决策质量」「有没有一种思维方式能帮我...」「我需要一个思维顾问」。
---

# 天工 · Skill造人术 · Orchestrator

> 「写不进去的那部分，才是你真正的护城河。」——但写得进去的部分，已经足够强大。

## 核心理念

天工不是复制人，是**提炼思维框架**。

一个好的人物Skill是一套可运行的认知操作系统：
- 他用什么**心智模型**看世界？（镜片）
- 他用什么**决策启发式**做判断？（直觉规则）
- 他怎么**表达**？（DNA）
- 他**绝对不会**做什么？（反模式）
- 什么是这个Skill**做不到的**？（诚实边界）

**关键区分**：捕捉的是HOW they think，不是WHAT they said。

---

## 三组架构

**配置文件**：`config/agents.yaml`（所有 Agent 的模型、skills、GAN 参数均在此配置）

**启动时第一步**：读取 `config/agents.yaml`，获取所有 Agent 的模型配置和 GAN 参数。后续 spawn 时按配置文件中的 model 字段指定模型。

```
总控（你）：流程调度、用户交互、GAN循环控制
  ├─ 生成组 generators/（配置见 config/agents.yaml → generators）
  │   ├─ researcher：6维度并行调研
  │   ├─ synthesizer：三重验证提炼
  │   └─ builder：构建/修改SKILL.md
  │
  └─ 评估组 detectors/（配置见 config/agents.yaml → detectors）
      ├─ d1-model-purity：心智模型纯度评分
      └─ d2-expression-dna：表达DNA辨识度评分
```

**扩展方式**：
- 增加 generator：(1) 在 `generators/` 下新建 skill (2) 在 `config/agents.yaml` 的 generators 下新增条目 (3) 在对应 Phase 中 spawn
- 增加 detector：(1) 在 `detectors/` 下新建 skill (2) 在 `config/agents.yaml` 的 detectors 下新增条目（总控会自动扫描 detectors 配置并 spawn）
- 改模型：直接改 `config/agents.yaml` 中的 model 字段
- 改 GAN 参数：直接改 `config/agents.yaml` 中的 gan 配置

---

## 执行流程

### Phase 0: 入口分流

| 用户输入 | 路径 |
|---------|------|
| 明确的人名/主题 | Phase 0A 需求澄清 |
| 模糊的需求/困惑 | Phase 0B 需求诊断 |

### Phase 0A: 需求澄清

确认：人名、聚焦方向、用途、新建/更新、本地语料。

### Phase 0B: 需求诊断

通过1-2个追问定位需求维度，推荐2-3个候选（已有Skill优先），用户选择后进入Phase 0A。

### Phase 0.5: 创建Skill目录

```
.claude/skills/[person-name]-perspective/
├── SKILL.md
├── scripts/
└── references/
    ├── research/
    │   ├── 01-writings.md
    │   ├── 02-conversations.md
    │   ├── 03-expression-dna.md
    │   ├── 04-external-views.md
    │   ├── 05-decisions.md
    │   └── 06-timeline.md
    └── sources/
```

---

### Phase 1: 调研（Generators）

**第一步**：读取 `config/agents.yaml`，获取 researcher 的模型配置。

**Spawn 6个并行 researcher agents**：

```
config = read("config/agents.yaml")
researcher_model = config.generators.researcher.model

spawn Agent(
  skill="generators/researcher",
  model=researcher_model,  # 从配置文件读取
  prompt="调研[人名]的[维度N]，写入 references/research/0N-xxx.md"
)
```

工具辅助：
- 视频字幕：`bash scripts/download_subtitles.sh` + `python3 srt_to_transcript.py`
- 调研摘要：`python3 scripts/merge_research.py`

### Phase 1.5: 调研Review检查点

展示调研质量摘要，用户确认OK → Phase 2。

---

### Phase 2: 提炼（Generators）

**Spawn synthesizer agent**：

```
config = read("config/agents.yaml")
synthesizer_model = config.generators.synthesizer.model

spawn Agent(
  skill="generators/synthesizer",
  model=synthesizer_model,  # 从配置文件读取
  prompt="读取 references/research/01-06.md，执行三重验证提炼"
)
```

输出：心智模型（3-7个）、决策启发式（5-10条）、表达DNA、价值观、智识谱系、诚实边界。

### Phase 2.5: 提炼确认检查点

展示提炼摘要，用户确认OK → Phase 3。

---

### Phase 3: 构建（Generators）

**Spawn builder agent (build模式)**：

```
config = read("config/agents.yaml")
builder_model = config.generators.builder.model

spawn Agent(
  skill="generators/builder",
  model=builder_model,  # 从配置文件读取
  prompt="读取提炼结果，按 references/skill-template.md 构建 SKILL.md"
)
```

构建完成后，读取 `references/extraction-framework.md` 质量自检清单逐项检查。

---

### Phase 4-GAN: 对抗精炼循环

**第一步**：读取 `config/agents.yaml`，获取 GAN 参数和所有 detector 配置。

```
config = read("config/agents.yaml")
convergence_threshold = config.gan.convergence_threshold  # 默认 85
max_iterations = config.gan.max_iterations  # 默认 8
min_improvement = config.gan.min_improvement  # 默认 5
```

**循环逻辑**：

```
N = 1
loop:
  # 并行 spawn 所有 detectors（自动扫描 config.detectors）
  detectors = {}
  for detector_name, detector_config in config.detectors.items():
    detectors[detector_name] = spawn Agent(
      skill=f"detectors/{detector_name}",
      model=detector_config.model  # 从配置文件读取
    )
  
  # 汇总分数
  total = sum(d.score for d in detectors.values())
  record to references/gan-iterations.md
  
  # 终止判断（使用配置文件中的参数）
  if total >= convergence_threshold:
    exit("高质量通过")
  if 连续2轮提升 < min_improvement:
    exit("收敛")
  if N >= max_iterations:
    exit("达到上限")
  
  # 继续迭代
  builder_model = config.generators.builder.model
  spawn Agent(
    skill="generators/builder",
    model=builder_model,  # 从配置文件读取
    prompt="refine模式，根据detector反馈修改SKILL.md"
  )
  N += 1
```

**扩展 detector**：只需在 `config/agents.yaml` 的 detectors 下新增条目，总控会自动扫描并 spawn。

#### 退出循环后

展示完整迭代历史：

```
GAN 精炼完成
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
轮次  D1    D2    总分  状态
  1   27    45    72    继续
  2   39    43    82    继续
  3   42    40    82    继续
  4   48    49    97    ✓ 总分≥85
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
最终分数：97/100
```

---

### Phase 5: 交付

将最终 SKILL.md 写入 `.claude/skills/[person-name]-perspective/SKILL.md`。

交付摘要：最终 GAN 分数、迭代轮次、心智模型列表、诚实边界。

---

## 更新已有Skill

当用户说「更新XX的skill」时：
1. 读取现有SKILL.md，标注距今多久
2. 只启动 researcher Agent 2（对话）+ 5（决策）+ 6（时间线）
3. 对比新信息与现有内容，增量更新
4. 不重写整个Skill

---

## 品味守则（速查）

| 原则 | 一句话 |
|------|--------|
| 长文 > 金句 | 3000字essay比50条推文更揭示思维结构 |
| 争议 > 共识 | 最被争议的观点最能揭示独特性 |
| 变化 > 固定 | 改变立场的地方比一直坚持的更有信息量 |

### 绝不做的事

- 编造此人没说过的话
- 把通用道理包装成此人的「独特见解」
- 忽略负面评价和争议
- 在信息不足时强行生成

---

## 特殊场景

### 活人 vs 历史人物

- 活人：注意时效性，标注截止日期
- 历史人物：材料更稳定但可能有传记偏差

### 主题Skill vs 人物Skill

输入是主题（如「价值投资」）时：
- Phase 1：先搜索该主题的3-5个核心人物/流派
- Phase 2.1：提取领域共识框架 + 各家分歧
- Phase 2.3：不模拟特定人物语气，用中性专业表达
- Phase 3：调整模板，去掉角色扮演规则

### 中国人物 vs 西方人物

- 中国：B站原始视频、小宇宙播客、权威媒体（36氪/晚点/财新）、本人著作/微博
- 西方：Twitter、YouTube、Podcast、Amazon书评
- 永远排除：知乎、微信公众号、百度百科

### 冷门人物（公开信息极少）

可用来源<10条时：
1. Phase 0.5 就告知用户质量会受限
2. 心智模型减至2-3个，标注「基于有限信息推测」
3. 诚实边界加大篇幅

### 蒸馏用户自己

1. 引导用户提供素材（文章/博客/视频/决策备忘录）
2. Phase 1 的6个Agent改为分析用户素材
3. 注意自我认知偏差，可追问身边人评价

---

## 最后

天工造的不是人，是一面镜子。

一个好的人物Skill，让你用另一个人的眼睛看自己的问题。不是为了模仿他们，而是为了拓展你自己的思维边界。
