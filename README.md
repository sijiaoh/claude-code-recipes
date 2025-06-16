# Claude 编程规范合集

这是一个个人 Claude 设定合集，提供针对不同编程语言和框架的模块化配置文件。建议作为 git submodule 添加到项目中，并从项目的 CLAUDE.md 引用相应的规范文件。

## 项目结构

```
/
├── base.md                    # 通用编程最佳实践和基础规范
├── oop.md                     # 面向对象编程最佳实践
├── python.md                  # Python 语言特定规范
├── mvc.md                     # MVC 架构最佳实践
├── [其他语言].md             # 其他语言特定规则（根据需要扩展）
├── [其他框架].md             # 其他框架特定规则（根据需要扩展）
└── .claude/
    └── settings.local.json    # Claude Code 权限配置
```

## 文件说明

- **base.md** - 通用编程最佳实践，适用于所有项目
- **oop.md** - 面向对象编程设计原则和最佳实践
- **python.md** - Python 语言特定的编程规范和最佳实践
- **mvc.md** - MVC 架构模式的最佳实践和设计原则
- **[其他文件]** - 其他语言和框架的规范（根据需要扩展）

## 使用方法

### 1. 添加为 Submodule
```bash
# 在项目根目录添加 submodule
git submodule add https://github.com/your-username/claude-code-recipes .claude-recipes
```

### 2. 在项目 CLAUDE.md 中引用
```markdown
# CLAUDE.md

## 编程最佳实践

**重要**: Claude Code 在处理此项目时必须严格遵循以下规范：

### 自动规范检测
Claude Code 应根据项目特征自动判断并遵循相应的规范文件：

- **.claude-recipes/base.md** - 所有项目必须遵循的通用编程最佳实践
- **检测编程语言** - 根据项目中的文件扩展名、package.json、requirements.txt、pyproject.toml 等自动检测使用的编程语言，并遵循对应的语言规范文件（如 .claude-recipes/python.md）
- **检测架构模式** - 根据项目结构和代码模式检测使用的架构，并遵循对应规范：
  - 发现 Model/View/Controller 结构时遵循 .claude-recipes/mvc.md
  - 发现面向对象编程模式时遵循 .claude-recipes/oop.md
- **检测框架** - 根据依赖文件、配置文件、目录结构等检测使用的框架，并遵循对应的框架规范文件

### 规范优先级
1. **.claude-recipes/base.md** - 基础规范，始终适用
2. **语言特定规范** - 如 .claude-recipes/python.md，覆盖 base.md 中的语言相关部分
3. **架构模式规范** - 如 .claude-recipes/mvc.md、.claude-recipes/oop.md，提供架构层面的指导
4. **框架特定规范** - 具体框架的最佳实践，优先级最高
```

### 3. 更新 Submodule
```bash
# 获取最新的规范文件
git submodule update --remote .claude-recipes
```

## 示例用法

### Python + MVC 项目
```markdown
# 项目 CLAUDE.md

## 编程最佳实践
请严格遵循以下规则文件中的所有指导：
- .claude-recipes/base.md - 通用编程最佳实践
- .claude-recipes/oop.md - 面向对象编程最佳实践
- .claude-recipes/python.md - Python 语言最佳实践
- .claude-recipes/mvc.md - MVC 架构指导
```

### 面向对象项目
```markdown
# 项目 CLAUDE.md

## 编程最佳实践
请严格遵循以下规则文件中的所有指导：
- .claude-recipes/base.md - 通用编程最佳实践
- .claude-recipes/oop.md - 面向对象编程最佳实践
- .claude-recipes/python.md - Python 语言最佳实践
```

## 自定义配置

可以根据项目特殊需求在项目的 CLAUDE.md 中添加额外的项目特定规则，这些规则会与 submodule 中的通用规范一起生效。