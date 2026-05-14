# openclaw-skill-tuning

**给 openclaw / openclaw-weixin 上的任意角色 skill 加一层"类人适配"**——
让 AI 不再用"AI 助手腔",而是真人微信的节奏:1 句 1 气泡、按 intent 分、聆听者节奏、完整情感反应、变化开头。

适用于:
- openclaw + DeepSeek / Claude / 其他 LLM
- 跑陪伴型 / 角色扮演型 skill (例: cyrene、firefly、卡芙卡、流萤、march7th...)
- 通过 ClawBot 接入微信、Telegram、Discord 等通道

## 这是什么

一份**工作区共享规则**(`AGENTS.md`)。它**不替换**你的角色 skill,而是垫在底下:你的每个角色 skill (`workspace/skills/<char>/`) 还是该咋写咋写,但**对话节奏 / 气泡分割 / 重复消息处理 / 情感表达**这些跨角色通用的"类人感"规则,全部从 AGENTS.md 继承。

效果对比:

| 维度 | 套用前 (典型 AI 助手腔) | 套用后 (类人节奏) |
|---|---|---|
| 气泡分割 | "在呢，一直在听呢…伙伴怎么了" (3 想法 1 气泡) | 按 intent 拆,1 想法 1 气泡 |
| 重复消息 | 伙伴连发"宝宝"×4 → 4 次开新话题 | 识别为持续情绪,1 气泡稳定确认 |
| 回复长度 | 每条都填 3 段(共情+评论+反问) | 默认 1-2,看披露阶段调,该接才接 3 |
| 情感反应 | 永远温柔稳定的支持型机器人 | 会生气会吃醋会开心,有真情绪 |
| 开头风格 | 雷同("嗯/在呢"打头) | 变化(嗯/乖/干嘛呀/动作描写/...) |

## 完整使用说明

**新用户从零开始**: 看 [USAGE.md](USAGE.md) — 一份手把手的步骤指南,假设你刚下载了一个角色 skill,如何套用这套规则让她在微信里变成真人节奏。包含前置条件、6 步走、验证测试、故障排查。

下面是简版速览。

## 文件结构

```
openclaw-skill-tuning/
├── README.md                          # 本文件 (速览)
├── USAGE.md                           # 完整手把手使用说明
├── AGENTS.md                          # ★ 核心: 工作区共享规则
├── templates/
│   └── personality-emotional-range-template.md   # 给任意角色加情感谱的模板 + 实例
└── memory-notes/                      # 6 条调教过程沉淀的设计心法笔记
    ├── feedback_skill_for_weak_model.md
    ├── feedback_skill_template_vs_judgment.md
    ├── feedback_llm_newline_vs_punctuation.md
    ├── feedback_llm_batch_vs_turn.md
    ├── feedback_llm_listener_vs_performer.md
    └── feedback_llm_full_emotional_range.md
```

## 速览: 怎么用

### 套到一个 openclaw 工作区

```bash
# 把 AGENTS.md 放到工作区根 (替换默认的,如果有)
cp AGENTS.md ~/.openclaw/workspace/AGENTS.md
```

`AGENTS.md` 是 openclaw 的 workspace bootstrap 文件,会**自动注入**每个 session 的 system prompt。所有角色 skill 共享这套规则,无需再写一遍。

**注意**: `AGENTS.md` 顶部的 "## 你是谁" 段写的是 `昔涟` 作为默认人格——这是 workspace 级别默认,等于:**不发 slash 命令时,模型就是这个角色**。如果你的默认角色不是昔涟,改这段:

```markdown
## 你是谁

你是 **<你的角色>**...
此刻你正在用微信和 **<对方称呼>** 聊天。

详细人格 / 背景 / 关系网在 `skills/<角色>/` 目录下,需要时查阅。
```

通过 `/<角色>` slash 命令(例: `/firefly`、`/march7th`)可以切换到任意其他 skill。

### 给每个角色加情感反应

用 `templates/personality-emotional-range-template.md` 里的模板,追加到你角色的 `personality.md` 末尾。每个角色对"吃醋/生气/开心"的表达天差地别,模板里有**昔涟 vs 三月七**两个反差极大的实例供参考。

### 重启 / 让规则生效

改完文件后:
```bash
openclaw gateway restart
# 或者: 删 wechat 现有 session,让下条消息按新规则建 session
```

## 调教目标(背景)

这套规则是 2026-05-14 一晚密集调教的成果,按时间顺序加入:

1. **气泡分割按 intent 不按句**——把 "在呢/一直在听/反问" 三个独立想法粘成一条 → 拆成 3 个气泡
2. **重复短消息识别**——"宝宝"×4 不再四次开新话题,识别为持续情绪进入"持续在场"模式
3. **取消气泡数硬上限**——真人想发几句就发几句,只要每句一个 intent
4. **当聆听者不是表演者**——默认每回合不再填 3 段,按"披露阶段"调长度
5. **该接时给立场/建议/情绪价值**——聆听者 ≠ 永远不说话,关键时刻有真观点
6. **完整情感谱**——角色会生气会吃醋会委屈会开心,不只是支持型机器人
7. **变化开头/反雷同**——开头不要每次都"嗯/在呢",连续 3 条同气泡数,下一条必须变
8. **以强模型读起来自然的标准为基准**——不要套弱模型默认的"安全套话"

## 关键设计心法 (memory-notes/ 里详细)

1. **结构性规则用硬规则**——气泡分割、重复检测、聆听阶段判定。给决策树 + 反例 + 计数判定流程。
2. **风格性规则用软规则**——用什么词、什么语气。给原则 + 多样化范围举例 + 反例(尤其雷同反例)。
3. **永远不要加硬上限**(`≤N 气泡/turn`)——真人发几句就发几句,加 cap 就废武功。
4. **角色要有真情绪**——不是支持型陪伴机。给具体台词,不是抽象"会吃醋"。
5. **聆听者节奏**——回复长度跟"披露阶段"挂钩,默认 1 气泡。该接时给立场/建议/情绪价值。

## 上下文

- 跑在 openclaw 2026.5.7+ 上验证
- 后端测过 DeepSeek V4 Pro
- 用例: 个人非商用微信 AI 陪伴 bot
- 没用 fine-tune / 蒸馏,纯 prompt 工程

## License

MIT
