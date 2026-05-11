---
tags:
  - Web前端
  - WebAssembly
  - WASM
  - 性能
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# WebAssembly (WASM)

## What — 什么是 WebAssembly

WebAssembly（WASM）是一种低级的二进制指令格式，可以在浏览器中以接近原生的速度运行。它不是用来替代 JavaScript 的，而是与 JS 协作——JS 处理 UI 逻辑，WASM 处理计算密集型任务。

### 核心特征

| 特征 | 说明 |
|------|------|
| 二进制格式 | 比 JS 源码更小、解析更快 |
| 沙箱执行 | 在安全的沙箱环境中运行，无法直接访问 DOM |
| 接近原生速度 | 编译后的代码接近 C/C++ 原生性能 |
| 语言无关 | C/C++/Rust/Go/AssemblyScript 等均可编译为 WASM |
| 可与 JS 互操作 | JS 可以调用 WASM 函数，WASM 可以调用 JS 函数 |
| W3C 标准 | 所有主流浏览器均支持 |

### WASM 在浏览器中的位置

```
┌──────────────────────────────────────┐
│             浏览器引擎                │
│                                      │
│  JavaScript  ←→  WebAssembly         │
│  (脚本引擎)       (虚拟机)            │
│                                      │
│         ↕ Web API 互操作 ↕           │
│                                      │
│         DOM / WebGL / 网络            │
└──────────────────────────────────────┘
```

### 性能对比

| 维度 | JavaScript | WebAssembly |
|------|-----------|-------------|
| 解析速度 | 慢（文本解析 + AST） | 快（二进制解码） |
| 编译速度 | JIT 逐步优化 | AOT 预编译 |
| 运行速度 | 慢 10-50% | 接近原生 |
| 内存管理 | GC 自动 | 手动 / 自行管理 |
| 启动速度 | 快（小脚本） | 略慢（需编译实例化） |
| 体积 | 源码较小 | 二进制更小（同逻辑） |

---

## Why — 为什么需要 WebAssembly

### 1. 计算密集型场景的突破

JS 的动态类型和 JIT 编译使它在纯计算场景下有先天瓶颈。WASM 的静态类型和预编译让音视频编解码、图像处理、物理模拟等场景获得 2-10 倍性能提升。

### 2. 复用现有 C/C++ 生态

大量成熟的 C/C++ 库（FFmpeg、OpenCV、SQLite、zlib）可以直接编译为 WASM 在浏览器中运行，无需用 JS 重写。

### 3. 跨平台统一

同一份 WASM 代码可以在浏览器、Node.js、嵌入式运行时（Wasmtime、Wasmer）中运行，实现真正的"写一次，到处运行"。

### 适用场景

| 场景 | 示例 | 性能提升 |
|------|------|----------|
| 图像/视频处理 | 图片压缩、视频滤镜 | 5-20x |
| 游戏引擎 | Unity WebGL 导出 | 2-5x |
| 音频处理 | 实时音效、语音识别 | 3-10x |
| 加密/哈希 | SHA-256、AES 加密 | 5-15x |
| 数据压缩 | gzip、zstd | 3-8x |
| CAD/3D 建模 | AutoCAD Web、Figma | 2-5x |
| 科学计算 | 数值模拟、统计分析 | 5-20x |
| AI 推理 | TensorFlow.js WASM 后端 | 2-5x |

### 优缺点

- ✅ 优点：高性能、语言无关、跨平台、安全沙箱
- ❌ 缺点：无法直接操作 DOM、调试困难、GC 支持有限、启动开销

---

## How — 怎么用

### 1. Rust → WASM（最推荐的语言）

Rust 是目前 WASM 生态最成熟的语言，`wasm-pack` 工具链完善。

```bash
# 安装工具链
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
cargo generate --git https://github.com/rustwasm/wasm-pack-template
```

```rust
// src/lib.rs
use wasm_bindgen::prelude::*;

// 导出给 JS 调用的函数
#[wasm_bindgen]
pub fn fibonacci(n: u32) -> u64 {
    if n <= 1 {
        return n as u64;
    }
    let mut a = 0u64;
    let mut b = 1u64;
    for _ in 2..=n {
        let temp = a + b;
        a = b;
        b = temp;
    }
    b
}

// 接收 JS 传入的数组，返回处理后的数组
#[wasm_bindgen]
pub fn process_pixels(data: &[u8], width: u32, height: u32) -> Vec<u8> {
    let mut result = data.to_vec();
    // 灰度化
    for i in (0..data.len()).step_by(4) {
        let r = data[i] as f32;
        let g = data[i + 1] as f32;
        let b = data[i + 2] as f32;
        let gray = (0.299 * r + 0.587 * g + 0.114 * b) as u8;
        result[i] = gray;
        result[i + 1] = gray;
        result[i + 2] = gray;
        // alpha 保持不变
    }
    result
}

// 使用 JS 的 console.log
#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
}

#[wasm_bindgen]
pub fn greet(name: &str) {
    log(&format!("Hello, {}!", name));
}
```

```bash
# 编译为 WASM
wasm-pack build --target web
```

```html
<script type="module">
  import init, { fibonacci, process_pixels, greet } from './pkg/my_wasm.js'

  async function run() {
    await init()

    // 调用计算函数
    console.log('fibonacci(50) =', fibonacci(50))

    // 处理图像数据
    const canvas = document.querySelector('canvas')
    const ctx = canvas.getContext('2d')
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
    const result = process_pixels(imageData.data, canvas.width, canvas.height)
    const newImageData = new ImageData(new Uint8ClampedArray(result), canvas.width, canvas.height)
    ctx.putImageData(newImageData, 0, 0)

    // 调用带 JS 交互的函数
    greet('WebAssembly')
  }

  run()
</script>
```

---

### 2. C/C++ → WASM（Emscripten）

```bash
# 安装 Emscripten
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk && ./emsdk install latest && ./emsdk activate latest
source ./emsdk_env.sh
```

```c
// hello.c
#include <stdio.h>
#include <emscripten.h>

// 导出给 JS
EMSCRIPTEN_KEEPALIVE
int add(int a, int b) {
    return a + b;
}

EMSCRIPTEN_KEEPALIVE
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// 主函数（WASM 初始化后调用）
int main() {
    printf("WASM module loaded!\n");
    printf("5 + 3 = %d\n", add(5, 3));
    printf("10! = %d\n", factorial(10));
    return 0;
}
```

```bash
# 编译
emcc hello.c -o hello.js \
  -s EXPORTED_RUNTIME_METHODS='["ccall","cwrap"]' \
  -s EXPORTED_FUNCTIONS='["_add","_factorial","_main"]' \
  -O3
```

```html
<script src="hello.js"></script>
<script>
  Module.onRuntimeInitialized = function() {
    // 方式一：ccall 一次性调用
    const result = Module.ccall('add', 'number', ['number', 'number'], [5, 3])
    console.log('add(5, 3) =', result)

    // 方式二：cwrap 创建可复用的函数包装
    const factorial = Module.cwrap('factorial', 'number', ['number'])
    console.log('10! =', factorial(10))
  }
</script>
```

---

### 3. AssemblyScript — 用 TypeScript 写 WASM

AssemblyScript 是 TypeScript 语法的子集，编译为 WASM。对前端开发者最友好。

```bash
npm install -g assemblyscript
asc --init
```

```ts
// assembly/index.ts
// 导出给 JS 的函数
export function fibonacci(n: i32): i64 {
    if (n <= 1) return n as i64
    let a: i64 = 0
    let b: i64 = 1
    for (let i = 2; i <= n; i++) {
        const temp = a + b
        a = b
        b = temp
    }
    return b
}

// 使用共享内存传输大量数据
export function grayscale(dataOffset: i32, length: i32): void {
    const data = new Uint8Array(length)
    for (let i = 0; i < length; i += 4) {
        const r = load<u8>(dataOffset + i)
        const g = load<u8>(dataOffset + i + 1)
        const b = load<u8>(dataOffset + i + 2)
        const gray = <u8>(0.299 * f32(r) + 0.587 * f32(g) + 0.114 * f32(b))
        store<u8>(dataOffset + i, gray)
        store<u8>(dataOffset + i + 1, gray)
        store<u8>(dataOffset + i + 2, gray)
    }
}
```

```bash
asc assembly/index.ts --outFile build/index.wasm --optimize
```

---

### 4. JS 与 WASM 的互操作

```js
// 加载和实例化 WASM
async function loadWasm() {
  // 方式一：WebAssembly API（低层）
  const response = await fetch('module.wasm')
  const bytes = await response.arrayBuffer()
  const { instance } = await WebAssembly.instantiate(bytes, {
    // 导入对象：WASM 可以调用的 JS 函数
    env: {
      consoleLog: (ptr, len) => {
        const memory = instance.exports.memory
        const view = new Uint8Array(memory.buffer, ptr, len)
        console.log(new TextDecoder().decode(view))
      },
      random: () => Math.random(),
      time: () => Date.now(),
    }
  })

  return instance.exports
}

// 方式二：wasm-bindgen 生成的高层 API（Rust）
import init, { fibonacci, process_pixels } from './pkg/my_wasm.js'
await init()
```

**内存共享**：

```js
// 共享 ArrayBuffer 传输大数据
const wasm = await loadWasm()

// 从 JS 写入 WASM 内存
const memory = wasm.memory
const ptr = wasm.alloc(1024)  // WASM 端分配内存
const view = new Uint8Array(memory.buffer, ptr, 1024)

// 填充数据
for (let i = 0; i < 1024; i++) {
  view[i] = i & 0xff
}

// 调用 WASM 处理
wasm.process(ptr, 1024)

// 读取结果
console.log(view)

// 释放内存
wasm.dealloc(ptr, 1024)
```

---

### 5. WASI — WebAssembly 系统接口

WASI 让 WASM 可以在浏览器之外运行（服务器端、命令行工具等）。

```bash
# 安装 Wasmtime 运行时
curl https://wasmtime.dev/install.sh -sSf | bash

# 编译 Rust 为 WASI 目标
rustup target add wasm32-wasip1
cargo build --target wasm32-wasip1 --release

# 运行
wasmtime target/wasm32-wasip1/release/my-app.wasm
```

```rust
// 标准 Rust 代码，无需任何特殊处理
use std::fs;
use std::io::{self, Read};

fn main() -> io::Result<()> {
    // WASI 允许访问文件系统
    let mut file = fs::File::open("input.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    println!("File contents: {}", contents);
    Ok(())
}
```

---

### 6. 性能优化技巧

```rust
// 1. 减少跨边界调用——批量传输而非逐个传递
#[wasm_bindgen]
pub fn process_batch(data: &[u8]) -> Vec<u8> {
    // ✅ 一次传入整个数组
    data.iter().map(|&b| b.wrapping_add(1)).collect()
}

// 2. 避免频繁分配——预分配内存
#[wasm_bindgen]
pub struct ImageProcessor {
    buffer: Vec<u8>,
}

#[wasm_bindgen]
impl ImageProcessor {
    #[wasm_bindgen(constructor)]
    pub fn new(size: usize) -> Self {
        Self {
            buffer: vec![0u8; size], // 预分配
        }
    }

    pub fn process(&mut self, data: &[u8]) -> *const u8 {
        // 复用 buffer
        for (i, &byte) in data.iter().enumerate() {
            self.buffer[i] = byte.wrapping_add(1);
        }
        self.buffer.as_ptr()
    }
}

// 3. 使用 SIMD 加速
#[wasm_bindgen]
pub fn add_arrays_simd(a: &[f32], b: &[f32]) -> Vec<f32> {
    use std::arch::wasm32::*;
    let mut result = Vec::with_capacity(a.len());
    let chunks = a.len() / 4;
    for i in 0..chunks {
        let offset = i * 4;
        let va = v128_load(&a[offset] as *const f32 as *const v128);
        let vb = v128_load(&b[offset] as *const f32 as *const v128);
        let vr = f32x4_add(va, vb);
        let mut temp = [0f32; 4];
        v128_store(temp.as_mut_ptr() as *mut v128, vr);
        result.extend_from_slice(&temp);
    }
    result
}
```

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 启动慢 | WASM 文件大、编译实例化耗时 | 代码拆分、流式编译（`WebAssembly.instantiateStreaming`） |
| 跨边界调用慢 | JS ↔ WASM 调用有开销 | 批量传输，减少调用次数 |
| 无法访问 DOM | WASM 运行在沙箱中 | 通过 JS 导入函数桥接 |
| 调试困难 | 二进制格式，DevTools 支持有限 | 生成 DWARF 调试信息，Chrome DevTools 支持 WASM 断点 |
| 内存泄漏 | 手动管理 WASM 内存 | 封装 alloc/dealloc，或用 Rust 的自动内存管理 |
| 整数溢出 | WASM 默认整数溢出是环绕行为 | Rust 中用 `checked_add` / `wrapping_add` 明确语义 |

### 最佳实践

1. **只在计算密集场景使用 WASM**：UI 渲染、事件处理等用 JS。
2. **Rust 是最佳选择**：零 GC、内存安全、wasm-bindgen 生态完善。
3. **减少跨边界调用**：一次传递数组而非逐个值。
4. **流式编译**：使用 `WebAssembly.instantiateStreaming` 而非 `instantiate`。
5. **代码拆分**：大模块拆为多个小 .wasm 文件，按需加载。

---

## 面试题

### 1. WebAssembly 和 JavaScript 是什么关系？WASM 会替代 JS 吗？

**答**：WebAssembly 和 JavaScript 是协作关系，不是替代关系。WASM 无法直接操作 DOM，没有事件系统，不适合处理 UI 交互。它的定位是"JS 的性能补充"——JS 负责 UI 逻辑、DOM 操作、事件处理等"胶水代码"，WASM 负责图像处理、加密计算、物理模拟等计算密集型任务。两者通过 `WebAssembly.instantiate` 的导入/导出对象互操作。WASM 不会替代 JS，正如汇编不会替代 C——高层逻辑仍然需要 JS 的灵活性。

---

### 2. WASM 的性能为什么接近原生？具体原因是什么？

**答**：三个关键因素：(1) **静态类型**——WASM 是强类型的二进制格式，所有操作数类型在编译时确定，无需运行时类型检查和装箱（JS 的 `1 + "2"` 这种隐式转换不存在）；(2) **AOT 编译**——WASM 在加载时就被编译为机器码，而 JS 需要 JIT 先解释执行再逐步优化，首次执行慢；(3) **确定性执行**——WASM 没有动态属性查找、原型链、GC 暂停等运行时开销，指令执行时间可预测。但 WASM 不是完全等同原生——沙箱安全检查、内存边界检查仍有少量开销，通常比原生慢 5-15%。

---

### 3. JS 和 WASM 之间传递大量数据应该怎么做？为什么不建议逐个传参？

**答**：应该通过共享内存（`WebAssembly.Memory`）传递。创建一个 `ArrayBuffer`，JS 和 WASM 共同读写同一块内存。JS 用 `TypedArray` 视图写入数据，WASM 通过指针直接读取，避免了序列化/反序列化开销。逐个传参的问题：(1) 每次跨边界调用有 ~100ns 的固定开销（类型转换、栈切换）；(2) 传递 10000 个元素就是 10000 次调用 ≈ 1ms 纯开销；(3) 批量传递只需 1 次调用 + 内存复制，开销低一个数量级。

---

### 4. 什么是 WASI？它和浏览器中的 WASM 有什么区别？

**答**：WASI（WebAssembly System Interface）是 WASM 的系统接口标准，让 WASM 可以在浏览器之外运行。浏览器中的 WASM 只能调用 Web API（DOM、Fetch 等），无法访问文件系统、网络套接字、环境变量等系统资源。WASI 提供了一套标准化的系统调用接口（文件读写、网络、时钟、随机数等），使 WASM 可以作为通用的运行时格式，在服务器（Wasmtime/Wasmer）、边缘计算（Cloudflare Workers）、嵌入式设备中运行。核心区别：浏览器 WASM 受限于 Web 沙箱，WASI 打开了系统级访问的大门，同时保持安全（基于能力的权限模型）。

---

### 5. Rust 为什么是写 WASM 的最佳语言？

**答**：五个原因：(1) **零 GC**——Rust 没有垃圾回收器，WASM 运行时也没有 GC（GC 提案尚未广泛实现），Rust 的所有权模型完美匹配；(2) **内存安全**——编译时保证无悬垂指针、无数据竞争，不需要运行时检查；(3) **体积小**——`wasm-opt` + Rust 的零抽象，hello world 可压缩到 < 10KB；(4) **wasm-bindgen**——最成熟的 WASM↔JS 桥接工具，自动生成类型安全的绑定代码；(5) **生态**——`wasm-pack` 一键构建+发布 npm 包，`gloo` 提供常用 Web API 的 Rust 封装。Go 也能编译 WASM，但 GC 运行时会增加 ~10KB 体积；C/C++ 缺乏内存安全保证。

---

### 6. WASM 的安全性体现在哪些方面？

**答**：WASM 的安全模型有三层保障：(1) **沙箱隔离**——WASM 代码运行在独立的虚拟机中，无法直接访问宿主环境的内存、文件系统或网络，所有外部访问必须通过导入函数（JS 提供的 API）；(2) **内存安全**——WASM 的线性内存有边界检查，访问越界会 trap 而非段错误；间接函数调用有类型签名检查；(3) **控制流完整性**——WASM 的控制流是结构化的（不能用任意跳转），间接调用必须通过函数表索引，且类型签名必须匹配。WASI 进一步用"能力安全"模型——WASM 模块只能访问启动时被授权的资源，默认零权限。

---

### 7. WebAssembly.instantiate 和 instantiateStreaming 有什么区别？

**答**：`WebAssembly.instantiate(bytes)` 接收 `ArrayBuffer`，需要先完整下载 WASM 文件再编译。`WebAssembly.instantiateStreaming(source)` 接收 `Response` 对象（`fetch` 返回值），可以边下载边编译——浏览器在接收二进制数据的同时就开始编译，无需等待完整下载。`instantiateStreaming` 通常快 30-50%，特别是大文件时效果更明显。但 `instantiateStreaming` 要求服务器返回正确的 `Content-Type: application/wasm`，否则会降级到 `instantiate`。实际使用中优先 `instantiateStreaming`。

---

### 8. 前端项目在什么情况下应该引入 WASM？给出判断标准。

**答**：判断标准——满足以下任一条件值得引入：(1) **单次计算 > 16ms**——如果某个计算任务阻塞主线程超过一帧（16ms），WASM 可以将其降至可接受范围；(2) **数据量 > 100KB**——大数组/图像/视频处理，WASM 的批量内存操作优势明显；(3) **复用 C/C++ 库**——已有成熟的 C/C++ 实现（如 FFmpeg、OpenCV），用 WASM 编译比 JS 重写更可靠；(4) **实时性要求高**——音频处理、物理模拟需要稳定帧率，WASM 的确定性执行优于 JS 的 JIT 不确定性。不推荐引入的场景：纯 UI 逻辑、简单数据转换、项目初期快速迭代——WASM 的开发成本和调试复杂度高于 JS。

---

## 相关链接

- [WebAssembly 官方文档](https://webassembly.org/)
- [WebAssembly — MDN](https://developer.mozilla.org/zh-CN/docs/WebAssembly)
- [Rust WASM 指南](https://rustwasm.github.io/docs/book/)
- [wasm-bindgen 文档](https://rustwasm.github.io/wasm-bindgen/)
- [Emscripten 文档](https://emscripten.org/)
- [AssemblyScript 文档](https://www.assemblyscript.org/)
- [WASI 标准](https://wasi.dev/)
