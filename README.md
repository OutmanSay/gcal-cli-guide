# gcal-cli-guide

用命令行管理 Google 日历 | Manage Google Calendar from the terminal with `gog` CLI

基于 [gogcli](https://github.com/AdrianLunworton/gogcli)，提供完整的中文使用指南和 AI Agent 集成模板。

## 为什么用命令行管日历

- 批量操作：一条命令查看一周日程，不用在 app 里翻来翻去
- AI 集成：配合 Claude Code / OpenClaw 等 AI agent，用自然语言管日历
- 脚本自动化：cron 定时提醒、日报生成、日程统计
- 速度快：不用打开浏览器，终端里几秒搞定

## 安装

```bash
# macOS
brew install gogcli

# 验证
gog --version
```

## 首次认证

```bash
# 初始化 OAuth 认证（会打开浏览器登录 Google 账号）
gog auth login

# 验证连接
gog calendar events --today
```

认证信息保存在 `~/Library/Application Support/gogcli/`，后续无需重复登录。

## 命令速查

### 查看日程

```bash
# 今天日程
gog calendar events --today

# 明天日程
gog calendar events --tomorrow

# 本周日程
gog calendar events --week

# 指定日期
gog calendar events --from 2026-04-05 --to 2026-04-06

# 未来 N 天
gog calendar events --days 7

# 所有日历的事件
gog calendar events --today --all

# 搜索事件
gog calendar search "开会"

# JSON 输出（便于脚本处理）
gog calendar events --today -j
```

### 创建事件

```bash
# 有时间段的事件
gog calendar create primary --summary "团队周会" \
  --from "2026-04-07T14:00:00+08:00" \
  --to "2026-04-07T15:00:00+08:00"

# 全天事件
gog calendar create primary --summary "写周报" \
  --from "2026-04-07" --to "2026-04-07" --all-day

# 带地点和描述
gog calendar create primary --summary "客户拜访" \
  --from "2026-04-07T10:00:00+08:00" \
  --to "2026-04-07T11:30:00+08:00" \
  --location "星巴克 国贸店" \
  --description "讨论 Q2 合作方案"

# 带 Google Meet 视频会议
gog calendar create primary --summary "远程会议" \
  --from "2026-04-07T09:00:00+08:00" \
  --to "2026-04-07T10:00:00+08:00" \
  --with-meet

# 添加参会者
gog calendar create primary --summary "项目对齐" \
  --from "2026-04-07T15:00:00+08:00" \
  --to "2026-04-07T16:00:00+08:00" \
  --attendees "alice@example.com,bob@example.com"

# 设置提醒
gog calendar create primary --summary "提交报告" \
  --from "2026-04-07T17:00:00+08:00" \
  --to "2026-04-07T17:30:00+08:00" \
  --reminder "popup:30m"

# 周期性事件
gog calendar create primary --summary "每周站会" \
  --from "2026-04-07T09:30:00+08:00" \
  --to "2026-04-07T09:45:00+08:00" \
  --rrule "RRULE:FREQ=WEEKLY;BYDAY=MO"
```

### 修改事件

```bash
# 先查到事件 ID
gog calendar events --today -j

# 改时间
gog calendar update primary <eventId> \
  --from "2026-04-07T15:00:00+08:00" \
  --to "2026-04-07T16:00:00+08:00"

# 改标题
gog calendar update primary <eventId> --summary "新标题"

# 改地点
gog calendar update primary <eventId> --location "新地点"

# 添加参会者（保留已有）
gog calendar update primary <eventId> --add-attendee "charlie@example.com"
```

### 删除事件

```bash
# 删除指定事件
gog calendar delete primary <eventId>

# 跳过确认
gog calendar delete primary <eventId> -y
```

### 其他实用命令

```bash
# 查看所有日历
gog calendar calendars

# 查看日历颜色
gog calendar colors

# 查看冲突事件
gog calendar conflicts --from "2026-04-07" --to "2026-04-11"

# 设置专注时间
gog calendar focus-time primary \
  --from "2026-04-07T14:00:00+08:00" \
  --to "2026-04-07T16:00:00+08:00"

# 设置外出
gog calendar out-of-office primary \
  --from "2026-04-10" --to "2026-04-12"

# 多账号支持
gog calendar events --today -a work@company.com
gog calendar events --today -a personal@gmail.com
```

## AI Agent 集成

本项目提供一份 AI Agent Skill 模板（`skill.md`），可直接用于 Claude Code、OpenClaw 等 AI agent 平台。

### 使用场景

| 你说 | Agent 执行 |
|------|-----------|
| "今天安排是什么" | `gog calendar events --today` |
| "明天有什么会" | `gog calendar events --tomorrow` |
| "排个会，下午2点到3点" | `gog calendar create primary --summary "会议" --from ... --to ...` |
| "把周会挪到10点" | `gog calendar update primary <id> --from ...` |
| "日程记录：刚开完会" | 推断时间 → 创建已完成事件 |
| "搜一下上周跟张总的会" | `gog calendar search "张总"` |

### 配置方法

将 `skill.md` 复制到你的 AI agent 工作目录中。不同平台的配置方式：

**Claude Code** — 放到项目 `CLAUDE.md` 中引用或直接作为上下文

**OpenClaw** — 放到 `skills/gcal/SKILL.md`

## 实用技巧

### 日程记录（记已完成的事）

不只是提前排日程，还可以事后记录。用 AI agent 说"日程记录 刚开完会"，agent 会：
1. 推断时间（"刚才" → 当前时间往前推 1 小时）
2. 创建带 ✅ 前缀的事件

这样你的日历既是计划，也是记录。

### 脚本自动化

```bash
# 每天早上推送今日日程
gog calendar events --today -p | mail -s "今日日程" you@email.com

# 导出本周日程为 JSON
gog calendar events --week -j > week-events.json

# 检查今天是否有冲突
gog calendar conflicts --from today --to tomorrow
```

## 时间格式

`gog` 支持多种时间格式：

| 格式 | 示例 |
|------|------|
| RFC3339 | `2026-04-07T14:00:00+08:00` |
| 日期 | `2026-04-07` |
| 相对时间 | `today`, `tomorrow`, `monday` |
| 自然语言 | `this week`, `next monday` |

创建/修改事件时建议使用 RFC3339 格式以确保时区正确。

## License

MIT
