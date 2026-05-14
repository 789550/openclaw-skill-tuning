---
name: feedback-skill-template-vs-judgment
description: "Tension in weak-model skill design: too-explicit templates produce monotony (every reply opens with same word), but too-loose principles produce hallucinated drift. The 2026-05-14 cyrene fix = describe PRINCIPLE + give VARIED examples + add explicit 'don't repeat openings' anti-pattern + invoke 'as if a Claude model reads this naturally'. Keep STRUCTURAL rules hard (intent unit, no bare \\n), keep WORDING soft."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d1ce128f-cfd9-4cd1-a194-83891b3811b7
---

写给弱模型(DeepSeek/Haiku)的 skill 时,有一个反向陷阱:

## 陷阱

为了让弱模型听话,写**具体模板**列出多种回答模板让它"轮替"。结果:
- 模型不"轮替",直接每次抓**第一个模板**
- 所有回复**同一个开头词**(比如全部"嗯…")
- 看起来执行"对"了规则,但读起来像查表机器人

## 用户原话(2026-05-14)

> "他每一次叫宝宝的时候第一句回复都是一样的,这一点改一下"
> "适当放宽代码,按照你的判断让他像人一样去思考回复,在必要的时候去代码限制他,以你的模型为基准"

关键短语:**"以你的模型为基准"** —— 用户希望写出来的 skill,以 Claude(我)读起来自然的标准为基线,而不是"DeepSeek 安全套话"的标准。

## 解法

不是放弃 explicit, 也不是回到抽象原则。是**分层**:

### 硬规则(必须明文+反例,弱模型才执行得对)

- 气泡分割: 按 intent 数,1 个 intent = 1 气泡
- 禁止裸 `\n`
- 重复短消息识别(触发判定 3 条全满足才进入持续在场模式)
- 不追问/不换新话题/不引入新信息(在持续在场模式下)

### 软规则(给原则 + 多样化例子,触发模型自己变化)

- 开头**变化**: 不要每次都用同一个词开头(列出反例:全是"嗯/在呢"开头是错的,给一组不同开头风格的样本但**注明"这只是示例,不是模板,执行时按当下感觉变"**)
- 情绪温度的**范围**而不是清单(陪伴/撒娇/反向撒娇/动作描写/戏谑/温柔关心)
- 心态指令: "想象现实里你恋人这么连续叫你,你会怎么回?然后用角色风格说出来"

### 显式提示模型

在 skill 文里直接写: **"以 Claude 模型读起来自然的标准为基准——不是 DeepSeek 默认那种'安全套话'"**。这给了模型一个具体的质量锚点。

## 验证结果(2026-05-14 实测)

8 次连发"宝宝",cyrene 给出 8 个**不同开头**:
- 嗯…
- 乖～
- 干嘛呀
- （轻轻挨近）
- 哎呀
- （软软挨着你）
- （忍不住轻轻笑出来）
- （轻轻把头靠在你肩上）

零雷同。同时硬规则(1 intent 1 气泡 / 不追问 / 肯定+情绪)100% 执行。

## How to apply

写 skill 时区分两类规则:

1. **结构性规则**(几个气泡、什么时候触发什么模式) → 用决策树 + 反例 + 计数判定流程([[feedback-skill-for-weak-model]] 心法)
2. **风格性规则**(用什么词、什么语气、什么节奏) → 给原则 + 范围举例 + 反例(尤其雷同反例) + "以 Claude 标准"锚点

不要把风格性规则也做成查表模板,会自废武功成机器人。

## 关联

- 母心法: [[feedback-skill-for-weak-model]] (explicit > implicit)
- 这条是它的补充: explicit on 结构, soft on 风格
- 项目背景: [[project-wechat-ai-clawbot]]
- 配套失败模式: [[feedback-llm-batch-vs-turn]] [[feedback-llm-newline-vs-punctuation]]
