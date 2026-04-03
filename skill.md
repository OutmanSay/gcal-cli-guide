# Google Calendar AI Agent Skill

通过 `gog` CLI 管理 Google Calendar 的 AI Agent 技能模板。

## When to Use

触发场景：
- "日程记录 XXX" / "记下日程 XXX"
- "今天安排是什么" / "查一下日程" / "看看日历"
- "排到下午2点" / "安排到几点"
- "把XX挪到10点"
- "明天有什么安排"
- "搜一下上周的会"

## When NOT to Use

- 待办/任务管理 → 用任务管理工具
- 记账 → 用记账工具

## 命令

### 查看日程

```bash
# 今天日程
gog calendar events --today

# 明天日程
gog calendar events --tomorrow

# 本周日程
gog calendar events --week

# 指定日期范围
gog calendar events --from 2026-04-05 --to 2026-04-06

# 未来 N 天
gog calendar events --days 7

# 搜索事件
gog calendar search "关键词"
```

### 创建事件

```bash
# 有时间段的事件
gog calendar create primary --summary "事件名" \
  --from "2026-04-07T14:00:00+08:00" \
  --to "2026-04-07T15:00:00+08:00"

# 全天事件
gog calendar create primary --summary "事件名" \
  --from "2026-04-07" --to "2026-04-07" --all-day
```

### 修改事件

```bash
# 先用 events 查到事件 ID，再 update
gog calendar events --today -j
gog calendar update primary <eventId> --from "新时间" --to "新结束时间"
gog calendar update primary <eventId> --summary "新标题"
```

### 删除事件

```bash
gog calendar delete primary <eventId>
```

### 记录已完成的事

当用户说"日程记录 刚开完会"或"我刚完成了XX"时：

1. **推断时间**：
   - "刚才" / "刚" → 当前时间往前推合理时长（默认1小时）
   - "上午开的" → 推断上午时间
   - "8点半做的" → 08:30
   - 不确定就问

2. **创建已完成事件**：
```bash
gog calendar create primary --summary "✅ 事件名" \
  --from "日期T开始时间+08:00" --to "日期T结束时间+08:00"
```

3. **回复**：`✅ 日程已记录：事件名（HH:MM - HH:MM）`

## 交互规则

| 用户说 | 执行 |
|--------|------|
| "今天安排" / "看看日程" / "查日历" | `gog calendar events --today` |
| "明天有什么" | `gog calendar events --tomorrow` |
| "本周日程" | `gog calendar events --week` |
| "日程记录 刚开完会" | 推断时间 → 创建 ✅ 事件 |
| "排到下午2点到4点" | `gog calendar create primary --summary ... --from ... --to ...` |
| "把XX挪到10点" | `gog calendar update primary <eventId> --from ...` |
| "搜一下跟张总的会" | `gog calendar search "张总"` |

## 安全

- 只读写日历事件
- 不删除非用户指定的事件
- 修改/删除前先确认事件信息
