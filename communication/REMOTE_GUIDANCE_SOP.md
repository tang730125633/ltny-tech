# Claude 远程指导 OpenClaw SOP

最后更新：2026-03-28

## 目标

通过同一个 GitHub 仓库，实现 Claude（导师）对 OpenClaw（执行端）的异步远程指导与闭环复盘。

## 角色分工

- Claude：下发任务、审阅结果、给反馈、维护模板与技能
- OpenClaw：拉取任务、执行产出、如实汇报、暴露问题

## 标准闭环（每轮都按此执行）

1. Claude 端下发任务  
在 `communication/claude-to-xiaol.md` 新增一条指令（编号 + 目标 + 验收标准 + 截止时间）。

2. Claude 推送  
提交并推送到远端仓库。

3. OpenClaw 拉取并执行  
先 `git pull --rebase`，再读取 `communication/claude-to-xiaol.md` 最新指令并执行。

4. OpenClaw 产出与汇报  
- 结果文档放到 `briefings/output/`（按日期命名）
- 汇报写入 `communication/xiaol-to-claude.md`
- 遇到失败或无更新时，必须如实写明，不允许旧闻补位

5. OpenClaw 推送  
提交并推送产出与汇报。

6. Claude 审阅并复盘  
- 审阅后把修订稿放 `briefings/reviewed/`
- 在 `communication/claude-to-xiaol.md` 记录反馈和下一轮动作

## 指令模板（Claude 写给 OpenClaw）

```md
### #00X — [任务名]（YYYY-MM-DD）
**状态**: ⏳ 进行中
**目标**: [一句话目标]
**验收标准**:
1. [可验证结果1]
2. [可验证结果2]
**截止时间**: YYYY-MM-DD HH:mm（Asia/Shanghai）
**备注**: [限制条件/禁忌]
```

## 汇报模板（OpenClaw 写给 Claude）

```md
## 日期：YYYY-MM-DD
## 状态：✅ 成功 / ⚠️ 部分完成 / ❌ 失败
## 任务：[任务描述]
## 结果：[简要说明]
## 证据：[产出文件路径或关键数据]
## 问题：[遇到的问题，没有则写"无"]
## 下一步：[建议导师下发的后续动作]
```

## 质量红线

1. 不编造数据与来源
2. 无新增就写“无新增”，不拿旧内容填充
3. 汇报必须包含证据路径
4. 状态必须区分“已发送 / 已接收 / 已完成”

## 常用命令

```bash
# 拉取最新指导
git pull --rebase origin main

# 查看导师指令
sed -n '1,220p' communication/claude-to-xiaol.md

# 提交并推送结果
git add .
git commit -m "feat: update daily briefing and report"
git push origin main
```
