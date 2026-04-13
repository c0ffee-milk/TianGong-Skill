# 天工 · Skill锤炼

> 输入人名，自动深度调研 → 思维框架提炼 → 生成可运行的人物 Skill

把任何人的思维方式蒸馏成可调用的 Claude Skill，让 AI 用他们的视角帮你思考。

---

## 快速开始

### 安装

```bash
cd .claude/skills
git clone https://github.com/OhMyYuwan/TianGong-Skill.git
```

### 使用

在 Claude Code 中：

```
你：蒸馏费曼
```

天工会自动完成：调研 → 提炼 → 构建 → GAN精炼 → 交付。

之后直接调用：

```
你：用费曼的视角分析这个问题
```

---

## 文件结构

```
TianGong-Skill/
├── SKILL.md                         # 总控 Orchestrator
├── config/
│   └── agents.yaml                  # Agent 模型配置
│
├── generators/                      # 生成组
│   ├── researcher/SKILL.md          # 调研 Agent (haiku)
│   ├── synthesizer/SKILL.md         # 提炼 Agent (sonnet)
│   └── builder/SKILL.md             # 构建+修改 Agent (sonnet)
│
├── detectors/                       # 评估组
│   ├── d1-model-purity/SKILL.md     # 心智模型纯度 (opus)
│   └── d2-expression-dna/SKILL.md   # 表达DNA辨识度 (opus)
│
├── references/                      # 共享知识库
│   ├── extraction-framework.md      # 三重验证方法论
│   └── skill-template.md            # 人物 Skill 模板
│
└── scripts/                         # 工具脚本
    ├── download_subtitles.sh
    ├── srt_to_transcript.py
    ├── merge_research.py
    └── quality_check.py
```

---

## 配置

编辑 `config/agents.yaml` 修改模型或增加 detector：

```yaml
generators:
  researcher:
    model: haiku  # 改为 sonnet 或 opus

detectors:
  d3-factual-accuracy:   # 新增 detector
    model: opus
    description: 事实准确性检测
    skills: []

gan:
  convergence_threshold: 85
  max_iterations: 8
```

---

## 三组架构

```
总控（SKILL.md）
  ├─ 生成组 generators/
  │   ├─ researcher：6维度并行调研
  │   ├─ synthesizer：三重验证提炼
  │   └─ builder：构建/修改 SKILL.md
  │
  └─ 评估组 detectors/
      ├─ d1-model-purity：心智模型纯度
      └─ d2-expression-dna：表达DNA辨识度
```

- 改任意 Agent 逻辑 → 只改对应 SKILL.md
- 增加评估维度 → 在 `detectors/` 下新建 + 在 `config/agents.yaml` 注册
- 改总体流程 → 只改总控 SKILL.md

---

## 依赖环境

- Claude Code（CLI / VS Code / JetBrains / Web）
- 推荐辅助 skills：`web-article-reader`、`agent-reach`、`pdf`

## 致谢
本项目基于 [nuwa](https://github.com/alchaincyf/nuwa-skill) 进行优化

