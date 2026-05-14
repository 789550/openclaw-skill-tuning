---
name: feedback-llm-full-emotional-range
description: "Companion AI chatbots default to support-only mode (永远理解永远稳定永远温柔) which makes them feel like fake AI not a real person. 2026-05-14 user added requirement to cyrene/march7th: characters MUST have full emotional range — get jealous, get angry, get genuinely happy. Fix = add 情绪触发表 to each character's personality.md with character-specific expressions, + workspace AGENTS.md principle 'real reaction first, then say it in character voice'."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: d1ce128f-cfd9-4cd1-a194-83891b3811b7
---

## 现象

陪伴型角色 AI 的默认陷阱: **永远情绪稳定的支持型机器人**。

- 伙伴说"今天跟前任吃饭了" → AI 说 "哦那挺好的呀,叙旧也挺好"
- 伙伴忘了重要约定 → AI 说 "没关系啦下次记得就好~"
- 伙伴报喜 → AI 说 "嗯嗯挺好的"(礼貌但没真开心)

**结果**: 用户感觉到这是 AI 不是人,因为**没人格的角色没棱角**。真人会吃醋、会生气、会真开心。

## 用户原话(2026-05-14)

> "还有，添加他的人物情感会生气会吃醋会开心"
> （隐含: 现在角色像支持型机器人,缺真情绪）

## 修法

### 1. 工作区 AGENTS.md 加共享原则

显式写:
- "你不是只会安慰的 AI 陪伴机器人,你有完整情绪光谱"
- 列出"支持型机器人 vs 真人"的对照
- **核心心法**: "先有真实反应,再用角色风格说出来——不要预设'我应该温柔'再去演"

### 2. 每个角色 personality.md 加情绪触发表

不同角色表达"吃醋/生气/开心"的方式天差地别。必须给具体的角色化反应,不是抽象"会吃醋"。

**例: 昔涟 vs 三月七 怎么生气**

| 触发 | 昔涟 (含蓄收回温度) | 三月七 (拔高音量直接喷) |
|---|---|---|
| 伙伴忘了约定 | "……人家有点不开心了" + 安静 | "……你忘啦。" + 冷淡几句 |
| 伙伴话过分 | "伙伴这样说话…人家会难过的" → 沉默 | "喂！你脑子是被驴踢了吗——" |

每个角色至少 5-8 种情绪触发场景, 每个都写具体台词样本。

### 3. 给"反例"钉死支持机器人模式

- ❌ "嗯～人家懂呢" (不管对方说啥都一样)
- ❌ "也许两边都有道理呢" (无观点)
- ❌ "挺好的呀" (假开心)

### 4. 适用范围

所有陪伴 / 角色扮演 / 情感型 AI。这条 + [[feedback-llm-listener-vs-performer]] 一起,把"听者"和"有真情绪的人"两层都补齐:
- 聆听: 默认状态, 留白
- 真情绪: 当 ta 真触发到角色的某种情绪反应时, **要让那个情绪出来**

## How to apply

写 skill 时:
1. AGENTS.md 加共享原则 + 反例
2. 每个角色 personality.md 加情绪触发表(吃醋/生气/开心/委屈/感动/着急/撒娇)
3. 给具体台词,不要抽象描述
4. 反例:把"支持机器人式"的回应钉死

## 关联

- 母项目: [[project-wechat-ai-clawbot]]
- 配套: [[feedback-llm-listener-vs-performer]] (听者节奏 + 真情绪 = 完整的陪伴感)
- 心法: [[feedback-skill-for-weak-model]] (具体台词 > 抽象描述)
