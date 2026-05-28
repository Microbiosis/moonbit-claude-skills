---
name: moonbit-ffi
description: "MoonBit FFI（外部函数接口）互操作技能。提供 MoonBit 与 JavaScript、C、Wasm 等后端的互操作参考，包括外部类型声明、函数导入导出、回调处理、内存管理等。TRIGGER when: 用户需要 MoonBit 与外部语言互操作、声明外部函数、处理 FFI 回调、或调试 FFI 相关问题时触发。SKIP: 纯 MoonBit 代码开发。"
---

# MoonBit FFI 互操作技能

本技能提供 MoonBit 与外部语言（JavaScript、C、Wasm）互操作的完整参考。

## 触发条件

当用户执行以下操作时触发此技能：
- 声明外部函数或类型
- 实现 MoonBit 与 JavaScript/C 的互操作
- 处理 FFI 回调
- 调试 FFI 相关问题
- 了解不同后端的 ABI 差异

## 跳过条件

以下情况不触发此技能：
- 纯 MoonBit 代码开发（使用 `moonbit-language` 技能）
- 仅询问测试（使用 `moonbit-testing` 技能）

---

## 后端支持

MoonBit 支持五个后端：

| 后端 | 说明 | FFI 支持 |
|------|------|----------|
| Wasm | WebAssembly | ✓ |
| Wasm GC | Wasm GC 提案 | ✓ |
| JavaScript | JS 代码生成 | ✓ |
| C | C 代码生成 | ✓ |
| LLVM | 实验性 | ✗ |

---

## 声明外部类型

使用 `#external` 属性声明外部类型：

```moonbit
#external
type ExternalRef
```

| 后端 | 解释 |
|------|------|
| Wasm / Wasm GC | `externref` |
| JavaScript | JavaScript 值 |
| C | `void*` |

---

## 声明外部函数

### Wasm & Wasm GC

**从宿主导入：**

```moonbit
fn cos(d : Double) -> Double = "math" "cos"
```

**内联 Wasm 语法：**

```moonbit
extern "wasm" fn identity(d : Double) -> Double =
  #|(func (param f64) (result f64))
```

### JavaScript

**按模块和函数名导入：**

```moonbit
fn cos(d : Double) -> Double = "Math" "cos"
```

生成类似 `(d) => Math.cos(d)` 的代码。

**内联 JavaScript lambda：**

```moonbit
extern "js" fn cos(d : Double) -> Double =
  #|(d) => Math.cos(d)
```

### C

**按函数名导入：**

```moonbit
extern "C" fn put_char(ch : UInt) -> Unit = "function_name"
```

**动态链接配置（moon.pkg）：**

```json
{
  "cc-link-flags": "-lm"
}
```

**使用 C 桩文件：**

```json
{
  "native-stub": ["stub.c"]
}
```

在桩文件中包含 `moonbit.h`（位于 `~/.moon/include`）以获取类型定义和工具函数。

---

## 类型表示（ABI）

### Wasm & Wasm GC

| MoonBit 类型 | ABI |
|---|---|
| `Bool` | `i32` |
| `Int` | `i32` |
| `UInt` | `i32` |
| `Int64` | `i64` |
| `UInt64` | `i64` |
| `Float` | `f32` |
| `Double` | `f64` |
| 常量 `enum` | `i32` |
| `#external type T` | `externref` |
| `FuncRef[T]` | `funcref` |

### JavaScript

| MoonBit 类型 | ABI |
|---|---|
| `Bool` | `boolean` |
| `Int`, `UInt`, `Float`, `Double` | `number` |
| 常量 `enum` | `number` |
| `#external type T` | `any` |
| `String` | `string` |
| `FixedArray[Byte]`/`Bytes` | `Uint8Array` |
| `FixedArray[T]`/`Array[T]` | `T[]` |
| `FuncRef[T]` | `Function` |

### C

| MoonBit 类型 | ABI |
|---|---|
| `Bool`, `Int` | `int32_t` |
| `UInt` | `uint32_t` |
| `Int64` | `int64_t` |
| `UInt64` | `uint64_t` |
| `Float` | `float` |
| `Double` | `double` |
| 常量 `enum` | `int32_t` |
| 抽象类型 (`type T`) | 指针（必须是有效的 MoonBit 对象） |
| `#external type T` | `void*` |
| `FixedArray[Byte]`/`Bytes` | `uint8_t*` |
| `FixedArray[T]` | `T*` |
| `FuncRef[T]` | 函数指针 |

---

## 回调处理

### FuncRef[T] — 闭包（非捕获）函数

表示不捕获自由变量的函数。如果在期望 `FuncRef[T]` 的地方使用闭包，会产生类型错误。

```moonbit
fn callback(f : FuncRef[Int -> Int]) -> Int {
  f(42)
}
```

### 闭包（捕获函数）

**Wasm/Wasm GC：**

回调作为 `externref` 传递。模块导入 `moonbit:ffi`/`make_closure` — 一个将 funcref 与闭包对象绑定的宿主函数。

宿主实现示例：
```javascript
{
  "moonbit:ffi": {
    "make_closure": (funcref, closure) => funcref.bind(null, closure)
  }
}
```

**JavaScript：**

原生支持闭包，无需特殊处理。

**C：**

使用 `FuncRef` 的技巧将回调函数与闭包数据分离，利用 C 的模式将额外数据与回调一起传递。

---

## 自定义整数值（常量枚举）

```moonbit
enum SpecialNumbers {
  Zero = 0
  One
  Two
  Three
  Ten = 10
  FourtyTwo = 42
}
```

未指定的构造函数默认为前一个值加一（第一个为零）。这对于绑定 C 库标志特别有用。

---

## 导出函数

公开的、非方法的、非多态函数可以通过 `moon.pkg` 链接配置导出：

```json
{
  "options": {
    "link": {
      "wasm": {
        "exports": ["add", "fib:test"]
      },
      "js": {
        "exports": ["add", "fib:test"],
        "format": "esm"
      }
    }
  }
}
```

这里 `fib` 以名称 `test` 导出。JS 后端还支持 `format` 选项（`cjs`、`esm` 或 `iife`）。

---

## 生命周期管理

MoonBit 对 Wasm 和 C 后端使用**引用计数**，对 Wasm GC 和 JavaScript 后端使用**运行时 GC**。

### 外部对象（仅 C 后端）

`moonbit.h` 提供 `moonbit_make_external_object` 用于通过 MoonBit 的内存管理管理外部资源生命周期。当对象不再存活时，提供的 `finalize` 函数会被调用。

```c
void* moonbit_make_external_object(
  void* data,
  void (*finalize)(void*)
);
```

finalize 函数"不能丢弃对象本身，因为这由 MoonBit 运行时处理"。

### MoonBit 对象传递给宿主（C 和 Wasm 后端）

MoonBit 默认使用**所有权调用约定** — 被调用者负责 `decref` 其参数。

| 事件 | 操作 |
|---|---|
| 读取字段/元素 | 无操作 |
| 存储到数据结构 | `incref` |
| 传递给 MoonBit 函数 | `incref` |
| 传递给其他外部函数 | 无操作 |
| 返回 | 无操作 |
| 作用域结束（未返回） | `decref` |

### `#borrow` 和 `#owned` 属性

这些属性控制 FFI 参数的所有权语义。默认正在迁移到 `#borrow`。

**`#borrow`：** 调用的函数不需要对参数进行 `decref`。当 FFI 函数仅在本地读取时有用。

| 事件 | 操作 |
|---|---|
| 读取字段/元素 | 无操作 |
| 存储到数据结构 | `incref` |
| 传递给 MoonBit 函数 | `incref` |
| 传递给其他 C / `#borrow` 函数 | 无操作 |
| 返回 | `incref` |
| 作用域结束（未返回） | 无操作 |

**`#owned`：** 参数被 FFI 函数存储，需要稍后手动 `decref`。常见用例：注册回调时闭包是 owned 的。

### 托管类型

**始终拆箱（无需生命周期管理）：** 内置数值类型（`Int`、`Double` 等）和常量枚举。

**始终装箱并引用计数：** `FixedArray[T]`、`Bytes`、`String` 和抽象类型（`type T`）。

**外部类型**（`#external type T`）是装箱的但表示外部指针 — MoonBit 不对它们执行引用计数。

---

## 最佳实践

1. **选择合适的后端**：根据目标平台选择 Wasm、JS 或 C
2. **类型安全**：使用 `#external` 声明外部类型，避免类型转换错误
3. **内存管理**：理解所有权语义，正确使用 `#borrow` 和 `#owned`
4. **错误处理**：外部函数应该返回 `Unit` 当没有返回值时
5. **测试 FFI**：编写测试验证外部函数的行为

---

## 常见陷阱

1. **不支持多态外部函数**：MoonBit 不支持多态的外部函数
2. **LLVM 后端不支持 FFI**：实验性的 LLVM 后端不支持 FFI
3. **引用计数 vs GC**：不同后端的内存管理策略不同
4. **闭包 vs FuncRef**：理解两者的区别和使用场景
5. **ABI 差异**：不同后端的类型表示可能不同
