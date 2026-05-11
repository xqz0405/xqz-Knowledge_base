---
tags:
  - Web前端
  - JavaScript
  - DOM
  - 事件
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# DOM与事件机制

## What — 是什么

> DOM（Document Object Model）是 HTML 文档的编程接口；事件机制是浏览器处理用户交互和异步通知的核心系统，包括捕获、目标和冒泡三个阶段。

**核心概念：**

- **DOM 树**：文档被解析为节点树（Element/Text/Comment 节点）
- **事件流**：捕获阶段（window→target）→ 目标阶段 → 冒泡阶段（target→window）
- **事件委托**：在父元素监听，利用冒泡处理子元素事件
- **Event 对象**：`e.target`（触发元素）、`e.currentTarget`（监听元素）、`e.preventDefault()`、`e.stopPropagation()`

**关键特性：**

- `addEventListener` 第三个参数控制捕获/冒泡
- `once: true` 自动移除监听
- `passive: true` 优化滚动性能
- 自定义事件：`new CustomEvent('name', { detail: data })`

## Why — 为什么

**适用场景：**

- 用户交互处理（点击、输入、滚动）
- 动态 DOM 操作
- 组件间通信（自定义事件）
- 表单验证

**对比替代方案：**

| 维度 | 原生 DOM 事件 | React 合成事件 | Vue 事件 |
|------|-------------|--------------|---------|
| 跨浏览器 | 需兼容处理 | React 统一 | Vue 统一 |
| 事件委托 | 手动实现 | 自动委托到 root | 自动优化 |
| 性能 | 原生最快 | 合成层开销 | 合成层开销 |

**优缺点：**

- ✅ 优点：
  - 事件委托减少监听器数量，性能好
  - 冒泡机制天然适合 UI 组件层级
  - 自定义事件实现松耦合通信
- ❌ 缺点：
  - 频繁 DOM 操作性能差
  - 内存泄漏风险（忘记移除监听）
  - 事件流调试不直观

## How — 怎么用

### 快速上手

```javascript
// 添加事件监听
button.addEventListener('click', handleClick, { once: true });

// 事件委托
list.addEventListener('click', (e) => {
    const item = e.target.closest('.list-item');
    if (!item) return;
    console.log('clicked:', item.dataset.id);
});

// 自定义事件
const event = new CustomEvent('user-login', { detail: { userId: 1 } });
window.dispatchEvent(event);
```

### 代码示例

**事件委托（列表动态项）：**

```javascript
// ❌ 给每个 item 绑事件，新增 item 需重新绑定
items.forEach(item => item.addEventListener('click', handleClick));

// ✅ 委托到父元素，动态 item 自动生效
document.querySelector('.list').addEventListener('click', (e) => {
    const item = e.target.closest('[data-id]');
    if (item) {
        handleItemClick(item.dataset.id);
    }
});
```

**防止重复点击：**

```javascript
function throttle(fn, delay) {
    let lastCall = 0;
    return function (...args) {
        const now = Date.now();
        if (now - lastCall >= delay) {
            lastCall = now;
            fn.apply(this, args);
        }
    };
}

button.addEventListener('click', throttle(submitForm, 1000));
```

**滚动性能优化：**

```javascript
// passive: true 告诉浏览器不会调用 preventDefault
// 允许浏览器立即开始滚动，不等 JS 执行完
window.addEventListener('scroll', handleScroll, { passive: true });

// ❌ 不加 passive，滚动可能卡顿
window.addEventListener('touchstart', handleTouch); // 阻塞滚动
```

**MutationObserver 监听 DOM 变化：**

```javascript
const observer = new MutationObserver((mutations) => {
    mutations.forEach(m => {
        if (m.type === 'childList') {
            console.log('新增节点:', m.addedNodes);
        }
    });
});

observer.observe(container, {
    childList: true,
    subtree: true,
});

// 停止观察
observer.disconnect();
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 事件监听泄漏 | 组件销毁时未移除监听 | 保存引用，`removeEventListener` 或用 `once: true` |
| `e.target` 不是期望元素 | 事件冒泡，子元素触发 | 用 `e.target.closest()` 找到目标 |
| `stopPropagation` 阻断委托 | 冒泡被中断 | 只在必要时 stop，优先用条件判断 |
| scroll/touch 卡顿 | 监听器阻塞了浏览器滚动 | 加 `{ passive: true }` |

### 最佳实践

- 优先用事件委托，减少监听器数量
- 组件销毁时移除事件监听
- scroll/touch 事件加 `passive: true`
- 用 `closest()` 替代逐层判断 `e.target`

## 面试题

**Q1: 事件冒泡和捕获的区别是什么？**
> DOM 事件流分为三个阶段：捕获阶段（从 window 向下传到目标元素）、目标阶段、冒泡阶段（从目标元素向上传到 window）。`addEventListener` 第三个参数为 `true` 时在捕获阶段监听，为 `false`（默认）时在冒泡阶段监听。实际开发中主要使用冒泡阶段。

**Q2: 什么是事件委托？有什么优势？**
> 事件委托是利用事件冒泡，在父元素上统一监听子元素的事件。优势：1) 减少事件监听器数量，节省内存；2) 动态添加的子元素自动生效，无需重新绑定；3) 代码更简洁。实现方式：在父元素监听事件，通过 `e.target.closest()` 找到实际触发的子元素。

**Q3: `e.target` 和 `e.currentTarget` 有什么区别？**
> `e.target` 是事件的实际触发元素（最内层被点击的元素），在事件传播过程中不变。`e.currentTarget` 是当前绑定事件监听器的元素（即 `addEventListener` 绑定的元素），在冒泡/捕获过程中会变化。在事件委托中，`e.target` 是子元素，`e.currentTarget` 是父元素。

**Q4: `e.preventDefault()` 和 `e.stopPropagation()` 有什么区别？**
> `e.preventDefault()` 阻止元素的默认行为（如链接跳转、表单提交），不阻止事件继续传播。`e.stopPropagation()` 阻止事件继续冒泡或捕获，但不阻止默认行为。两者互不影响，可单独或同时使用。

---

**相关链接：**
- [[Promise与异步]]
- [[浏览器渲染原理]]
- [[ES6+核心特性]]
