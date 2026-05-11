---
tags:
  - Go
  - 并发
  - Channel
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Channel与Select

## What — 是什么

> Channel 是 Go 语言中 Goroutine 间通信的管道，遵循 CSP 模型"不要通过共享内存来通信，而要通过通信来共享内存"。Select 用于同时监听多个 Channel 操作。

**核心概念：**

- **无缓冲 Channel**：`make(chan T)` — 发送和接收必须同时就绪，同步语义
- **缓冲 Channel**：`make(chan T, n)` — 缓冲区满前发送不阻塞
- **Select**：多路复用，随机选取一个就绪的 case 执行
- **方向限定**：`chan<- T`（只发送）、`<-chan T`（只接收）

**关键特性：**

- Channel 是一等公民，可赋值、传参、作为结构体字段
- 关闭 Channel 后仍可读取（返回零值），不可再发送
- Select 的 default 分支实现非阻塞操作

## Why — 为什么

**适用场景：**

- Goroutine 间数据传递
- 信号通知（关闭 channel 广播）
- 并行任务结果收集
- 超时控制（配合 `time.After`）

**对比替代方案：**

| 维度 | Channel | Mutex | atomic |
|------|---------|-------|--------|
| 语义 | 通信/数据流 | 共享状态保护 | 单变量原子操作 |
| 适用场景 | 数据传递、信号 | 临界区保护 | 计数器/标志位 |
| 复杂度 | 中 | 高（易死锁） | 低 |
| 调试 | 较容易 | 较困难 | 中等 |

**优缺点：**

- ✅ 优点：
  - 语义清晰，表达"数据流"而非"锁"
  - 天然避免数据竞争
  - 与 select 配合实现超时/取消
- ❌ 缺点：
  - 有性能开销（锁 + 内存分配）
  - Channel 泄漏风险（无人接收/发送）
  - 不适合保护复杂共享状态

## How — 怎么用

### 快速上手

```go
// 无缓冲 Channel
ch := make(chan string)

go func() { ch <- "hello" }()
msg := <-ch // 阻塞直到收到
fmt.Println(msg)

// 缓冲 Channel
bufCh := make(chan int, 3)
bufCh <- 1
bufCh <- 2
fmt.Println(<-bufCh) // 1
```

### 代码示例

**Select 多路复用：**

```go
func fanIn(ch1, ch2 <-chan string) <-chan string {
    merged := make(chan string)
    go func() {
        defer close(merged)
        for ch1 != nil || ch2 != nil {
            select {
            case v, ok := <-ch1:
                if !ok { ch1 = nil; continue }
                merged <- v
            case v, ok := <-ch2:
                if !ok { ch2 = nil; continue }
                merged <- v
            }
        }
    }()
    return merged
}
```

**超时控制：**

```go
select {
case result := <-doWork():
    fmt.Println("got:", result)
case <-time.After(2 * time.Second):
    fmt.Println("timeout!")
}
```

**非阻塞操作：**

```go
select {
case msg := <-ch:
    fmt.Println("received:", msg)
default:
    fmt.Println("no message available")
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 向已关闭 Channel 发送数据 | close 后不能再写入 | 只有发送方关闭 Channel，用 `sync.Once` 保证只关闭一次 |
| Channel 死锁 | 所有 Goroutine 都在等待发送/接收 | 确保有对应的发送/接收方，使用缓冲 Channel 解耦 |
| Goroutine 泄漏 | Channel 无人接收，发送方永远阻塞 | 使用 context 取消，或 buffered channel |
| Select 空选择 | 所有 case 都未就绪且无 default | 添加 default 或 time.After 分支 |

### 最佳实践

- 发送方负责关闭 Channel，接收方不要关闭
- 用 `v, ok := <-ch` 检测 Channel 是否关闭
- 优先用 Channel 传递数据，Mutex 保护状态
- 生产代码中 Channel 操作都应有超时或取消机制

## 面试题

**Q1: 无缓冲 Channel 和缓冲 Channel 的区别是什么？**
> 无缓冲 Channel（`make(chan T)`）发送和接收必须同时就绪，是同步的；缓冲 Channel（`make(chan T, n)`）缓冲区未满时发送不阻塞，是异步的。无缓冲适合强制同步语义（如信号通知），缓冲适合解耦生产消费速率。

**Q2: Select 有哪些重要特性？**
> ①多个 case 同时就绪时随机选取一个执行，保证公平性；②无 case 就绪且无 default 时阻塞；③有 default 时实现非阻塞操作；④可用 `v, ok := <-ch` 检测 channel 关闭；⑤空 select（`select{}`）永远阻塞。

**Q3: Channel 关闭有哪些规则？为什么只能发送方关闭？**
> 规则：①关闭后不能再发送（panic）；②关闭后仍可读取，返回零值和 ok=false；③重复关闭会 panic；④关闭 nil channel 会 panic。只由发送方关闭是因为接收方不知道是否还有数据待发送，提前关闭会导致发送方 panic，这是"单一职责"原则的体现。

**Q4: 如何检测 Go 程序中的竞态条件？**
> 使用 `go test -race` 或 `go build -race` 启用竞态检测器。它基于 ThreadSanitizer 实现，在运行时动态检测对共享变量的非同步并发访问。生产环境不建议开启（有 5-10 倍性能开销），但应作为 CI 必选项。

---

**相关链接：**
