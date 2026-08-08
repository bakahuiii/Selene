# 按日合并 JSON

Windows `0.5.4` 每次同步维护后都会生成一份派生的按日时间线。它读取
Windows 持久 Android 归档和 Windows 采集目录中的不可变
`SELENE-v1-.../context-events.json`，然后写入：

```text
%LOCALAPPDATA%\SELENE\Archive\Merged\SELENE-merged-v1-YYYY-MM-DD.json
```

日期使用 Windows 系统时区。每条事件按 `startAt` 归入一个日期；如果
`startAt` 无法解析，则保守地使用来源快照的 `generatedAt`。

## 原始数据与幂等性

- Android 原始快照继续保留在 `Archive\SELENE-v1-*`，原有 ACK 和一天保留期
  清理规则不变。
- Windows 原始快照继续保留在用户选择的导出目录。
- 日合并文件是派生数据。晚到数据出现时会原子替换当天文件，但不会把合并文件
  再当作下一次合并的输入。
- 任一来源扫描失败时，不发布不完整的新文件；上一份健康的日文件保持不变，错误会
  显示在 Windows 同步状态中。

事件按稳定 `id` 去重。同一个 ID 在重叠快照中重复出现时，保留 `capturedAt` 更新的
版本；事件对象本身不删字段、不重新舍入，也不改变坐标和速度信息。

## 文件结构

日文件继续使用 `selene-context-events/v1` 和
`producer.layout: immutable-snapshot-v1`，因此现有 HYPERION 导入器仍可读取。新增的
`aggregation.schema` 为 `selene-daily-merge/v1`，用于说明日期、来源快照数量、原始
事件数量、去重后的事件数量和重复数量。

`events` 数组保留原始事件 JSON。Android `0.5.3` 快照无需迁移；合并文件的 producer
版本为 `0.5.4`。

## 测试

Windows 契约测试覆盖 Android `0.5.3` 输入、Android/Windows 混合来源、稳定 ID 去重、
HYPERION schema 兼容性，以及原始 Android 目录不被修改。
