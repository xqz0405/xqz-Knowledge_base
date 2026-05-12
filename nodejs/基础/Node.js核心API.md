---
tags:
  - Node.js
  - 核心API
  - fs
  - path
date: 2026-05-12
status: 已完成
difficulty: 入门
---

# Node.js 核心 API

## What — 是什么

> Node.js 核心 API 是内置模块提供的编程接口，无需安装第三方包即可使用，涵盖文件系统、路径处理、URL 解析、工具函数等基础能力。

**核心概念：**

- **fs**：文件系统操作，`fs/promises` 提供 Promise 版本，支持同步/异步/流式三种模式
- **path**：跨平台路径处理，`join`/`resolve`/`dirname`/`basename`/`extname` 等
- **url**：URL 解析和构建，`new URL()` + `url.fileURLToPath()`
- **util**：工具函数，`promisify`/`callbackify`/`inspect`/`isDeepStrictEqual`
- **querystring**：查询字符串解析（已弃用，推荐 `URLSearchParams`）
- **assert**：断言测试，`assert.strictEqual`/`deepStrictEqual`/`rejects`
- **perf_hooks**：性能测量，`performance.now()`/`monitorEventLoopDelay`
- **ABI 稳定性**：Node.js API 稳定性等级（Stability: 1-3），3 为最稳定

**关键特性：**

- `fs/promises` 是推荐的 fs 使用方式（async/await 友好）
- `path.join()` 处理跨平台路径分隔符（`/` vs `\`）
- `util.promisify()` 将回调风格函数转为 Promise
- `new URL()` 原生解析 URL，替代 `url.parse()`
- `perf_hooks` 提供高精度计时和事件循环监控

## Why — 为什么

**适用场景：**

- 文件操作：读写配置、日志、模板
- 路径处理：构建跨平台的文件路径
- URL 处理：解析请求 URL、构建 API 地址
- 工具函数：promisify 回调函数、格式化对象
- 性能测量：基准测试、事件循环延迟监控

**对比方案：**

| 维度 | Node.js 核心 API | 第三方库 |
|------|-----------------|---------|
| 安装 | 无需 | npm install |
| 稳定性 | 高（Node.js 团队维护） | 取决于社区 |
| 功能 | 基础 | 丰富 |
| 性能 | 优化到极致 | 可能更慢 |

## How — 怎么用

### 快速上手

```javascript
const fs = require('fs/promises');
const path = require('path');
const { URL } = require('url');
const { promisify } = require('util');

// fs/promise
const data = await fs.readFile(path.join(__dirname, 'config.json'), 'utf8');
await fs.writeFile('output.txt', 'hello');

// path
const fullPath = path.resolve('src', 'utils', 'helper.js'); // 绝对路径
const ext = path.extname('file.tar.gz'); // '.gz'
const dir = path.dirname('/a/b/c.txt'); // '/a/b'

// URL
const url = new URL('https://example.com/api?q=hello&page=1');
url.searchParams.get('q'); // 'hello'

// promisify
const readFile = promisify(require('fs').readFile);
const content = await readFile('data.txt', 'utf8');
```

### 代码示例

```javascript
const fs = require('fs/promises');
const path = require('path');

// 递归创建目录
await fs.mkdir(path.join('logs', '2026', '05'), { recursive: true });

// 目录遍历
async function walkDir(dir) {
    const entries = await fs.readdir(dir, { withFileTypes: true });
    const files = [];
    for (const entry of entries) {
        const fullPath = path.join(dir, entry.name);
        if (entry.isDirectory()) files.push(...await walkDir(fullPath));
        else files.push(fullPath);
    }
    return files;
}

// 文件监控
const { watch } = require('fs');
watch('./src', { recursive: true }, (eventType, filename) => {
    console.log(`${eventType}: ${filename}`);
});

// 临时文件
const os = require('os');
const tmpFile = path.join(os.tmpdir(), `app-${Date.now()}.tmp`);
await fs.writeFile(tmpFile, data);
// 使用后清理
await fs.unlink(tmpFile);
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 路径拼接跨平台出错 | Windows 用 `\` Linux 用 `/` | 始终用 `path.join()` |
| `__dirname` ESM 中不可用 | ESM 没有 CJS 全局变量 | `path.dirname(fileURLToPath(import.meta.url))` |
| `fs.exists` 已弃用 | 语义不清（不存在可能是权限问题） | 用 `fs.access()` 或 try/catch `fs.stat()` |
| `url.parse` 已弃用 | 不符合 WHATWG 标准 | 用 `new URL()` |

### 最佳实践

- 文件操作使用 `fs/promises`，不用回调版本
- 路径拼接用 `path.join()`/`path.resolve()`，不手动拼字符串
- URL 解析用 `new URL()`，不用 `url.parse()`
- 回调风格 API 用 `util.promisify()` 转换

## 面试题

**Q1: fs 的三种操作模式有什么区别？**
> ① 同步模式（`fs.readFileSync`）：阻塞事件循环，简单但影响性能，只适合启动时读取配置。② 异步回调（`fs.readFile(path, cb)`）：非阻塞，但回调嵌套不优雅。③ 异步 Promise（`fs/promises`）：`await fs.readFile()`，推荐方式，非阻塞 + 代码清晰。性能：异步比同步好（不阻塞），Promise 和回调性能几乎相同。

**Q2: path.join 和 path.resolve 的区别？**
> `path.join('a', 'b', 'c')` 拼接路径片段，返回 `'a/b/c'`（相对路径）。`path.resolve('a', 'b', 'c')` 从右到左拼接，直到遇到绝对路径，等价于 `cd a && cd b && cd c` 后的 `pwd`，返回绝对路径。`path.resolve('src', 'index.js')` → `/current/working/dir/src/index.js`。`path.join` 不解析 `..` 为绝对路径，`path.resolve` 会。

**Q3: util.promisify 的原理？**
> `promisify` 将 `(err, result) => {}` 回调风格的函数包装为返回 Promise 的函数。原理：① 返回新函数；② 新函数调用原始函数，传入的回调内部：如果 `err` 存在则 `reject(err)`，否则 `resolve(result)`；③ 处理 `(err, result1, result2)` 多返回值——合并为 `{ result1, result2 }` 对象。限制：只适用于最后一个参数是 `(err, result) => {}` 回调的函数。

---

**相关链接：**
- [[Stream与Buffer]]
- [[模块系统]]
- [[文件系统操作]]
- Node.js API 文档：https://nodejs.org/api/
