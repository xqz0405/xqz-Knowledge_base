---
tags:
  - Node.js
  - Stream
  - Buffer
date: 2026-05-12
status: 已完成
difficulty: 进阶
---

# Stream与Buffer

## What — 是什么

> Stream（流）是 Node.js 中处理流式数据的核心抽象，将数据拆分为小块逐段处理，避免一次性将全部数据加载到内存。Buffer 是 Node.js 中处理二进制数据的全局类，用于在 V8 堆外分配原始内存，是流和文件 I/O 的基础单元。

**核心概念：**

- **四种流类型**：
  - **Readable**：可读流，数据的来源（如 fs.createReadStream、HTTP 请求体）
  - **Writable**：可写流，数据的目的地（如 fs.createWriteStream、HTTP 响应体）
  - **Duplex**：双工流，同时可读可写，读写独立（如 TCP Socket、net.Socket）
  - **Transform**：转换流，读写之间可对数据做变换（如 zlib.createGzip、crypto.createCipher）
- **Buffer**：固定大小的原始二进制数据缓冲区，类似定长字节数组，是 Uint8Array 的子类
- **编码（Encoding）**：Buffer 与字符串之间的转换规则（utf8、ascii、hex、base64 等）
- **管道（Pipe）**：将可读流输出直接连接到可写流输入，`readable.pipe(writable)`
- **Pipeline**：`stream.pipeline` 的增强版，自动处理错误传播和流清理，推荐替代 `.pipe()` 链
- **Backpressure（反压）**：当消费端处理速度慢于生产端时，通过暂停读取实现流量控制，防止内存膨胀

**关键特性：**

- 流是 EventEmitter 的子类，通过事件驱动数据流转（data、end、error、close）
- Buffer 大小创建后不可变，修改内容需在现有 Buffer 上操作
- 流有两种模式：流动模式（Flowing）和暂停模式（Paused），通过 `.pipe()` 或 `resume()`/`pause()` 切换
- Backpressure 是流控的核心机制，保证内存安全

## 运行机制

### 内存模型

**Buffer 内存分配：**

- Buffer 的内存不在 V8 命名空间中分配，而是通过 C++ 层在 V8 堆外申请
- 小 Buffer（≤ poolSize / 2，即 4KB 以内）：采用 Slab 分配机制——先申请一块 8KB 的 Slab，多个小 Buffer 共享同一 Slab，减少系统调用
- 大 Buffer（> 4KB）：直接单独分配一块完整内存，不使用 Slab
- `Buffer.alloc(size)`：分配指定大小的 Buffer，内存清零（安全但稍慢）
- `Buffer.allocUnsafe(size)`：分配但不清零，可能包含旧数据（快但不安全）
- `Buffer.from(source)`：从数组、字符串、另一个 Buffer 等创建新 Buffer

**Slab 机制详解：**

```
┌──────────────── 8KB Slab ────────────────┐
│ Buffer A (2KB) │ Buffer B (1KB) │ 空闲... │
└───────────────────────────────────────────┘
↑ 共享同一底层 ArrayBuffer
├─ slab 状态：partial（部分使用）
├─ offset：记录当前分配位置
└─ 当 Slab 空间不足时，创建新 Slab
```

- Slab 由 `bufferPool` 管理，全局只维护一个活跃 Slab
- `allocUnsafe` 会从当前 Slab 中切片，多个小 Buffer 引用同一个 ArrayBuffer 的不同偏移区间
- 这意味着 `allocUnsafe` 创建的 Buffer 可能通过底层 ArrayBuffer 互相"看到"数据，存在安全风险

### 执行模型

**流动模式 vs 暂停模式：**

| 维度 | 流动模式（Flowing） | 暂停模式（Paused） |
|------|-------------------|-------------------|
| 数据获取 | 自动推送（data 事件） | 手动拉取（read() 方法） |
| 触发方式 | 添加 data 监听 / 调用 resume() / 调用 pipe() | 默认初始状态 / 调用 pause() / 移除 data 监听 |
| 背压处理 | 自动（write() 返回 false 时暂停） | 手动（需自行调用 read() 控制） |
| 适用场景 | 高速连续数据流（视频、网络） | 需要精确控制读取节奏 |

**模式切换：**

```javascript
const readable = fs.createReadStream('./data.txt');

// 默认暂停模式 → 添加 data 监听进入流动模式
readable.on('data', chunk => { /* 自动推送 */ });

// 流动模式 → 调用 pause() 切回暂停模式
readable.pause();

// 暂停模式 → 调用 resume() 切回流动模式
readable.resume();

// 暂停模式下手动读取
readable.on('readable', () => {
    let chunk;
    while (null !== (chunk = readable.read())) {
        process(chunk);
    }
});
```

### 并发模型

**Backpressure 反压机制：**

反压是流控的核心——当可写端的写入速度跟不上可读端的推送速度时，系统自动暂停可读端，等待可写端消化后恢复。

```
Readable ──push──▶ Writable
   ↑                  │
   │   write() 返回    │
   │   false（缓冲区满）│
   └─── pause() ──────┘

         ↓ 缓冲区排空后 ↓

Readable ──resume──▶ 继续推送
```

- `writable.write(chunk)` 返回 `true` 表示可以继续写，返回 `false` 表示缓冲区已满（超过 highWaterMark）
- `.pipe()` 内部自动处理反压：write 返回 false 时暂停 readable，writable 触发 `drain` 事件时恢复 readable
- 自行处理流时必须手动实现反压逻辑，否则会导致内存持续增长

## 类型系统

### Buffer 的类型

- Buffer 是 `Uint8Array` 的子类，继承其所有方法（slice、subarray 等）
- Buffer 的底层内存由 C++ 的 `ArrayBuffer` 分配，V8 堆外管理
- 与 `Uint8Array` 的区别：

| 特性 | Buffer | Uint8Array |
|------|--------|------------|
| 内存位置 | V8 堆外（C++ 层分配） | V8 堆内（TypedArray） |
| 创建方式 | Buffer.alloc / allocUnsafe / from | new Uint8Array(length) |
| 字符串转换 | buf.toString('utf8') 等内置方法 | 需借助 TextDecoder |
| 字节操作 | buf.readInt16LE / writeUInt32BE 等 | 无多字节读写方法 |
| 继承关系 | extends Uint8Array | extends TypedArray |
| Slab 共享 | 小 Buffer 共享 Slab | 每个实例独立 ArrayBuffer |

### 编码类型

| 编码 | 说明 | 示例 |
|------|------|------|
| utf8 | 默认编码，多字节 Unicode | `buf.write('你好', 'utf8')` |
| ascii | 7 位 ASCII，忽略高位 | 仅支持 0~127 |
| hex | 十六进制字符串，每字节 2 字符 | `'48656c6c6f'` → `'Hello'` |
| base64 | Base64 编码，常用于传输二进制 | 图片/Data URL 编码 |
| binary / latin1 | 每字节对应一个字符（ISO-8859-1） | 低级二进制处理 |
| ucs2 / utf16le | 2 字节小端序 Unicode | Windows 环境常见 |

**编码转换示例：**

```javascript
// 字符串 → Buffer → 不同编码输出
const buf = Buffer.from('Hello 世界', 'utf8');
console.log(buf.toString('utf8'));   // 'Hello 世界'
console.log(buf.toString('hex'));    // '48656c6c6f20e4b896e7958c'
console.log(buf.toString('base64')); // 'SGVsbG8g5LiW55WM'

// Base64 解码
const decoded = Buffer.from('SGVsbG8g5LiW55WM', 'base64');
console.log(decoded.toString('utf8')); // 'Hello 世界'

// Hex 解码
const fromHex = Buffer.from('48656c6c6f', 'hex');
console.log(fromHex.toString('utf8')); // 'Hello'
```

## Why — 为什么

**适用场景：**

- **大文件处理**：读取/复制/转码 GB 级文件，内存占用恒定
- **网络传输**：HTTP 请求/响应体流式传输，支持进度反馈
- **数据转换**：压缩/加密/格式转换链式处理

**一次性读取 vs 流式处理对比：**

| 维度 | 一次性读取（fs.readFile） | 流式处理（fs.createReadStream） |
|------|-------------------------|-------------------------------|
| 内存占用 | 文件多大占多少（可能 OOM） | 恒定（≈ highWaterMark，默认 64KB） |
| 响应速度 | 必须全部读完才能处理 | 第一个 chunk 到达即开始处理 |
| 适用文件大小 | 小文件（< 几十 MB） | 任意大小（GB、TB 级） |
| 错误恢复 | 全部重来 | 可断点续传 |

**优缺点：**

- ✅ 优点：
  - 内存效率极高，恒定占用处理无限数据
  - 首字节延迟低，第一个 chunk 到达即可开始处理
  - 管道组合灵活，链式处理（读 → 解压 → 解密 → 写）
  - 背压机制保证系统稳定，自动调节流速

- ❌ 缺点：
  - 概念复杂，需要理解流动/暂停模式、背压、事件驱动
  - 错误处理繁琐，`.pipe()` 不会自动传播错误
  - 调试困难，流是异步的，数据分段流转不易追踪
  - 不适合需要随机访问整个数据的场景（如排序、搜索）

## How — 怎么用

### 快速上手

```javascript
const fs = require('node:fs');
const { pipeline } = require('node:stream/promises');

// 流式复制文件
async function copyFile(src, dest) {
    await pipeline(
        fs.createReadStream(src),
        fs.createWriteStream(dest)
    );
    console.log('复制完成');
}

// Buffer 基本操作
const buf = Buffer.alloc(8);         // 8 字节，清零
buf.writeUInt32BE(0x12345678, 0);    // 大端写入 32 位整数
console.log(buf.toString('hex'));    // '1234567800000000'
console.log(buf.readUInt32BE(0));    // 0x12345678
```

### 代码示例

**1. 文件流复制：**

```javascript
const fs = require('node:fs');
const { pipeline } = require('node:stream/promises');

// ---- 方式一：pipeline（推荐） ----
async function copyWithPipeline(src, dest) {
    const startTime = Date.now();
    let bytesCopied = 0;

    await pipeline(
        fs.createReadStream(src, { highWaterMark: 64 * 1024 }),
        async function* (source) {
            for await (const chunk of source) {
                bytesCopied += chunk.length;
                yield chunk;
            }
        },
        fs.createWriteStream(dest)
    );

    const elapsed = Date.now() - startTime;
    const mb = (bytesCopied / 1024 / 1024).toFixed(2);
    console.log(`复制完成: ${mb} MB, 耗时 ${elapsed} ms`);
}

// ---- 方式二：pipe（旧写法，需手动处理错误） ----
function copyWithPipe(src, dest, callback) {
    const readStream = fs.createReadStream(src);
    const writeStream = fs.createWriteStream(dest);

    readStream.pipe(writeStream);

    writeStream.on('finish', () => callback(null));
    writeStream.on('error', (err) => {
        readStream.destroy();
        callback(err);
    });
    readStream.on('error', (err) => {
        writeStream.destroy();
        callback(err);
    });
}

// ---- 方式三：手动背压控制（了解原理） ----
function copyWithBackpressure(src, dest, callback) {
    const readStream = fs.createReadStream(src, { highWaterMark: 16 * 1024 });
    const writeStream = fs.createWriteStream(dest, { highWaterMark: 16 * 1024 });

    function write() {
        let chunk;
        let canContinue = true;

        while (canContinue && (chunk = readStream.read()) !== null) {
            canContinue = writeStream.write(chunk);
        }

        if (!canContinue) {
            writeStream.once('drain', onDrain);
        }
    }

    function onDrain() {
        readStream.resume();
    }

    readStream.on('readable', write);
    readStream.on('end', () => {
        writeStream.end();
        callback(null);
    });

    readStream.on('error', callback);
    writeStream.on('error', callback);
}
```

**2. Transform 流处理 CSV：**

```javascript
const { Transform } = require('node:stream');
const { pipeline } = require('node:stream/promises');
const fs = require('node:fs');

// ---- CSV 行 → JSON 对象的 Transform 流 ----
class CsvToJsonTransform extends Transform {
    constructor(options = {}) {
        super({ ...options, objectMode: true }); // 对象模式
        this.headers = null;
        this.lineBuffer = '';
    }

    _transform(chunk, encoding, callback) {
        const lines = (this.lineBuffer + chunk.toString()).split('\n');
        this.lineBuffer = lines.pop(); // 保留不完整的最后一行

        for (const line of lines) {
            const trimmed = line.trim();
            if (!trimmed) continue;

            const fields = trimmed.split(',');

            if (!this.headers) {
                this.headers = fields; // 首行作为表头
                continue;
            }

            const obj = {};
            this.headers.forEach((header, i) => {
                obj[header.trim()] = (fields[i] || '').trim();
            });

            this.push(obj); // 输出 JSON 对象
        }

        callback();
    }

    _flush(callback) {
        // 处理剩余的行
        if (this.lineBuffer.trim()) {
            const fields = this.lineBuffer.split(',');
            if (this.headers) {
                const obj = {};
                this.headers.forEach((header, i) => {
                    obj[header.trim()] = (fields[i] || '').trim();
                });
                this.push(obj);
            }
        }
        callback();
    }
}

// ---- JSON 对象 → SQL INSERT 的 Transform 流 ----
class JsonToSqlTransform extends Transform {
    constructor(tableName, options = {}) {
        super({ ...options, objectMode: true });
        this.tableName = tableName;
    }

    _transform(record, encoding, callback) {
        const columns = Object.keys(record).join(', ');
        const values = Object.values(record)
            .map(v => `'${String(v).replace(/'/g, "''")}'`)
            .join(', ');
        const sql = `INSERT INTO ${this.tableName} (${columns}) VALUES (${values});\n`;
        callback(null, sql);
    }
}

// ---- 使用管道链处理 CSV → JSON → SQL ----
async function processCsvFile(inputPath, outputPath) {
    await pipeline(
        fs.createReadStream(inputPath, { encoding: 'utf8' }),
        new CsvToJsonTransform(),
        new JsonToSqlTransform('users'),
        fs.createWriteStream(outputPath)
    );
    console.log('CSV → SQL 转换完成');
}

// 示例输入 CSV:
// name,age,email
// Alice,30,alice@example.com
// Bob,25,bob@example.com

// 示例输出 SQL:
// INSERT INTO users (name, age, email) VALUES ('Alice', '30', 'alice@example.com');
// INSERT INTO users (name, age, email) VALUES ('Bob', '25', 'bob@example.com');
```

**3. Pipeline 管道链：**

```javascript
const fs = require('node:fs');
const zlib = require('node:zlib');
const crypto = require('node:crypto');
const { pipeline } = require('node:stream/promises');

// ---- 文件 → 压缩 → 加密 → 写入（链式管道） ----
async function compressAndEncrypt(inputPath, outputPath, password) {
    const key = crypto.scryptSync(password, 'salt', 32);
    const iv = crypto.randomBytes(16);

    // 先写入 IV（解密时需要）
    const writeStream = fs.createWriteStream(outputPath);
    writeStream.write(iv);

    await pipeline(
        fs.createReadStream(inputPath),
        zlib.createGzip(),             // 压缩
        crypto.createCipheriv('aes-256-cbc', key, iv), // 加密
        writeStream                     // 写入文件
    );

    console.log('压缩加密完成');
}

// ---- 解密 → 解压 → 写入 ----
async function decryptAndDecompress(inputPath, outputPath, password) {
    const key = crypto.scryptSync(password, 'salt', 32);

    // 读取 IV
    const readStream = fs.createReadStream(inputPath);
    const iv = await readStreamToBuffer(readStream, 16);

    await pipeline(
        readStream,
        crypto.createDecipheriv('aes-256-cbc', key, iv), // 解密
        zlib.createGunzip(),                               // 解压
        fs.createWriteStream(outputPath)
    );

    console.log('解密解压完成');
}

// 辅助：从流中读取指定字节数
function readStreamToBuffer(stream, size) {
    return new Promise((resolve, reject) => {
        const buf = Buffer.alloc(size);
        let offset = 0;

        function onReadable() {
            let chunk;
            while (offset < size && (chunk = stream.read(size - offset)) !== null) {
                chunk.copy(buf, offset);
                offset += chunk.length;
            }
            if (offset >= size) {
                stream.off('readable', onReadable);
                resolve(buf);
            }
        }

        stream.on('readable', onReadable);
        stream.on('error', reject);
    });
}

// ---- pipeline + 进度追踪 ----
async function copyWithProgress(src, dest) {
    const stat = await fs.promises.stat(src);
    const totalSize = stat.size;
    let transferred = 0;

    await pipeline(
        fs.createReadStream(src),
        new Transform({
            transform(chunk, encoding, callback) {
                transferred += chunk.length;
                const percent = ((transferred / totalSize) * 100).toFixed(1);
                if (transferred % (1024 * 1024) < chunk.length) {
                    process.stdout.write(`\r进度: ${percent}%`);
                }
                callback(null, chunk); // 原样透传
            }
        }),
        fs.createWriteStream(dest)
    );

    console.log(`\n完成: ${(totalSize / 1024 / 1024).toFixed(2)} MB`);
}
```

**4. Backpressure 处理：**

```javascript
const { Readable, Writable } = require('node:stream');

// ---- 模拟慢消费者 + 快生产者 ----
class FastProducer extends Readable {
    constructor() {
        super({ highWaterMark: 16 * 1024 }); // 16KB 缓冲区
        this.counter = 0;
        this.maxItems = 1000;
    }

    _read(size) {
        // 快速生产数据
        const produce = () => {
            let canPush = true;
            while (canPush && this.counter < this.maxItems) {
                const data = Buffer.alloc(1024, `chunk-${this.counter}`);
                canPush = this.push(data);
                this.counter++;
            }

            if (this.counter >= this.maxItems) {
                this.push(null); // 结束
            }

            // push 返回 false → 背压生效，等待 drain
            if (!canPush) {
                console.log(`[背压] 生产暂停在 chunk #${this.counter}，等待消费`);
            }
        };

        // 使用 setImmediate 避免同步递归
        setImmediate(produce);
    }
}

class SlowConsumer extends Writable {
    constructor() {
        super({ highWaterMark: 8 * 1024 }); // 8KB 缓冲区
        this.consumed = 0;
    }

    _write(chunk, encoding, callback) {
        // 模拟慢速消费（每块 10ms）
        setTimeout(() => {
            this.consumed++;
            if (this.consumed % 50 === 0) {
                console.log(`[消费] 已处理 ${this.consumed} 块`);
            }
            callback();
        }, 10);
    }
}

// ---- 手动实现管道 + 背压（理解 pipe 原理） ----
function pipeWithManualBackpressure(readable, writable) {
    readable.on('data', (chunk) => {
        const canWrite = writable.write(chunk);
        if (!canWrite) {
            readable.pause(); // 背压：暂停读取
            writable.once('drain', () => {
                readable.resume(); // 消费完成：恢复读取
            });
        }
    });

    readable.on('end', () => {
        writable.end();
    });

    readable.on('error', (err) => writable.destroy(err));
    writable.on('error', (err) => readable.destroy(err));
}

// ---- 使用 .pipe() 自动处理背压 ----
function autoBackpressure() {
    const producer = new FastProducer();
    const consumer = new SlowConsumer();

    producer.pipe(consumer); // pipe 内部自动处理背压

    consumer.on('finish', () => {
        console.log(`完成！消费了 ${consumer.consumed} 块`);
    });
}

// ---- highWaterMark 对性能的影响 ----
function testHighWaterMark() {
    const sizes = [1024, 16 * 1024, 64 * 1024, 256 * 1024];

    for (const hwm of sizes) {
        const start = Date.now();
        let chunks = 0;

        const readable = new Readable({
            highWaterMark: hwm,
            read() {
                if (chunks < 10000) {
                    this.push(Buffer.alloc(1024, 'x'));
                    chunks++;
                } else {
                    this.push(null);
                }
            }
        });

        const writable = new Writable({
            highWaterMark: hwm,
            write(chunk, enc, cb) { setImmediate(cb); }
        });

        readable.pipe(writable);
        writable.on('finish', () => {
            console.log(`hwm=${hwm}: ${Date.now() - start}ms`);
        });
    }
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `.pipe()` 不传播错误 | pipe 只转发数据和结束信号，不转发 error 事件 | 使用 `stream.pipeline()` 替代，自动传播错误并清理资源 |
| 流未关闭导致内存泄漏 | 忘记调用 `.end()` / `.destroy()`，流对象无法被 GC | 始终在 error/end 回调中清理流；使用 pipeline 自动管理生命周期 |
| `setEncoding` 后 chunk 变字符串 | 可读流调用 `setEncoding('utf8')` 后 data 事件输出字符串 | 如需 Buffer，不要调用 setEncoding；手动 `Buffer.from(chunk)` |
| Transform 流 objectMode 不匹配 | 上下游 objectMode 不一致，导致数据类型错误 | 确保管道链中每个流的 objectMode 一致，或在 Transform 中分别设置 readableObjectMode/writableObjectMode |
| 高速生产导致内存膨胀 | 未处理背压，数据堆积在可写端内部缓冲区 | 始终检查 write() 返回值，返回 false 时暂停读取；使用 pipe 自动处理 |
| Buffer.allocUnsafe 泄露敏感数据 | 未清零的 Buffer 可能包含之前的密码、密钥等 | 生产环境优先用 `Buffer.alloc()`；如需性能必须用 allocUnsafe，确保覆写所有字节 |

### 最佳实践

1. **使用 `stream.pipeline()` 替代 `.pipe()` 链**：pipeline 自动传播错误、清理资源，避免内存泄漏；推荐 `stream/promises` 版本配合 async/await
2. **设置合理的 highWaterMark**：默认 16KB（对象模式 16 个），大文件场景可提高到 64KB~256KB 减少系统调用；注意过高会占用更多内存
3. **使用 objectMode 处理结构化数据**：当流需要传递对象而非 Buffer 时（如 CSV 解析、数据库行），开启 `objectMode: true`
4. **大文件必须用流，禁止 readFile**：超过几十 MB 的文件用 `createReadStream` + `pipeline`，保证内存恒定
5. **始终处理流的 error 事件**：未监听 error 会触发 unhandledException 导致进程崩溃；使用 pipeline 时也建议在外层 try/catch
6. **善用 `for await...of` 消费可读流**：异步迭代器模式比 data 事件更安全，自动处理背压和流关闭
7. **Buffer 优先使用 `Buffer.alloc()` 和 `Buffer.from()`**：避免 `new Buffer()` 构造函数（已废弃），allocUnsafe 仅在性能热点且有覆写保障时使用

## 面试题

**Q1: Node.js 的四种流类型分别是什么？各自的特点和应用场景？**
> (1) Readable：可读流，数据来源，如 fs.createReadStream、process.stdin、HTTP 请求体。特点是只能读不能写，通过 data 事件或 read() 方法消费数据。(2) Writable：可写流，数据目的地，如 fs.createWriteStream、process.stdout、HTTP 响应体。特点是只能写不能读，通过 write() 写入、end() 结束。(3) Duplex：双工流，可读可写且读写独立，如 TCP Socket（net.Socket）。读写各有独立缓冲区，互不影响。(4) Transform：转换流，是 Duplex 的子类，输出是输入的变换结果，如 zlib.createGzip（压缩）、crypto.createCipher（加密）。Transform 流的 _transform 方法接收输入、产出变换后的输出。

**Q2: Backpressure（反压）的原理是什么？为什么重要？**
> 反压是流控机制：当可写端的消费速度跟不上可读端的生产速度时，可写端的内部缓冲区填满（超过 highWaterMark），`write()` 返回 false，可读端收到信号后暂停读取（`pause()`），待可写端缓冲区排空触发 `drain` 事件后，可读端恢复读取（`resume()`）。反压的重要性：如果没有反压，数据会在可写端内部缓冲区无限堆积，导致内存持续增长最终 OOM。`.pipe()` 和 `pipeline()` 内部自动实现反压；手动处理流时必须自行检查 write() 返回值并暂停/恢复读取。

**Q3: pipeline 和 pipe 有什么区别？为什么推荐使用 pipeline？**
> `.pipe()` 的三个缺陷：(1) 不传播错误——可读端的 error 不会传递给可写端，需要为每个流单独监听 error；(2) 不自动清理——出错时不会销毁链中的流，导致资源泄漏；(3) 无法 await。`stream.pipeline()` 的改进：(1) 任一流出错自动传播到回调/await，并销毁链中所有流；(2) 正常完成时自动关闭所有流；(3) `stream/promises` 版本支持 async/await。结论：任何生产代码都应使用 `pipeline` 替代 `.pipe()` 链。

**Q4: Buffer 和 Uint8Array 的关系是什么？有什么区别？**
> Buffer 是 Uint8Array 的子类（`Buffer.prototype.__proto__ === Uint8Array.prototype`），继承其所有方法和属性。关键区别：(1) 内存分配——Buffer 在 V8 堆外由 C++ 层分配，Uint8Array 在 V8 堆内；(2) Slab 机制——小 Buffer 共享同一个 8KB Slab（底层 ArrayBuffer），Uint8Array 每个实例有独立 ArrayBuffer；(3) 便捷方法——Buffer 提供 toString（编码转换）、readInt32LE/writeUInt16BE（多字节操作）等，Uint8Array 无这些方法；(4) 创建方式——Buffer 通过 alloc/allocUnsafe/from 创建，`new Buffer()` 已废弃。在 Node.js API 中，Buffer 和 Uint8Array 通常可互换使用。

**Q5: 流的流动模式和暂停模式有什么区别？如何切换？**
> 流动模式：数据自动推送，通过 data 事件接收，调用 resume() 或 pipe() 或添加 data 监听器进入。暂停模式：数据需手动拉取，调用 read() 方法获取，是可读流的默认初始状态，调用 pause() 或移除所有 data 监听器进入。切换：暂停→流动：添加 data 监听 / 调用 resume() / 调用 pipe()；流动→暂停：调用 pause() / 移除 data 监听。最佳实践：需要自动处理时用流动模式（配合 pipe），需要精细控制读取节奏时用暂停模式（配合 readable 事件 + read() 方法）。

**Q6: highWaterMark 的作用是什么？如何设置？**
> highWaterMark 是流内部缓冲区的高水位线，控制缓冲区大小上限。Readable 流：缓冲区中最多存储 highWaterMark 字节的数据（对象模式下为对象个数），默认 16KB（对象模式 16 个）。Writable 流：write() 返回 false 的阈值，超过后触发反压，默认 16KB。设置方式：创建流时通过选项传入 `fs.createReadStream(path, { highWaterMark: 64 * 1024 })`。影响：值太小→频繁暂停/恢复，增加系统调用开销；值太大→占用更多内存。经验值：小文件 16KB 足够，大文件/高吞吐场景 64KB~256KB，极高吞吐可到 1MB。

**Q7: Transform 流的实现原理是什么？如何自定义 Transform 流？**
> Transform 流继承自 Duplex，但读写不独立——输出是输入的变换结果。内部机制：写入端接收数据 → 存入内部缓冲 → 调用 `_transform(chunk, encoding, callback)` 处理 → 通过 `this.push(transformedChunk)` 输出到读取端。自定义步骤：(1) 继承 Transform 类；(2) 实现 `_transform(chunk, encoding, callback)` 方法，在其中对 chunk 做变换并调用 `this.push(result)` 输出，处理完后调用 callback；(3) 可选实现 `_flush(callback)` 处理流结束时剩余的数据。还可分别设置 `readableObjectMode` 和 `writableObjectMode` 处理不同类型的数据。

**Q8: Stream 导致内存泄漏的常见原因和防范措施？**
> 常见原因：(1) 未销毁流——流出错后未调用 destroy()，内部缓冲区无法释放；(2) `.pipe()` 不传播错误——上游出错后下游不知道，流挂起；(3) 闭包引用——data 事件回调中引用了大对象，流未结束时无法 GC；(4) 高水位线过大——缓冲区设置过高，数据堆积；(5) 未结束的可写流——忘记调用 end()，流持续占用资源。防范措施：(1) 使用 pipeline 自动管理流生命周期；(2) 始终监听 error 事件或在 pipeline 的 catch 中处理；(3) 流处理完成后置 null 释放大对象引用；(4) 设置合理 highWaterMark；(5) 使用 `for await...of` 消费可读流，自动清理。

---

**相关链接：**
- [[事件循环]]
- [[异步编程]]
- [[HTTP服务构建]]
- Node.js Stream 文档：https://nodejs.org/api/stream.html
