---
tags:
  - Web前端
  - Web APIs
  - 浏览器API
  - WAAPI
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# 浏览器高级API

## What — 是什么

浏览器高级API是指现代浏览器内置的、超越基础DOM操作的原生接口集合。它们直接暴露浏览器底层能力，让Web应用无需依赖第三方库即可实现动画、观察、通信、性能监控等高级功能。

### API 分类总览

| 类别 | API | 核心能力 |
|------|-----|----------|
| **Observer 系列** | Intersection Observer | 元素可见性检测 |
| | Resize Observer | 元素尺寸变化监测 |
| | Mutation Observer | DOM 变动监测 |
| **动画与媒体** | Web Animations API (WAAPI) | 原生动画控制 |
| | Web Speech API | 语音识别与合成 |
| | Fullscreen API | 全屏显示控制 |
| **通信与协作** | SharedWorker | 跨标签页共享Worker |
| | BroadcastChannel | 跨标签页消息广播 |
| | Web Lock API | 跨标签页资源锁 |
| **系统交互** | Notification API | 系统通知推送 |
| | Geolocation API | 地理位置获取 |
| | Clipboard API | 剪贴板读写 |
| **性能监控** | Performance API | 性能指标采集 |

---

### 1. Web Animations API (WAAPI)

WAAPI 是浏览器原生动画引擎的 JavaScript 接口，是 CSS Animations 和 CSS Transitions 底层实现的同一套模型。它让你用 JS 精确控制动画的播放、暂停、反向、变速等。

**核心概念：**
- `element.animate(keyframes, options)` — 快捷创建动画
- `Animation` 对象 — 播放控制器
- `AnimationTimeline` — 时间轴（默认为 `document.timeline`）
- `KeyframeEffect` — 关键帧效果

```js
// 基础用法：element.animate()
const animation = element.animate(
  [
    { transform: 'translateX(0px)', opacity: 1 },
    { transform: 'translateX(300px)', opacity: 0.5 }
  ],
  {
    duration: 1000,
    easing: 'ease-in-out',
    fill: 'forwards',
    iterations: Infinity,
    direction: 'alternate'
  }
);

// Animation 对象控制
animation.pause();           // 暂停
animation.play();            // 播放
animation.reverse();         // 反向播放
animation.finish();          // 跳到终点
animation.cancel();          // 取消动画
animation.playbackRate = 2;  // 2倍速
animation.currentTime = 500; // 跳到500ms处

// 监听事件
animation.onfinish = () => console.log('动画结束');
animation.oncancel = () => console.log('动画被取消');

// 使用 Animation 构造函数（更灵活）
const keyframes = new KeyframeEffect(
  element,
  [
    { transform: 'scale(1)' },
    { transform: 'scale(1.5)' },
    { transform: 'scale(1)' }
  ],
  { duration: 600, easing: 'linear' }
);
const anim = new Animation(keyframes, document.timeline);
anim.play();

// 动画承诺（Promise）
element.animate(/* ... */, { duration: 1000 }).finished.then(() => {
  console.log('动画完成后执行');
});

// 批量获取动画
element.getAnimations().forEach(anim => anim.pause());
```

---

### 2. Intersection Observer

异步检测目标元素与祖先元素或视口的交叉状态，比 scroll 事件监听高效得多（浏览器内部优化，不阻塞主线程）。

```js
// 基础：图片懒加载
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src; // 替换 data-src -> src
        img.classList.remove('lazy');
        observer.unobserve(img);   // 加载后停止观察
      }
    });
  },
  {
    root: null,           // 视口为根（默认）
    rootMargin: '100px',  // 提前100px触发
    threshold: 0.1        // 10%可见时触发
  }
);

document.querySelectorAll('img.lazy').forEach(img => observer.observe(img));

// 进阶：无限滚动
const scrollObserver = new IntersectionObserver(
  (entries) => {
    if (entries[0].isIntersecting) {
      loadMoreItems();
    }
  },
  { rootMargin: '200px' } // 提前200px预加载
);
scrollObserver.observe(document.querySelector('#sentinel'));

// 进阶：曝光统计（精确阈值控制）
const exposureObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.intersectionRatio >= 0.5) {
        trackExposure(entry.target.id, entry.time);
      }
    });
  },
  { threshold: [0, 0.25, 0.5, 0.75, 1] } // 多阈值
);

// 清理
observer.disconnect(); // 停止所有观察
```

---

### 3. Resize Observer

监听元素尺寸变化（不仅是 window，任意 DOM 元素），比 `window.resize` 事件精确得多，且不会在每帧都触发（有去抖机制）。

```js
// 基础：响应式组件
const resizeObserver = new ResizeObserver((entries) => {
  for (const entry of entries) {
    const { width, height } = entry.contentRect;
    const target = entry.target;

    if (width < 600) {
      target.classList.add('compact');
    } else {
      target.classList.remove('compact');
    }

    // 也可使用 borderBoxSize（更精确）
    if (entry.borderBoxSize?.length) {
      const boxSize = entry.borderBoxSize[0];
      console.log(`border-box: ${boxSize.inlineSize}x${boxSize.blockSize}`);
    }
  }
});

resizeObserver.observe(document.querySelector('.responsive-container'));

// 进阶：自定义图表自适应
const chartObserver = new ResizeObserver((entries) => {
  for (const entry of entries) {
    const { width, height } = entry.contentRect;
    resizeChart(entry.target, width, height);
  }
});
document.querySelectorAll('.chart-container').forEach(el => chartObserver.observe(el));

// 取消观察
resizeObserver.unobserve(element);
resizeObserver.disconnect();
```

---

### 4. Mutation Observer

监听 DOM 节点的属性变化、子节点增删、文本内容修改。替代了已废弃的 Mutation Events（同步触发，性能差）。

```js
// 基础：监听子节点变化
const mutationObserver = new MutationObserver((mutationsList) => {
  for (const mutation of mutationsList) {
    switch (mutation.type) {
      case 'childList':
        console.log('新增节点:', mutation.addedNodes);
        console.log('删除节点:', mutation.removedNodes);
        break;
      case 'attributes':
        console.log(`属性 ${mutation.attributeName} 变为 ${mutation.target.getAttribute(mutation.attributeName)}`);
        break;
      case 'characterData':
        console.log('文本变化:', mutation.target.textContent);
        break;
    }
  }
});

mutationObserver.observe(targetNode, {
  childList: true,        // 监听子节点增删
  attributes: true,       // 监听属性变化
  characterData: true,    // 监听文本变化
  subtree: true,          // 监听所有后代
  attributeFilter: ['class', 'data-status'], // 只监听指定属性
  attributeOldValue: true, // 记录旧值
  characterDataOldValue: true
});

// 获取变更记录（断开前）
const records = mutationObserver.takeRecords();
mutationObserver.disconnect();
```

---

### 5. Notification API

在操作系统层面显示通知，需要用户授权。常与 Service Worker / Push API 配合实现推送通知。

```js
// 请求权限
async function requestNotificationPermission() {
  if (!('Notification' in window)) {
    console.log('浏览器不支持通知');
    return;
  }
  const permission = await Notification.requestPermission();
  // 'granted' | 'denied' | 'default'
  return permission;
}

// 发送通知
function showNotification(title, options = {}) {
  if (Notification.permission !== 'granted') return;

  const notification = new Notification(title, {
    body: '这是通知内容',
    icon: '/icon.png',
    badge: '/badge.png',
    image: '/preview.jpg',       // 大图
    tag: 'message-1',            // 相同tag会替换旧通知
    renotify: true,              // tag相同时仍提示
    requireInteraction: true,    // 不自动关闭，需用户操作
    silent: false,
    vibrate: [200, 100, 200],    // 振动模式
    data: { url: '/messages/1' } // 自定义数据
  });

  notification.onclick = (event) => {
    window.focus();
    window.open(notification.data.url);
    notification.close();
  };

  notification.onclose = () => console.log('通知关闭');
  notification.onerror = () => console.log('通知出错');
}

// 配合 Service Worker（推送通知）
navigator.serviceWorker.ready.then(registration => {
  registration.showNotification('推送消息', {
    body: '来自服务端的推送',
    actions: [
      { action: 'reply', title: '回复' },
      { action: 'ignore', title: '忽略' }
    ]
  });
});
```

---

### 6. Geolocation API

获取用户地理位置，支持单次获取和持续追踪。**必须在 HTTPS 或 localhost 下使用**。

```js
// 单次获取位置
navigator.geolocation.getCurrentPosition(
  (position) => {
    const { latitude, longitude, accuracy, altitude, speed } = position.coords;
    const timestamp = position.timestamp;
    console.log(`纬度: ${latitude}, 经度: ${longitude}`);
    console.log(`精度: ${accuracy}m`);
  },
  (error) => {
    switch (error.code) {
      case error.PERMISSION_DENIED:    console.log('用户拒绝'); break;
      case error.POSITION_UNAVAILABLE: console.log('位置不可用'); break;
      case error.TIMEOUT:              console.log('超时'); break;
    }
  },
  {
    enableHighAccuracy: true,  // 高精度模式（更耗电）
    timeout: 10000,            // 超时时间
    maximumAge: 0              // 不使用缓存
  }
);

// 持续追踪位置
const watchId = navigator.geolocation.watchPosition(
  (position) => {
    updateMapMarker(position.coords.latitude, position.coords.longitude);
  },
  handleError,
  { enableHighAccuracy: true, maximumAge: 5000 }
);

// 停止追踪
navigator.geolocation.clearWatch(watchId);
```

---

### 7. Clipboard API

异步读写剪贴板，替代 `document.execCommand('copy')`。支持文本、图片、自定义 MIME 类型。

```js
// 写入文本
async function copyText(text) {
  try {
    await navigator.clipboard.writeText(text);
    console.log('已复制');
  } catch (err) {
    // 降级方案
    const textarea = document.createElement('textarea');
    textarea.value = text;
    textarea.style.position = 'fixed';
    textarea.style.opacity = '0';
    document.body.appendChild(textarea);
    textarea.select();
    document.execCommand('copy');
    document.body.removeChild(textarea);
  }
}

// 读取文本
async function pasteText() {
  try {
    const text = await navigator.clipboard.readText();
    return text;
  } catch (err) {
    console.log('读取剪贴板被拒绝:', err);
  }
}

// 写入富内容（图片等）
async function copyImage(blob) {
  try {
    await navigator.clipboard.write([
      new ClipboardItem({ 'image/png': blob })
    ]);
  } catch (err) {
    console.log('复制图片失败:', err);
  }
}

// 读取剪贴板内容
async function readClipboard() {
  try {
    const items = await navigator.clipboard.read();
    for (const item of items) {
      if (item.types.includes('image/png')) {
        const blob = await item.getType('image/png');
        return URL.createObjectURL(blob);
      }
      if (item.types.includes('text/plain')) {
        const text = await item.getType('text/plain');
        return await text.text();
      }
    }
  } catch (err) {
    console.log('读取失败:', err);
  }
}

// 监听粘贴事件（配合 ClipboardEvent）
document.addEventListener('paste', (e) => {
  const items = e.clipboardData.items;
  for (const item of items) {
    if (item.type.startsWith('image/')) {
      const file = item.getAsFile();
      handlePastedImage(file);
    }
  }
});
```

---

### 8. Web Speech API

包含两部分：**SpeechRecognition**（语音转文字）和 **SpeechSynthesis**（文字转语音）。

```js
// === 语音识别 ===
const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

if (!SpeechRecognition) {
  console.log('浏览器不支持语音识别');
} else {
  const recognition = new SpeechRecognition();
  recognition.lang = 'zh-CN';            // 语言
  recognition.continuous = true;          // 持续识别
  recognition.interimResults = true;      // 返回临时结果
  recognition.maxAlternatives = 1;

  recognition.onresult = (event) => {
    let interimTranscript = '';
    let finalTranscript = '';

    for (let i = event.resultIndex; i < event.results.length; i++) {
      const transcript = event.results[i][0].transcript;
      if (event.results[i].isFinal) {
        finalTranscript += transcript;
      } else {
        interimTranscript += transcript;
      }
    }
    console.log('临时:', interimTranscript);
    console.log('最终:', finalTranscript);
  };

  recognition.onerror = (event) => {
    console.log('识别错误:', event.error);
  };

  recognition.onend = () => {
    console.log('识别结束');
    // continuous 模式下需手动重启
    // recognition.start();
  };

  recognition.start();
  // recognition.stop();
  // recognition.abort();
}

// === 语音合成 ===
function speak(text, lang = 'zh-CN') {
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.lang = lang;
  utterance.rate = 1.0;    // 语速 0.1-10
  utterance.pitch = 1.0;   // 音调 0-2
  utterance.volume = 1.0;  // 音量 0-1

  // 选择语音
  const voices = speechSynthesis.getVoices();
  const targetVoice = voices.find(v => v.lang === lang);
  if (targetVoice) utterance.voice = targetVoice;

  utterance.onstart = () => console.log('开始朗读');
  utterance.onend = () => console.log('朗读结束');
  utterance.onerror = (e) => console.log('朗读出错:', e);

  speechSynthesis.speak(utterance);
}

// 语音列表可能异步加载
speechSynthesis.onvoiceschanged = () => {
  const voices = speechSynthesis.getVoices();
  console.log('可用语音:', voices.map(v => `${v.name} (${v.lang})`));
};

// 控制方法
speechSynthesis.pause();
speechSynthesis.resume();
speechSynthesis.cancel();
```

---

### 9. Fullscreen API

将任意元素全屏显示，常用于视频播放器、图片查看器、数据大屏等场景。

```js
// 进入全屏
async function enterFullscreen(element) {
  try {
    await element.requestFullscreen();
    // 也可指定导航栏隐藏选项
    // await element.requestFullscreen({ navigationUI: 'hide' });
  } catch (err) {
    console.log('全屏请求被拒绝:', err);
  }
}

// 退出全屏
async function exitFullscreen() {
  if (document.fullscreenElement) {
    await document.exitFullscreen();
  }
}

// 切换全屏
function toggleFullscreen(element) {
  if (document.fullscreenElement) {
    document.exitFullscreen();
  } else {
    element.requestFullscreen();
  }
}

// 监听全屏变化
document.addEventListener('fullscreenchange', () => {
  if (document.fullscreenElement) {
    console.log('进入全屏:', document.fullscreenElement);
  } else {
    console.log('退出全屏');
  }
});

// 监听全屏错误
document.addEventListener('fullscreenerror', (event) => {
  console.log('全屏错误:', event);
});

// CSS 伪类配合
// :fullscreen { background: black; }
// ::backdrop { background: rgba(0, 0, 0, 0.5); }  // 全屏背景
```

---

### 10. SharedWorker & BroadcastChannel

两种跨标签页通信方案，适用场景不同。

```js
// === BroadcastChannel（简单消息广播）===

// 标签页 A：发送消息
const channel = new BroadcastChannel('app-sync');
channel.postMessage({ type: 'LOGOUT', userId: '123' });
channel.postMessage({ type: 'UPDATE_CART', items: [] });

// 标签页 B：接收消息
const channel = new BroadcastChannel('app-sync');
channel.onmessage = (event) => {
  const { type, ...data } = event.data;
  switch (type) {
    case 'LOGOUT':
      clearUserSession();
      break;
    case 'UPDATE_CART':
      refreshCart(data.items);
      break;
  }
};

// 关闭
channel.close();

// === SharedWorker（共享Worker，复杂场景）===

// shared-worker.js
const connections = [];

self.onconnect = (event) => {
  const port = event.ports[0];
  connections.push(port);

  port.onmessage = (e) => {
    // 广播给所有连接
    connections.forEach(p => {
      if (p !== port) p.postMessage(e.data);
    });
  };

  port.start();
};

// 主线程：连接 SharedWorker
const worker = new SharedWorker('shared-worker.js', 'app-worker');

worker.port.onmessage = (event) => {
  console.log('收到跨标签页消息:', event.data);
};

worker.port.start();
worker.port.postMessage({ from: 'tab-A', msg: 'hello' });
```

**两者对比：**

| 特性 | BroadcastChannel | SharedWorker |
|------|-----------------|--------------|
| 通信模式 | 发布/订阅广播 | 共享计算+消息传递 |
| 复杂度 | 低 | 中 |
| 状态共享 | 无 | Worker 内可共享状态 |
| 兼容性 | 较好 | 较好 |
| 适用场景 | 简单通知、同步 | 共享状态、复杂计算 |

---

### 11. Web Lock API

跨标签页的资源互斥锁，确保同一时刻只有一个标签页能操作某个资源。适合 IndexedDB 写入、定时任务防重复等场景。

```js
// 请求锁
async function writeWithLock(data) {
  try {
    await navigator.locks.request('db-write', async (lock) => {
      // 持有锁期间执行操作
      await writeToIndexedDB(data);
      // 函数返回后锁自动释放
    });
  } catch (err) {
    if (err.name === 'AbortError') {
      console.log('锁请求被中止');
    }
  }
}

// 带选项的锁请求
await navigator.locks.request('my-resource', {
  mode: 'exclusive',  // 'exclusive'(默认) 或 'shared'
  ifAvailable: true,  // 锁不可用时立即返回null，不等待
  signal: abortController.signal // 可中途取消
}, async (lock) => {
  if (!lock) {
    console.log('锁不可用，跳过');
    return;
  }
  await doWork();
});

// 共享锁（多读单写）
await navigator.locks.request('read-resource', { mode: 'shared' }, async () => {
  await readData(); // 多个标签页可同时持有共享锁
});

// 查询锁状态
const state = await navigator.locks.query();
console.log('当前持有的锁:', state.held);
console.log('等待中的锁:', state.pending);

// 监控锁等待
const controller = new AbortController();
setTimeout(() => controller.abort(), 5000); // 5秒超时

await navigator.locks.request('my-lock', { signal: controller.signal }, async () => {
  await longRunningTask();
});
```

---

### 12. Performance API

浏览器性能监控的统一接口，包含多个子API：PerformanceObserver、Navigation Timing、Resource Timing、Long Tasks、Paint Timing 等。

```js
// === Navigation Timing（页面加载性能）===
const [nav] = performance.getEntriesByType('navigation');
if (nav) {
  console.log('DNS查询:', nav.domainLookupEnd - nav.domainLookupStart, 'ms');
  console.log('TCP连接:', nav.connectEnd - nav.connectStart, 'ms');
  console.log('请求耗时:', nav.responseEnd - nav.requestStart, 'ms');
  console.log('DOM解析:', nav.domContentLoadedEventEnd - nav.domInteractive, 'ms');
  console.log('首字节时间(TTFB):', nav.responseStart - nav.requestStart, 'ms');
  console.log('页面完全加载:', nav.loadEventEnd - nav.startTime, 'ms');
}

// === Resource Timing（资源加载性能）===
const resources = performance.getEntriesByType('resource');
resources.forEach(r => {
  console.log(`${r.name}: ${r.duration.toFixed(0)}ms (${r.initiatorType})`);
});

// === PerformanceObserver（监听性能条目）===
// 监听长任务（阻塞主线程>50ms）
const longTaskObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.warn(`长任务: ${entry.duration.toFixed(0)}ms`, entry);
    // 可上报到监控系统
    reportLongTask({
      duration: entry.duration,
      startTime: entry.startTime,
      name: entry.name
    });
  }
});
longTaskObserver.observe({ type: 'longtask', buffered: true });

// 监听首次绘制
const paintObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.name === 'first-paint') {
      console.log(`FP: ${entry.startTime.toFixed(0)}ms`);
    }
    if (entry.name === 'first-contentful-paint') {
      console.log(`FCP: ${entry.startTime.toFixed(0)}ms`);
    }
  }
});
paintObserver.observe({ type: 'paint', buffered: true });

// 监听布局偏移（CLS）
let clsValue = 0;
const layoutShiftObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) { // 排除用户输入导致的偏移
      clsValue += entry.value;
    }
  }
  console.log('当前CLS:', clsValue);
});
layoutShiftObserver.observe({ type: 'layout-shift', buffered: true });

// 监听最大内容绘制（LCP）
const lcpObserver = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1];
  console.log(`LCP: ${lastEntry.startTime.toFixed(0)}ms`, lastEntry.element);
});
lcpObserver.observe({ type: 'largest-contentful-paint', buffered: true });

// === 自定义性能标记 ===
performance.mark('api-start');
await fetchData();
performance.mark('api-end');
performance.measure('api-duration', 'api-start', 'api-end');

const [measure] = performance.getEntriesByName('api-duration');
console.log(`API耗时: ${measure.duration.toFixed(0)}ms`);

// 清理
performance.clearMarks();
performance.clearMeasures();
```

---

## Why — 为什么

### 为什么优先使用浏览器原生API

| 维度 | 原生API | 第三方库 |
|------|---------|----------|
| **包体积** | 0 KB | 5KB - 50KB+ |
| **性能** | 浏览器引擎级优化 | JS 层面模拟 |
| **兼容性** | 随浏览器更新 | 需手动升级依赖 |
| **安全性** | 浏览器权限管控 | 无系统级安全机制 |
| **维护成本** | 无需维护 | 依赖社区维护 |
| **一致性** | 与浏览器行为一致 | 可能存在行为差异 |

### 各API替代的典型库

| 原生API | 替代的库/方案 | 原生优势 |
|---------|-------------|---------|
| WAAPI | GSAP, anime.js, Framer Motion | 零依赖、与CSS动画同引擎、硬件加速 |
| Intersection Observer | scroll 事件 + getBoundingClientRect | 不阻塞主线程、无布局抖动 |
| Resize Observer | window.resize + getComputedStyle | 可监听任意元素、自动去抖 |
| Mutation Observer | MutationEvents / 轮询 | 异步微任务、批量回调 |
| Clipboard API | execCommand('copy') | 异步、支持富内容、不依赖DOM选区 |
| Geolocation API | 第三方定位SDK | 免费、原生GPS支持 |
| Web Speech API | 付费云服务 | 免费、离线可用（部分浏览器） |
| BroadcastChannel | localStorage 事件 / postMessage | 专用通道、类型安全 |

### 什么时候不用原生API

1. **兼容性不足时**：如 Web Speech API 在 Firefox/Safari 上支持有限，生产环境可用云服务
2. **功能差距大时**：如 WAAPI 缺乏 SVG path 动画、复杂时间线编排，GSAP 仍更强大
3. **开发效率优先时**：如简单动画场景，CSS Animation 可能比 WAAPI 代码更简洁
4. **需要 Polyfill 时**：某些旧浏览器需 polyfill，引入 polyfill 的体积可能与库相当

---

## How — 怎么用

### 综合实战：带动画的图片懒加载

```js
// 结合 Intersection Observer + WAAPI 实现优雅的懒加载
class LazyImageLoader {
  constructor(selector = 'img[data-src]') {
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersection(entries),
      { rootMargin: '50px', threshold: 0.01 }
    );
    document.querySelectorAll(selector).forEach(img => this.observer.observe(img));
  }

  handleIntersection(entries) {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      const img = entry.target;
      this.observer.unobserve(img);
      this.loadImage(img);
    });
  }

  async loadImage(img) {
    // 淡入动画
    const animation = img.animate(
      [{ opacity: 0 }, { opacity: 1 }],
      { duration: 400, easing: 'ease-in' }
    );

    img.src = img.dataset.src;
    img.removeAttribute('data-src');

    try {
      await animation.finished;
    } catch {
      // 动画取消，忽略
    }
  }

  destroy() {
    this.observer.disconnect();
  }
}

new LazyImageLoader();
```

### 综合实战：跨标签页状态同步

```js
// 结合 BroadcastChannel + Web Lock API
class CrossTabSync {
  constructor(channelName) {
    this.channel = new BroadcastChannel(channelName);
    this.channel.onmessage = (e) => this.handleMessage(e.data);
  }

  // 安全写入（防并发）
  async safeWrite(key, updater) {
    await navigator.locks.request(`lock:${key}`, async () => {
      const current = await readFromDB(key);
      const updated = updater(current);
      await writeToDB(key, updated);
      this.channel.postMessage({ type: 'UPDATED', key, value: updated });
    });
  }

  handleMessage(data) {
    switch (data.type) {
      case 'UPDATED':
        refreshUI(data.key, data.value);
        break;
    }
  }

  destroy() {
    this.channel.close();
  }
}
```

### 综合实战：性能监控仪表盘

```js
class PerformanceMonitor {
  constructor() {
    this.metrics = {};
    this.observers = [];
  }

  start() {
    this.observePaint();
    this.observeLongTasks();
    this.observeLayoutShift();
    this.observeLCP();
    this.observeNavigation();
    return this.metrics;
  }

  observePaint() {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        this.metrics[entry.name] = entry.startTime;
      }
    });
    observer.observe({ type: 'paint', buffered: true });
    this.observers.push(observer);
  }

  observeLongTasks() {
    this.metrics.longTasks = [];
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        this.metrics.longTasks.push({
          duration: entry.duration,
          startTime: entry.startTime
        });
      }
    });
    observer.observe({ type: 'longtask', buffered: true });
    this.observers.push(observer);
  }

  observeLayoutShift() {
    this.metrics.cls = 0;
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (!entry.hadRecentInput) this.metrics.cls += entry.value;
      }
    });
    observer.observe({ type: 'layout-shift', buffered: true });
    this.observers.push(observer);
  }

  observeLCP() {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      this.metrics.lcp = entries[entries.length - 1].startTime;
    });
    observer.observe({ type: 'largest-contentful-paint', buffered: true });
    this.observers.push(observer);
  }

  observeNavigation() {
    const [nav] = performance.getEntriesByType('navigation');
    if (nav) {
      this.metrics.ttfb = nav.responseStart - nav.requestStart;
      this.metrics.domContentLoaded = nav.domContentLoadedEventEnd - nav.startTime;
      this.metrics.fullLoad = nav.loadEventEnd - nav.startTime;
    }
  }

  report() {
    console.table(this.metrics);
    return this.metrics;
  }

  stop() {
    this.observers.forEach(o => o.disconnect());
    this.observers = [];
  }
}

const monitor = new PerformanceMonitor();
monitor.start();
```

### 常见陷阱

| API | 陷阱 | 正确做法 |
|-----|------|---------|
| Intersection Observer | `rootMargin` 写百分比无效 | 使用像素值如 `'100px'` |
| Intersection Observer | 回调中直接修改 DOM 导致循环触发 | 先 `unobserve` 再修改 |
| Resize Observer | 回调中修改被观察元素尺寸 | 修改前先 `unobserve`，修改后重新 `observe` |
| Mutation Observer | 忘记 `disconnect()` 导致内存泄漏 | 组件卸载时必须调用 |
| Notification API | 在用户交互外调用 `requestPermission()` | 浏览器可能阻止，应在点击事件中调用 |
| Geolocation API | HTTP 环境下调用 | 必须在 HTTPS 或 localhost 下使用 |
| Clipboard API | 没有用户手势就调用 `write` | 需在点击/按键等用户手势内调用 |
| Web Speech API | `getVoices()` 在页面加载时返回空 | 监听 `voiceschanged` 事件后再获取 |
| WAAPI | `fill: 'forwards'` 后元素仍占据原始样式 | 动画结束后手动设置最终样式或使用 `commitStyles()` |
| Web Lock API | 锁回调中抛异常 | 锁仍会释放，但异常会传播到 `request()` 的调用方 |
| BroadcastChannel | 同一标签页能收到自己的消息 | 不能，只跨标签页/iframe |
| Fullscreen API | `requestFullscreen()` 不在用户手势中调用 | 需在点击等用户交互事件中调用 |

### 最佳实践

1. **Observer 统一管理**：在框架组件中，于 `mounted`/`useEffect` 创建 Observer，`unmounted`/`cleanup` 时 `disconnect`
2. **API 可用性检测**：始终先检查 `if ('IntersectionObserver' in window)` 再使用
3. **降级策略**：原生API不可用时回退到兼容方案（如 scroll 事件懒加载）
4. **性能监控前置**：PerformanceObserver 的 `buffered: true` 可获取页面加载前的条目
5. **权限请求时机**：Notification / Geolocation 等需权限的API，在用户明确意图时再请求
6. **Lock 粒度控制**：Web Lock 的锁名应具体（`'cart:userId:write'`），避免过粗锁粒度

---

## 面试题

### 1. Intersection Observer 相比 scroll 事件监听有什么优势？

**答案：** Intersection Observer 有三大核心优势：(1) **性能**：由浏览器内部优化，不在主线程上频繁计算，而 scroll 事件每次触发都需要调用 `getBoundingClientRect()` 导致强制同步布局（布局抖动）；(2) **语义清晰**：直接告知元素是否可见及可见比例，无需手动计算；(3) **自动去抖**：浏览器合并多次变化为一次回调，而 scroll 事件可能在一帧内触发多次。典型应用是图片懒加载和无限滚动。

---

### 2. WAAPI 与 CSS Animation 有什么区别？各自适用场景？

**答案：** CSS Animation 声明式写法简洁，适合预定义的、固定的动画。WAAPI 提供命令式控制能力：(1) 可动态暂停/播放/反向/变速（`playbackRate`）；(2) 支持 Promise（`animation.finished`）；(3) 可在运行时修改关键帧；(4) 可获取所有动画并统一管理（`getAnimations()`）。复杂交互动画（如拖拽释放回弹、滚动驱动动画）用 WAAPI 更合适；简单过渡动画用 CSS 更简洁。两者共享同一底层引擎。

---

### 3. Mutation Observer 与废弃的 Mutation Events 有何区别？

**答案：** Mutation Events（如 `DOMNodeInserted`）是同步触发的，每次 DOM 变化都立即触发回调，在大量 DOM 操作时严重阻塞主线程，且可能导致无限递归。Mutation Observer 改为异步微任务模式，将多次 DOM 变化收集后批量回调，大幅减少回调次数。同时 Mutation Observer 支持精确配置（`attributeFilter`、`subtree` 等），而 Mutation Events 无法过滤。Mutation Observer 还可通过 `takeRecords()` 在 disconnect 前获取未处理的记录。

---

### 4. 如何实现跨标签页通信？比较各方案。

**答案：** 主要方案：(1) **BroadcastChannel**：最简洁，专用通道、类型安全、同源即可，但不能同标签页自收；(2) **SharedWorker**：可共享状态，适合多标签页协作场景，但实现较复杂；(3) **localStorage 事件**：兼容性最好，同一 `localStorage.setItem()` 在其他标签页触发 `storage` 事件，但只能传字符串且容量有限；(4) **postMessage**：需获取目标窗口引用，适合 iframe 通信。简单通知选 BroadcastChannel，共享计算选 SharedWorker，兼容旧浏览器选 localStorage 事件。

---

### 5. Web Lock API 解决什么问题？`exclusive` 和 `shared` 模式有什么区别？

**答案：** Web Lock API 解决跨标签页的资源互斥问题。典型场景：多个标签页同时写 IndexedDB 导致数据冲突、定时任务防止多标签页重复执行。`exclusive` 模式下同一锁名同时只允许一个请求持有（读写锁的"写锁"）；`shared` 模式允许多个请求同时持有（"读锁"），但与 `exclusive` 互斥。这实现了经典的"多读单写"模式。锁在回调函数返回后自动释放，即使异常也不会死锁。

---

### 6. Performance API 如何监控 Core Web Vitals 指标？

**答案：** Core Web Vitals 三个指标均可通过 PerformanceObserver 监控：(1) **LCP**（最大内容绘制）：`type: 'largest-contentful-paint'`，取最后一条记录的 `startTime`；(2) **FID/INP**（首次输入延迟/交互延迟）：`type: 'event'`，计算 `processingStart - startTime`；(3) **CLS**（累积布局偏移）：`type: 'layout-shift'`，累加所有 `hadRecentInput === false` 的 `value`。加上 `buffered: true` 可获取页面加载前就发生的条目。Nav Timing 提供 TTFB 指标。

---

### 7. Clipboard API 与 `execCommand('copy')` 有什么区别？

**答案：** (1) **异步 vs 同步**：Clipboard API 返回 Promise，`execCommand` 同步阻塞；(2) **功能范围**：Clipboard API 支持写入/读取图片等富内容（`ClipboardItem`），`execCommand` 只能复制文本；(3) **安全性**：Clipboard API 有权限系统（`navigator.permissions`），`execCommand` 无明确权限模型；(4) **依赖条件**：`execCommand` 需要先选中 DOM 内容，Clipboard API 不需要；(5) **状态**：`execCommand` 已标记为废弃（deprecated）。生产环境应优先用 Clipboard API，`execCommand` 仅作降级。

---

### 8. 如何在 Vue/React 组件中正确使用 Observer API 避免内存泄漏？

**答案：** 核心原则是在组件卸载时 `disconnect()`。在 Vue 3 中：

```js
// Vue 3 Composition API
onMounted(() => {
  const observer = new IntersectionObserver(callback, options);
  observer.observe(targetRef.value);
  onUnmounted(() => observer.disconnect());
});
```

在 React 中：

```jsx
useEffect(() => {
  const observer = new IntersectionObserver(callback, options);
  observer.observe(ref.current);
  return () => observer.disconnect(); // cleanup
}, []);
```

常见错误：(1) 忘记 cleanup 导致 Observer 持有已卸载组件的 DOM 引用；(2) 在 callback 中修改被观察元素尺寸导致 Resize Observer 循环触发（应先 unobserve 再修改）；(3) 在 SSR 环境中直接使用（应先判断 `typeof window !== 'undefined'`）。

---

## Related links

- [MDN - Web Animations API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Animations_API)
- [MDN - Intersection Observer](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)
- [MDN - Resize Observer](https://developer.mozilla.org/zh-CN/docs/Web/API/ResizeObserver)
- [MDN - Mutation Observer](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)
- [MDN - Notification API](https://developer.mozilla.org/zh-CN/docs/Web/API/Notifications_API)
- [MDN - Geolocation API](https://developer.mozilla.org/zh-CN/docs/Web/API/Geolocation_API)
- [MDN - Clipboard API](https://developer.mozilla.org/zh-CN/docs/Web/API/Clipboard_API)
- [MDN - Web Speech API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Speech_API)
- [MDN - Fullscreen API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fullscreen_API)
- [MDN - BroadcastChannel](https://developer.mozilla.org/zh-CN/docs/Web/API/Broadcast_Channel_API)
- [MDN - Web Lock API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Locks_API)
- [MDN - Performance API](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance_API)
- [web.dev - Core Web Vitals](https://web.dev/vitals/)
- [W3C - Web Animations 规范](https://www.w3.org/TR/web-animations/)
