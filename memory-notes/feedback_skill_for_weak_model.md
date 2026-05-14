---
name: feedback-skill-for-weak-model
description: "When designing skills/prompts to be EXECUTED by a weak model (DeepSeek/Haiku) while AUTHORED by a strong model (Claude), the design discipline inverts: explicit > implicit, examples > descriptions, decision trees > judgment, anti-patterns > positive-only"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2eae3019-8f00-46dc-ab60-cec1351ef7d6
---

**核心心法**:写 skill 时假设执行者是个**只会照字面查表的一根筋**,不是能领悟意图的同行。

## 设计原则对照

| 给 Claude 写的 skill | 给 DeepSeek/Haiku 等弱模型写的 skill |
|---|---|
| 描述性原则:"保持温柔" | 明文规则:"禁止使用'我命令你'等强硬语气;每条消息必须包含 1-3 个语气助词(呀/呢/嗯/哎呀/~)" |
| "用你的判断" | "按下表 1-5 条逐项核对" |
| "避免出戏" | 列出出戏的 10 种典型表现 + 每种的回应模板 |
| 100 字 prompt | 可能要 5000 字,把所有 corner case 列出来 |
| 抽象原则 | 具体场景的 if-then 表 |
| 短规则,模型自己补全 | **同一条规则在多处冗余强调** |
| 给目标 | 给目标 + 反例 + 决策步骤 |

## 工程类比

"写代码要假设接手维护的人是个连环杀手,而且知道你家住哪。"
**写 skill 要假设执行的模型每次都用最字面、最不动脑的方式理解你写的每个字。**

## **Why:**

用户在 [[project-wechat-ai-clawbot]] 项目里反复跑通这个洞察。
- 起初我帮他写的 cyrene skill 全是 Claude 视角的"描述性人设"——Claude 跑没问题,切到 DeepSeek 就到处出戏
- 用户明确指出:"你模型能力太强大了,DeepSeek 做不到"
- 然后:"我要让 Claude 写 skill,然后用低端模型跑"
- → 设计目标从"写 prompt 给 LLM 用" 翻转成 "写**给弱模型读得懂的明文指令集**"

这次会话里我们做的所有改动都是这个原则的实例:
- AGENTS.md 的"气泡分割"从一行扩成带正反例 + 决策流程的整页 → [[feedback-llm-newline-vs-punctuation]]
- 三条 LLM 角色扮演失败模式被明文化 → 失败模式手册三件套

## How to apply

**给低端模型写 skill 时,逐条检查:**

1. **形容词审计**:扫一遍 skill,凡是用形容词描述行为("温柔""克制""自然")的地方,**全部换成可量化的规则**(字数 / 句末标点 / 词汇白黑名单 / if-then)
2. **抽象指令审计**:"按情况判断""灵活处理""根据语境"这种话,**全部改写成具体步骤**或者用判定表替换
3. **反例必给**:除了说"该做 X",必须配"不要做 Y(典型错误样本)"
4. **关键规则冗余**:重要的规则在不同文件、不同章节**重复至少 2 次**,因为弱模型注意力不稳
5. **决策流程**:复杂判断改成"问自己 1, 2, 3"的流程,而不是依赖模型综合判断
6. **失败模式分类化**:积累每种失败模式 → 加进 "禁止 X" 规则列表

**审计现有 skill 的工作流**:

- 用 Claude(或更强模型)读所有文件,**专门挑"对弱模型不友好"的句子**
- 每条问题给出:原文 + 为什么弱模型看不懂 + 具体改写建议
- 用户审过再统一动文件

## 关联

- 母项目:[[project-wechat-ai-clawbot]]
- 失败模式三件套是这个心法的实例化:[[feedback-llm-semantic-redundancy]] [[feedback-llm-batch-vs-turn]] [[feedback-llm-newline-vs-punctuation]]
