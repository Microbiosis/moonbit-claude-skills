---
name: moonbit-language
description: "MoonBit 语言开发技能。提供 MoonBit 语言的核心语法、数据类型、控制结构、错误处理、方法与 traits、泛型、模式匹配等完整参考。TRIGGER when: 用户编写 MoonBit 代码、询问 MoonBit 语法、需要 MoonBit 代码示例、调试 MoonBit 程序、或需要了解 MoonBit 语言特性时触发。SKIP: 非 MoonBit 语言的开发任务。"
---

# MoonBit 语言开发技能

本技能提供 MoonBit 语言的完整开发参考，包括语法、数据类型、控制结构、错误处理、方法系统等核心内容。

## 触发条件

当用户执行以下操作时触发此技能：
- 编写或修改 MoonBit 代码（`.mbt` 文件）
- 询问 MoonBit 语言语法或特性
- 需要 MoonBit 代码示例
- 调试 MoonBit 程序
- 了解 MoonBit 的类型系统、错误处理、模式匹配等

## 跳过条件

以下情况不触发此技能：
- 非 MoonBit 语言的开发任务
- 仅询问工具链配置（使用 `moonbit-toolchain` 技能）
- 仅询问 FFI 互操作（使用 `moonbit-ffi` 技能）
- 仅询问测试相关（使用 `moonbit-testing` 技能）

---

## 内置数据类型

### Unit 类型
`Unit` 表示"没有有意义的值"，仅有一个值写作 `()`。类似 C/Java 的 `void`，但是真实类型。

```moonbit
fn do_nothing() -> Unit {
  ()
}
```

### Boolean
布尔类型有两个值：`true` 和 `false`。使用 `!` 取反。

```moonbit
let a = true
let b = !a  // false
```

### 数值类型

| 类型 | 描述 | 示例 |
|------|------|------|
| `Int16` | 16位有符号整数 | `(42 : Int16)` |
| `Int` | 32位有符号整数（默认） | `42` |
| `Int64` | 64位有符号整数 | `1000L` |
| `UInt16` | 16位无符号整数 | `(14 : UInt16)` |
| `UInt` | 32位无符号整数 | `14U` |
| `UInt64` | 64位无符号整数 | `14UL` |
| `Double` | 64位浮点数（IEEE754） | `3.14` |
| `Float` | 32位浮点数 | `(3.14 : Float)` |
| `BigInt` | 任意大整数 | `10000000000000000000000N` |

数值字面量支持：
- 十进制：`42`
- 二进制：`0b101010`
- 八进制：`0o52`
- 十六进制：`0x2A`
- 下划线增强可读性：`1_000_000`

### String
`String` 持有 UTF-16 码单元序列。

```moonbit
let s1 = "Hello, World!"
let s2 = #|
  这是原始字符串
  不需要转义
|#
let name = "MoonBit"
let s3 = $"Hello, {name}!"  // 字符串插值
```

### Char
`Char` 表示一个 Unicode 码点。

```moonbit
let c = 'A'
```

### Byte 与 Bytes

```moonbit
let b = b'A'  // Byte 类型
let bs = b"Hello"  // Bytes 类型
```

### Tuple
元组用圆括号 `()` 构造，元素有序。

```moonbit
let t = (1, true, "hello")
let (a, b, c) = t  // 模式匹配解构
```

### Option 与 Result

```moonbit
let some_value : Int? = Some(42)
let none_value : Int? = None

let ok_result : Result[Int, String] = Ok(42)
let err_result : Result[Int, String] = Err("error")
```

### Array

```moonbit
let arr = [1, 2, 3, 4, 5]
let first = arr[0]  // 索引访问
let slice = arr[1:3]  // 切片
```

### Map

```moonbit
let m = Map::of([("a", 1), ("b", 2)])
let value = m["a"]  // Some(1)
```

---

## 函数

### 顶层函数

```moonbit
fn add(x : Int, y : Int) -> Int {
  x + y
}
```

### 命名参数

```moonbit
fn greet(name~ : String, greeting~ : String) -> String {
  $"\{greeting}, \{name}!"
}

// 调用
let msg = greet(name="World", greeting="Hello")
let msg2 = greet(greeting="Hi", name="MoonBit")  // 可以任意顺序
```

### 可选参数

```moonbit
fn create_user(name~ : String, age? : Int = 0) -> String {
  $"\{name} is \{age} years old"
}
```

### 箭头函数

```moonbit
let add = fn(x, y) { x + y }
let add_short = (x, y) => x + y
```

### 部分应用

```moonbit
let add5 = add(5, _)  // 部分应用
let result = add5(3)  // 8
```

---

## 控制结构

### 条件表达式

```moonbit
let x = 10
let result = if x > 0 {
  "positive"
} else {
  "non-positive"
}
```

### Match 表达式

```moonbit
let x = 1
let desc = match x {
  0 => "zero"
  1 => "one"
  _ => "other"
}
```

### While 循环

```moonbit
let mut i = 0
while i < 10 {
  i = i + 1
  if i == 5 { continue }
  if i == 8 { break }
}
```

### For 循环

```moonbit
for i = 0; i < 10; i = i + 1 {
  // C 风格 for 循环
}
```

### for .. in 循环

```moonbit
for x in [1, 2, 3, 4, 5] {
  // 遍历数组
}

for i in 0..<10 {
  // 范围遍历
}

for (k, v) in Map::of([("a", 1), ("b", 2)]) {
  // 遍历 Map
}
```

### 列表推导

```moonbit
let squares = [for i in 0..<10 => i * i]
let evens = [for i in 0..<100 if i % 2 == 0 => i]
```

### Defer 表达式

```moonbit
fn read_file() -> Unit {
  let file = open_file()
  defer {
    close_file(file)  // 无论如何都会执行
  }
  // 使用文件
}
```

---

## 自定义数据类型

### Struct

```moonbit
struct Point {
  x : Double
  y : Double
  mut z : Double  // 可变字段
}

// 创建
let p = { x: 1.0, y: 2.0, z: 3.0 }

// 访问
let x = p.x

// 修改可变字段
p.z = 4.0

// 结构体更新语法
let p2 = { ..p, x: 10.0 }
```

### Enum

```moonbit
enum Color {
  Red
  Green
  Blue
  RGB(Int, Int, Int)
}

let c = Color::RGB(255, 0, 0)
match c {
  Color::Red => "red"
  Color::Green => "green"
  Color::Blue => "blue"
  Color::RGB(r, g, b) => $"\{r}, \{g}, \{b}"
}
```

### Type Alias

```moonbit
type UserId = Int
type UserName = String
```

---

## 模式匹配

### 基本模式

```moonbit
match x {
  0 => "zero"
  1 | 2 => "one or two"  // 或模式
  _ => "other"  // 通配符
}
```

### 数组模式

```moonbit
match arr {
  [] => "empty"
  [x] => $"single: \{x}"
  [x, y, ..] => $"starts with \{x} and \{y}"
}
```

### 守卫条件

```moonbit
match x {
  n if n > 0 => "positive"
  n if n < 0 => "negative"
  _ => "zero"
}
```

### is 表达式

```moonbit
if x is Some(value) {
  // 使用 value
}

if x is [first, ..rest] {
  // 使用 first 和 rest
}
```

---

## 错误处理

### 抛出错误

```moonbit
enum MyError {
  NotFound
  InvalidInput(String)
}

fn process(input : String) -> String!MyError {
  if input.is_empty() {
    raise MyError::InvalidInput("empty input")
  }
  "ok"
}
```

### try ... catch

```moonbit
fn handle_error() -> String {
  try {
    process("")
  } catch {
    MyError::NotFound => "not found"
    MyError::InvalidInput(msg) => $"invalid: \{msg}"
  }
}
```

### 转换为 Result

```moonbit
let result = try? process("input")  // Result[String, MyError]
```

### Failure 内置错误

```moonbit
fn risky() -> String! {
  fail("something went wrong")  // 使用内置 Failure 类型
}
```

---

## 方法与 Traits

### 定义方法

```moonbit
struct Point {
  x : Double
  y : Double
}

fn Point::distance(self : Point, other : Point) -> Double {
  let dx = self.x - other.x
  let dy = self.y - other.y
  (dx * dx + dy * dy).sqrt()
}

// 使用
let p1 = { x: 0.0, y: 0.0 }
let p2 = { x: 3.0, y: 4.0 }
let d = p1.distance(p2)  // 5.0
```

### 定义 Trait

```moonbit
trait Show {
  to_string(Self) -> String
}

impl Show for Point with to_string(self : Point) -> String {
  $"(\{self.x}, \{self.y})"
}
```

### 内置 Traits

```moonbit
// Eq - 相等性比较
trait Eq {
  op_equal(Self, Self) -> Bool
}

// Compare - 比较
trait Compare : Eq {
  compare(Self, Self) -> Int
}

// Hash - 哈希
trait Hash {
  hash_combine(Self, Hasher) -> Unit
}

// Show - 显示
trait Show {
  output(Self, Logger) -> Unit
  to_string(Self) -> String
}

// Default - 默认值
trait Default {
  default() -> Self
}
```

### Derive 自动实现

```moonbit
struct Point {
  x : Double
  y : Double
} derive(Eq, Hash, Show, Compare)
```

---

## 泛型

```moonbit
fn identity[T](x : T) -> T {
  x
}

struct Pair[A, B] {
  first : A
  second : B
}

fn Pair::swap[A, B](self : Pair[A, B]) -> Pair[B, A] {
  { first: self.second, second: self.first }
}
```

---

## 特殊语法

### 管道操作符

```moonbit
let result = x |> f(y)  // 等价于 f(x, y)
let result2 = f <| x    // 等价于 f(x)
```

### 级联操作符

```moonbit
let arr = []
arr..push(1)
   ..push(2)
   ..push(3)
// 等价于 { arr.push(1); arr.push(2); arr.push(3); arr }
```

### 展开操作符

```moonbit
let arr1 = [1, 2, 3]
let arr2 = [4, 5, 6]
let combined = [..arr1, ..arr2]  // [1, 2, 3, 4, 5, 6]
```

### 正则字面量

```moonbit
let pattern = re"\d+"
if "hello123" =~ pattern {
  // 匹配成功
}
```

### TODO 语法

```moonbit
fn not_implemented_yet() -> Int {
  ...  // 编译器会标记为未实现
}
```

---

## 迭代器

```moonbit
let iter = [1, 2, 3, 4, 5].iter()
let doubled = iter.map(fn(x) { x * 2 })
let sum = iter.fold(init=0, fn(acc, x) { acc + x })
```

---

## 最佳实践

1. **使用显式类型标注**：顶层函数的参数和返回值需要显式类型
2. **善用模式匹配**：比 if-else 更安全，编译器会检查完整性
3. **使用 `Option` 和 `Result`**：而不是抛出异常
4. **利用 derive**：自动实现常用 traits
5. **使用命名参数**：提高代码可读性
6. **使用管道操作符**：让数据流更清晰

---

## 常见陷阱

1. **数组模式中的 `..`**：只能出现一次
2. **字符串插值**：表达式不能包含换行、`{}` 或 `"`
3. **FixedArray 初始化**：`FixedArray::make(size, value)` 会共享同一对象，使用 `makei` 避免
4. **迭代器是单次使用**：消耗后需重新获取
5. **Map 字面量的键**：必须是常量
