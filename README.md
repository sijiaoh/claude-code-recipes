# Claude 编程规范合集

这是一个个人 Claude 设定合集，提供针对不同编程语言和框架的模块化配置文件。建议作为 git submodule 添加到项目中，并从项目的 CLAUDE.md 引用相应的规范文件。

## 项目结构

```
/
├── base.md                    # 通用编程最佳实践和基础规范
├── oop.md                     # 面向对象编程最佳实践
├── python.md                  # Python 语言特定规范
├── typescript.md              # TypeScript 语言特定规范
├── mvc.md                     # MVC 架构最佳实践
└── .claude/
    └── settings.local.json    # Claude Code 权限配置
```

## 文件说明

- **base.md** - 通用编程最佳实践，适用于所有项目
- **oop.md** - 面向对象编程设计原则和最佳实践
- **python.md** - Python 语言特定的编程规范和最佳实践
- **typescript.md** - TypeScript 语言特定的编程规范和最佳实践
- **mvc.md** - MVC 架构模式的最佳实践和设计原则

## 使用方法

### 1. 添加为 Submodule
```bash
# 在项目根目录添加 submodule
git submodule add https://github.com/sijiaoh/claude-code-recipes .claude-recipes
```

### 2. 在项目 CLAUDE.md 中引用
```markdown
# CLAUDE.md

## 编程最佳实践
请严格遵循以下规则文件中的所有指导：
- .claude-recipes/base.md - 通用编程最佳实践
- .claude-recipes/oop.md - 面向对象编程最佳实践
- .claude-recipes/python.md - Python 语言最佳实践
- .claude-recipes/mvc.md - MVC 架构指导
```

### 3. 更新 Submodule
```bash
# 获取最新的规范文件
git submodule update --remote .claude-recipes
```

## 示例用法

### TypeScript + MVC 项目
```markdown
# 项目 CLAUDE.md

## 编程最佳实践
请严格遵循以下规则文件中的所有指导：
- .claude-recipes/base.md - 通用编程最佳实践
- .claude-recipes/oop.md - 面向对象编程最佳实践
- .claude-recipes/typescript.md - TypeScript 语言最佳实践
- .claude-recipes/mvc.md - MVC 架构指导
```

### 简单项目（仅基础规范）
```markdown
# 项目 CLAUDE.md

## 编程最佳实践
请严格遵循以下规则文件中的所有指导：
- .claude-recipes/base.md - 通用编程最佳实践
```

## 自定义配置

可以根据项目特殊需求在项目的 CLAUDE.md 中添加额外的项目特定规则，这些规则会与 submodule 中的通用规范一起生效。