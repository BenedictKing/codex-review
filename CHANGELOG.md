# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- 改进任务难度评估标准：更精确的 xhigh/high 判定逻辑
  - 新增总变更量计算（新增+删除）≥ 500 行触发 xhigh
  - 新增单项指标：新增 ≥ 300 行或删除 ≥ 300 行触发 xhigh
  - 添加详细的解析规则和决策逻辑说明
  - 提供 6 个典型案例验证（包括 20 文件/1327 行变更场景）
  - 修正 git diff --stat 解析规则，正确处理边缘情况：
    - 单数形式："1 file changed"
    - 缺失字段：当插入或删除为 0 时 Git 会省略该字段
    - 纯重命名：可能显示 0 或完全省略插入/删除
- 版本号更新：2.1.3 → 2.1.4

## [2.1.0] - 2026-01-19

### Added

- 初始版本发布
- 双技能架构：主技能 (codex-review) + 子技能 (codex-runner)
- 自动 CHANGELOG 生成功能
- 智能难度评估（根据变更规模自动调整审核深度）
- 自动暂存新增文件（避免 codex P1 错误）
- 支持多种项目类型：Go、Node、Python
- Lint 集成（go fmt、npm lint、black 等）
- 双模式审核：未提交变更 / 最新提交
- 意图驱动审核（结合 CHANGELOG 和代码变更）
- 完整的中英文文档
- MIT 许可证

### Technical Details

- 使用 `context: fork` 分离审核上下文，减少 Token 消耗
- 支持 `model_reasoning_effort=high/xhigh` 配置
- 自动超时调整：一般任务 10 分钟，困难任务 30 分钟
