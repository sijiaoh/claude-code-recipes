# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 编程规范

**重要**: Claude Code 在处理此项目时必须严格遵循 `base.md` 中定义的所有编程规则和最佳实践，包括但不限于：
- 测试驱动开发 (TDD)
- 代码可读性和 DRY 原则
- 性能优化和错误处理
- 安全编程实践
- 模块化设计和代码组织

## 仓库概述

这是一个个人 Claude 设定合集，提供针对不同编程语言和框架的模块化配置文件。用户可以根据项目需求选择合适的配置文件，复制到自己的项目中使用。

## 项目结构

```
/
├── base.md                    # 核心规则和基础指令
├── [语言名].md               # 语言特定规则 (如 python.md, javascript.md)  
├── [框架名].md               # 框架特定规则 (如 react.md, django.md)
└── .claude/
    └── settings.local.json    # Claude Code 权限配置
```

## 文件说明

- **base.md** - 包含通用编程最佳实践，适用于所有项目
- **[语言名].md** - 特定编程语言的规则和约定 (如 `python.md`, `rust.md`)
- **[框架名].md** - 特定框架的开发指导 (如 `fastapi.md`, `nextjs.md`)

## 使用方法

### 1. 选择配置文件
根据你的项目技术栈选择需要的配置文件：
- 所有项目都建议使用 `base.md`
- 根据编程语言选择对应的 `[语言名].md`
- 根据使用的框架选择对应的 `[框架名].md`

### 2. 复制到项目
将选中的 markdown 文件复制到你的项目目录中，通常放在项目根目录。

### 3. 引用配置
在你的项目 `CLAUDE.md` 中引用这些配置文件：

```markdown
# CLAUDE.md

## 编程规范
请严格遵循以下规则文件中的所有指导：
- base.md - 通用编程最佳实践
- python.md - Python 语言规范
- fastapi.md - FastAPI 框架指导
```

### 4. 自定义配置
可以根据项目特殊需求修改和扩展复制的配置文件。

## 示例用法

### Python + FastAPI 项目
```bash
# 复制基础配置
cp base.md /your-project/
cp python.md /your-project/
cp fastapi.md /your-project/
```

### JavaScript + React 项目  
```bash
# 复制基础配置
cp base.md /your-project/
cp javascript.md /your-project/
cp react.md /your-project/
```