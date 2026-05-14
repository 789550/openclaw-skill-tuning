---
name: project-openclaw-weixin-bubble-delivery
description: "openclaw-weixin 插件的 sendWeixinOutbound 不识别 \\n\\n 作为消息分隔符——把整段含\\n\\n的 text 当一条微信消息发出,WeChat 客户端把 \\n\\n 渲染成消息内换行(竖排),不是独立气泡。模型层的多气泡分割技术上正确,但视觉效果在微信端丢失。2026-05-14 试过 channel.js patch 但因模型卡 tool_call 没验证成功,已回退。两条解法路径: (1) message_tool 模式 = config + 模型 prompt 改造; (2) 改插件 channel.js 直接拆 \\n\\n。"
metadata: 
  node_type: memory
  type: project
  originSessionId: d1ce128f-cfd9-4cd1-a194-83891b3811b7
---

## 现象

模型输出 `bubble1\n\n bubble2` (按 AGENTS.md 规则正确分气泡)。在 TUI 上看是 2 个气泡,但在微信端看是**一条消息内带空行换行**。

## 根因 (2026-05-14 诊断)

openclaw-weixin 插件源码: `~/.openclaw/npm/node_modules/@tencent-weixin/openclaw-weixin/dist/src/channel.js`

关键函数 `sendWeixinOutbound`:

```javascript
filteredText = sendingResult.text;
const result = await sendMessageWeixin({ to: params.to, text: filteredText, ... });
```

**它把整个 filteredText (含 `\n\n`) 一次性发给 wechat API**。WeChat 把 `\n\n` 当作消息内的换行符渲染。

验证: 比对 outbound 日志 `textLen=N` 与 jsonl 里的 assistant 回复:
- assistant: 2 气泡 [8 字, 10 字] = 8 + `\n\n`(2) + 10 = 20 字 
- outbound: 1 次 send,textLen=20

完全对上 → 插件确实不拆。

## 不是问题的:

- openclaw core 的 reply pipeline 不在这里拆——它把 model 输出原封不动传给插件
- `applyWeixinMessageSendingHook` 拦截器只能改 text 或 cancel,不能 split + 多次发
- `textChunkLimit: 4000` 只在 > 4000 字时拆,跟 `\n\n` 无关

## 是问题的:

`messages.directChat.visibleReplies` 默认 = `"automatic"` = "把 model text 当一条发"。 

要拆需要切到 `"message_tool"` 模式,让模型显式调用 `message(action=send, text=...)` 工具一条一条发。

## 试过的修法 (失败)

**channel.js 直接 patch**:
```javascript
const bubbleParts = filteredText.split(/\n{2,}/).filter(s => s.trim());
for (const bubble of bubbleParts) {
    await sendMessageWeixin({...text: bubble.trim()...});
    if (i < bubbleParts.length - 1) await new Promise(r => setTimeout(r, 600));
}
```

实测时**没收到回复**——但看 gateway err.log 真实原因是模型卡在 tool_call(reading skill files)阶段,根本没走到 send。patch 本身可能没问题,只是没机会被触发。已回退至原版。

## 下一次该怎么修

**选项 A (优先)**: 走 openclaw 原生 `messages.directChat.visibleReplies = "message_tool"`
- 改 openclaw.json config
- 加 prompt 教模型用 message 工具
- 模型学新模式,但 supported,不会被 npm update 覆盖

**选项 B**: 重试 channel.js patch
- 加 try/catch + 降级到原行为
- 加详细日志确认 patch 被触发
- 改 `~/.openclaw/npm/node_modules/@tencent-weixin/openclaw-weixin/dist/src/channel.js`
- 备份位置: 同目录 `.bak.bubble-patch` 后缀
- 缺点: npm update 会覆盖

**选项 C (摆烂)**: 接受当前现状——模型分气泡正确,微信端渲染为单消息换行,不算大问题

## 关联

- 母项目: [[project-wechat-ai-clawbot]]
- 提示侧: [[feedback-llm-newline-vs-punctuation]](模型层分气泡的 prompt 设计)
