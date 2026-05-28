---
name: moonbit-toolchain
description: "MoonBit 工具链技能。提供 moon 构建工具、项目管理、包管理、格式化、文档生成等完整参考。TRIGGER when: 用户使用 moon 命令、创建 MoonBit 项目、管理依赖、格式化代码、生成文档、或需要了解 MoonBit 工具链时触发。SKIP: 纯 MoonBit 代码编写（使用 moonbit-language 技能）。"
---

# MoonBit 工具链技能

本技能提供 MoonBit 工具链的完整参考，包括构建工具、项目管理、包管理等核心内容。

## 触发条件

当用户执行以下操作时触发此技能：
- 使用 `moon` 命令
- 创建新的 MoonBit 项目
- 管理项目依赖
- 格式化代码
- 生成文档
- 配置工作区

## 跳过条件

以下情况不触发此技能：
- 纯 MoonBit 代码编写（使用 `moonbit-language` 技能）
- 仅询问测试（使用 `moonbit-testing` 技能）
- 仅询问 FFI（使用 `moonbit-ffi` 技能）

---

## 核心命令

### 项目创建

```bash
# 创建新模块
moon new <PATH>

# 指定用户名和模块名
moon new --user <USER> --name <NAME> <PATH>
```

### 构建与检查

```bash
# 构建当前包
moon build

# 监视文件变化并自动构建
moon build --watch

# 检查当前包（不构建对象文件）
moon check

# 监视文件变化并自动检查
moon check --watch

# 检查特定包
moon check --package-path <PACKAGE_PATH>

# 以 JSON 格式输出
moon check --output-json

# 解释错误码
moon check --explain

# 检查单个文件
moon check <SINGLE_FILE.mbt>
```

### 运行与测试

```bash
# 运行主包
moon run <PACKAGE_OR_MBT_FILE> [ARGS]...

# 只构建不运行
moon run --build-only <PACKAGE>

# 测试当前包
moon test

# 测试特定包
moon test --package <PACKAGE>

# 测试特定文件
moon test --package <PACKAGE> --file <FILE>

# 运行特定测试（按索引）
moon test --package <PACKAGE> --file <FILE> --index <INDEX>

# 更新快照
moon test --update

# 只构建不运行测试
moon test --build-only

# 顺序执行（不并行）
moon test --no-parallelize

# 以 JSON 格式输出失败信息
moon test --test-failure-json

# 运行文档测试
moon test --doc
```

### 代码格式化

```bash
# 格式化源代码
moon fmt

# 只检查不修改
moon fmt --check

# 排序输入文件
moon fmt --sort-input

# 设置块样式
moon fmt --block-style true
```

### 文档生成

```bash
# 生成文档
moon doc

# 启动文档服务器
moon doc --serve

# 绑定地址
moon doc --bind <BIND>

# 端口
moon doc --port <PORT>
```

### 包管理

```bash
# 添加依赖
moon add <PACKAGE_PATH>

# 添加二进制依赖
moon add --bin <PACKAGE_PATH>

# 移除依赖
moon remove <PACKAGE_PATH>

# 安装依赖
moon install

# 显示依赖树
moon tree

# 更新包注册索引
moon update
```

### 发布

```bash
# 登录账户
moon login

# 注册账户
moon register

# 发布当前模块
moon publish

# 打包当前模块
moon package
```

### 其他命令

```bash
# 清理构建目录
moon clean

# 生成公共接口文件
moon info

# 运行基准测试
moon bench

# 代码覆盖率
moon coverage analyze
moon coverage report
moon coverage clean

# 升级工具链
moon upgrade

# 生成 shell 补全
moon shell-completion --shell bash
moon shell-completion --shell powershell

# 打印版本信息
moon version
```

---

## 通用选项

以下选项在 build/check/run/test/bench 中通用：

| 选项 | 说明 |
|------|------|
| `--std` | 启用标准库（默认） |
| `--nostd` | 禁用标准库 |
| `-g`, `--debug` | 发出调试信息 |
| `--release` | 发布模式编译 |
| `--strip` / `--no-strip` | 启用/禁用剥离调试信息 |
| `--target <TARGET>` | 输出目标：`wasm`, `wasm-gc`, `js`, `native`, `llvm`, `all` |
| `--enable-coverage` | 启用覆盖率插桩 |
| `--sort-input` | 排序输入文件 |
| `--output-wat` | 输出 WAT 而不是 WASM |
| `-d`, `--deny-warn` | 将所有警告视为错误 |
| `--no-render` | 不渲染来自 moonc 的诊断 |
| `--warn-list <WARN_LIST>` | 警告列表配置 |
| `--alert-list <ALERT_LIST>` | 警报列表配置 |
| `-j`, `--jobs <JOBS>` | 最大并行任务数 |
| `--frozen` | 不同步依赖；假设本地依赖是最新的 |

---

## 项目结构

### 工作区（Workspace）

工作区是 MoonBit 项目的顶层组织单位：

```
my-project/
├── moon.mod.json          # 模块配置
├── moon.pkg               # 包配置
├── src/
│   ├── main.mbt           # 主入口
│   └── lib.mbt            # 库代码
└── test/
    └── main_test.mbt      # 测试代码
```

### 模块配置（moon.mod.json）

```json
{
  "name": "username/module-name",
  "version": "0.1.0",
  "readme": "README.md",
  "repository": "",
  "license": "MIT",
  "keywords": [],
  "deps": {
    "username/dependency": "0.1.0"
  }
}
```

### 包配置（moon.pkg）

```json
{
  "import": ["username/dependency"],
  "wbtest-import": ["username/test-dependency"],
  "test-import": ["username/test-dependency"]
}
```

---

## 包管理详解

### 添加依赖

```bash
# 添加最新版本
moon add username/package

# 添加特定版本
moon add username/package@0.1.0

# 添加二进制依赖
moon add --username/package
```

### 依赖树

```bash
# 查看完整依赖树
moon tree

# 输出示例：
# my-project
# ├── username/dependency@0.1.0
# │   └── another/dep@0.2.0
# └── username/other-dep@0.3.0
```

### 版本约束

MoonBit 使用语义化版本控制：

- `0.1.0` — 精确版本
- `^0.1.0` — 兼容版本（默认）
- `>=0.1.0` — 最小版本
- `0.1.0..0.2.0` — 版本范围

---

## 目标平台

### Wasm

```bash
moon build --target wasm
moon build --target wasm-gc
```

### JavaScript

```bash
moon build --target js
```

生成的 JS 文件可以是 CommonJS、ES 模块或 IIFE，根据配置决定。

### Native

```bash
moon build --target native
```

### LLVM（实验性）

```bash
moon build --target llvm
```

---

## 调试与诊断

### 调试信息

```bash
# 启用调试信息
moon build --debug

# 发布模式（优化）
moon build --release
```

### 错误诊断

```bash
# 检查错误
moon check

# 输出 JSON 格式
moon check --output-json

# 解释错误码
moon check --explain

# 不渲染诊断
moon check --no-render
```

### 警告配置

```bash
# 将警告视为错误
moon build --deny-warn

# 配置警告列表
moon build --warn-list "unused-var,shadowed-var"
```

---

## 最佳实践

1. **使用 `moon new` 创建项目**：确保正确的项目结构
2. **定期运行 `moon test`**：保持代码质量
3. **使用 `moon fmt`**：保持代码风格一致
4. **使用 `moon doc`**：生成和查看文档
5. **使用 `moon tree`**：了解依赖关系
6. **使用 `--watch` 模式**：开发时自动构建/检查
7. **使用 `--release` 构建发布版本**：启用优化

---

## 常见陷阱

1. **依赖版本冲突**：使用 `moon tree` 检查依赖关系
2. **构建缓存**：使用 `moon clean` 清理构建缓存
3. **目标平台差异**：不同后端的 ABI 可能不同
4. **快照测试**：使用 `--update` 更新快照
5. **并行测试**：默认并行运行，确保测试独立
