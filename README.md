# Codex Review Skill

[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://skillsmp.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

English | [简体中文](./README_CN.md)

> 🚀 Professional code review skill integrated with Codex AI, automatically collects change context and generates CHANGELOG.

## Introduction

Codex Review is an intelligent Claude Code skill that integrates with Codex AI review tool. It automatically analyzes Git changes, generates CHANGELOG entries, and performs comprehensive code reviews with intent-driven analysis.

### Key Features

- ✨ **Smart Context Collection**: Automatically analyzes Git changes and collects complete code modification context
- 📝 **Auto CHANGELOG Generation**: Detects when CHANGELOG is not updated and automatically generates standard entries
- 🔄 **Dual Review Modes**:
  - Uncommitted changes → Review all workspace modifications
  - Clean workspace → Review latest commit
- 🎯 **Intelligent Difficulty Assessment**: Automatically adjusts review depth and timeout based on change scale
- 🔧 **Lint Integration**: Automatically executes code formatting and static checks
- 💡 **Intent-Driven Review**: Combines CHANGELOG descriptions with code changes for more accurate review suggestions
- 🏗️ **Efficient Architecture**: Uses dual-skill architecture to reduce token consumption

## Quick Start

Set up in 5 minutes

## Installation

### Option 1: Install via skills CLI (Recommended)

```bash
# Install globally to all detected agents (Claude Code, Cursor, Codex, etc.)
npx skills add -g BenedictKing/codex-review

# Or install to current project only
npx skills add BenedictKing/codex-review
```

The skill will be automatically installed to `~/.claude/skills/codex-review` and loaded by Claude Code.

### Option 2: Manual Installation via Git Clone

```bash
# Clone to Claude Code's skills directory
git clone https://github.com/BenedictKing/codex-review.git ~/.claude/skills/codex-review

# Or clone to your preferred location
git clone https://github.com/BenedictKing/codex-review.git
cd codex-review
```

## Prerequisites

### Required

- **Git Repository**: Must be executed in a Git repository directory
- **Codex CLI**: Need to install and configure [Codex](https://codex.ai/) command-line tool
- **CHANGELOG.md**: Project root directory needs a CHANGELOG.md file

### Optional (Based on Project Type)

- **Go Projects**: `go fmt`, `go vet`
- **Node Projects**: `npm run lint`
- **Python Projects**: `black`, `ruff`

## Usage

### Via Slash Command

```bash
/codex-review
```

### Via Natural Language

```
"code review"
"review my code"
"check the code"
"代码审核"
```

## How It Works

### 1. Check Workspace Status

Automatically detects if there are uncommitted changes and decides the review mode.

### 2. CHANGELOG Check & Auto-Generation

If CHANGELOG.md is not updated, the skill will:
1. Analyze `git diff` to get complete changes
2. Automatically generate standard CHANGELOG entries
3. Use Edit tool to write to file
4. Continue with review process

**Auto-generated CHANGELOG format**:

```markdown
## [Unreleased]

### Added / Changed / Fixed

- Feature description: What problem was solved or what functionality was implemented
- Affected files: Main modified files/modules
```

### 3. Stage New Files (v2.1.0)

**Automatically adds all new files to git staging area to avoid codex P1 errors.**

```bash
# Safely stage all new files (handles empty lists and special filenames)
git ls-files --others --exclude-standard -z | while IFS= read -r -d '' f; do git add -- "$f"; done
```

**Features:**
- Uses null character separation to correctly handle filenames with spaces/newlines
- Uses `--` separator to correctly handle filenames starting with `-`
- Loop body doesn't execute when there are no new files, safely skips
- Only processes new files, doesn't stage modified files
- Automatically excludes files in .gitignore

### 4. Intelligent Difficulty Assessment

Automatically selects review configuration based on change scale:

| Difficulty | Trigger Conditions | Configuration | Timeout |
|-----------|-------------------|---------------|---------|
| **Hard Task** | Modified files ≥ 10<br>Code changes ≥ 500 lines<br>Core architecture changes | `model_reasoning_effort=xhigh` | 30 minutes |
| **Normal Task** | Other cases | `model_reasoning_effort=high` | 10 minutes |

### 5. Lint + Codex Review

Automatically executes project-specific Lint tools:

- **Go Projects**: `go fmt ./... && go vet ./...`
- **Node Projects**: `npm run lint:fix`
- **Python Projects**: `black . && ruff check --fix .`

Then calls `codex review` for AI code review.

### 6. Self-Correction

If Codex finds inconsistencies between CHANGELOG description and code logic:
- **Code error** → Fix code
- **Inaccurate description** → Update CHANGELOG

## Dual-Skill Architecture

This project uses a **two-stage architecture**:

```
User Request → Main Skill (codex-review)
                ↓ Check workspace + Update CHANGELOG
           Task Tool → Sub-Skill (codex-runner)
                ↓ Execute Lint + codex review (independent context)
           Main Skill ← Return review results
                ↓ Self-correction if needed
           User ← Review results + Suggestions
```

**Why this design?**

| Aspect | Main Skill | Sub-Skill |
|--------|-----------|-----------|
| Context | Full conversation | Fork (independent) |
| Purpose | Intent analysis + CHANGELOG | Lint + Review execution |
| Token usage | Higher | Lower |
| Execution | Sequential | Independent |

**Benefits:**
- Main skill needs to understand user intent and conversation history (requires context)
- Lint and review execution don't need conversation history (avoids wasting tokens)
- Separation improves efficiency and reduces costs

### Core Philosophy: Intent vs Implementation

Simply running `codex review --uncommitted` only lets AI see "what was done (Implementation)".

By recording intent first (CHANGELOG), you're telling AI "what you want to do (Intention)".

**"Code changes + Intent description" as input together is the most efficient way to improve AI code review quality.**

## Use Cases

### Case 1: Daily Development Review

```
User: Modified a few files, want to review
Skill:
1. Detected 3 files modified, 200 lines changed
2. Checked CHANGELOG not updated, auto-generated entry
3. Executed go fmt && go vet
4. Called codex review --uncommitted --config model_reasoning_effort=high
5. Returned review results and improvement suggestions
```

### Case 2: Large-Scale Refactoring Review

```
User: Refactored entire module, need comprehensive review
Skill:
1. Detected 15 files modified, 800 lines changed
2. Determined as hard task
3. Auto-generated CHANGELOG entry
4. Executed Lint
5. Called codex review --uncommitted --config model_reasoning_effort=xhigh
6. 30-minute timeout, deep review
```

### Case 3: Review Latest Commit

```
User: Workspace clean, want to review recent commit
Skill:
1. Detected clean workspace
2. Directly called codex review --commit HEAD
3. Returned review results
```

## Codex Review Command Reference

### Basic Syntax

```bash
codex review [OPTIONS] [PROMPT]
```

**Note**: `[PROMPT]` parameter cannot be used with `--uncommitted`, `--base`, or `--commit`.

### Common Options

| Option | Description | Example |
|--------|-------------|---------|
| `--uncommitted` | Review all uncommitted changes in workspace | `codex review --uncommitted` |
| `--base <BRANCH>` | Review changes relative to specified base branch | `codex review --base main` |
| `--commit <SHA>` | Review changes introduced by specified commit | `codex review --commit HEAD` |
| `--title <TITLE>` | Optional commit title, displayed in review summary | `codex review --uncommitted --title "feat: add JSON parser"` |
| `-c, --config <key=value>` | Override configuration values | `codex review --uncommitted -c model="o3"` |

### Usage Examples

```bash
# 1. Review all uncommitted changes (most common)
codex review --uncommitted

# 2. Review latest commit
codex review --commit HEAD

# 3. Review specific commit
codex review --commit abc1234

# 4. Review all changes in current branch relative to main
codex review --base main

# 5. Review with specific model
codex review --uncommitted -c model="o3"

# 6. Adjust reasoning depth
codex review --uncommitted -c model_reasoning_effort=xhigh
```

### Important Limitations

- `--uncommitted`, `--base`, `--commit` are mutually exclusive
- Must be executed in a Git repository directory
- CHANGELOG.md must be in uncommitted changes, otherwise Codex cannot see intent description

## CHANGELOG Format

Recommended to use [Keep a Changelog](https://keepachangelog.com/) format:

```markdown
# Changelog

## [Unreleased]

### Added
- New feature description

### Changed
- Modification description

### Fixed
- Bug fix description

## [1.0.0] - 2026-01-19

### Added
- Initial release
```

## Troubleshooting

### Codex Command Not Found

```bash
# Install Codex CLI
npm install -g @codex/cli

# Or follow official documentation
# https://codex.ai/docs/installation
```

### CHANGELOG Not Detected

Ensure CHANGELOG.md exists in project root:

```bash
# Create CHANGELOG.md
touch CHANGELOG.md

# Add basic structure
cat > CHANGELOG.md << 'EOF'
# Changelog

## [Unreleased]

### Added
- Initial version
EOF
```

### Review Timeout

For large-scale changes, the skill automatically adjusts timeout:
- Hard tasks: 30 minutes
- Normal tasks: 10 minutes

If still timing out, consider splitting changes into smaller commits.

## Contributing

Welcome to submit Issues and Pull Requests!

## License

MIT License

## Related Links

- [Codex CLI Documentation](https://codex.ai/docs)
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills)
- [SKILL.md - Detailed Skill Documentation](./.claude/skills/codex-review/SKILL.md)
- [codex-runner.md - Sub-Skill Documentation](./.claude/skills/codex-review/codex-runner.md)
- [Project Repository](https://github.com/BenedictKing/codex-review)

---

🎉 Setup complete! Now you can enjoy professional AI code reviews!
