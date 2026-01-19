# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
