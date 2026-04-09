

真的非常推荐这个 FlowBuilder，这是zhang zara在 GitHub 分享的 skill
https://github.com/zarazhangrui/follow-builders
可能值得借鉴的点包括：
1. 定时发送消息
2. 绑定渠道

还有分享几个同样的，我很喜欢的博主们各自发表的 skill


这个 Obsidian CLI 是必须得装上的，因为我需要靠 AI 去观察我的笔记，以及连接我的日常生活和工作，其中，它们的关系图谱结构是我非常喜欢的一种可视化的展示方式。


#Context

每一次的系统操作，尤其是跨软件或网页的操作，你都需要把成果可视化出来。

比如：
1. 创建了一个 HTML 文件：最终结果需要把它打开，在我的电脑上呈现出来。
2. 创建了一个 PPT 或 Word 文档：你需要把它打开让我看到。
3. 创建了多维表格：同样需要把它打开，或者通过飞书 CLI 发到群里，让大家都能看到你做的成果，方便验收。

也就是说，你需要把一切人类看不到的、你已经执行的操作，想办法可视化到人类的眼前。



（重点强调的）（容易忽视的细节）
在6小时大报时提及到飞书多维表格的时候，需要保持在context当中带上飞书文档的链接：
Claude 更新作为表格投递任务的时候，复杂任务一定要带上 GitHub 项目的链接
早报和每 6 小时大报的内容需要进行归档系统




OpenClaw定时机制：**[https://github.com/tang730125633/agent-orchestration-docs](https://github.com/tang730125633/agent-orchestration-docs)**
│
├── 每30分钟 ──→ 技术健康检查（A~F）
│                  ├── 全绿 → 静默（写日志，不推飞书）
│                  └── 有异常 → Agent 主动调用飞书发告警
│
└── 每6小时 ──→ 综合汇报（06:00 / 12:00 / 18:00 / 00:00）
       ├── 第一段：业务进展（读表 → 更新 → 汇报三大系统）
       ├── 第二段：技术自检（A~F 结果并入报告）
       └── OpenClaw 自动把报告推送到飞书群

找到了。跟**戴总企业服务**相关的 GitHub 仓库有以下这些：

---

## 戴总企业服务 AI 系统 — GitHub 仓库汇总

| #   | 仓库                                                                                    | 用途                                                          | 最近更新  |
| --- | ------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ----- |
| 1   | [dai-knowledge-base](https://github.com/tang730125633/dai-knowledge-base)             | 🔵 **知识库系统** — 电力行业知识库架构（发输变配用×全生命周期×国标/行标/企标）              | 04-07 |
| 2   | [energy-daily-brief-skill](https://github.com/tang730125633/energy-daily-brief-skill) | 🟠 **早报系统** — 能源行业日报 OpenClaw Skill，自动抓取+推送企业微信             | 04-07 |
| 3   | [openclaw-workspace-index](https://github.com/tang730125633/openclaw-workspace-index) | 📊 **轮询机制+大报** — 小婷/小琳工作目录索引、综合汇报模板、历史报告存档                  | 04-07 |
| 4   | [agent-orchestration-docs](https://github.com/tang730125633/agent-orchestration-docs) | 🤖 **多Agent编排** — ChatGPT/Claude Code/OpenClaw 多Agent协作方案文档 | 04-06 |
| 5   | [business-advisor](https://github.com/tang730125633/business-advisor)                 | 🟣 **人格系统** — 基于24本经管书籍的企业经营咨询 OpenClaw Skill               | 04-05 |
| 6   | [zero-carbon-power-ai](https://github.com/tang730125633/zero-carbon-power-ai) 🔒      | 🏭 **零碳电力圈** — AI早报+行业知识库爬取（整合项目）                           | 03-30 |
| 7   | [ltny-tech](https://github.com/tang730125633/ltny-tech)                               | 🏢 **公司官网** — 湖北零碳能源科技有限公司技术文档知识库                           | 03-27 |

---

**共 7 个仓库**，覆盖戴总企业服务的三大核心系统（早报/知识库/人格）+ 轮询协作机制 + 多Agent编排 + 公司官网。

![[Pasted image 20260407055044.png]]



现在可以把完整的启动流程告诉你了。

---

## OpenClaw启动时发生了什么

**第一步 — App 唤醒**

你双击 OpenClaw.app，macOS 加载这个 125MB 的原生二进制文件。它做的第一件事：

```
读取 ~/.openclaw/openclaw.json   ← 整个系统的"大脑蓝图"
```

这一个文件告诉它所有事情：用哪个端口、用哪些模型、连哪些渠道、记忆存在哪。

---

**第二步 — 网关启动**

App 读完配置后，立刻孵化出 `openclaw-gateway` 进程，在本机开一个 HTTP 服务：

```
127.0.0.1:18789   ← 这就是你整个系统的神经中枢
```

网关启动时还要读：

```
agents/main/agent/auth.json    ← API 密钥（OpenAI/Kimi/GLM...）
agents/main/agent/models.json  ← 可用模型列表
cron/jobs.json                 ← 加载所有定时任务
extensions/feishu/             ← 启动飞书插件，建立长连接
memory/main.sqlite             ← 加载知识库数据库
agents/main/qmd/index.sqlite   ← 加载向量索引（QMD）
```

完成后网关就绪，发出 `{"ok":true,"status":"live"}`。

---

**第三步 — TUI 连接**

`openclaw-tui` 进程启动，用一个 **内部 Token** 向网关做认证：

```
Authorization: Bearer 4f1bf4c76953b4c384...（你的本机密钥）
```

认证通过后，TUI 就成了网关的一个「屏幕」，把网关里发生的一切渲染成你在终端看到的界面。

---

## 你说话之后发生了什么

```
你打字
   ↓
TUI 把文字打包成请求 → 发送到 localhost:18789
   ↓
网关收到 → 检查是否需要从知识库检索（QMD 向量搜索）
   ↓
组装上下文（系统提示 + 知识库片段 + 对话历史）
   ↓
发 HTTPS 请求到 → OpenRouter/Gemini/Kimi/Claude API（看当前主力模型）
   ↓                    ↑
   ↓          若失败自动切换 Fallback 模型（配了8个备用）
   ↓
API 返回流式 token（streaming）
   ↓
网关把 token 实时转发给 TUI
   ↓
你看到文字一个一个出现
   ↓
Agent 决定要执行工具（读文件/发飞书/跑脚本）
   ↓
网关 → 检查 exec-approvals.json（这个操作有没有授权？）
   ↓
执行 shell 命令 / 调用 lark-cli / 写文件
   ↓
结果返回给模型 → 模型生成下一轮回复
```

---

## 为什么它能操控你的电脑

关键在 `exec-approvals.json`。这个文件相当于"授权白名单"，允许 Agent 通过 unix socket 执行经过授权的 shell 命令。

```
Agent 想执行命令
   ↓
发到 exec-approvals.sock（本地 socket）
   ↓
检查是否在授权列表 → 有授权 → 直接执行
                    → 无授权 → 弹出确认框给你审批
   ↓
命令在你的 macOS 本地运行，有完整文件系统权限
```

这就是为什么它能读文件、写文件、运行脚本、调 API，像一个真正在你电脑上工作的助手。

---

## 一句话总结整个魔法

> `openclaw.json` 是剧本，`gateway` 是导演，`模型 API` 是演员，`TUI` 是舞台，`exec-approvals` 是安全门，`~/.openclaw/` 是整个剧团的后台。

你和它说话，本质上是在和一个运行在你电脑本地的、连接着全球 AI 模型的**私有 AI 操作系统**对话。