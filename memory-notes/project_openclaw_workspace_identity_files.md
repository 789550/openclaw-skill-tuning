---
name: project-openclaw-workspace-identity-files
description: "openclaw workspace 根目录的 SOUL.md / IDENTITY.md / USER.md / TOOLS.md / HEARTBEAT.md / MEMORY.md 会被自动注入每个 session 的 system prompt。如果这些文件硬编码了某个角色身份(比如 cyrene),即使 AGENTS.md 通用化、即使发 slash 命令切换 skill,模型仍然会被这些 workspace 文件锚定到那个默认身份。修法 = 把这些文件改成中性,角色身份只放在 skills/<char>/ 目录里。"
metadata: 
  node_type: memory
  type: project
  originSessionId: d1ce128f-cfd9-4cd1-a194-83891b3811b7
---

## 现象

openclaw 启动每个 session 时,会**自动注入**这一组 workspace 根目录文件到 system prompt:

```
AGENTS.md          - 共享规则
SOUL.md            - 声音/性格底色
IDENTITY.md        - 角色档案
USER.md            - 对方信息
TOOLS.md           - 可用工具
HEARTBEAT.md       - 心跳机制
MEMORY.md          - 长期记忆
BOOTSTRAP.md       - (可选)
```

可以在 `sessions.json` 的 `systemPromptReport.injectedWorkspaceFiles` 里看到这个列表。

## 陷阱

如果这些文件里有 cyrene-specific 内容(比如 SOUL.md 写 "你的声音(来自 cyrene): 自称'人家'、称对方'伙伴'",IDENTITY.md 标题就是"角色档案: 昔涟"),那么:

- 即使 AGENTS.md 说"身份由当前激活的 skill 决定"
- 即使发 slash 命令 `/march7th` 或 `/kafuka`
- 模型读到 SOUL.md/IDENTITY.md 里的硬编码后,**仍然把自己定位成 cyrene**,串味严重(用"人家"+"伙伴"演 kafuka)

模型的 thinking 里会直接说: "SOUL.md 和 IDENTITY.md 都指向昔涟,所以我是 cyrene"。

## 修法 (2026-05-14)

把 workspace 根目录的所有"身份相关"文件改成**完全中性**:

- **SOUL.md**: 改成"你的声音完全由当前激活的 skill 决定,看 `skills/<active>/personality.md`"
- **IDENTITY.md**: 改成"身份由当前激活的 skill 决定 + 列出可用角色 + 没 slash 时礼貌问对方想跟谁聊"
- **USER.md**: 改成对方档案模板,不抬头"关于人家的伙伴"(那是 cyrene 视角)

cyrene 专属的原版文件备份到 `skills/cyrene/_workspace-snapshots/` 作为参考。

## 验证

改完之后:
- 微信新 session 进来 → 模型问"想跟谁聊呀? 可选 cyrene/march7th/kafka"
- 发 `/kafuka` → 完全卡芙卡声音(自称"我"+ 称"孩子"),零"人家"串味
- 发 `/cyrene` → 完全昔涟声音(skill 文件够全,不需要 workspace 兜底)

## How to apply (写 openclaw 通用 skill 项目时)

1. **workspace 根目录文件只放共享规则**(AGENTS.md),其他身份相关文件全部中性化或留空
2. **角色身份完全在 `skills/<char>/`**——SKILL.md/profile.md/personality.md/interaction.md 必须自包含
3. **slash 命令完全决定身份**——不发命令时模型应该问,不应该有"默认角色"

这个原则是 [[project-wechat-ai-clawbot]] 项目里第二天发现的——做单一角色 bot 时硬编码 cyrene 没问题,做多角色通用项目时这是必踩的坑。

## 关联

- 母项目: [[project-wechat-ai-clawbot]]
- 关联: [[feedback-skill-template-vs-judgment]] (硬规则管结构,身份硬绑定是另一种"过度具体")
