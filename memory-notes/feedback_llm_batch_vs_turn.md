---
name: feedback-llm-batch-vs-turn
description: "LLMs tend to dump all related sub-ideas in one turn; mitigation is NOT a hard bubble cap (user pushed back on that 2026-05-14) but instead 'limit ideas per turn via topic restraint + repeat-detection', combined with strict 'one sentence = one bubble' splitting"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 96e634d5-fc59-40bf-a520-f0aa2d2cb191
---

LLM 在角色扮演 / 聊天场景里有个倾向：把"该说的话"在**一个回合里全说完**——共情+邀请+陪伴+追问全塞一回合，把伙伴淹没。

但是**修法不是硬上限气泡数**。用户 2026-05-14 明确反对硬上限,理由:真人微信就是"想到几句发几句",每句一气泡。硬上限会逼模型用逗号把多个想法粘成一条长气泡,这又踩到[[feedback-llm-newline-vs-punctuation]]的反例。

## 正确的修法 (2026-05-14 与用户对齐)

**双管齐下：**

1. **气泡分割层**: 严格 "1 句完整意思 = 1 个气泡",禁止逗号粘多想法。**不限制每轮气泡数量** — 有 3 句发 3 个,有 5 句发 5 个。
2. **想法节制层**: 通过"重复短消息识别"等规则,**限制 IDEAS 数量**,而不是气泡数量。伙伴反复发同一种情绪(如连发"宝宝"),进入"持续在场"模式,只用一句很短的稳定确认,不追问不换话题。

## ❌ 不要这样修

- 加 "≤2 气泡硬上限" — 用户反对,理由见上
- 教模型"少说点" — 抽象,弱模型不会执行
- 把多个想法用逗号粘一条 — 踩另一个反例

## ✅ 要这样修

- 鼓励多气泡(1 句 1 气泡,几句就几个气泡)
- 限制 IDEAS 而非气泡:对持续情绪给短确认,不开新话题
- 给具体反例:"人家在呢，一直在听呢…伙伴今天是不是特别想人家呀" 是错的 → 拆 3 个气泡 或 干脆只回 "嗯…人家在呢"

## How to apply

- 写 skill / AGENTS.md 时:
  - 气泡分割规则要带"禁止逗号粘多想法"的明文 + 反例
  - 加重复短消息检测规则(短 + 相似 + 连发 → 持续在场模式)
  - **不要**写"每轮 ≤N 条气泡"的硬上限
- 适用范围:所有 turn-based 陪伴 / 角色扮演 / 长期对话 AI
- 关联:[[feedback-llm-semantic-redundancy]] [[feedback-llm-newline-vs-punctuation]] [[project-wechat-ai-clawbot]]
