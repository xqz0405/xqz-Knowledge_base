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

> DOM（Document Object Model）是 HTML 文档的编程接口，将文档解析为节点树；事件机制是浏览器处理用户交互和异步通知的核心系统，包括捕获、目标和冒泡三个阶段。

### DOM 树的完整结构

DOM 将 HTML/XML 文档解析为一棵由节点（Node）组成的树结构。所有节点都继承自 `Node` 接口。

**核心 Node 类型：**

| 节点类型 | nodeType 值 | 说明 | 示例 |
|---------|------------|------|------|
| `Document` | 9 | 文档根节点 | `document` |
| `DocumentFragment` | 11 | 轻量级文档片段 | `document.createDocumentFragment()` |
| `Element` | 1 | 元素节点 | `<div>`, `<p>` |
| `Text` | 3 | 文本节点 | 标签间的文字内容 |
| `Comment` | 8 | 注释节点 | `<!-- 注释 -->` |
| `DocumentType` | 10 | DOCTYPE 声明 | `<!DOCTYPE html>` |
| `Attr` | 2 | 属性节点（已废弃，用 `getAttribute` 替代） | `id="app"` |
| `CDATASection` | 4 | CDATA 块（XML 中使用） | `<![CDATA[...]]>` |

**DocumentFragment 的特殊性：**

- 不是真实 DOM 树的一部分，存在于内存中
- 插入文档时，其子节点会被插入，自身不会进入 DOM
- 批量 DOM 操作的理想容器，减少重排重绘

```javascript
// DocumentFragment 批量插入
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    fragment.appendChild(li);
}
// 只触发一次重排
list.appendChild(fragment);
```

### DOM 节点关系

每个 DOM 节点都有一组属性描述其在树中的位置关系：

| 属性 | 说明 | 返回值 |
|------|------|--------|
| `parentNode` | 父节点 | Node 或 null |
| `parentElement` | 父元素节点 | Element 或 null |
| `childNodes` | 所有子节点（含文本、注释） | NodeList（动态） |
| `children` | 仅子元素节点 | HTMLCollection（动态） |
| `firstChild` | 第一个子节点 | Node 或 null |
| `firstElementChild` | 第一个子元素 | Element 或 null |
| `lastChild` | 最后一个子节点 | Node 或 null |
| `lastElementChild` | 最后一个子元素 | Element 或 null |
| `nextSibling` | 下一个兄弟节点 | Node 或 null |
| `nextElementSibling` | 下一个兄弟元素 | Element 或 null |
| `previousSibling` | 上一个兄弟节点 | Node 或 null |
| `previousElementSibling` | 上一个兄弟元素 | Element 或 null |

> 注意：`childNodes` 包含文本节点（空白缩进也会算）、注释节点等；`children` 只返回元素节点，日常开发优先用 `children` 系列。

```javascript
// 节点关系遍历示例
// <ul id="list">
//   <li>Item 1</li>
//   <li>Item 2</li>
//   <li>Item 3</li>
// </ul>

const list = document.getElementById('list');

// children 只拿元素
console.log(list.children.length);       // 3（三个 li）
console.log(list.childNodes.length);     // 7（3 个 li + 4 个文本节点/换行）

// 遍历元素节点
Array.from(list.children).forEach((li, index) => {
    console.log(li.textContent); // Item 1, Item 2, Item 3
});

// 兄弟遍历
let current = list.firstElementChild;
while (current) {
    console.log(current.textContent);
    current = current.nextElementSibling;
}
```

### 事件传播三阶段详解

DOM 事件传播遵循 **W3C 事件模型**，分为三个阶段：

```
捕获阶段        目标阶段        冒泡阶段
window          ┌─────┐         window
  ↓             │ target│          ↑
document        └─────┘        document
  ↓                               ↑
html                              html
  ↓                               ↑
body                              body
  ↓                               ↑
div                               div
  ↓                               ↑
button ────→ [目标阶段] ────→ button
```

1. **捕获阶段（Capture Phase）**：事件从 `window` 向下传递到目标元素的父元素。此阶段主要用于全局拦截或调试。
2. **目标阶段（Target Phase）**：事件到达目标元素本身。此时按注册顺序执行，不区分捕获/冒泡。
3. **冒泡阶段（Bubbling Phase）**：事件从目标元素向上冒泡回 `window`。日常开发主要使用此阶段。

```javascript
// 三阶段演示
const grandparent = document.querySelector('.grandparent');
const parent = document.querySelector('.parent');
const child = document.querySelector('.child');

// 捕获阶段
grandparent.addEventListener('click', () => {
    console.log('1. 祖父 - 捕获');
}, true);  // 第三个参数 true = 捕获阶段

parent.addEventListener('click', () => {
    console.log('2. 父 - 捕获');
}, true);

// 冒泡阶段
parent.addEventListener('click', () => {
    console.log('3. 父 - 冒泡');
}, false); // 默认 false

grandparent.addEventListener('click', () => {
    console.log('4. 祖父 - 冒泡');
}, false);

// 点击 child 输出：1 → 2 → 3 → 4
// 点击 parent 输出：1 → 2 → 3 → 4（目标阶段按注册顺序执行）
```

> 部分事件不冒泡：`focus`、`blur`、`load`、`unload`、`mouseenter`、`mouseleave`。对应的冒泡版本：`focusin`、`focusout`、`mouseover`、`mouseout`。

### 事件对象完整属性

事件处理函数接收一个 `Event` 对象，包含以下关键属性和方法：

| 属性/方法 | 类型 | 说明 |
|-----------|------|------|
| `e.target` | 属性 | 事件的实际触发元素 |
| `e.currentTarget` | 属性 | 当前绑定监听器的元素 |
| `e.type` | 属性 | 事件类型字符串（如 `'click'`） |
| `e.bubbles` | 属性 | 事件是否冒泡 |
| `e.cancelable` | 属性 | 事件是否可被 `preventDefault` |
| `e.defaultPrevented` | 属性 | 是否已调用 `preventDefault()` |
| `e.eventPhase` | 属性 | 当前阶段（1=捕获, 2=目标, 3=冒泡） |
| `e.timeStamp` | 属性 | 事件创建的时间戳（ms） |
| `e.isTrusted` | 属性 | 是否由用户操作触发（非 JS 派发） |
| `e.preventDefault()` | 方法 | 阻止默认行为 |
| `e.stopPropagation()` | 方法 | 阻止事件继续传播 |
| `e.stopImmediatePropagation()` | 方法 | 阻止传播且阻止同元素其他监听器 |
| `e.composedPath()` | 方法 | 返回事件传播路径（从 target 到 window） |

```javascript
// composedPath 获取完整传播路径
document.querySelector('.child').addEventListener('click', (e) => {
    const path = e.composedPath();
    // [child, parent, grandparent, body, html, document, window]
    console.log('事件传播路径:', path.map(el => el.tagName || el.constructor.name));
});

// stopImmediatePropagation 区别演示
btn.addEventListener('click', (e) => {
    e.stopImmediatePropagation();
    console.log('监听器 A'); // 会执行
});
btn.addEventListener('click', (e) => {
    console.log('监听器 B'); // 不会执行！被 A 阻断了
});
```

### 事件委托原理

事件委托（Event Delegation）基于事件冒泡机制：不在每个子元素上绑定事件，而是在父元素上统一监听，通过 `e.target` 判断实际触发的子元素。

```
传统方式：每个子元素绑定事件      委托方式：只在父元素绑定
┌─────────────────┐           ┌─────────────────┐
│  ul             │           │  ul [监听]       │
│  ┌───┐ ┌───┐   │           │  ┌───┐ ┌───┐   │
│  │li1│ │li2│   │           │  │li1│ │li2│   │
│  └───┘ └───┘   │           │  └───┘ └───┘   │
│  [监听] [监听]  │           │                 │
└─────────────────┘           └─────────────────┘
  N 个监听器                     1 个监听器
```

**核心原理：**
- 事件冒泡保证了子元素的事件会传到父元素
- `e.target` 指向实际触发的子元素
- `closest()` 可向上查找匹配的祖先元素

### 常用事件分类

| 分类 | 事件名 | 说明 |
|------|--------|------|
| **鼠标** | `click`, `dblclick`, `mousedown`, `mouseup`, `mousemove`, `mouseover`, `mouseout`, `mouseenter`, `mouseleave`, `contextmenu` | 鼠标交互，`enter/leave` 不冒泡 |
| **键盘** | `keydown`, `keyup`, `keypress`(已废弃) | 按键交互，`key`/`code` 属性获取按键信息 |
| **表单** | `submit`, `reset`, `change`, `input`, `focus`, `blur`, `focusin`, `focusout`, `invalid` | 表单元素交互与验证 |
| **触摸** | `touchstart`, `touchmove`, `touchend`, `touchcancel` | 移动端触控 |
| **拖拽** | `drag`, `dragstart`, `dragend`, `dragover`, `dragenter`, `dragleave`, `drop` | HTML5 拖放 API |
| **滚动** | `scroll`, `wheel` | 页面或元素滚动 |
| **窗口** | `resize`, `load`, `DOMContentLoaded`, `beforeunload`, `unload`, `hashchange`, `popstate` | 窗口/页面生命周期 |
| **媒体** | `play`, `pause`, `ended`, `timeupdate`, `volumechange`, `loadeddata`, `canplay` | 音视频播放控制 |
| **剪贴板** | `copy`, `cut`, `paste` | 剪贴板操作 |
| **网络** | `online`, `offline`, `error` | 网络状态 |

## Why — 为什么

**适用场景：**

- 用户交互处理（点击、输入、滚动）
- 动态 DOM 操作
- 组件间通信（自定义事件）
- 表单验证
- 响应式布局（resize 事件）
- 无限滚动与懒加载
- 拖拽交互

### 直接绑定 vs 事件委托对比

| 维度 | 直接绑定（每个子元素） | 事件委托（父元素统一） |
|------|----------------------|----------------------|
| 内存占用 | N 个监听器 = N 份内存 | 1 个监听器 |
| 动态元素 | 新增元素需重新绑定 | 自动生效，无需额外处理 |
| 初始化速度 | 大量绑定时较慢 | 极快，只绑定一次 |
| 代码复杂度 | 简单直接 | 需要 `closest()` 判断目标 |
| 事件精度 | 精确到元素 | 需要过滤非目标触发 |
| 移除清理 | 需逐一移除 | 只需移除一个监听器 |
| 适用场景 | 元素数量少且固定 | 元素数量多或动态增减 |

### 事件代理的性能优势

1. **内存节省**：1000 个列表项只需 1 个监听器，而非 1000 个
2. **初始化时间**：绑定 1 次 vs 绑定 1000 次
3. **GC 友好**：减少闭包引用，降低内存泄漏风险
4. **动态适配**：AJAX 加载新内容无需重新绑定事件

```javascript
// 性能对比：直接绑定 vs 事件委托
// 1000 个列表项的场景

// ❌ 直接绑定：创建 1000 个监听器
document.querySelectorAll('.item').forEach(item => {
    item.addEventListener('click', handleClick); // 1000 个闭包
});

// ✅ 事件委托：只创建 1 个监听器
document.querySelector('.list').addEventListener('click', (e) => {
    const item = e.target.closest('.item');
    if (item) handleClick(item);
});
```

### 为什么不推荐 inline event handler

内联事件处理器（如 `onclick="..."`）存在多个问题：

| 问题 | 说明 |
|------|------|
| 违反关注点分离 | HTML 和 JS 混合，难以维护 |
| 全局作用域污染 | 处理函数必须在全局作用域可访问 |
| 无法绑定多个处理器 | 后写的会覆盖先写的 |
| 无法使用事件选项 | 不支持 `once`、`passive`、`capture` |
| 无法方便地移除 | 需要 `element.onclick = null` |
| 字符串解析开销 | 每次触发需在全局作用域解析字符串 |
| CSP 安全限制 | 内容安全策略可能禁止内联脚本 |

```html
<!-- ❌ inline handler -->
<button onclick="handleClick()">Click</button>

<!-- ✅ addEventListener -->
<button id="btn">Click</button>
<script>
    document.getElementById('btn').addEventListener('click', handleClick);
</script>
```

### 对比替代方案

| 维度 | 原生 DOM 事件 | React 合成事件 | Vue 事件 |
|------|-------------|--------------|---------|
| 跨浏览器 | 需兼容处理 | React 统一 | Vue 统一 |
| 事件委托 | 手动实现 | 自动委托到 root | 自动优化 |
| 性能 | 原生最快 | 合成层开销 | 合成层开销 |
| 事件池 | 无 | React 16 有事件池复用 | 无 |
| 绑定方式 | `addEventListener` | JSX `onClick` | `@click` |

### 优缺点

- **优点：**
  - 事件委托减少监听器数量，性能好
  - 冒泡机制天然适合 UI 组件层级
  - 自定义事件实现松耦合通信
  - 标准化的事件模型，浏览器原生支持
  - Observer 系列 API 提供声明式的 DOM 监听能力

- **缺点：**
  - 频繁 DOM 操作性能差（触发重排重绘）
  - 内存泄漏风险（忘记移除监听）
  - 事件流调试不直观
  - `stopPropagation` 可能意外阻断委托链

## How — 怎么用

### 1. DOM 操作 CRUD

```javascript
// === 创建（Create） ===
// 创建元素
const div = document.createElement('div');
div.id = 'app';
div.className = 'container';
div.textContent = 'Hello'; // 安全，自动转义
div.innerHTML = '<span>内容</span>'; // 注意 XSS 风险

// 创建文本节点
const text = document.createTextNode('纯文本');

// 创建 DocumentFragment（批量操作）
const fragment = new DocumentFragment(); // 或 document.createDocumentFragment()

// === 查询（Read） ===
// 现代 API
document.querySelector('.item');              // 第一个匹配
document.querySelectorAll('.item');           // 所有匹配，返回 NodeList
document.getElementById('app');               // 按 ID（最快）
document.getElementsByClassName('item');      // 返回 HTMLCollection（动态）
document.getElementsByTagName('div');         // 返回 HTMLCollection（动态）

// 关系查询
element.closest('.parent');                   // 向上查找最近匹配祖先
element.matches('.active');                   // 是否匹配选择器
element.contains(otherElement);               // 是否包含后代节点

// === 修改（Update） ===
// 属性操作
element.setAttribute('data-id', '123');
element.getAttribute('data-id');              // '123'
element.removeAttribute('data-id');
element.hasAttribute('data-id');              // false

// dataset 操作（data-* 属性）
element.dataset.userId = '42';                // data-user-id="42"
console.log(element.dataset.userId);          // "42"

// class 操作
element.classList.add('active');
element.classList.remove('active');
element.classList.toggle('active');
element.classList.contains('active');
element.classList.replace('old', 'new');

// style 操作
element.style.cssText = 'color: red; font-size: 16px;';
element.style.setProperty('--custom-color', 'blue'); // CSS 变量
element.style.removeProperty('color');

// === 删除（Delete） ===
element.remove();                             // 直接移除自身
parent.removeChild(child);                    // 移除子节点（返回被移除节点）
parent.replaceChild(newChild, oldChild);      // 替换子节点
element.innerHTML = '';                        // 清空所有子节点

// 安全清空（移除事件监听）
while (element.firstChild) {
    element.removeChild(element.firstChild);
}
```

### 2. 事件监听完整用法

```javascript
// addEventListener 完整签名
// target.addEventListener(type, listener, options | useCapture)

// 基础用法
element.addEventListener('click', handler);

// 第三个参数：布尔值（旧语法）
element.addEventListener('click', handler, true);  // 捕获阶段

// 第三个参数：选项对象（推荐）
element.addEventListener('click', handler, {
    capture: false,   // 是否在捕获阶段触发（默认 false）
    once: true,       // 触发一次后自动移除（默认 false）
    passive: true,    // 声明不会调用 preventDefault（默认 false）
    signal: abortController.signal  // AbortController 信号，用于移除
});

// 移除监听器
element.removeEventListener('click', handler); // handler 必须是同一引用

// 使用 AbortController 移除（现代方式，推荐）
const controller = new AbortController();
element.addEventListener('click', handler, { signal: controller.signal });
// 需要移除时
controller.abort(); // 一次性移除通过此信号绑定的所有监听器

// 多个监听器的执行顺序
element.addEventListener('click', () => console.log('A'));
element.addEventListener('click', () => console.log('B'));
// 点击输出：A → B（按注册顺序执行）
```

### 3. 事件委托实战

```javascript
// 场景一：动态列表项点击
document.querySelector('.todo-list').addEventListener('click', (e) => {
    const item = e.target.closest('.todo-item');
    if (!item) return; // 点击的不是 todo-item

    const id = item.dataset.id;
    const action = e.target.closest('[data-action]')?.dataset.action;

    switch (action) {
        case 'toggle':
            item.classList.toggle('completed');
            break;
        case 'delete':
            item.remove();
            break;
        case 'edit':
            startEditing(item);
            break;
    }
});

// 场景二：动态加载的内容
const container = document.getElementById('feed');
container.addEventListener('click', (e) => {
    const likeBtn = e.target.closest('.like-btn');
    if (likeBtn) {
        likeBtn.classList.toggle('liked');
        updateLikeCount(likeBtn);
        return;
    }

    const shareBtn = e.target.closest('.share-btn');
    if (shareBtn) {
        handleShare(shareBtn.dataset.postId);
        return;
    }
});

// 动态添加的内容自动生效
function appendPost(post) {
    const html = `
        <div class="post" data-post-id="${post.id}">
            <button class="like-btn">Like</button>
            <button class="share-btn" data-post-id="${post.id}">Share</button>
        </div>
    `;
    container.insertAdjacentHTML('beforeend', html);
    // 无需重新绑定事件！
}
```

### 4. 自定义事件（CustomEvent）

```javascript
// 创建自定义事件
const event = new CustomEvent('user-login', {
    detail: { userId: 42, username: 'alice' }, // 携带数据
    bubbles: true,     // 是否冒泡（默认 false）
    cancelable: true,  // 是否可 preventDefault（默认 false）
});

// 派发事件
window.dispatchEvent(event);

// 监听自定义事件
window.addEventListener('user-login', (e) => {
    console.log('用户登录:', e.detail.userId, e.detail.username);
});

// === 实际应用：组件间通信 ===
// 简易事件总线
class EventBus {
    constructor() {
        this.events = new Map();
    }

    on(event, handler) {
        if (!this.events.has(event)) {
            this.events.set(event, []);
        }
        this.events.get(event).push(handler);
    }

    off(event, handler) {
        const handlers = this.events.get(event);
        if (handlers) {
            this.events.set(event, handlers.filter(h => h !== handler));
        }
    }

    emit(event, data) {
        const handlers = this.events.get(event);
        if (handlers) {
            handlers.forEach(h => h(data));
        }
    }
}

const bus = new EventBus();
bus.on('cart-updated', (data) => updateCartUI(data));
bus.emit('cart-updated', { count: 3, total: 99.9 });
```

### 5. 防抖/节流在事件中的应用

```javascript
// === 防抖（Debounce）：事件停止触发后延迟执行 ===
function debounce(fn, delay = 300) {
    let timer = null;
    return function (...args) {
        clearTimeout(timer);
        timer = setTimeout(() => fn.apply(this, args), delay);
    };
}

// 适用场景：搜索输入、窗口 resize
const searchInput = document.querySelector('#search');
searchInput.addEventListener('input', debounce((e) => {
    fetchSuggestions(e.target.value);
}, 300));

window.addEventListener('resize', debounce(() => {
    recalculateLayout();
}, 250));

// === 节流（Throttle）：固定时间间隔执行一次 ===
function throttle(fn, interval = 300) {
    let lastTime = 0;
    return function (...args) {
        const now = Date.now();
        if (now - lastTime >= interval) {
            lastTime = now;
            fn.apply(this, args);
        }
    };
}

// 带尾调用保证最后一次触发的节流
function throttleWithTrailing(fn, interval = 300) {
    let lastTime = 0;
    let timer = null;
    return function (...args) {
        const now = Date.now();
        const remaining = interval - (now - lastTime);

        clearTimeout(timer);

        if (remaining <= 0) {
            lastTime = now;
            fn.apply(this, args);
        } else {
            // 保证最后一次触发会被执行
            timer = setTimeout(() => {
                lastTime = Date.now();
                fn.apply(this, args);
            }, remaining);
        }
    };
}

// 适用场景：scroll、mousemove、按钮防连点
window.addEventListener('scroll', throttle(handleScroll, 100));
canvas.addEventListener('mousemove', throttle(drawTrail, 16)); // ~60fps
submitBtn.addEventListener('click', throttle(submitForm, 1000));
```

**防抖 vs 节流对比：**

| 维度 | 防抖（Debounce） | 节流（Throttle） |
|------|------------------|------------------|
| 执行时机 | 停止触发后执行 | 固定间隔执行 |
| 高频触发时 | 只执行最后一次 | 按间隔执行多次 |
| 适用场景 | 搜索输入、resize | 滚动、拖拽、resize |
| 比喻 | 电梯等人 | 红绿灯放行 |

### 6. passive 事件监听器

```javascript
// passive: true 的含义
// 告诉浏览器：监听器内部不会调用 e.preventDefault()
// 浏览器可以立即开始滚动/缩放，无需等待 JS 执行完毕

// ✅ 正确用法：scroll/touch 事件加 passive
window.addEventListener('scroll', handleScroll, { passive: true });
document.addEventListener('touchstart', handleTouchStart, { passive: true });
document.addEventListener('touchmove', handleTouchMove, { passive: true });
document.addEventListener('wheel', handleWheel, { passive: true });

// ❌ 错误用法：passive + preventDefault 矛盾
element.addEventListener('touchstart', (e) => {
    e.preventDefault(); // 控制台警告！passive 监听器不能 preventDefault
}, { passive: true });

// ❌ 需要阻止默认行为时不能加 passive
element.addEventListener('touchstart', (e) => {
    e.preventDefault(); // 阻止移动端 300ms 延迟等
}, { passive: false }); // 明确声明

// Chrome 中 scroll/touch 事件默认 passive: true
// 以下代码在 Chrome 中 passive 不生效（已经是默认行为）
window.addEventListener('touchstart', handler); // Chrome 默认 passive

// 检测浏览器是否默认 passive
let supportsPassive = false;
try {
    const opts = Object.defineProperty({}, 'passive', {
        get() { supportsPassive = true; }
    });
    window.addEventListener('test', null, opts);
} catch (e) {}
```

### 7. MutationObserver 监听 DOM 变化

```javascript
// MutationObserver：监听 DOM 变化（替代已废弃的 Mutation Events）
const observer = new MutationObserver((mutationsList, observer) => {
    for (const mutation of mutationsList) {
        switch (mutation.type) {
            case 'childList':
                console.log('子节点变化:');
                console.log('  新增:', mutation.addedNodes);
                console.log('  删除:', mutation.removedNodes);
                break;
            case 'attributes':
                console.log(`属性 ${mutation.attributeName} 变化:`,
                    mutation.oldValue, '→',
                    mutation.target.getAttribute(mutation.attributeName));
                break;
            case 'characterData':
                console.log('文本内容变化:', mutation.oldValue);
                break;
        }
    }
});

// 配置观察选项
observer.observe(targetElement, {
    childList: true,       // 观察子节点增删
    attributes: true,      // 观察属性变化
    characterData: true,   // 观察文本内容变化
    subtree: true,         // 观察所有后代节点
    attributeOldValue: true,       // 记录属性旧值
    characterDataOldValue: true,   // 记录文本旧值
    attributeFilter: ['class', 'data-status'], // 只观察指定属性
});

// 停止观察
observer.disconnect();

// 实战：监听 SPA 页面内容变化
const pageObserver = new MutationObserver(() => {
    if (document.querySelector('.dynamic-content')) {
        initDynamicFeatures();
    }
});
pageObserver.observe(document.body, { childList: true, subtree: true });

// 实战：监听属性变化实现响应式
const attrObserver = new MutationObserver((mutations) => {
    mutations.forEach(m => {
        if (m.attributeName === 'data-loading') {
            const isLoading = m.target.dataset.loading === 'true';
            toggleSpinner(isLoading);
        }
    });
});
attrObserver.observe(button, { attributes: true, attributeFilter: ['data-loading'] });
```

### 8. IntersectionObserver 懒加载

```javascript
// IntersectionObserver：监听元素是否进入视口
// 常用于：懒加载、无限滚动、曝光统计

const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            // 元素进入视口
            const img = entry.target;
            img.src = img.dataset.src;     // 加载真实图片
            img.classList.remove('lazy');   // 移除占位样式
            observer.unobserve(img);       // 停止观察（只加载一次）
        }
    });
}, {
    root: null,            // 视口作为根元素（默认）
    rootMargin: '100px',   // 提前 100px 触发（预加载）
    threshold: 0.1,        // 10% 可见时触发（可以是数组 [0, 0.25, 0.5, 0.75, 1]）
});

// 观察所有懒加载图片
document.querySelectorAll('img.lazy').forEach(img => {
    observer.observe(img);
});

// === 无限滚动 ===
const scrollObserver = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting) {
        loadMorePosts();
    }
}, {
    rootMargin: '200px', // 提前 200px 加载
});
scrollObserver.observe(document.querySelector('.sentinel'));

// === 元素曝光统计 ===
const exposureObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            const adId = entry.target.dataset.adId;
            trackExposure(adId);
            exposureObserver.unobserve(entry.target);
        }
    });
}, { threshold: 0.5 }); // 50% 可见才算曝光
```

### 9. 表单事件处理与验证

```javascript
// === 表单事件 ===
const form = document.querySelector('#register-form');

// input 事件：实时响应（每次值变化都触发）
form.querySelector('#username').addEventListener('input', (e) => {
    validateUsername(e.target.value); // 实时校验
});

// change 事件：值确认后触发（失焦或选择完成）
form.querySelector('#country').addEventListener('change', (e) => {
    updateProvinces(e.target.value);
});

// submit 事件：表单提交
form.addEventListener('submit', (e) => {
    e.preventDefault(); // 阻止默认提交

    // 验证
    if (!form.checkValidity()) {
        form.reportValidity(); // 显示浏览器默认验证提示
        return;
    }

    // 自定义验证
    const formData = new FormData(form);
    const data = Object.fromEntries(formData);
    submitRegistration(data);
});

// === HTML5 约束验证 API ===
// <input required minlength="3" pattern="[a-zA-Z]+" type="email">

const emailInput = form.querySelector('#email');

emailInput.addEventListener('input', () => {
    // checkValidity() 返回布尔值
    if (!emailInput.checkValidity()) {
        // validity 对象包含具体失败原因
        const { valueMissing, typeMismatch, tooShort, patternMismatch } = emailInput.validity;
        if (valueMissing) showEmailError('请输入邮箱');
        else if (typeMismatch) showEmailError('邮箱格式不正确');
    } else {
        clearEmailError();
    }
});

// 自定义验证消息
emailInput.setCustomValidity('该邮箱已被注册');
emailInput.reportValidity();
// 清除自定义消息
emailInput.setCustomValidity('');

// === 实时搜索（防抖 + input 事件） ===
const searchInput = document.querySelector('#search');
searchInput.addEventListener('input', debounce(async (e) => {
    const query = e.target.value.trim();
    if (query.length < 2) return;

    const results = await fetchSuggestions(query);
    renderSuggestions(results);
}, 300));
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 事件监听泄漏 | 组件销毁时未移除监听 | 保存引用，`removeEventListener` 或用 `once: true` / `AbortController` |
| `e.target` 不是期望元素 | 事件冒泡，子元素触发 | 用 `e.target.closest()` 找到目标 |
| `stopPropagation` 阻断委托 | 冒泡被中断 | 只在必要时 stop，优先用条件判断 |
| scroll/touch 卡顿 | 监听器阻塞了浏览器滚动 | 加 `{ passive: true }` |
| `removeEventListener` 无效 | handler 不是同一引用 | 保存引用，避免匿名函数 |
| `innerHTML` 导致事件丢失 | 替换 HTML 会销毁旧 DOM | 用 `insertAdjacentHTML` 或事件委托 |
| `change` 不触发 | `change` 在失焦时才触发 | 用 `input` 事件实时响应 |
| 移动端 300ms 点击延迟 | 双击缩放等待 | `<meta name="viewport" content="width=device-width">` 或 `touch-action: manipulation` |
| 事件冒泡被第三方库阻断 | 第三方组件调用了 `stopPropagation` | 使用捕获阶段监听（`capture: true`） |

### 最佳实践

- 优先用事件委托，减少监听器数量
- 组件销毁时移除事件监听（`AbortController` 或保存引用）
- scroll/touch 事件加 `passive: true`
- 用 `closest()` 替代逐层判断 `e.target`
- 搜索输入用防抖、滚动/拖拽用节流
- 表单验证优先用 HTML5 约束验证 API
- 避免使用 `inline event handler`
- 批量 DOM 操作使用 `DocumentFragment` 减少重排
- 用 `textContent` 替代 `innerHTML` 防 XSS

## 面试题

**Q1: 事件传播的三个阶段是什么？**
> DOM 事件传播分为三个阶段：1) **捕获阶段**（Capture Phase）：事件从 `window` 沿 DOM 树向下传播到目标元素的父元素；2) **目标阶段**（Target Phase）：事件到达目标元素本身，此时按监听器注册顺序执行，不区分捕获/冒泡；3) **冒泡阶段**（Bubbling Phase）：事件从目标元素沿 DOM 树向上冒泡回 `window`。`addEventListener` 第三个参数为 `true` 时在捕获阶段监听，为 `false`（默认）时在冒泡阶段监听。部分事件（如 `focus`/`blur`/`load`）不冒泡。

**Q2: target 和 currentTarget 有什么区别？**
> `e.target` 是事件的实际触发元素（最内层被点击的元素），在整个事件传播过程中始终指向同一个元素，不会改变。`e.currentTarget` 是当前正在执行事件监听器的元素（即 `addEventListener` 绑定的那个元素），在冒泡/捕获过程中会随着事件传播而变化。在事件委托中，`e.target` 是实际被点击的子元素，`e.currentTarget` 是绑定监听器的父元素。事件处理完成后，`e.currentTarget` 会被回收到事件池中（某些浏览器）。

**Q3: 什么是事件委托？有什么优势？**
> 事件委托是利用事件冒泡，在父元素上统一监听子元素的事件，通过 `e.target.closest()` 找到实际触发的子元素。优势：1) 减少事件监听器数量，节省内存；2) 动态添加的子元素自动生效，无需重新绑定；3) 代码更简洁，集中管理逻辑；4) 初始化速度快（只绑定一次）；5) 移除清理方便。适用场景：列表项点击、动态内容交互、批量操作等。

**Q4: `e.preventDefault()` 和 `e.stopPropagation()` 有什么区别？**
> `e.preventDefault()` 阻止元素的默认行为（如链接跳转、表单提交、复选框勾选），不阻止事件继续传播，其他监听器仍会执行。`e.stopPropagation()` 阻止事件继续冒泡或捕获，但不阻止默认行为，也不会阻止同一元素上其他监听器的执行。`e.stopImmediatePropagation()` 最强：阻止传播 + 阻止同元素其他监听器执行。三者互不影响，可单独或同时使用。

**Q5: 什么是 passive 事件监听器？为什么 scroll/touch 事件要用它？**
> `passive: true` 声明事件监听器内部不会调用 `e.preventDefault()`，浏览器因此可以立即开始滚动/缩放操作，无需等待 JS 执行完毕。对于 scroll 和 touch 事件，浏览器需要在主线程执行完监听器后才能确定是否要阻止默认滚动行为，这会导致滚动卡顿。加上 `passive: true` 后，浏览器可以在另一个线程立即开始滚动，显著提升滚动性能。Chrome 中 scroll/touch 事件已默认为 passive。注意：如果确实需要 `preventDefault`，则不能加 passive。

**Q6: MutationObserver 和 IntersectionObserver 各自适用什么场景？**
> **MutationObserver** 监听 DOM 节点的变化（子节点增删、属性变化、文本内容变化），适用于：监听第三方库的 DOM 修改、SPA 页面内容变化检测、属性驱动的响应式更新、自动化测试中监听 DOM 状态。**IntersectionObserver** 监听元素与视口（或指定根元素）的交叉状态，适用于：图片懒加载（进入视口再加载）、无限滚动（触底加载更多）、元素曝光统计（广告/内容可见性）、滚动动画触发。两者都是声明式 API，比传统的事件轮询或 scroll 监听性能更好。

**Q7: 如何防止事件监听器内存泄漏？**
> 常见方式：1) **保存引用**：将 handler 保存到变量，销毁时 `removeEventListener`；2) **`once: true`**：触发一次后自动移除；3) **`AbortController`**：传入 `signal`，调用 `controller.abort()` 一次性移除多个监听器（推荐）；4) **WeakMap**：用 WeakMap 存储 handler 引用，不阻止 GC；5) 事件委托减少监听器数量本身也降低泄漏风险。特别注意：匿名函数无法被 `removeEventListener` 移除，闭包引用的变量不会被 GC。

**Q8: 防抖（Debounce）和节流（Throttle）有什么区别？各适用什么场景？**
> 防抖：事件停止触发后延迟执行，如果延迟期间再次触发则重新计时，高频触发时只执行最后一次。适用于搜索输入联想、窗口 resize 重计算、表单实时保存。节流：固定时间间隔执行一次，高频触发时按固定频率执行。适用于 scroll 事件处理、mousemove 拖拽绘制、按钮防连点、resize 布局更新。记忆口诀：防抖等停再执行，节流按间隔执行。

---

**相关链接：**
- [[Promise与异步]]
- [[浏览器渲染原理]]
- [[ES6+核心特性]]
- [[性能优化实践]]
