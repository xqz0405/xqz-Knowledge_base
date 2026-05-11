---
tags:
  - Go
  - 并发
  - Goroutine
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Goroutine与调度

## What — 是什么

> Goroutine 是 Go 语言实现的轻量级用户态线程，由 Go 运行时调度而非操作系统。

**核心概念：**

- **G**（Goroutine）：用户态协程，初始栈仅 2KB（可动态增长至 1GB）
- **M**（Machine）：操作系统线程，真正执行代码的载体
- **P**（Processor）：逻辑处理器，持有一个本地运行队列，数量默认等于 CPU 核心数（`GOMAXPROCS`）

**关键特性：**

- 创建成本极低，可轻松创建数十万 Goroutine
- 栈大小动态伸缩，按需分配
- 调度在用户态完成，不需要内核态切换

**运行机制：**

- 内存模型：每个 G 有独立栈，堆内存由 GC 统一管理
- 执行模型：G-M-P 模型，G 在 P 的本地队列中等待被 M 执行
- 并发模型：CSP（Communicating Sequential Processes），通过 Channel 通信而非共享内存

**类型系统：**

- 类型分类：Goroutine 本身不是类型，`go` 关键字启动函数调用
- 类型转换规则：不适用
- 泛型/多态支持：不适用

## Why — 为什么

**适用场景：**

- 高并发网络服务（HTTP、gRPC）
- I/O 密集型任务（数据库查询、文件读写）
- 并行计算任务

**对比其他语言：**

| 维度 | Goroutine | Java 线程 | Python 协程 |
|------|-----------|----------|-------------|
| 性能 | 高（用户态调度） | 中（内核态线程） | 高（用户态） |
| 生态 | 强（内置并发原语） | 强（JUC 工具链） | 中（asyncio 生态较弱） |
| 上手难度 | 低（`go func()`） | 高（线程同步复杂） | 中（需理解 async/await） |
| 并发能力 | 极强（百万级） | 中（千级线程） | 强（百万级协程） |

**优缺点：**

- ✅ 优点：
  - 语法极简，`go f()` 一行启动
  - 调度器自动处理阻塞，无需手动让出 CPU
  - 栈动态增长，内存利用率高
- ❌ 缺点：
  - 调度不透明，调试并发问题较困难
  - 没有"协程优先级"，无法精细控制调度顺序
  - Goroutine 泄漏是常见问题，需要搭配 context 管理

## How — 怎么用

### 快速上手

```go
// 启动 Goroutine
go func() {
    fmt.Println("hello from goroutine")
}()

// 等待 Goroutine 完成
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        fmt.Printf("worker %d\n", id)
    }(i)
}
wg.Wait()
```

### 代码示例

**使用 context 控制超时和取消：**

```go
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()

go func(ctx context.Context) {
    select {
    case <-time.After(5 * time.Second):
        fmt.Println("work done")
    case <-ctx.Done():
        fmt.Println("cancelled:", ctx.Err()) // context deadline exceeded
    }
}(ctx)

time.Sleep(4 * time.Second)
```

**Goroutine 泄漏检测：**

```go
// 危险：Goroutine 永远不会退出
func leak() {
    ch := make(chan int)
    go func() {
        val := <-ch // 永远阻塞
        fmt.Println(val)
    }()
    // 函数返回后，Goroutine 仍在等待
}

// 修复：使用 context 或缓冲 channel
func noLeak() {
    ch := make(chan int, 1)
    ch <- 42 // 缓冲区保证不阻塞
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Goroutine 泄漏 | Channel 无发送方/接收方导致永久阻塞 | 使用 context 管理生命周期，配合 `runtime.NumGoroutine()` 监控 |
| 闭包捕获循环变量 | `go func() { use(i) }()` 中 `i` 共享最终值 | 参数传递 `go func(i int) { use(i) }(i)` |
| 数据竞争 | 多个 Goroutine 并发读写同一变量 | 使用 `go test -race` 检测，加锁或使用 Channel |
| 调度饥饿 | 长时间计算不主动让出 CPU | 使用 `runtime.Gosched()` 让出时间片 |

### 最佳实践

- 始终使用 `sync.WaitGroup` 或 `errgroup.Group` 等待 Goroutine 完成
- 用 `context.Context` 传递取消信号，而非全局变量
- 生产环境用 `runtime.NumGoroutine()` 做监控告警
- 使用 `go test -race` 作为 CI 必选项

## 面试题

**Q1: GMP 模型中 G、M、P 分别代表什么？为什么需要 P？**
> G 是 Goroutine（用户态协程），M 是 Machine（操作系统线程），P 是 Processor（逻辑处理器）。引入 P 是为了解耦 G 和 M 的绑定关系，P 持有本地运行队列，M 只需绑定 P 即可执行 G，避免了全局队列锁竞争，实现了工作窃取（work stealing）机制。

**Q2: Goroutine 和操作系统线程的区别是什么？**
> Goroutine 初始栈仅 2KB（线程通常 1-8MB），由 Go 运行时在用户态调度（线程由 OS 内核调度），创建和切换成本远低于线程。一个 Goroutine 阻塞时，绑定的 M 会与 P 解绑，P 寻找其他空闲 M 继续执行队列中的 G，不会浪费 CPU 资源。

**Q3: Go 调度器有哪些调度策略？**
> 主要有四种：①工作窃取（work stealing）：P 本地队列为空时从其他 P 偷 G；②hand off：M 阻塞时释放 P 给其他 M；③抢占式调度：基于 sysmon 协作抢占，Go 1.14+ 支持基于信号的异步抢占；④全局队列：P 本地队列为空时从全局队列获取 G。

**Q4: 什么是 Goroutine 泄漏？如何检测和避免？**
> Goroutine 泄漏指启动的 Goroutine 永远无法退出（如 channel 永远阻塞、缺少退出条件）。检测方法：使用 `runtime.NumGoroutine()` 监控数量变化，或用 pprof 的 goroutine profile。避免方法：始终用 context 管理生命周期、确保 channel 有对应的发送/接收方、为循环中的 select 添加退出条件。

---

**相关链接：**
- Go 官方博客：https://go.dev/blog/io2013-talk-concurrency
