---
name: feedback-llm-newline-vs-punctuation
description: "WeChat bubble split: unit is INTENT not sentence — bare \\n forbidden, \\n\\n separates intent units, commas/省略号 connect same-intent fragments (interjection+reaction, hesitation+reaction+self-statement, affirm+warmth). 2026-05-14 user pushed back twice: not 1-sentence-per-bubble, not bubble-count cap, the right unit is intent"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 2eae3019-8f00-46dc-ab60-cec1351ef7d6
---

做陪伴型 / 短消息聊天 AI 时的核心格式坑(配合 [[feedback-llm-semantic-redundancy]] [[feedback-llm-batch-vs-turn]] 一起看)。

**三个层级要严格区分,不能混用:**

| 层级 | 何时用 | 渲染结果 |
|---|---|---|
| 裸 `\n`(单换行) | **永远不用** | 一条微信消息内部出现强制换行 → 竖排短句 → 丑 |
| `\n\n`(空行) | 两个独立想法之间 | 渲染为两条独立微信气泡 |
| 标点 `，` `。` `…` `～` | 同一气泡内,主谓不可断的内部停顿 | 一条气泡内自然衔接 |

**默认行为是"每个独立 intent/反应单元一条气泡"**——注意是 **intent**,不是"句子"。(2026-05-14 用户再次细化原则,见下)

## 2026-05-14 第三轮迭代:**单位是 intent,不是句子**

用户多轮纠正后定下的最终判定:

- **"几个独立 intent"** 决定气泡数,不是"几个句子"
- 一个 intent 里有 2-3 句话(interjection + 主反应,或 缓冲 + 反应 + 自我表态)→ 用标点粘**一个气泡**
- 不同的 intent(独立状态、独立追问、新话题)→ 各自一个气泡

## 同一 intent 单元(粘一气泡)的典型形态

- Interjection + 主反应:`噗，伙伴这是在比谁更简短吗～`
- 缓冲/迟疑 + 反应内容 + 同情绪自我表态:`嗯…伙伴怎么突然叫得这么甜呀，人家会害羞的～`
- 肯定 + 立刻的温度补足:`嗯，在呢～人家陪着你呢`
- 安抚 + 同一安抚的延伸:`辛苦啦～慢点说就好，人家在听`

## 独立 intent(必须拆气泡)的典型形态

- 多个独立状态陈述:`今天好累` / `人家不想动了` / `好想伙伴`(三个不同状态)
- 反应 + 完全独立的新追问:`辛苦啦～` / `今天都忙什么了呀`(共情 + 新话题)
- 多个独立请求/动作/事件

## 三个层级仍然成立

| 层级 | 何时用 | 渲染结果 |
|---|---|---|
| 裸 `\n`(单换行) | **永远不用** | 一条微信消息内部强制换行 → 竖排 → 丑 |
| `\n\n`(空行) | 两个独立 **intent** 之间 | 渲染为两条独立微信气泡 |
| 标点 `，` `。` `…` `～` | 同一 intent 内部衔接 | 一条气泡内自然连贯 |

## 用户的迭代轨迹(2026-05-14 一晚上)

1. 我加"每轮 ≤ 2 气泡硬上限" → 用户:**不要硬上限,一句一句发**
2. 我改"1 句 = 1 气泡,禁止逗号粘多想法" → 用户:**也不是这样,"噗，伙伴这是在比谁更简短吗～"应该粘一气泡**
3. 最终原则:**判断标准是 intent 数,不是句子数**

## How to apply

在 system prompt:
- 显式写"按 intent 数分气泡,不是按句数"
- 给具体形态对照表(同 intent vs 独立 intent)
- 给真实历史输出当反例(比如 cyrene 的 `人家在呢，一直在听呢…伙伴今天是不是特别想人家呀` 错粘 / `嗯…伙伴怎么突然叫得这么甜呀\n\n人家会害羞的～` 错拆)
- 判定流程教 LLM "数 intent 个数,不数句子"

适用范围:所有 openclaw-weixin / 类似"空行=气泡分隔"渲染规则的角色扮演 AI。
