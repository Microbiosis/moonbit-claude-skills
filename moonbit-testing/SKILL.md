---
name: moonbit-testing
description: "MoonBit 测试开发技能。提供 MoonBit 测试框架、单元测试、快照测试、黑盒/白盒测试等完整参考。TRIGGER when: 用户编写 MoonBit 测试、运行测试、调试测试失败、或需要了解 MoonBit 测试最佳实践时触发。SKIP: 非 MoonBit 语言的测试任务。"
---

# MoonBit 测试开发技能

本技能提供 MoonBit 测试框架的完整参考，包括测试编写、快照测试、黑盒/白盒测试等核心内容。

## 触发条件

当用户执行以下操作时触发此技能：
- 编写 MoonBit 测试代码
- 运行 `moon test` 命令
- 调试测试失败
- 更新快照测试
- 了解 MoonBit 测试最佳实践

## 跳过条件

以下情况不触发此技能：
- 非 MoonBit 语言的测试任务
- 仅询问语言语法（使用 `moonbit-language` 技能）

---

## 测试代码块（Test Blocks）

MoonBit 提供 `test` 代码块用于编写内联测试。它本质上是一个返回 `Unit` 但可能抛出 `Error` 的函数。

### 基本语法

```moonbit
test "测试名称" {
  // 测试代码
  assert_eq(1 + 1, 2)
}
```

### 断言函数

```moonbit
// 相等性断言
assert_eq(actual, expected)

// 不等断言
assert_ne(actual, unexpected)

// 布尔断言
assert(condition)
assert(!condition)
```

### 测试 panic

测试名称以 `"panic"` 开头时，表示期望触发 panic：

```moonbit
test "panic: 除以零" {
  let _ = 1 / 0  // 期望 panic
}
```

---

## 快照测试（Snapshot Tests）

MoonBit 提供三种快照测试，均可通过 `moon test --update` 自动插入或更新。

### 1. Show 快照

使用 `inspect` 检查实现了 `Show` trait 的内容：

```moonbit
test "show 快照" {
  let point = { x: 1.0, y: 2.0 }
  inspect(point, content="point")
}
```

运行 `moon test --update` 后会自动生成快照文件。

### 2. JSON 快照

对于复杂数据结构，使用 JSON 快照更易读：

```moonbit
struct Config {
  name : String
  version : Int
  debug : Bool
} derive(ToJson)

test "json 快照" {
  let config = { name: "MyApp", version: 1, debug: true }
  @json.inspect(config, content=config)
}
```

### 3. 通用快照

使用 `@test.T::write` 和 `@test.T::writeln` 记录整个过程的输出：

```moonbit
test "通用快照" {
  @test.T::writeln("步骤 1: 初始化")
  @test.T::writeln("步骤 2: 处理")
  @test.T::writeln("步骤 3: 完成")
  @test.T::snapshot()  // 必须在末尾使用
}
```

快照文件保存在包的 `__snapshot__` 目录下。

---

## 黑盒测试与白盒测试

| 类型 | 访问范围 | 文件命名 |
|------|----------|----------|
| **白盒测试** | 包中所有成员 | 内联或 `*_wbtest.mbt` |
| **黑盒测试** | 仅公开成员 | `*_test.mbt` |

### 白盒测试（WhiteBox）

可访问包内所有成员，适合测试内部实现细节：

```moonbit
// 文件: internal_wbtest.mbt
test "测试内部函数" {
  // 可以访问私有函数和类型
  let result = internal_helper()
  assert_eq(result, expected)
}
```

### 黑盒测试（BlackBox）

仅访问公开成员，适合验证库的公共 API：

```moonbit
// 文件: api_test.mbt
test "测试公共 API" {
  // 只能访问公开的函数和类型
  let result = public_function()
  assert_eq(result, expected)
}
```

---

## 测试配置

### 包配置（moon.pkg）

```json
{
  "import": ["dependency1", "dependency2"],
  "wbtest-import": ["test-dependency1"],
  "test-import": ["test-dependency2"]
}
```

- `import`: 正常导入的包
- `wbtest-import`: 白盒测试额外导入的包
- `test-import`: 黑盒测试额外导入的包

---

## 运行测试

### 基本命令

```bash
# 运行所有测试
moon test

# 运行特定包的测试
moon test --package package_name

# 运行特定文件的测试
moon test --package package_name --file test_file.mbt

# 运行特定测试（按索引）
moon test --package package_name --file test_file.mbt --index 0

# 更新快照
moon test --update

# 限制快照更新次数
moon test --update --limit 100

# 只构建不运行
moon test --build-only

# 顺序执行（不并行）
moon test --no-parallelize

# 以 JSON 格式输出失败信息
moon test --test-failure-json
```

### 运行文档测试

```bash
moon test --doc
```

---

## 测试最佳实践

### 1. 测试命名规范

```moonbit
// 好的命名：描述行为
test "空数组应该返回长度 0" {
  assert_eq([].length(), 0)
}

// 好的命名：描述边界情况
test "负数输入应该抛出错误" {
  // ...
}

// panic 测试必须以 "panic" 开头
test "panic: 无效输入" {
  // ...
}
```

### 2. 使用快照测试

对于复杂输出，使用快照测试比手动断言更易维护：

```moonbit
test "复杂结构输出" {
  let result = process_complex_data()
  inspect(result, content=result)
}
```

### 3. 测试错误处理

```moonbit
test "错误处理" {
  let result = try? risky_operation()
  match result {
    Ok(value) => assert_eq(value, expected)
    Err(_) => assert(false)
  }
}
```

### 4. 测试边界条件

```moonbit
test "边界条件：空输入" {
  assert_eq(process(""), default_value)
}

test "边界条件：最大值" {
  assert_eq(process(Int::max_value()), expected)
}
```

### 5. 使用参数化测试（列表推导）

```moonbit
test "参数化测试" {
  let cases = [(1, "one"), (2, "two"), (3, "three")]
  for (input, expected) in cases {
    assert_eq(to_string(input), expected)
  }
}
```

---

## 调试测试失败

### 查看详细输出

```bash
# 查看失败的详细信息
moon test --test-failure-json

# 查看快照差异
moon test --update  # 会显示差异
```

### 更新快照

当预期输出改变时，更新快照：

```bash
moon test --update
```

### 临时跳过测试

使用 `skip` 属性临时跳过测试：

```moonbit
#skip
test "暂时跳过的测试" {
  // ...
}
```

---

## 测试覆盖率

### 生成覆盖率报告

```bash
# 运行覆盖率分析
moon coverage analyze

# 生成报告
moon coverage report

# 清理覆盖率数据
moon coverage clean
```

### 启用覆盖率

在构建时启用覆盖率：

```bash
moon build --enable-coverage
moon test --enable-coverage
```

---

## 常见陷阱

1. **快照文件位置**：快照保存在 `__snapshot__` 目录，需要提交到版本控制
2. **快照更新**：使用 `--update` 更新快照，不要手动修改
3. **测试并行化**：默认并行运行，确保测试之间没有依赖
4. **panic 测试**：名称必须以 `"panic"` 开头
5. **`@test.T::snapshot`**：必须在测试块末尾使用，因为它总是抛出异常
