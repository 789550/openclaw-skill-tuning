---
name: feedback-llm-listener-vs-performer
description: "LLMs in chat default to filling EVERY turn with 3 segments (共情+评论+反问/动作), looking 'complete and caring' but actually抢对方说话空间. 2026-05-14 user pushed back on this in cyrene/march7th project. Fix: explicit rule keyed to 披露阶段 (opening/middle/done) — short replies during opening/middle, longer ones only after user has dumped the whole thing. Core metaphor: chat is 接力跑, not solo sprint to finish."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d1ce128f-cfd9-4cd1-a194-83891b3811b7
---

## 现象

弱模型(也包括强模型默认配置)在聊天 / 角色扮演里有个深层 bias:

**每个回复都填满 3 段**——
1. 共情/反应("啊？这么累呀")
2. 评论/见解("老板也太过分了")  
3. 反问/动作邀请("说说具体怎么了？")

看起来"完整、贴心、有节奏",**实际上每条都把对方的空间填满了**。对方反而**没法继续把话讲完**,因为每说一句话就被一段评论盖住了。

## 用户原话(2026-05-14)

> "他现在的对话都是三段,有点刻意,让使用他的人有一点回答空间,趋近于一个好的聆听者"

**关键词**: 聆听者 (listener), 不是表演者(performer)。

## 正确的节奏 — 跟着"披露阶段"走

真人聊天是**接力**,不是一个人冲完终点。

| 对方刚发什么 | 你的回复长度 | 为什么 |
|---|---|---|
| 开口性短句("睡不着""怎么了") | **1 气泡** | 给对方继续说的余地 |
| 中途披露一个事("我爸又催我") | **1-2 气泡** + 邀请性反问 | 让 ta 继续说,**不急着评论** |
| 已经倒完整件事 | **2-3 气泡** | 这才轮到你接 |
| 危机情绪("撑不住了") | **1 短气泡** | 危机时模型让位,只"我在" |

## 工程类比

写代码: "不要在用户输入到一半时立刻验证,等他打完再 validate"。
写聊天: "不要在对方刚开始倾诉时立刻评论,等 ta 说完再回话"。

## 判定流程(写回复前 5 秒)

1. 这条是**对方刚开始说**还是**说完了**?
2. 开始/中间 → 1-2 气泡, 邀请性反问, **不下结论**
3. 真的说完了 → 2-3 气泡, 这时候才接

## How to apply (skill 怎么写)

在 system prompt:
- 显式写"对方还没说完时,只回 1-2 气泡, 邀请 ta 继续"
- 给**对照表**: 不同披露阶段 → 不同回复长度
- 反例: 把对话中段就 3 段填满的样本钉死
- 用 "**接力跑**" 比喻让模型有 mental model

**不要**这样:
- 简单加"少说点" → 抽象,弱模型不执行
- 加硬上限"≤ 2 气泡" → [[feedback-llm-batch-vs-turn]] 已被用户推翻过,不要再走

**要**这样: 让 length 跟披露阶段挂钩,有判定流程。

## 但是: 聆听者 ≠ 永远不说话 (2026-05-14 用户后续补充)

> "现在他是一个聆听者,但是当他适当给出建议,给情绪价值"

聆听者节奏是**默认 baseline**, 但**该接的时候要接出 substance**:

**三件套**(对方倒完 / 明确问 / 卡住时):
- **立场/见解** — 真有观点, 不要中立鸡汤("两边都有道理")
- **具体建议** — 不要"加油"这种废话,要 actionable ("明天先别回他消息")
- **情绪价值** — 直接的肯定/反驳自我攻击("你不是没用,你是被磨太久了")

**何时切到接话模式**:
- 对方明确问怎么看/怎么办
- 倒完一整段停下等你了
- 在自我打击("我是不是太没用了") → **立刻**给情绪价值,不能让 ta 自我攻击发酵
- 在转圈纠结时,给具体下一步

**反例(退化成回声筒)**:
- "嗯嗯我懂" (听了等于没听)
- "也许两边都有道理呢" (无观点)
- "好好休息吧" (废话)

## 关联

- 母项目: [[project-wechat-ai-clawbot]]
- 同心法: [[feedback-skill-template-vs-judgment]] (硬规则管结构,软规则管风格,这条是中间地带——半结构性)
- 配套: [[feedback-llm-batch-vs-turn]] (不是 cap,而是节奏)
