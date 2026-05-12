---
tags:
  - Web前端
  - Web Share
  - 分享
  - PWA
  - Web APIs
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Web Share API

## What — 是什么

> Web Share API 让 Web 应用调用系统原生分享面板，一键分享文本、链接、图片到其他应用（微信、QQ、短信、邮件等），无需为每个平台单独接入 SDK。Web Share Target API 则让 Web 应用注册为分享目标，接收其他应用的分享内容。

**核心概念：**

- **Web Share API（`navigator.share()`）**：调用系统分享面板，支持分享文本、URL、文件
- **Web Share Target API**：在 Web App Manifest 中声明分享目标，让 PWA 出现在系统分享列表中
- **share() 参数**：`title`、`text`、`url`、`files`，至少提供一个
- **canShare()**：检测当前设备是否支持分享指定内容
- **安全约束**：必须由用户操作触发（click），且需要 HTTPS

**系统分享面板示意：**

```
┌───────────────────────────────────┐
│         分享到                     │
├───────────────────────────────────┤
│  ┌─────┐  ┌─────┐  ┌─────┐      │
│  │微信  │  │QQ   │  │短信  │      │
│  └─────┘  └─────┘  └─────┘      │
│  ┌─────┐  ┌─────┐  ┌─────┐      │
│  │邮件  │  │复制  │  │更多  │      │
│  └─────┘  └─────┘  └─────┘      │
├───────────────────────────────────┤
│  标题: 文章标题                    │
│  内容: 文章摘要...                 │
│  链接: https://example.com/123   │
└───────────────────────────────────┘
```

**浏览器支持：**

| API | Chrome | Firefox | Safari | Edge |
|-----|--------|---------|--------|------|
| `navigator.share()` 文本/URL | 61+ (Android) | 79+ (Android) | 12.1+ (iOS) | 79+ |
| `navigator.share()` 文件 | 75+ (Android) | 不支持 | 15+ (iOS) | 79+ |
| Web Share Target API | 71+ (Android) | 不支持 | 不支持 | 79+ |
| 桌面端 share | 93+ | 不支持 | 12.1+ | 93+ |

> iOS Safari 支持最好（iOS 12.1+），Android Chrome 完整支持，桌面端 Chrome 93+ 部分支持。

## Why — 为什么

**适用场景：**

- 内容分享：文章、商品、活动页
- 社交传播：邀请链接、海报分享
- 文件分享：图片、PDF、CSV
- PWA 分享：让 PWA 与原生 App 一样出现在分享列表

**对比传统方案：**

| 维度 | Web Share API | JS-SDK 分享 | 自定义分享面板 |
|------|--------------|-------------|---------------|
| 系统集成 | 系统原生面板 | 各平台 SDK | 自建 UI |
| 覆盖应用 | 全部已安装应用 | 特定平台 | 复制链接 |
| 开发成本 | 极低 | 高（多平台 SDK） | 中 |
| 包体积 | 0 | 大（SDK 200KB+） | 小 |
| 图片分享 | 支持 | 部分支持 | 需另存 |
| 桌面端 | 部分支持 | 有限 | 支持 |
| 兼容性 | 移动端好 | 特定平台 | 全部 |

## How — 怎么用

### 基础分享

```ts
// 分享文本和链接
async function shareContent(data: {
  title?: string;
  text?: string;
  url?: string;
}): Promise<boolean> {
  if (!navigator.share) {
    console.warn('Web Share API 不支持');
    return false;
  }

  try {
    await navigator.share(data);
    console.log('分享成功');
    return true;
  } catch (error: any) {
    if (error.name === 'AbortError') {
      console.log('用户取消分享');
    } else {
      console.error('分享失败:', error);
    }
    return false;
  }
}

// 使用
shareContent({
  title: '这篇文章很棒',
  text: '推荐阅读：深入理解 React Server Components',
  url: 'https://example.com/article/123',
});
```

### 检测分享能力

```ts
// 检测是否支持分享
function isShareSupported(): boolean {
  return !!navigator.share;
}

// 检测是否支持分享文件
function isFileShareSupported(): boolean {
  return !!navigator.canShare?.({ files: [] });
}

// 检测是否支持分享特定内容
function canShareContent(data: ShareData): boolean {
  if (!navigator.canShare) return !!navigator.share;
  return navigator.canShare(data);
}

// 使用
const shareData = {
  title: '图片分享',
  files: [new File([blob], 'photo.png', { type: 'image/png' })],
};

if (canShareContent(shareData)) {
  navigator.share(shareData);
} else {
  // 降级：下载图片
  downloadImage(blob, 'photo.png');
}
```

### 分享文件

```ts
// 分享图片文件
async function shareImage(imageUrl: string, title: string): Promise<boolean> {
  try {
    // 1. 获取图片 Blob
    const response = await fetch(imageUrl);
    const blob = await response.blob();

    // 2. 创建 File 对象
    const file = new File([blob], 'shared-image.png', { type: 'image/png' });

    // 3. 检查是否支持文件分享
    const shareData = { title, files: [file] };

    if (!navigator.canShare?.(shareData)) {
      // 降级：复制链接
      await navigator.clipboard.writeText(imageUrl);
      showToast('链接已复制');
      return true;
    }

    await navigator.share(shareData);
    return true;
  } catch {
    return false;
  }
}

// 分享多个文件
async function shareMultipleFiles(files: File[], title: string) {
  const shareData = { title, files };

  if (!navigator.canShare?.(shareData)) {
    console.warn('不支持多文件分享');
    return;
  }

  await navigator.share(shareData);
}

// 分享 Canvas 图表
async function shareCanvasChart(canvas: HTMLCanvasElement, title: string) {
  const blob = await new Promise<Blob>((resolve) => {
    canvas.toBlob((b) => resolve(b!), 'image/png');
  });

  const file = new File([blob], `${title}.png`, { type: 'image/png' });
  const shareData = { title, files: [file] };

  if (navigator.canShare?.(shareData)) {
    await navigator.share(shareData);
  } else {
    // 降级：下载图片
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${title}.png`;
    a.click();
    URL.revokeObjectURL(url);
  }
}
```

### React 组件封装

```tsx
// components/ShareButton.tsx
interface ShareButtonProps {
  title?: string;
  text?: string;
  url?: string;
  files?: File[];
  fallback?: 'clipboard' | 'download' | 'none';
  children?: React.ReactNode;
}

function ShareButton({ title, text, url, files, fallback = 'clipboard', children }: ShareButtonProps) {
  const [sharing, setSharing] = useState(false);
  const [shared, setShared] = useState(false);

  const handleShare = async () => {
    setSharing(true);

    const shareData: ShareData = { title, text, url, files };

    // 支持 Web Share API
    if (navigator.share && (!files || navigator.canShare?.(shareData))) {
      try {
        await navigator.share(shareData);
        setShared(true);
        setTimeout(() => setShared(false), 2000);
      } catch (error: any) {
        if (error.name !== 'AbortError') {
          await handleFallback(shareData);
        }
      }
    } else {
      // 不支持 Web Share API
      await handleFallback(shareData);
    }

    setSharing(false);
  };

  const handleFallback = async (data: ShareData) => {
    if (fallback === 'clipboard' && data.url) {
      await navigator.clipboard.writeText(data.url);
      setShared(true);
      setTimeout(() => setShared(false), 2000);
    } else if (fallback === 'download' && data.files?.length) {
      const file = data.files[0];
      const a = document.createElement('a');
      a.href = URL.createObjectURL(file);
      a.download = file.name;
      a.click();
    }
  };

  return (
    <button onClick={handleShare} disabled={sharing}>
      {shared ? '已分享' : children || '分享'}
    </button>
  );
}

// 使用
function ArticlePage({ article }) {
  return (
    <div>
      <h1>{article.title}</h1>
      <p>{article.content}</p>

      <ShareButton
        title={article.title}
        text={article.excerpt}
        url={article.url}
        fallback="clipboard"
      >
        分享文章
      </ShareButton>

      {/* 分享图表图片 */}
      <ShareButton
        title="销售报表"
        files={[chartImageFile]}
        fallback="download"
      >
        分享报表
      </ShareButton>
    </div>
  );
}
```

### Web Share Target API（接收分享）

```json
// manifest.json — 注册为分享目标
{
  "name": "My App",
  "short_name": "MyApp",
  "start_url": "/",
  "display": "standalone",
  "share_target": {
    "action": "/share-target",
    "method": "POST",
    "enctype": "multipart/form-data",
    "params": {
      "title": "title",
      "text": "text",
      "url": "url",
      "files": [
        {
          "name": "images",
          "accept": ["image/png", "image/jpeg", "image/webp"]
        },
        {
          "name": "documents",
          "accept": ["application/pdf"]
        }
      ]
    }
  }
}
```

```ts
// 处理分享内容 — GET 方式（文本/URL）
// 用户从其他应用分享到本 App 时，浏览器导航到 /share-target?title=xxx&text=xxx&url=xxx

function handleShareTarget() {
  const params = new URLSearchParams(window.location.search);
  const title = params.get('title');
  const text = params.get('text');
  const url = params.get('url');

  if (title || text || url) {
    // 显示分享内容预览
    showSharePreview({ title, text, url });
    // 清除 URL 参数
    window.history.replaceState({}, '', '/share-target');
  }
}

// 处理分享内容 — POST 方式（文件）
// Service Worker 拦截 POST 请求，转为 GET 导航
// sw.js
self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);

  if (url.pathname === '/share-target' && event.request.method === 'POST') {
    event.respondWith(
      (async () => {
        const formData = await event.request.formData();
        const title = formData.get('title') as string;
        const text = formData.get('text') as string;
        const url = formData.get('url') as string;
        const images = formData.getAll('images') as File[];

        // 将文件存入 IndexedDB 或 Cache
        // 然后重定向到分享页面
        const params = new URLSearchParams({
          title: title || '',
          text: text || '',
          url: url || '',
          hasImages: String(images.length > 0),
        });

        return Response.redirect(`/share-preview?${params.toString()}`, 303);
      })()
    );
  }
});
```

### 自定义降级分享面板

```tsx
// 不支持 Web Share API 时的自定义面板
function SharePanel({ title, text, url, onClose }: {
  title?: string;
  text?: string;
  url?: string;
  onClose: () => void;
}) {
  const [copied, setCopied] = useState(false);

  const shareToWeibo = () => {
    window.open(`https://service.weibo.com/share/share.php?title=${encodeURIComponent(text || '')}&url=${encodeURIComponent(url || '')}`);
  };

  const shareToTwitter = () => {
    window.open(`https://twitter.com/intent/tweet?text=${encodeURIComponent(text || '')}&url=${encodeURIComponent(url || '')}`);
  };

  const copyLink = async () => {
    await navigator.clipboard.writeText(url || '');
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };

  return (
    <div className="share-panel-overlay" onClick={onClose}>
      <div className="share-panel" onClick={(e) => e.stopPropagation()}>
        <h3>分享到</h3>
        <div className="share-options">
          <button onClick={shareToWeibo}>微博</button>
          <button onClick={shareToTwitter}>Twitter</button>
          <button onClick={copyLink}>
            {copied ? '已复制' : '复制链接'}
          </button>
        </div>
        <button onClick={onClose}>取消</button>
      </div>
    </div>
  );
}

// 智能分享：优先原生，降级自定义
function SmartShare({ title, text, url }: ShareData) {
  const [showPanel, setShowPanel] = useState(false);

  const handleShare = async () => {
    if (navigator.share) {
      try {
        await navigator.share({ title, text, url });
      } catch (error: any) {
        if (error.name !== 'AbortError') {
          setShowPanel(true);
        }
      }
    } else {
      setShowPanel(true);
    }
  };

  return (
    <>
      <button onClick={handleShare}>分享</button>
      {showPanel && (
        <SharePanel
          title={title}
          text={text}
          url={url}
          onClose={() => setShowPanel(false)}
        />
      )}
    </>
  );
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| share() 报 NotAllowedError | 不在用户操作回调中 | 必须在 click 等事件中调用 |
| 桌面端不弹面板 | Chrome 桌面端 93+ 才支持 | 检测后降级到自定义面板 |
| 分享文件不显示 | 不支持该文件类型 | canShare() 预检，只支持 image/* 和部分文档 |
| iOS 分享面板少应用 | iOS 限制第三方 App 列表 | 系统行为，无法控制 |
| 分享后页面失焦 | 分享面板覆盖页面 | 正常行为，用户关闭面板后恢复 |
| Share Target 不生效 | PWA 未添加到主屏幕 | 用户必须先"添加到主屏幕" |
| POST 分享文件丢失 | Service Worker 未正确拦截 | SW 中处理 fetch 事件，存储文件后 redirect |
| 分享 URL 被 App 截断 | 某些 App 对 URL 长度有限制 | 使用短链服务 |
| canShare 返回 false | 文件类型不支持 | 检查 MIME 类型，只支持 image/* 和少量文档 |

### 最佳实践

- 优先使用 `navigator.share()`，不支持时降级到自定义面板或复制链接
- 用 `canShare()` 预检分享数据，避免调用后出错
- 必须在用户操作回调（click/touchend）中调用
- 分享数据提供完整：`title` + `text` + `url` 三件套
- 文件分享先检测 `canShare({ files })`，不支持时降级下载
- 桌面端 Chrome 93+ 支持原生分享，其他桌面浏览器降级处理
- Web Share Target 需要 PWA（添加到主屏幕），在 manifest.json 中配置
- 文件分享只支持 `image/*` 和少量文档格式，视频/音频不支持
- 处理 AbortError（用户取消）和其他错误的区别
- 分享成功/取消后不依赖 Promise resolve 区分（部分浏览器行为不一致）

## 面试题

**Q1: Web Share API 和传统社交 SDK 分享（如微信 JS-SDK）有什么区别？**
> 核心区别：Web Share API 调用系统原生分享面板，覆盖所有已安装应用（微信、QQ、短信、邮件等），无需任何 SDK；社交 SDK 只能分享到特定平台（如微信 JS-SDK 只能分享到微信），且需要引入 SDK、配置 appId、签名验证等。Web Share API 的优势：零依赖、零体积、覆盖面广、一次开发全平台通用。劣势：无法自定义分享面板 UI、无法统计分享到哪个平台、桌面端支持有限。选择：通用分享 → Web Share API；微信专属功能（如朋友圈分享卡片的标题/图片定制）→ 微信 JS-SDK。

**Q2: Web Share API 的安全限制有哪些？**
> 三个限制：① 用户操作触发：`navigator.share()` 必须在用户操作（click/touchend）回调中调用，不能在 setTimeout/fetch 回调中调用，防止网站偷偷弹出分享面板；② HTTPS：必须在安全上下文（HTTPS 或 localhost）中才能使用；③ 文件类型限制：只能分享特定 MIME 类型的文件（image/* 和部分文档），不能分享任意文件。另外，`navigator.share()` 返回的 Promise resolve 不代表用户真的完成了分享（用户可能取消），只能确认面板被打开过。

**Q3: Web Share Target API 是怎么让 PWA 接收分享的？**
> 流程：① 在 Web App Manifest 中声明 `share_target` 配置（action URL、接受的文件类型等）；② 用户将 PWA "添加到主屏幕"后，系统会把该 PWA 注册为分享目标；③ 其他应用分享内容时，系统分享面板中出现该 PWA 的图标；④ 用户选择 PWA 后，系统向 `action` URL 发送请求（GET 方式传文本/URL，POST 方式传文件）；⑤ Service Worker 拦截 POST 请求，提取分享数据，存储文件，重定向到预览页面。关键：必须添加到主屏幕才能成为分享目标，普通网页不会出现在分享列表中。

**Q4: 如何在不支持 Web Share API 的浏览器中实现分享？**
> 三层降级：① Web Share API（首选）：`navigator.share()` 调用系统原生面板；② 自定义分享面板：列出常用社交平台的 URL Scheme（微博、Twitter、Facebook 等），点击跳转对应平台的分享页面；③ 复制链接：`navigator.clipboard.writeText()` 将链接复制到剪贴板，提示用户手动粘贴分享。代码逻辑：`if (navigator.share) { share() } else if (isMobile) { showCustomPanel() } else { copyLink() }`。移动端优先尝试原生分享，桌面端直接复制链接最实用。

**Q5: canShare() 方法的作用是什么？**
> `navigator.canShare(data)` 检测当前浏览器是否支持分享指定的数据，返回 boolean。主要用途：① 检测文件分享支持：`canShare({ files: [file] })` 返回 false 说明不支持文件分享，需要降级；② 检测特定文件类型：`canShare({ files: [new File([], 'test.pdf', { type: 'application/pdf' })] })` 检测 PDF 是否可分享；③ 避免调用 share() 后报错。注意：canShare() 返回 true 不保证 share() 一定成功（可能因权限等原因失败），但返回 false 时一定不要调用 share()。

**Q6: 如何实现 PWA 既作为分享者又作为分享目标？**
> 两个 API 配合：① 分享出去：PWA 内使用 `navigator.share()` 将内容分享到其他应用；② 接收分享：在 manifest.json 中配置 `share_target`，让 PWA 出现在系统分享列表中。典型流程：用户在图库中选择图片 → 分享 → 选择 PWA → PWA 接收图片 → 编辑 → 通过 `navigator.share()` 再分享到社交平台。注意：分享目标只在 PWA 添加到主屏幕后生效；POST 方式接收文件需 Service Worker 拦截处理；两个 API 独立使用，不要求同时配置。

**Q7: 桌面端浏览器对 Web Share API 的支持情况如何？**
> Chrome 93+ 桌面端支持 `navigator.share()`，但行为与移动端不同：① 不弹出系统分享面板，而是弹出一个包含"复制链接"、"邮件"、"附近分享"等选项的简化面板；② 文件分享支持有限；③ Firefox/Safari 桌面端不支持。建议：桌面端优先用"复制链接"方案，而非 Web Share API——用户期望桌面端有更多控制权，直接复制链接到剪贴板体验更好。检测逻辑：`const isMobile = /Mobi|Android/i.test(navigator.userAgent)`，移动端用 share，桌面端用 clipboard。

**Q8: 分享图片时如何处理不同格式的兼容性？**
> Web Share API 文件分享的限制：① 只支持 `image/png`、`image/jpeg`、`image/webp`、`image/gif` 和少量文档格式；② 部分浏览器只接受 PNG/JPEG；③ 视频和音频不支持。处理策略：① 预检：`canShare({ files: [new File([], 'test.png', { type: 'image/png' })] })` 检测格式是否支持；② 格式转换：WebP 图片转为 PNG（Canvas toBlob），确保兼容性；③ 降级：不支持文件分享时，提供"下载图片"或"复制图片链接"替代；④ 尺寸限制：部分平台对分享图片大小有限制（如 < 10MB），大图需压缩后再分享。

---

**相关链接：**
- [[Service Worker与PWA]]
- [[Notification与Push API]]
- [[Clipboard API与剪贴板操作]]
- [[File API与文件处理]]
- [[浏览器高级API]]
