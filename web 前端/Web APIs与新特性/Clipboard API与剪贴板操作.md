---
tags:
  - Web前端
  - Clipboard
  - 剪贴板
  - 复制粘贴
  - Web APIs
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Clipboard API与剪贴板操作

## What — 是什么

> Clipboard API 是现代浏览器提供的异步剪贴板读写接口，替代了旧的 `document.execCommand('copy')` 同步方案。支持读写文本、HTML、图片等多种格式，且在权限模型下运行，比旧方案更安全更可靠。

**核心概念：**

- **Clipboard API（`navigator.clipboard`）**：异步读写剪贴板，返回 Promise，支持多种 MIME 类型
- **ClipboardItem**：剪贴板条目，一个条目可包含同一数据的多格式表示（如 text/plain + text/html）
- **Clipboard Permission**：读取需要用户授权（`clipboard-read`），写入通常由用户操作触发（如点击）
- **execCommand（旧方案）**：同步复制/粘贴，已废弃，不稳定且不支持图片
- **Copy/Paste 事件**：`copy`/`cut`/`paste` 事件，可拦截并修改剪贴板内容

**新旧方案对比：**

| 维度 | execCommand | Clipboard API |
|------|------------|---------------|
| 异步 | 同步（阻塞） | 异步（Promise） |
| 权限 | 无明确权限 | 需要权限或用户操作 |
| 图片 | 不支持 | 支持（Blob） |
| HTML | 不支持 | 支持（text/html） |
| 读取 | 不支持 | 支持（需权限） |
| 稳定性 | 不稳定，可能失败 | 稳定 |
| 安全 | 无安全模型 | 权限 + 安全上下文 |
| 状态 | 已废弃 | 推荐 |

**浏览器支持：**

| API | Chrome | Firefox | Safari | Edge |
|-----|--------|---------|--------|------|
| `navigator.clipboard.writeText` | 66+ | 63+ | 13.1+ | 79+ |
| `navigator.clipboard.readText` | 66+ | 63+ | 13.1+ | 79+ |
| `navigator.clipboard.write` (多格式) | 76+ | 87+ | 13.1+ | 79+ |
| `navigator.clipboard.read` (多格式) | 76+ | 87+ | 13.1+ | 79+ |
| 图片剪贴板 | 76+ | 87+ | 13.1+ | 79+ |

> 要求安全上下文（HTTPS 或 localhost）。

## Why — 为什么

**适用场景：**

- 一键复制（邀请码、链接、代码片段）
- 富文本复制（保留格式的 HTML 内容）
- 图片复制到剪贴板（截图、图表导出）
- 粘贴拦截和处理（表单粘贴格式化、图片粘贴上传）
- 拖拽 + 剪贴板配合（拖拽复制文件/图片）

**常见业务需求：**

| 需求 | 方案 |
|------|------|
| 复制文本 | `clipboard.writeText()` |
| 复制富文本 | `clipboard.write()` + text/html |
| 复制图片 | `clipboard.write()` + image/png |
| 读取剪贴板文本 | `clipboard.readText()` |
| 拦截粘贴内容 | `paste` 事件 |
| 粘贴图片上传 | `paste` 事件 + DataTransfer |
| 复制 Canvas 图表 | Canvas → Blob → clipboard.write() |

## How — 怎么用

### 复制文本

```ts
// 最常用：复制文本
async function copyText(text: string): Promise<boolean> {
  try {
    await navigator.clipboard.writeText(text);
    return true;
  } catch (error) {
    // 降级到旧方案
    return fallbackCopyText(text);
  }
}

// 旧方案降级
function fallbackCopyText(text: string): boolean {
  const textarea = document.createElement('textarea');
  textarea.value = text;
  textarea.style.position = 'fixed';
  textarea.style.left = '-9999px';
  textarea.style.top = '-9999px';
  textarea.style.opacity = '0';
  document.body.appendChild(textarea);
  textarea.select();

  try {
    const result = document.execCommand('copy');
    document.body.removeChild(textarea);
    return result;
  } catch {
    document.body.removeChild(textarea);
    return false;
  }
}

// React 组件
function CopyButton({ text }: { text: string }) {
  const [copied, setCopied] = useState(false);

  const handleCopy = async () => {
    const success = await copyText(text);
    if (success) {
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    }
  };

  return (
    <button onClick={handleCopy}>
      {copied ? '已复制' : '复制'}
    </button>
  );
}
```

### 读取剪贴板

```ts
// 读取文本
async function readClipboardText(): Promise<string | null> {
  try {
    const text = await navigator.clipboard.readText();
    return text;
  } catch (error) {
    // 权限被拒绝或浏览器不支持
    console.warn('读取剪贴板失败:', error);
    return null;
  }
}

// 检查剪贴板权限
async function checkClipboardPermission(): Promise<boolean> {
  try {
    const result = await navigator.permissions.query({ name: 'clipboard-read' as PermissionName });
    return result.state === 'granted';
  } catch {
    return false;
  }
}

// 使用：粘贴按钮
function PasteButton() {
  const [pastedText, setPastedText] = useState('');

  const handlePaste = async () => {
    const text = await readClipboardText();
    if (text) setPastedText(text);
  };

  return (
    <div>
      <button onClick={handlePaste}>从剪贴板粘贴</button>
      <input value={pastedText} onChange={(e) => setPastedText(e.target.value)} />
    </div>
  );
}
```

### 复制富文本（HTML）

```ts
// 复制 HTML 格式内容（同时提供纯文本降级）
async function copyRichText(html: string, plainText?: string): Promise<boolean> {
  try {
    const blob = new Blob([html], { type: 'text/html' });
    const textBlob = new Blob([plainText || html.replace(/<[^>]*>/g, '')], { type: 'text/plain' });

    await navigator.clipboard.write([
      new ClipboardItem({
        'text/html': blob,
        'text/plain': textBlob,
      }),
    ]);
    return true;
  } catch {
    // 降级：只复制纯文本
    return copyText(plainText || html.replace(/<[^>]*>/g, ''));
  }
}

// 使用：复制格式化文章
function CopyArticleButton({ article }: { article: { title: string; content: string } }) {
  const handleCopy = async () => {
    const html = `<h2>${article.title}</h2><div>${article.content}</div>`;
    const plainText = `${article.title}\n\n${article.content.replace(/<[^>]*>/g, '')}`;
    const success = await copyRichText(html, plainText);

    if (success) {
      // 粘贴到邮件/文档时保留格式，粘贴到记事本时为纯文本
      showToast('已复制，支持富文本粘贴');
    }
  };

  return <button onClick={handleCopy}>复制文章</button>;
}
```

### 复制图片

```ts
// 从 URL 复制图片到剪贴板
async function copyImageFromUrl(url: string): Promise<boolean> {
  try {
    const response = await fetch(url);
    const blob = await response.blob();

    if (!blob.type.startsWith('image/')) {
      throw new Error('Not an image');
    }

    // Clipboard API 要求 PNG 格式（部分浏览器）
    const pngBlob = blob.type === 'image/png' ? blob : await convertToPng(blob);

    await navigator.clipboard.write([
      new ClipboardItem({ 'image/png': pngBlob }),
    ]);
    return true;
  } catch (error) {
    console.error('复制图片失败:', error);
    return false;
  }
}

// 从 Canvas 复制图表
async function copyCanvasToClipboard(canvas: HTMLCanvasElement): Promise<boolean> {
  try {
    const blob = await new Promise<Blob>((resolve, reject) => {
      canvas.toBlob((b) => {
        if (b) resolve(b);
        else reject(new Error('Canvas toBlob failed'));
      }, 'image/png');
    });

    await navigator.clipboard.write([
      new ClipboardItem({ 'image/png': blob }),
    ]);
    return true;
  } catch {
    return false;
  }
}

// React：ECharts 图表复制
function ChartWithCopy({ chartRef }: { chartRef: React.RefObject<ReactECharts> }) {
  const [copied, setCopied] = useState(false);

  const handleCopyChart = async () => {
    const echartsInstance = chartRef.current?.getEchartsInstance();
    if (!echartsInstance) return;

    // ECharts 提供的获取 Canvas 方法
    const canvas = echartsInstance.getDom().querySelector('canvas');
    if (!canvas) return;

    const success = await copyCanvasToClipboard(canvas);
    if (success) {
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    }
  };

  return (
    <div>
      <ReactECharts ref={chartRef} option={chartOption} />
      <button onClick={handleCopyChart}>
        {copied ? '已复制图片' : '复制图表'}
      </button>
    </div>
  );
}

// Blob 格式转换
async function convertToPng(blob: Blob): Promise<Blob> {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.onload = () => {
      const canvas = document.createElement('canvas');
      canvas.width = img.naturalWidth;
      canvas.height = img.naturalHeight;
      const ctx = canvas.getContext('2d')!;
      ctx.drawImage(img, 0, 0);
      canvas.toBlob((pngBlob) => {
        if (pngBlob) resolve(pngBlob);
        else reject(new Error('Conversion failed'));
        URL.revokeObjectURL(img.src);
      }, 'image/png');
    };
    img.onerror = reject;
    img.src = URL.createObjectURL(blob);
  });
}
```

### 拦截复制/粘贴事件

```ts
// 拦截复制事件：自定义复制内容
element.addEventListener('copy', (event) => {
  event.preventDefault();
  const selection = window.getSelection()?.toString() || '';
  const htmlContent = `<blockquote>${selection}</blockquote><p>— 来源: ${location.href}</p>`;
  const plainContent = `${selection}\n\n— 来源: ${location.href}`;

  event.clipboardData?.setData('text/html', htmlContent);
  event.clipboardData?.setData('text/plain', plainContent);
});

// 拦截剪切事件
element.addEventListener('cut', (event) => {
  event.preventDefault();
  const selection = window.getSelection()?.toString() || '';
  navigator.clipboard.writeText(selection);
  // 清除选中内容
  document.execCommand('delete');
});

// 拦截粘贴事件：格式化粘贴内容
const input = document.querySelector('input');

input?.addEventListener('paste', (event) => {
  // 场景1：只允许纯文本（去除格式）
  event.preventDefault();
  const text = event.clipboardData?.getData('text/plain') || '';
  document.execCommand('insertText', false, text);

  // 场景2：手机号自动格式化
  // 粘贴 "13812345678" → "138 1234 5678"
  // const raw = text.replace(/\D/g, '');
  // if (raw.length === 11) {
  //   const formatted = raw.replace(/(\d{3})(\d{4})(\d{4})/, '$1 $2 $3');
  //   document.execCommand('insertText', false, formatted);
  // }
});
```

### 粘贴图片上传

```tsx
// 粘贴图片到输入框并上传
function ImagePasteInput() {
  const [images, setImages] = useState<string[]>([]);

  const handlePaste = useCallback(async (event: React.ClipboardEvent) => {
    const items = event.clipboardData?.items;
    if (!items) return;

    for (const item of Array.from(items)) {
      if (item.type.startsWith('image/')) {
        event.preventDefault();
        const file = item.getAsFile();
        if (!file) continue;

        // 预览
        const url = URL.createObjectURL(file);
        setImages((prev) => [...prev, url]);

        // 上传
        const formData = new FormData();
        formData.append('image', file, `paste-${Date.now()}.png`);

        try {
          const result = await fetch('/api/upload', { method: 'POST', body: formData });
          const { url: uploadedUrl } = await result.json();
          // 替换预览 URL 为上传后的 URL
          setImages((prev) => prev.map((u) => u === url ? uploadedUrl : u));
        } catch {
          setImages((prev) => prev.filter((u) => u !== url));
        }
      }
    }
  }, []);

  return (
    <div>
      <textarea
        onPaste={handlePaste}
        placeholder="粘贴图片或输入文字..."
      />
      <div className="pasted-images">
        {images.map((url, i) => (
          <img key={i} src={url} alt={`粘贴图片 ${i + 1}`} />
        ))}
      </div>
    </div>
  );
}
```

### 拖拽 + 剪贴板

```tsx
// 拖拽文件到页面并复制到剪贴板
function DragAndCopy() {
  const [dragging, setDragging] = useState(false);

  const handleDrop = useCallback(async (e: React.DragEvent) => {
    e.preventDefault();
    setDragging(false);

    const file = e.dataTransfer.files[0];
    if (!file || !file.type.startsWith('image/')) return;

    // 复制到剪贴板
    try {
      const pngBlob = file.type === 'image/png' ? file : await convertToPng(file);
      await navigator.clipboard.write([
        new ClipboardItem({ 'image/png': pngBlob }),
      ]);
      showToast('图片已复制到剪贴板');
    } catch {
      showToast('复制失败');
    }
  }, []);

  return (
    <div
      onDragOver={(e) => { e.preventDefault(); setDragging(true); }}
      onDragLeave={() => setDragging(false)}
      onDrop={handleDrop}
      style={{ border: dragging ? '2px dashed #1677ff' : '2px dashed #ddd', padding: 48, textAlign: 'center' }}
    >
      拖拽图片到此处，自动复制到剪贴板
    </div>
  );
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `navigator.clipboard` 为 undefined | 非 HTTPS 或非用户操作触发 | 必须在安全上下文 + 用户操作回调中调用 |
| writeText 报错 | 不在用户操作回调中 | 在 click/keydown 等事件处理函数中调用 |
| 图片复制失败 | 非 PNG 格式 | 转换为 PNG：Canvas toBlob('image/png') |
| ClipboardItem 构造失败 | Blob 类型不支持 | 只支持 text/plain、text/html、image/png |
| 读取剪贴板被拒 | 未授权 `clipboard-read` | 权限需用户授权，或用 paste 事件替代 |
| iOS Safari 部分不支持 | 旧版本限制 | 降级到 execCommand |
| 复制后在某些应用粘贴乱码 | HTML 格式不兼容 | 同时提供 text/plain 降级 |
| Canvas toBlob 返回 null | Canvas 跨域污染 | 图片需同源或 CORS 配置 |
| 复制大量文本卡顿 | writeText 同步处理 | Clipboard API 异步，不会卡顿；execCommand 会 |
| 拦截 paste 后输入框无法输入 | preventDefault 阻止默认行为 | 手动 insertText 或不阻止默认行为 |

### 最佳实践

- 始终在用户操作（click/keydown）回调中调用 Clipboard API
- 提供旧方案 `execCommand('copy')` 降级，兼容旧浏览器
- 复制富文本时同时提供 `text/plain` 和 `text/html`，确保粘贴到任何应用都有合理输出
- 图片复制优先转为 PNG 格式，兼容性最好
- 读取剪贴板需用户授权，优先用 `paste` 事件替代主动读取
- 粘贴图片上传时，先预览（ObjectURL）再上传，上传后替换 URL
- 复制后给明确反馈（"已复制"提示 + 按钮状态变化）
- 拦截 paste 事件时，注意 `preventDefault` 后需要手动处理输入
- Canvas 跨域图片不能 toBlob，需确保图片同源或配置 CORS

## 面试题

**Q1: Clipboard API 和 execCommand('copy') 有什么区别？为什么推荐 Clipboard API？**
> 五个区别：① 异步 vs 同步：Clipboard API 返回 Promise 不阻塞，execCommand 同步阻塞主线程；② 权限模型：Clipboard API 有明确权限（clipboard-read/write），execCommand 无权限控制；③ 数据类型：Clipboard API 支持文本、HTML、图片，execCommand 只支持文本；④ 读取能力：Clipboard API 可读取剪贴板内容，execCommand 只能写入；⑤ 稳定性：execCommand 已废弃，返回值不一定准确（可能返回 true 但实际未复制），Clipboard API 通过 Promise 明确成功/失败。推荐 Clipboard API 的核心原因：更安全（权限控制）、更可靠（Promise 结果）、更强大（多格式/读取）。

**Q2: 为什么 Clipboard API 必须在用户操作回调中调用？**
> 浏览器的安全策略：防止网页在用户不知情的情况下偷偷读写剪贴板（恶意网站可能读取用户复制的密码、银行卡号等敏感信息）。用户操作（click/keydown/touch）证明用户主动触发了剪贴板操作，浏览器才允许 API 调用。如果在异步回调（如 setTimeout、fetch 回调）中调用，浏览器认为不是用户主动操作，会抛出 `NotAllowedError`。例外：读取剪贴板可以通过 `clipboard-read` 权限授权后无需用户操作（但需用户手动授权弹窗）。

**Q3: 如何实现"复制富文本"——粘贴到邮件保留格式，粘贴到记事本为纯文本？**
> 使用 `ClipboardItem` 同时写入 `text/html` 和 `text/plain` 两种格式：`new ClipboardItem({ 'text/html': htmlBlob, 'text/plain': textBlob })`。粘贴时，应用会根据自身支持的格式选择：邮件/Word 支持 HTML → 使用 text/html 格式，保留格式；记事本/终端只支持纯文本 → 使用 text/plain 格式。这是剪贴板的多格式表示（Multiple Representation）机制：同一数据的不同表示，粘贴方按优先级选择。

**Q4: 如何实现粘贴图片并上传？**
> 监听元素的 `paste` 事件，从 `event.clipboardData.items` 中查找 `type.startsWith('image/')` 的条目，调用 `item.getAsFile()` 获取 File 对象，然后：① 用 `URL.createObjectURL(file)` 创建本地预览 URL；② 用 `FormData.append('image', file)` 构建上传表单；③ `fetch('/api/upload', { method: 'POST', body: formData })` 上传；④ 上传成功后替换预览 URL 为服务端返回的正式 URL。注意：paste 事件中必须同步获取 file，异步操作（如 await 后）clipboardData 会被清空。

**Q5: Canvas 图片复制到剪贴板的流程是什么？跨域图片有什么限制？**
> 流程：① `canvas.toBlob(callback, 'image/png')` 将 Canvas 转为 PNG Blob；② `new ClipboardItem({ 'image/png': blob })` 创建剪贴板条目；③ `navigator.clipboard.write([item])` 写入剪贴板。跨域限制：如果 Canvas 中绘制了跨域图片（未配置 CORS），Canvas 会被"污染"（tainted），调用 `toBlob()` 会抛出 SecurityError。解决：图片服务器设置 `Access-Control-Allow-Origin`，`img.crossOrigin = 'anonymous'`，或在服务端代理图片。

**Q6: 如何检测和处理剪贴板权限？**
> 两种方式：① `navigator.permissions.query({ name: 'clipboard-read' })` 查询权限状态（granted/prompt/denied）；② 直接调用 API 捕获 `NotAllowedError`。写入权限（clipboard-write）通常在用户操作回调中自动授予，不需要显式查询。读取权限（clipboard-read）需要用户授权，流程：检查权限状态 → 如果是 prompt 则调用 API 触发授权弹窗 → 如果是 denied 则引导用户手动修改浏览器设置。注意：`clipboard-read` 权限名称在某些浏览器中未实现，直接 try/catch 更稳妥。

**Q7: 剪贴板操作在移动端有什么限制？**
> 限制：① iOS Safari：`navigator.clipboard.writeText` 只在用户操作回调中有效（同桌面），`clipboard.readText` 可能不被支持；② Android Chrome：支持良好，但 `clipboard.read` 可能需要权限弹窗；③ 触摸操作：长按触发的系统复制菜单与 Clipboard API 独立，互不影响；④ 输入法：部分输入法会拦截 paste 事件，导致自定义粘贴处理不生效；⑤ 安全策略：部分浏览器限制非焦点元素的剪贴板操作。建议：移动端优先用 `writeText`（兼容性最好），复杂操作提供"长按复制"的降级方案。

**Q8: 如何实现"复制带引用来源"的文本？**
> 两种方式：① Clipboard API：`navigator.clipboard.write([new ClipboardItem({ 'text/html': htmlBlob, 'text/plain': textBlob })])`，HTML 中包含 `<blockquote>` + 来源链接，纯文本中附上来源 URL；② copy 事件拦截：监听 `copy` 事件，`event.preventDefault()` 阻止默认行为，用 `event.clipboardData.setData('text/html', ...)` 和 `setData('text/plain', ...)` 写入自定义内容。优势：粘贴到邮件/文档时显示格式化的引用块，粘贴到纯文本编辑器时显示纯文本 + 来源 URL。

---

**相关链接：**
- [[File API与文件处理]]
- [[Fetch API与请求模式]]
- [[浏览器高级API]]
- [[Web安全XSS与CSRF]]
