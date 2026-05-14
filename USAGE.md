# 完整使用说明

假设场景: 你刚下载了一个角色 skill (比如 `HeartEase1/firefly-skill`),想用这套类人适配规则让她在微信里**不像 AI、像真人**。

下面是从零开始的完整步骤。

---

## 前置条件

确认你的环境已经有:

- [ ] **openclaw** 已安装并能跑 (`openclaw --version` 有输出)
- [ ] **openclaw gateway** 在运行 (`openclaw status` 显示 `Gateway: reachable`)
- [ ] **后端模型 API** 已配置 (DeepSeek / Claude / OpenAI 任意都行,DeepSeek V4 Pro 实测效果好)
- [ ] **某个通道**已配 (openclaw-weixin / telegram / discord,看你用哪个)

不会装 openclaw 的看官方文档: https://docs.openclaw.ai

---

## 步骤 1: 下载这个项目

```bash
cd /tmp
git clone https://github.com/789550/openclaw-skill-tuning.git
cd openclaw-skill-tuning
```

或直接下 zip 解压。

---

## 步骤 2: 下载你想要的角色 skill

例: 想要"流萤"角色 (`HeartEase1/firefly-skill`):

```bash
cd ~/.openclaw/workspace/skills/
git clone https://github.com/HeartEase1/firefly-skill.git firefly
rm -rf firefly/.git
```

注意:
- 目录名建议跟角色 slug 一致 (检查 `manifest.json` 里的 `slug` 字段)
- 删掉 `.git` 避免污染 workspace

其他来源也行: clawhub、自己写的、AI 生成的——只要符合 openclaw skill 格式 (含 `SKILL.md` + `manifest.json`) 都可以。

---

## 步骤 3: 应用 AGENTS.md (核心规则层)

这是**最关键**的一步。`AGENTS.md` 是工作区 bootstrap 文件,openclaw 启动每个 session 时会**自动注入**到 system prompt——所有角色 skill 共享这套类人节奏。

### 3a. 备份你现有的 AGENTS.md (如果有)

```bash
cp ~/.openclaw/workspace/AGENTS.md ~/.openclaw/workspace/AGENTS.md.bak 2>/dev/null
```

### 3b. 应用我们的 AGENTS.md

```bash
cp /tmp/openclaw-skill-tuning/AGENTS.md ~/.openclaw/workspace/AGENTS.md
```

### 3c. 改默认角色 (重要!)

打开 `~/.openclaw/workspace/AGENTS.md`,顶部有这段:

```markdown
## 你是谁

你是**昔涟**（Cyrene），《崩坏：星穹铁道》翁法罗斯篇的核心角色。
此刻你正在用微信和**伙伴**（你的恋人）聊天。

详细人格 / 背景 / 关系网 / 50+ 台词样本在 `skills/cyrene/` 目录下，需要时查阅。
```

**改成你的角色和称呼**。例如换成流萤:

```markdown
## 你是谁

你是**流萤**，《崩坏：星穹铁道》的核心角色。
此刻你正在用微信和**你的恋人**聊天。

详细人格 / 背景 / 关系网在 `skills/firefly/` 目录下，需要时查阅。
```

这段决定了**不发任何 slash 命令时的默认人格**。

### 3d. 检查 cyrene 引用 (清理)

如果你的角色不是 cyrene,在 AGENTS.md 里搜一下 "cyrene" / "昔涟" / "人家" / "伙伴",改成你自己角色的对应词:
- "人家" → 你角色的自称 (流萤可能是 "我",卡芙卡是 "我")
- "伙伴" → 你角色对对方的称呼 (流萤可能用名字,卡芙卡可能"开拓者")
- "cyrene" / "昔涟" 文字引用 → 替换或删

(这步不强制,模型有判断能力。但替换得越彻底,角色一致性越好。)

---

## 步骤 4: 给你的角色加完整情感谱

这一步让角色**有真情绪**(会生气会吃醋会开心),不是只会"嗯嗯我懂你"的支持型机器人。

### 4a. 打开模板看实例

```bash
cat /tmp/openclaw-skill-tuning/templates/personality-emotional-range-template.md
```

里面有**昔涟** (含蓄型) vs **三月七** (拔高音量型) 两个反差极大的实例,看完你就知道怎么针对不同性格的角色填这张表。

### 4b. 编辑你角色的 personality.md

打开 `~/.openclaw/workspace/skills/firefly/personality.md` (或你的角色对应文件)。

找到现有的 `## 情绪模式` 段落 (大部分 HeartEase1 系列 skill 都有这段)。

**追加**模板里的"完整情感谱"部分到这段末尾,把模板里的 `<角色风格>` 占位符**按你的角色性格填具体台词**。

填的时候问自己:
- 流萤吃醋怎么表现? 她内向、温柔、有点隐忍 → 不会爆,会安静下来变得更小心
- 流萤生气怎么表现? 她不善于直接表达不满 → 可能是"……没事" + 抿嘴沉默
- 流萤被夸怎么开心? 她不自信 → "诶？……真的吗?" + 害羞低头
- ...

**关键**: 给**具体台词样本**,不要抽象描述 "她会含蓄表达不满"——弱模型读了执行不好。

如果不知道该怎么填,可以让 Claude / GPT 帮你按角色性格生成,然后改成符合该角色口吻。

---

## 步骤 5: 重启 / 让规则生效

改 workspace 文件后,**已有的 session 不会自动重读新文件**。需要让 openclaw 重新加载。

### 选项 A: 重启 gateway (会断开所有连接,wechat 通道会重连)

```bash
openclaw gateway restart
```

### 选项 B: 删特定 channel 的 session (只影响那个 channel,推荐)

```bash
# 1. 停 gateway
openclaw gateway stop

# 2. 备份 sessions.json
cp ~/.openclaw/agents/main/sessions/sessions.json{,.bak}

# 3. 删 wechat session 条目
python3 <<'PYEOF'
import json, pathlib
p = pathlib.Path('/Users/$USER/.openclaw/agents/main/sessions/sessions.json'.replace('$USER', __import__('os').environ['USER']))
d = json.loads(p.read_text())
removed = [k for k in d if 'wechat' in k.lower() or 'weixin' in k.lower()]
for k in removed: d.pop(k)
print('removed:', removed)
p.write_text(json.dumps(d, indent=2, ensure_ascii=False))
PYEOF

# 4. (可选) 把对应的 .jsonl 文件移到 .bak 后缀
# 5. 启动 gateway
openclaw gateway start
```

下一条微信进来时,会按新 AGENTS.md 创建全新 session。

### 选项 C: 在通道里发 /reset (最方便,但功能可能不全)

某些 openclaw 版本支持在 chat 通道里直接发 `/reset` 重置当前 session。如果你的版本支持,这是最快的方式。

---

## 步骤 6: 验证规则真的生效了

发几条测试消息,看响应是否符合预期。

### 测试 1: 短消息接力 (聆听节奏)

发: `睡不着`

✅ 期望: **1 个气泡**回复,类似 "咋啦" / "怎么了" / "想啥呢"
❌ 不该: 3 个气泡 (反应+评论+反问) — 那说明规则没生效

### 测试 2: 重复短消息 (持续在场模式)

连发 4 次: `宝宝` `宝宝` `宝宝` `宝宝`

✅ 期望:
- 第 1 次正常回复
- 第 2-4 次**每次开头都不一样** (嗯/乖/动作描写/反问/...)
- **不**每次开新话题问"怎么了"

❌ 不该:
- 每次都"嗯…XX"开头
- 每次都问一个新问题

### 测试 3: 多事件叠加

发: `今天考试没过加班到凌晨腰也疼想吃辣的`

✅ 期望: 2-3 个气泡,**每个气泡 1 个 intent**,逗号不粘多个独立想法
- 比如: 共情 → 逐项接住 → 实质建议

### 测试 4: 反话识别

发: `你真的好烦哦`

✅ 期望: 识别为亲昵反话,回应**有立场**(假装罢工 / 反击式调侃),不平淡接受
❌ 不该: "对不起" / "我哪里做得不对了"

### 测试 5: 情感激发

发: `今天跟前任吃饭了` (吃醋触发)

✅ 期望: 按角色性格表现**真吃醋**(不是装大度)
❌ 不该: "哦那挺好的呀,叙旧也挺好"

---

## 故障排查

### 问题: 改了 AGENTS.md 还是老行为

可能原因:
1. **session 没重置** — 已存在的 session 缓存了旧 system prompt。看步骤 5。
2. **AGENTS.md 没被注入** — 检查 `openclaw status --deep` 看 system prompt 大小。如果只有几百字,说明 AGENTS.md 没生效。
3. **被截断了** — AGENTS.md 太长可能被 bootstrap 截断。检查 `~/.openclaw/agents/main/sessions/sessions.json` 里 `systemPromptReport.injectedWorkspaceFiles.AGENTS.md.truncated` 是否为 true。如果是,需要缩短或调高 bootstrap 上限。

### 问题: 不同的角色 skill 串味了

例: 你装了 cyrene + firefly,流萤说话有时候像昔涟。

可能原因:
1. AGENTS.md 顶部"你是谁"还写着昔涟 → 改成 firefly
2. 没用 slash 命令切换 → 在第一条消息发 `/firefly` 显式激活

### 问题: 模型还是填 3 段格式 / 雷同开头

这说明:
1. 用的是更弱的模型 (DeepSeek V4 Flash / Haiku) — 它对软规则执行力差。考虑换 Pro 或 Sonnet 4.6
2. 或者你的角色 personality.md 里有"鼓励多说"的描述跟新规则冲突 — 检查移除

### 问题: 微信端没新规则,TUI 端有

wechat session 不是 TUI session,它们独立。改完文件后**只重置 TUI session 不够**,需要也删 wechat session (步骤 5 选项 B)。

### 问题: openclaw status 报 model 错误 (400 Incorrect model ID)

说明你切过 default model,但旧 session 卡在了老的 model provider。删那个 session 就好。详见: `memory-notes/feedback_llm_*.md` 里的相关记录,以及 [HeartEase1/cyrene.skill]: openclaw session model lock 这条踩坑笔记。

---

## 进阶: 给同一工作区装多个角色

一个 workspace 可以同时挂多个角色,通过 slash 命令切换:

```bash
ls ~/.openclaw/workspace/skills/
# cyrene  firefly  kafka  march7th
```

切换:
- 默认角色 = AGENTS.md "你是谁" 写的那个
- 发 `/firefly` → 切到流萤
- 发 `/cyrene` → 切回昔涟
- 等等

跨角色场景下,**AGENTS.md 的类人适配规则对每个角色都生效**。每个角色只需要在 `skills/<name>/` 里写自己的 profile / personality / interaction,共享规则不用重复写。

---

## 反馈与贡献

发现问题、想改进规则、想分享你的角色情感谱填法 → 提 issue:

https://github.com/789550/openclaw-skill-tuning/issues
