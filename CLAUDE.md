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

这是一个个人 Claude 设定合集，提供针对不同编程语言和框架的模块化配置文件。该系统允许灵活组合 Claude 指令，以适应特定的开发环境。

## 项目结构

```
/
├── base.md                    # 核心规则和基础指令
├── [语言名].md               # 语言特定规则 (如 python.md, javascript.md)  
├── [框架名].md               # 框架特定规则 (如 react.md, django.md)
├── build.yaml                # 合并配置规则
├── build/                    # 生成的合并文件
│   └── [语言名]_[框架名].md
└── .claude/
    └── settings.local.json    # Claude Code 权限配置
```

## 合并系统

项目使用基于 YAML 的合并系统 (`build.yaml`) 来组合基础规则与语言和框架特定指令。合并配置支持：

### YAML 结构示例
```yaml
merge_rules:
  - name: "python_django"
    base_files:
      - "base.md"
    include_files:
      - "python.md"
      - "django.md"
    output: "build/python_django.md"
    order: ["base", "language", "framework"]
  
  - name: "javascript_react"
    base_files:
      - "base.md"
    include_files:
      - "javascript.md"
      - "react.md"
    output: "build/javascript_react.md"
    order: ["base", "language", "framework"]

settings:
  separator: "\n\n---\n\n"
  preserve_comments: true
  merge_headers: false
```

### 合并规则配置

- **base_files**: 总是包含的基础文件 (通常是 `base.md`)
- **include_files**: 要合并的特定语言/框架文件
- **output**: 构建目录中的目标文件路径
- **order**: 合并优先级顺序 (基础 → 语言 → 框架)
- **separator**: 在合并部分之间插入的文本
- **preserve_comments**: 在合并文件中保留 YAML/HTML 注释
- **merge_headers**: 合并重复的标题部分

## 文件命名约定

- `base.md` - 核心 Claude 指令和通用规则
- `[语言名].md` - 语言特定规则 (如 `python.md`, `rust.md`)
- `[框架名].md` - 框架特定规则 (如 `fastapi.md`, `nextjs.md`)
- `[语言名]_[框架名].md` - 在 `build/` 中生成的合并文件

## 使用方法

1. **添加新语言规则**: 创建包含语言特定指令的 `[语言名].md`
2. **添加框架规则**: 创建包含框架特定指令的 `[框架名].md`
3. **配置合并**: 在 `build.yaml` 中更新新的合并组合
4. **构建合并文件**: 运行构建过程生成合并配置
5. **使用合并文件**: 在项目中引用生成的文件如 `build/python_django.md`

## 开发命令

随着系统演进，在此部分添加构建/合并命令：

```bash
# 示例构建命令 (待实现)
# python build.py
# 或
# npm run build
```

## 权限配置

仓库配置了 Claude Code 权限，允许：
- 读写 markdown 文件的文件操作
- 构建过程执行
- 合并操作的目录遍历