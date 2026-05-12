---
tags:
  - Web前端
  - Web APIs
  - File API
  - 文件处理
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# File API 与文件处理

## What — 什么是 File API

File API 是浏览器提供的文件处理接口体系，允许前端直接读取、创建、上传、下载、预览和编辑本地文件，无需依赖服务器中转。它由一组互相关联的 API 组成，覆盖了文件处理的完整生命周期。

### 核心对象与 API

| 对象/API | 说明 |
|----------|------|
| `Blob` | 二进制大对象，文件的底层数据表示 |
| `File` | 继承自 Blob，增加文件名、修改时间等元信息 |
| `FileReader` | 异步读取文件内容（文本/二进制/Data URL） |
| `URL.createObjectURL()` | 为 Blob/File 生成临时 URL，用于预览 |
| `File System Access API` | 读写本地文件系统的现代 API |
| `CompressionStream` | 浏览器原生压缩/解压流 |
| `Clipboard API` | 剪贴板读写文件（如图片） |
| `DataTransfer` | 拖拽操作中传递文件数据 |

### 最简示例

```js
// 选择文件并读取内容
const input = document.querySelector('input[type="file"]');
input.addEventListener('change', async () => {
  const file = input.files[0];
  console.log(`文件名: ${file.name}, 大小: ${file.size}B, 类型: ${file.type}`);
  const text = await file.text();
  console.log('文件内容:', text);
});
```

### 演进历史

| 时间 | 里程碑 |
|------|--------|
| 2000 | `<input type="file">` 仅能提交表单，JS 无法访问文件内容 |
| 2010 | File API（File/FileReader/Blob）W3C 工作草案，JS 首次能读文件 |
| 2013 | `URL.createObjectURL` 广泛支持，文件预览成为可能 |
| 2016 | `Blob.stream()` / `blob.text()` 等 Promise 方法加入 |
| 2020 | File System Access API（Chrome 86），可直接读写本地文件 |
| 2022 | CompressionStream / DecompressionStream 原生压缩支持 |

---

## Why — 为什么需要 File API

### 1. 文件上传体验优化

传统表单文件上传只能提交，无法预览、校验、压缩。File API 让前端在上传前完成全部预处理：

```js
// 上传前预览 + 压缩 + 校验
const input = document.querySelector('input[type="file"]');
input.addEventListener('change', async () => {
  const file = input.files[0];

  // 校验文件大小
  if (file.size > 10 * 1024 * 1024) {
    alert('文件不能超过 10MB');
    return;
  }

  // 校验文件类型
  if (!file.type.startsWith('image/')) {
    alert('请选择图片文件');
    return;
  }

  // 压缩图片后上传
  const compressed = await compressImage(file, 0.7);
  await uploadFile(compressed);
});
```

### 2. 文件预览免服务端

传统方案需要先上传到服务器再返回 URL 预览，File API 实现纯前端预览：

```js
// 纯前端图片预览，无需上传
const url = URL.createObjectURL(file);
imgElement.src = url;

// 纯前端文本预览
const text = await file.text();
preElement.textContent = text;
```

### 3. 本地文件编辑

File System Access API 让 Web 应用像桌面应用一样直接编辑本地文件：

```js
// VS Code for Web 的核心能力：直接读写本地文件
const handle = await window.showOpenFilePicker();
const file = await handle.getFile();
const content = await file.text();
// 编辑后写回
const writable = await handle.createWritable();
await writable.write(editedContent);
await writable.close();
```

### 4. 大文件处理能力

分片读取、流式处理、断点续传让前端可以处理 GB 级文件：

```js
// 流式读取大文件，不阻塞内存
const stream = file.stream();
const reader = stream.getReader();
while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  processChunk(value); // 逐块处理
}
```

---

## FileReader vs blob.arrayBuffer() vs File System Access API

| 维度 | FileReader | Blob Promise 方法 | File System Access API |
|------|-----------|-------------------|----------------------|
| **异步模式** | 事件回调（onload/onerror） | Promise/async-await | Promise/async-await |
| **读取方式** | readAsText/ArrayBuffer/DataURL/BinaryString | blob.text()/arrayBuffer()/stream() | file.text()/arrayBuffer() + 可写流 |
| **进度监听** | 支持 onprogress | 不支持 | 不支持 |
| **多次读取** | 每次需重新创建实例 | 可重复调用 | 可重复调用 |
| **写入能力** | 无 | 无 | 有（createWritable） |
| **文件选择** | 需配合 input 或拖拽 | 需配合 input 或拖拽 | showOpenFilePicker/showDirectoryPicker |
| **目录操作** | 不支持 | 不支持 | 支持遍历目录 |
| **兼容性** | 全浏览器（IE10+） | 现代浏览器 | 仅 Chromium（Chrome/Edge） |
| **适用场景** | 需要进度/多种读取格式 | 简单读取、现代项目 | 本地编辑器、文件管理器 |

### 何时选什么

| 场景 | 推荐 | 理由 |
|------|------|------|
| 简单读取文件内容 | `blob.text()` / `blob.arrayBuffer()` | 代码最简洁，Promise 风格 |
| 需要读取进度 | FileReader | 唯一支持 onprogress |
| 需要读取为 Data URL | FileReader `readAsDataURL` | Blob 方法无直接替代 |
| 本地文件编辑器 | File System Access API | 唯一能写回本地文件的方案 |
| 大文件流式处理 | `blob.stream()` | 流式读取，内存友好 |
| 兼容性要求高 | FileReader | IE10+ 全支持 |

---

## How — 怎么用

### 1. File 与 Blob 对象

#### Blob 构造与属性

```js
// Blob 构造函数：new Blob(array, options)
const blob1 = new Blob(['Hello, World!'], { type: 'text/plain' });
const blob2 = new Blob(['<h1>Title</h1>'], { type: 'text/html' });
const blob3 = new Blob([JSON.stringify({ name: '张三' })], { type: 'application/json' });

// 合并多个 Blob
const combined = new Blob([blob1, blob2], { type: 'text/plain' });

// Blob 属性
console.log(blob1.size); // 13（字节数）
console.log(blob1.type); // 'text/plain'
```

#### Blob.slice 分片

```js
// slice(start, end, contentType) —— 切割 Blob
const fullText = new Blob(['Hello, World! This is a test.']);
const firstWord = fullText.slice(0, 5); // 'Hello'

// 大文件分片上传
const file = input.files[0];
const CHUNK_SIZE = 5 * 1024 * 1024; // 5MB
const totalChunks = Math.ceil(file.size / CHUNK_SIZE);

for (let i = 0; i < totalChunks; i++) {
  const start = i * CHUNK_SIZE;
  const end = Math.min(start + CHUNK_SIZE, file.size);
  const chunk = file.slice(start, end);
  // 上传分片...
}
```

#### File 对象与属性

File 继承自 Blob，增加了文件元信息：

```js
// File 来源：input 或拖拽
const input = document.querySelector('input[type="file"]');
input.addEventListener('change', () => {
  const file = input.files[0];

  // File 继承 Blob 的属性
  console.log(file.size); // 文件大小（字节）
  console.log(file.type); // MIME 类型，如 'image/png'

  // File 独有属性
  console.log(file.name);         // 文件名：'photo.png'
  console.log(file.lastModified); // 最后修改时间戳：1715500800000
  console.log(file.lastModifiedDate); // Date 对象（已废弃，用 lastModified）
});
```

#### 手动创建 File 对象

```js
// new File(array, name, options)
const file = new File(
  ['Hello, World!'],
  'hello.txt',
  { type: 'text/plain', lastModified: Date.now() }
);

console.log(file.name); // 'hello.txt'
console.log(file.type); // 'text/plain'
```

#### MIME 类型速查

| 类型 | MIME | 说明 |
|------|------|------|
| 纯文本 | `text/plain` | .txt |
| HTML | `text/html` | .html |
| CSV | `text/csv` | .csv |
| JSON | `application/json` | .json |
| PNG | `image/png` | .png |
| JPEG | `image/jpeg` | .jpg |
| GIF | `image/gif` | .gif |
| WebP | `image/webp` | .webp |
| SVG | `image/svg+xml` | .svg |
| PDF | `application/pdf` | .pdf |
| ZIP | `application/zip` | .zip |
| MP4 | `video/mp4` | .mp4 |
| WebM | `video/webm` | .webm |
| MP3 | `audio/mpeg` | .mp3 |
| Excel | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | .xlsx |

---

### 2. FileReader 读取文件

FileReader 是传统的文件读取 API，基于事件回调模型。

#### 四种读取方法

```js
const reader = new FileReader();
const file = input.files[0];

// 1. readAsText —— 读取为文本字符串
reader.readAsText(file);           // 默认 UTF-8
reader.readAsText(file, 'GBK');    // 指定编码

// 2. readAsArrayBuffer —— 读取为 ArrayBuffer（二进制）
reader.readAsArrayBuffer(file);

// 3. readAsDataURL —— 读取为 Base64 Data URL
reader.readAsDataURL(file);  // 'data:image/png;base64,iVBOR...'

// 4. readAsBinaryString —— 读取为二进制字符串（已废弃，用 ArrayBuffer）
reader.readAsBinaryString(file);
```

#### 事件模型

```js
const reader = new FileReader();

reader.onload = (e) => {
  console.log('读取完成:', e.target.result);
};

reader.onerror = (e) => {
  console.error('读取失败:', reader.error);
};

reader.onprogress = (e) => {
  if (e.lengthComputable) {
    const percent = Math.round((e.loaded / e.total) * 100);
    console.log(`读取进度: ${percent}%`);
  }
};

reader.onloadstart = () => {
  console.log('开始读取');
};

reader.onloadend = () => {
  console.log('读取结束（无论成功失败）');
};

reader.onabort = () => {
  console.log('读取被中止');
};

// 中止读取
// reader.abort();
```

#### Promise 封装

```js
function readFileAsText(file, encoding = 'UTF-8') {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = () => reject(reader.error);
    reader.readAsText(file, encoding);
  });
}

function readFileAsDataURL(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = () => reject(reader.error);
    reader.readAsDataURL(file);
  });
}

function readFileAsArrayBuffer(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = () => reject(reader.error);
    reader.readAsArrayBuffer(file);
  });
}

// 使用
const text = await readFileAsText(file);
const dataURL = await readFileAsDataURL(file);
const buffer = await readFileAsArrayBuffer(file);
```

#### 多文件读取

```js
// 并行读取多个文件
async function readMultipleFiles(files) {
  const promises = Array.from(files).map(file => readFileAsDataURL(file));
  const results = await Promise.all(promises);
  return results; // Data URL 数组
}

// 使用
input.addEventListener('change', async () => {
  const urls = await readMultipleFiles(input.files);
  urls.forEach((url, i) => {
    const img = document.createElement('img');
    img.src = url;
    document.body.appendChild(img);
  });
});
```

#### 带进度的读取

```js
function readFileWithProgress(file, onProgress) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = () => reject(reader.error);
    reader.onprogress = (e) => {
      if (e.lengthComputable) {
        onProgress(Math.round((e.loaded / e.total) * 100));
      }
    };
    reader.readAsArrayBuffer(file);
  });
}

// 使用
const buffer = await readFileWithProgress(file, (percent) => {
  progressBar.style.width = `${percent}%`;
  progressText.textContent = `${percent}%`;
});
```

---

### 3. Blob 流式读取

Blob 提供了更现代的 Promise 方法，推荐用于简单读取场景。

#### blob.text() / blob.arrayBuffer() / blob.stream()

```js
const file = input.files[0];

// blob.text() —— 读取为文本（Promise）
const text = await file.text();
console.log(text);

// blob.arrayBuffer() —— 读取为 ArrayBuffer（Promise）
const buffer = await file.arrayBuffer();
const view = new DataView(buffer);
console.log(view.getUint32(0)); // 读取前4字节

// blob.stream() —— 获取 ReadableStream
const stream = file.stream();
const reader = stream.getReader();
while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  console.log(`收到 ${value.length} 字节`, value);
}
```

#### 分片读取大文件

```js
// 分片读取，避免一次性加载大文件到内存
async function readLargeFileInChunks(file, chunkSize = 1024 * 1024) {
  const results = [];
  let offset = 0;

  while (offset < file.size) {
    const end = Math.min(offset + chunkSize, file.size);
    const chunk = file.slice(offset, end);
    const chunkBuffer = await chunk.arrayBuffer();
    results.push(chunkBuffer);
    offset = end;
    console.log(`已读取: ${Math.round((offset / file.size) * 100)}%`);
  }

  return results;
}
```

#### 流式逐行读取文本文件

```js
async function readTextFileLineByLine(file) {
  const reader = file.stream()
    .pipeThrough(new TextDecoderStream())
    .getReader();

  let buffer = '';
  const lines = [];

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += value;
    const parts = buffer.split('\n');
    buffer = parts.pop(); // 最后一行可能不完整

    for (const line of parts) {
      lines.push(line);
      // 逐行处理，无需等待整个文件加载
      processLine(line);
    }
  }

  if (buffer) lines.push(buffer);
  return lines;
}
```

#### Blob 与 ArrayBuffer 互转

```js
// Blob -> ArrayBuffer
const buffer = await blob.arrayBuffer();

// ArrayBuffer -> Blob
const blob = new Blob([arrayBuffer], { type: 'application/octet-stream' });

// Blob -> Uint8Array
const buffer = await blob.arrayBuffer();
const uint8 = new Uint8Array(buffer);

// Uint8Array -> Blob
const blob = new Blob([uint8], { type: 'application/octet-stream' });
```

---

### 4. URL.createObjectURL 与 revokeObjectURL

`createObjectURL` 为 Blob/File 生成一个指向内存中数据的临时 URL，可用于预览图片、视频、PDF 等。

#### 基本用法

```js
const file = input.files[0];

// 创建临时 URL
const url = URL.createObjectURL(file);
console.log(url); // 'blob:https://example.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'

// 用于预览
imgElement.src = url;
videoElement.src = url;

// 重要：用完必须释放，否则内存泄漏
URL.revokeObjectURL(url);
```

#### 图片预览

```js
const input = document.querySelector('#file-input');
const preview = document.querySelector('#preview');

input.addEventListener('change', () => {
  // 清除上一个预览
  if (preview.src.startsWith('blob:')) {
    URL.revokeObjectURL(preview.src);
  }

  const file = input.files[0];
  if (file && file.type.startsWith('image/')) {
    const url = URL.createObjectURL(file);
    preview.src = url;
    preview.onload = () => {
      URL.revokeObjectURL(url); // 图片加载完成后立即释放
    };
  }
});
```

#### 多图片预览列表

```js
const input = document.querySelector('#file-input');
const gallery = document.querySelector('#gallery');

input.addEventListener('change', () => {
  // 清除旧预览
  gallery.querySelectorAll('img').forEach(img => {
    if (img.src.startsWith('blob:')) {
      URL.revokeObjectURL(img.src);
    }
  });
  gallery.innerHTML = '';

  Array.from(input.files).forEach(file => {
    if (!file.type.startsWith('image/')) return;

    const url = URL.createObjectURL(file);
    const img = document.createElement('img');
    img.src = url;
    img.alt = file.name;
    img.style.maxWidth = '200px';
    img.onload = () => URL.revokeObjectURL(url);
    gallery.appendChild(img);
  });
});
```

#### 视频预览与缩略图

```js
// 视频预览
const videoUrl = URL.createObjectURL(videoFile);
videoElement.src = videoUrl;

// 提取视频缩略图
function getVideoThumbnail(videoFile) {
  return new Promise((resolve) => {
    const video = document.createElement('video');
    const url = URL.createObjectURL(videoFile);
    video.src = url;

    video.addEventListener('loadeddata', () => {
      video.currentTime = 1; // 跳到第1秒
    });

    video.addEventListener('seeked', () => {
      const canvas = document.createElement('canvas');
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      canvas.getContext('2d').drawImage(video, 0, 0);
      URL.revokeObjectURL(url);
      resolve(canvas.toDataURL('image/jpeg', 0.8));
    });
  });
}
```

#### PDF 预览

```js
const file = input.files[0];
const url = URL.createObjectURL(file);

// 方式1：iframe
const iframe = document.createElement('iframe');
iframe.src = url;
iframe.width = '100%';
iframe.height = '600px';
document.body.appendChild(iframe);

// 方式2：object 标签
const obj = document.createElement('object');
obj.data = url;
obj.type = 'application/pdf';
obj.width = '100%';
obj.height = '600px';
document.body.appendChild(obj);

// 方式3：新窗口打开
window.open(url);
```

#### 内存管理

```js
// 错误：忘记 revokeObjectURL 导致内存泄漏
function leakyPreview(file) {
  const url = URL.createObjectURL(file);
  img.src = url; // URL 永远不会被释放！
}

// 正确：用完即释放
function safePreview(file) {
  const url = URL.createObjectURL(file);
  img.src = url;
  img.onload = () => URL.revokeObjectURL(url);
}

// 正确：组件卸载时统一释放（React）
useEffect(() => {
  const urls = files.map(f => URL.createObjectURL(f));
  setPreviewUrls(urls);

  return () => {
    urls.forEach(url => URL.revokeObjectURL(url));
  };
}, [files]);
```

#### createObjectURL vs readAsDataURL

| 维度 | createObjectURL | readAsDataURL |
|------|----------------|---------------|
| 返回值 | `blob:` 临时 URL | `data:` Base64 字符串 |
| 内存占用 | 低（指向原始 Blob 引用） | 高（Base64 比原始数据大 33%） |
| 生命周期 | 需手动 revoke | 无需管理 |
| 适用场景 | 预览、临时引用 | 小文件内嵌、Canvas 输出、传输 |
| 大文件支持 | 好 | 差（大文件 Base64 字符串极长） |
| 性能 | 快 | 慢（需要 Base64 编码） |

---

### 5. 文件上传

#### input file 基础

```html
<!-- 基本文件选择 -->
<input type="file" id="fileInput">

<!-- 限制文件类型 -->
<input type="file" accept="image/*">
<input type="file" accept=".jpg,.png,.gif">
<input type="file" accept="image/png,image/jpeg">
<input type="file" accept=".pdf,.doc,.docx">

<!-- 多文件选择 -->
<input type="file" multiple>

<!-- 移动端调起相机 -->
<input type="file" accept="image/*" capture="environment">  <!-- 后置摄像头 -->
<input type="file" accept="image/*" capture="user">          <!-- 前置摄像头 -->
<input type="file" accept="video/*" capture="environment">   <!-- 录像 -->

<!-- 组合 -->
<input type="file" id="fileInput" accept="image/*" multiple>
```

#### accept / multiple / capture 属性详解

| 属性 | 值 | 说明 |
|------|---|------|
| `accept` | `image/*` | 接受所有图片 |
| `accept` | `.jpg,.png` | 接受指定扩展名 |
| `accept` | `application/pdf` | 接受指定 MIME 类型 |
| `multiple` | 无值 | 允许选择多个文件 |
| `capture` | `user` | 前置摄像头（移动端） |
| `capture` | `environment` | 后置摄像头（移动端） |

> 注意：`accept` 只是建议，用户仍可切换文件选择器选择其他类型。必须在 JS 中二次校验。

#### FormData 上传

```js
// 单文件上传
async function uploadFile(file) {
  const formData = new FormData();
  formData.append('file', file);
  formData.append('description', '用户上传');

  const response = await fetch('/api/upload', {
    method: 'POST',
    body: formData // 不设置 Content-Type，浏览器自动添加 boundary
  });

  return response.json();
}

// 多文件上传
async function uploadFiles(files) {
  const formData = new FormData();
  for (const file of files) {
    formData.append('files', file); // 同名 append，后端接收数组
  }

  const response = await fetch('/api/upload/batch', {
    method: 'POST',
    body: formData
  });

  return response.json();
}

// 携带额外字段
async function uploadWithMetadata(file, metadata) {
  const formData = new FormData();
  formData.append('file', file);
  formData.append('userId', metadata.userId);
  formData.append('category', metadata.category);
  formData.append('tags', JSON.stringify(metadata.tags));

  const response = await fetch('/api/upload', {
    method: 'POST',
    body: formData
  });

  return response.json();
}
```

#### 拖拽上传

```html
<div id="dropZone" class="drop-zone">
  <p>拖拽文件到此处，或点击选择文件</p>
  <input type="file" id="fileInput" multiple hidden>
</div>
<div id="fileList"></div>
```

```js
const dropZone = document.getElementById('dropZone');
const fileInput = document.getElementById('fileInput');

// 点击触发文件选择
dropZone.addEventListener('click', () => fileInput.click());

// 拖拽事件
dropZone.addEventListener('dragenter', (e) => {
  e.preventDefault();
  dropZone.classList.add('drag-over');
});

dropZone.addEventListener('dragover', (e) => {
  e.preventDefault(); // 必须阻止默认行为，否则浏览器会打开文件
  e.dataTransfer.dropEffect = 'copy';
});

dropZone.addEventListener('dragleave', (e) => {
  // 避免拖入子元素时误触发
  if (!dropZone.contains(e.relatedTarget)) {
    dropZone.classList.remove('drag-over');
  }
});

dropZone.addEventListener('drop', async (e) => {
  e.preventDefault();
  dropZone.classList.remove('drag-over');

  const files = e.dataTransfer.files;
  await handleFiles(files);
});

// input 选择
fileInput.addEventListener('change', () => {
  handleFiles(fileInput.files);
});

async function handleFiles(files) {
  for (const file of files) {
    console.log(`${file.name} (${formatSize(file.size)})`);
  }

  const formData = new FormData();
  for (const file of files) {
    formData.append('files', file);
  }

  const result = await fetch('/api/upload', {
    method: 'POST',
    body: formData
  }).then(r => r.json());

  console.log('上传结果:', result);
}

function formatSize(bytes) {
  if (bytes < 1024) return bytes + ' B';
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB';
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB';
}
```

#### 上传进度监听

Fetch 不支持上传进度，需要用 XHR 或改用 XMLHttpRequest：

```js
function uploadWithProgress(url, formData, onProgress) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('POST', url);

    // 上传进度
    xhr.upload.onprogress = (e) => {
      if (e.lengthComputable) {
        const percent = Math.round((e.loaded / e.total) * 100);
        onProgress(percent);
      }
    };

    xhr.upload.onload = () => {
      console.log('上传完成');
    };

    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`上传失败: HTTP ${xhr.status}`));
      }
    };

    xhr.onerror = () => reject(new Error('网络错误'));
    xhr.onabort = () => reject(new Error('上传取消'));
    xhr.send(formData);
  });
}

// 使用
const result = await uploadWithProgress('/api/upload', formData, (percent) => {
  progressBar.style.width = `${percent}%`;
  progressText.textContent = `${percent}%`;
});
```

#### 大文件分片上传 + 断点续传

```js
class ChunkUploader {
  constructor(options = {}) {
    this.chunkSize = options.chunkSize || 5 * 1024 * 1024; // 默认5MB
    this.concurrency = options.concurrency || 3;            // 并发数
    this.uploadUrl = options.uploadUrl || '/api/upload/chunk';
    this.mergeUrl = options.mergeUrl || '/api/upload/merge';
  }

  // 计算文件哈希（用于标识文件）
  async calculateHash(file) {
    const buffer = await file.arrayBuffer();
    const hashBuffer = await crypto.subtle.digest('SHA-256', buffer);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
  }

  // 查询已上传的分片（断点续传）
  async getUploadedChunks(fileHash) {
    try {
      const res = await fetch(`/api/upload/progress?hash=${fileHash}`);
      const data = await res.json();
      return new Set(data.uploadedChunks || []); // [0, 1, 3, 5]
    } catch {
      return new Set();
    }
  }

  // 上传单个分片
  async uploadChunk(chunk, chunkIndex, fileHash, fileName) {
    const formData = new FormData();
    formData.append('file', chunk);
    formData.append('hash', fileHash);
    formData.append('chunkIndex', chunkIndex);
    formData.append('filename', fileName);

    const res = await fetch(this.uploadUrl, {
      method: 'POST',
      body: formData
    });

    if (!res.ok) throw new Error(`分片 ${chunkIndex} 上传失败`);
    return chunkIndex;
  }

  // 并发控制上传
  async uploadChunks(file, fileHash, uploadedChunks, onProgress) {
    const totalChunks = Math.ceil(file.size / this.chunkSize);
    let uploaded = uploadedChunks.size;
    const pendingChunks = [];

    for (let i = 0; i < totalChunks; i++) {
      if (uploadedChunks.has(i)) continue; // 跳过已上传
      const start = i * this.chunkSize;
      const end = Math.min(start + this.chunkSize, file.size);
      pendingChunks.push({ index: i, blob: file.slice(start, end) });
    }

    // 并发上传
    let taskIndex = 0;
    const runTask = async () => {
      while (taskIndex < pendingChunks.length) {
        const { index, blob } = pendingChunks[taskIndex++];
        try {
          await this.uploadChunk(blob, index, fileHash, file.name);
          uploaded++;
          onProgress?.(Math.round((uploaded / totalChunks) * 100));
        } catch (error) {
          // 重试一次
          await this.uploadChunk(blob, index, fileHash, file.name);
          uploaded++;
          onProgress?.(Math.round((uploaded / totalChunks) * 100));
        }
      }
    };

    await Promise.all(
      Array.from({ length: Math.min(this.concurrency, pendingChunks.length) }, () => runTask())
    );
  }

  // 合并分片
  async mergeChunks(fileHash, fileName, totalChunks) {
    const res = await fetch(this.mergeUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        hash: fileHash,
        filename: fileName,
        totalChunks,
        chunkSize: this.chunkSize
      })
    });
    return res.json();
  }

  // 完整上传流程
  async upload(file, onProgress) {
    // 1. 计算哈希
    const fileHash = await this.calculateHash(file);

    // 2. 查询已上传分片（断点续传）
    const uploadedChunks = await this.getUploadedChunks(fileHash);

    // 3. 上传未完成的分片
    const totalChunks = Math.ceil(file.size / this.chunkSize);
    await this.uploadChunks(file, fileHash, uploadedChunks, onProgress);

    // 4. 合并
    return this.mergeChunks(fileHash, file.name, totalChunks);
  }
}

// 使用
const uploader = new ChunkUploader({
  chunkSize: 5 * 1024 * 1024,
  concurrency: 3
});

const result = await uploader.upload(largeFile, (percent) => {
  progressBar.style.width = `${percent}%`;
  console.log(`上传进度: ${percent}%`);
});
console.log('上传完成:', result);
```

#### 精简版大文件哈希（抽样计算）

```js
// 大文件全量计算 SHA-256 太慢，采用抽样策略
async function calculateHashSampled(file) {
  const SAMPLE_SIZE = 2 * 1024 * 1024; // 按每2MB取一段
  const chunks = [];
  let offset = 0;

  while (offset < file.size) {
    const end = Math.min(offset + SAMPLE_SIZE, file.size);
    // 每段取头部2KB + 尾部2KB
    const chunk = file.slice(offset, end);
    const head = chunk.slice(0, 2048);
    const tail = chunk.slice(Math.max(0, chunk.size - 2048));
    chunks.push(await head.arrayBuffer());
    chunks.push(await tail.arrayBuffer());
    offset = end;
  }

  // 加入文件大小和总块数信息
  const info = new TextEncoder().encode(`${file.size}-${file.name}-${file.lastModified}`);
  chunks.push(info.buffer);

  const combined = new Blob(chunks);
  const buffer = await combined.arrayBuffer();
  const hashBuffer = await crypto.subtle.digest('SHA-256', buffer);
  return Array.from(new Uint8Array(hashBuffer))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
}
```

---

### 6. 文件下载

#### Blob + URL 下载

```js
// 纯前端生成文件并下载
function downloadFile(content, filename, mimeType = 'text/plain') {
  const blob = new Blob([content], { type: mimeType });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

// 下载文本
downloadFile('Hello, World!', 'hello.txt', 'text/plain');

// 下载 JSON
downloadFile(JSON.stringify(data, null, 2), 'data.json', 'application/json');

// 下载 CSV
const csv = '姓名,年龄\n张三,25\n李四,30';
downloadFile(csv, 'data.csv', 'text/csv');

// 下载 HTML
downloadFile('<h1>Hello</h1>', 'page.html', 'text/html');
```

#### a 标签 download 属性

```html
<!-- 静态下载链接 -->
<a href="/files/document.pdf" download>下载文档</a>

<!-- 指定下载文件名 -->
<a href="/files/report.xlsx" download="月度报表.xlsx">下载报表</a>

<!-- 注意：跨域链接的 download 属性被忽略 -->
<!-- 以下不生效，浏览器会导航而非下载 -->
<a href="https://other.com/file.pdf" download>不会下载</a>
```

> `download` 属性的限制：仅对同源 URL 和 `blob:` / `data:` URL 有效。跨域链接的 download 属性会被浏览器忽略。

#### 流式下载

```js
// 流式构建大文件并下载（不阻塞内存）
async function downloadLargeFile() {
  const { writable, readable } = new TransformStream();
  const writer = writable.getWriter();

  // 后台逐步写入数据
  (async () => {
    for (let i = 0; i < 1000; i++) {
      const chunk = generateChunk(i);
      await writer.write(new TextEncoder().encode(chunk));
    }
    await writer.close();
  })();

  // 转为 Blob 下载
  const blob = await new Response(readable).blob();
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'large-file.csv';
  a.click();
  URL.revokeObjectURL(url);
}
```

#### fetch 下载 + 进度

```js
async function downloadWithProgress(url, filename, onProgress) {
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`下载失败: HTTP ${response.status}`);
  }

  const contentLength = +response.headers.get('Content-Length');
  let receivedLength = 0;

  const reader = response.body.getReader();
  const chunks = [];

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    chunks.push(value);
    receivedLength += value.length;

    if (contentLength && onProgress) {
      const percent = Math.round((receivedLength / contentLength) * 100);
      onProgress(percent, receivedLength, contentLength);
    }
  }

  // 合并分片并触发下载
  const blob = new Blob(chunks);
  const objectUrl = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = objectUrl;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(objectUrl);
}

// 使用
await downloadWithProgress(
  '/api/files/report.pdf',
  '报表.pdf',
  (percent, loaded, total) => {
    progressBar.style.width = `${percent}%`;
    progressText.textContent = `${percent}% (${formatSize(loaded)}/${formatSize(total)})`;
  }
);
```

#### ArrayBuffer / Uint8Array 下载

```js
// 下载二进制数据
function downloadBinary(uint8Array, filename, mimeType) {
  const blob = new Blob([uint8Array], { type: mimeType });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}

// 下载 Canvas 图像
function downloadCanvas(canvas, filename = 'image.png') {
  canvas.toBlob((blob) => {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.click();
    URL.revokeObjectURL(url);
  });
}
```

---

### 7. 文件预览

#### 图片预览 + Canvas 压缩

```js
// 图片预览
function previewImage(file, imgElement) {
  const url = URL.createObjectURL(file);
  imgElement.src = url;
  imgElement.onload = () => URL.revokeObjectURL(url);
}

// Canvas 压缩图片
function compressImage(file, quality = 0.7, maxWidth = 1920) {
  return new Promise((resolve) => {
    const img = new Image();
    const url = URL.createObjectURL(file);
    img.src = url;

    img.onload = () => {
      URL.revokeObjectURL(url);

      // 计算压缩后尺寸
      let { width, height } = img;
      if (width > maxWidth) {
        height = (height * maxWidth) / width;
        width = maxWidth;
      }

      // 绘制到 Canvas
      const canvas = document.createElement('canvas');
      canvas.width = width;
      canvas.height = height;
      const ctx = canvas.getContext('2d');
      ctx.drawImage(img, 0, 0, width, height);

      // 导出压缩后的 Blob
      canvas.toBlob(
        (blob) => resolve(blob),
        file.type || 'image/jpeg',
        quality
      );
    };
  });
}

// 使用
const compressed = await compressImage(file, 0.6, 1200);
console.log(`原始: ${file.size}B, 压缩后: ${compressed.size}B`);
```

#### 图片裁剪预览

```js
function cropImage(file, cropRect) {
  return new Promise((resolve) => {
    const img = new Image();
    const url = URL.createObjectURL(file);
    img.src = url;

    img.onload = () => {
      URL.revokeObjectURL(url);

      const canvas = document.createElement('canvas');
      canvas.width = cropRect.width;
      canvas.height = cropRect.height;
      const ctx = canvas.getContext('2d');
      ctx.drawImage(
        img,
        cropRect.x, cropRect.y, cropRect.width, cropRect.height,
        0, 0, cropRect.width, cropRect.height
      );

      canvas.toBlob((blob) => resolve(blob), 'image/jpeg', 0.9);
    };
  });
}
```

#### PDF 预览

```js
// 方式1：iframe 嵌入（最简单）
function previewPDF(file, container) {
  const url = URL.createObjectURL(file);
  const iframe = document.createElement('iframe');
  iframe.src = url;
  iframe.width = '100%';
  iframe.height = '100%';
  iframe.style.border = 'none';
  container.appendChild(iframe);
  // 释放时机：container 被清空时
}

// 方式2：object 标签
function previewPDFWithObject(file, container) {
  const url = URL.createObjectURL(file);
  const obj = document.createElement('object');
  obj.data = url;
  obj.type = 'application/pdf';
  obj.width = '100%';
  obj.height = '100%';
  container.appendChild(obj);
}

// 方式3：pdf.js 渲染到 Canvas（精确控制）
// 需引入 pdf.js 库
async function previewPDFWithPdfJs(file, canvas) {
  const url = URL.createObjectURL(file);
  const pdf = await pdfjsLib.getDocument(url).promise;
  const page = await pdf.getPage(1);

  const viewport = page.getViewport({ scale: 1.5 });
  canvas.width = viewport.width;
  canvas.height = viewport.height;

  const ctx = canvas.getContext('2d');
  await page.render({ canvasContext: ctx, viewport }).promise;
  URL.revokeObjectURL(url);
}
```

#### 音视频预览

```js
// 音频预览
function previewAudio(file, container) {
  const url = URL.createObjectURL(file);
  const audio = document.createElement('audio');
  audio.src = url;
  audio.controls = true;
  container.appendChild(audio);
}

// 视频预览
function previewVideo(file, container) {
  const url = URL.createObjectURL(file);
  const video = document.createElement('video');
  video.src = url;
  video.controls = true;
  video.style.maxWidth = '100%';
  container.appendChild(video);
}

// 获取视频元信息
function getVideoMeta(file) {
  return new Promise((resolve) => {
    const video = document.createElement('video');
    const url = URL.createObjectURL(file);
    video.preload = 'metadata';

    video.onloadedmetadata = () => {
      URL.revokeObjectURL(url);
      resolve({
        duration: video.duration,    // 时长（秒）
        width: video.videoWidth,     // 宽度
        height: video.videoHeight    // 高度
      });
    };

    video.src = url;
  });
}
```

#### 文本 / CSV / Excel 预览

```js
// 文本文件预览
async function previewText(file, element) {
  const text = await file.text();
  element.textContent = text;
}

// CSV 预览（解析为表格）
async function previewCSV(file, container) {
  const text = await file.text();
  const rows = text.trim().split('\n').map(row => row.split(','));

  const table = document.createElement('table');
  table.border = '1';

  rows.forEach((row, i) => {
    const tr = document.createElement('tr');
    row.forEach(cell => {
      const td = document.createElement(i === 0 ? 'th' : 'td');
      td.textContent = cell.trim().replace(/^"|"$/g, '');
      tr.appendChild(td);
    });
    table.appendChild(tr);
  });

  container.appendChild(table);
}

// Excel 预览（需 SheetJS 库）
async function previewExcel(file, container) {
  const buffer = await file.arrayBuffer();
  const workbook = XLSX.read(buffer, { type: 'array' });
  const sheet = workbook.Sheets[workbook.SheetNames[0]];
  const html = XLSX.utils.sheet_to_html(sheet);
  container.innerHTML = html;
}

// Markdown 预览（需 marked 库）
async function previewMarkdown(file, element) {
  const text = await file.text();
  element.innerHTML = marked.parse(text);
}

// 代码高亮预览（需 highlight.js）
async function previewCode(file, element, language) {
  const text = await file.text();
  element.textContent = text;
  hljs.highlightElement(element);
}
```

---

### 8. 压缩与解压

#### CompressionStream / DecompressionStream（原生）

```js
// 压缩数据
async function compressData(data, format = 'gzip') {
  const stream = new Blob([data]).stream();
  const compressed = stream.pipeThrough(new CompressionStream(format));
  return await new Response(compressed).arrayBuffer();
}

// 解压数据
async function decompressData(compressedData, format = 'gzip') {
  const stream = new Blob([compressedData]).stream();
  const decompressed = stream.pipeThrough(new DecompressionStream(format));
  return await new Response(decompressed).text();
}

// 支持的格式：'gzip' | 'deflate' | 'deflate-raw'

// 使用
const original = '这是一段需要压缩的文本，包含重复内容，重复内容...'.repeat(100);
const compressed = await compressData(original, 'gzip');
console.log(`原始: ${original.length}B, 压缩后: ${compressed.byteLength}B`);
const decompressed = await decompressData(compressed, 'gzip');
console.log('解压后长度:', decompressed.length);
```

#### 压缩文件并下载

```js
async function compressAndDownload(content, filename) {
  const stream = new Blob([content]).stream();
  const compressed = stream.pipeThrough(new CompressionStream('gzip'));
  const blob = await new Response(compressed).blob();

  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename + '.gz';
  a.click();
  URL.revokeObjectURL(url);
}
```

#### JSZip 客户端压缩

```js
// JSZip —— 客户端 ZIP 压缩/解压
// <script src="https://cdn.jsdelivr.net/npm/jszip@3/dist/jszip.min.js"></script>

// 创建 ZIP
async function createZip(files) {
  const zip = new JSZip();

  for (const file of files) {
    const content = await file.arrayBuffer();
    zip.file(file.name, content);
  }

  // 也可以添加文件夹
  zip.folder('images');

  // 生成 ZIP Blob
  const blob = await zip.generateAsync(
    { type: 'blob', compression: 'DEFLATE', compressionOptions: { level: 6 } },
    (metadata) => {
      console.log(`压缩进度: ${metadata.percent.toFixed(1)}%`);
    }
  );

  return blob;
}

// 使用
const zipBlob = await createZip(fileList);
const url = URL.createObjectURL(zipBlob);
a.href = url;
a.download = 'archive.zip';
a.click();
URL.revokeObjectURL(url);

// 解压 ZIP
async function extractZip(zipFile) {
  const buffer = await zipFile.arrayBuffer();
  const zip = await JSZip.loadAsync(buffer);
  const files = [];

  for (const [path, entry] of Object.entries(zip.files)) {
    if (!entry.dir) {
      const content = await entry.async('blob');
      files.push({ path, content });
    }
  }

  return files;
}

// 使用
const extracted = await extractZip(zipFile);
for (const { path, content } of extracted) {
  console.log(`解压: ${path}, 大小: ${content.size}B`);
}
```

---

### 9. File System Access API

File System Access API 是浏览器中最强大的文件操作 API，允许 Web 应用像桌面应用一样直接读写本地文件系统。

> 兼容性：目前仅 Chromium 浏览器（Chrome 86+、Edge 86+）支持。Firefox 和 Safari 不支持。

#### showOpenFilePicker —— 选择文件

```js
// 选择单个文件
async function openFile() {
  const [handle] = await window.showOpenFilePicker({
    types: [
      {
        description: '文本文件',
        accept: {
          'text/plain': ['.txt'],
          'text/markdown': ['.md'],
          'text/csv': ['.csv']
        }
      },
      {
        description: '图片文件',
        accept: {
          'image/*': ['.png', '.jpg', '.gif', '.webp']
        }
      }
    ],
    multiple: false,     // 是否多选
    excludeAcceptAllOption: false // 是否隐藏"所有文件"选项
  });

  const file = await handle.getFile();
  console.log(`打开文件: ${file.name}, 大小: ${file.size}B`);
  const content = await file.text();
  return { handle, content };
}

// 选择多个文件
async function openMultipleFiles() {
  const handles = await window.showOpenFilePicker({ multiple: true });
  const files = await Promise.all(handles.map(h => h.getFile()));
  return files;
}
```

#### showDirectoryPicker —— 选择目录

```js
async function openDirectory() {
  const dirHandle = await window.showDirectoryPicker({ mode: 'read' });

  // 遍历目录
  for await (const entry of dirHandle.values()) {
    if (entry.kind === 'file') {
      const file = await entry.getFile();
      console.log(`文件: ${file.name} (${file.size}B)`);
    } else if (entry.kind === 'directory') {
      console.log(`目录: ${entry.name}`);
    }
  }
}

// 递归遍历目录
async function listAllFiles(dirHandle, path = '') {
  const files = [];
  for await (const entry of dirHandle.values()) {
    const entryPath = path ? `${path}/${entry.name}` : entry.name;
    if (entry.kind === 'file') {
      files.push({ path: entryPath, handle: entry });
    } else if (entry.kind === 'directory') {
      const subFiles = await listAllFiles(entry, entryPath);
      files.push(...subFiles);
    }
  }
  return files;
}
```

#### 读写本地文件

```js
// 读取文件
async function readFile(handle) {
  const file = await handle.getFile();
  return await file.text();
}

// 写入文件（创建或覆盖）
async function writeFile(handle, content) {
  const writable = await handle.createWritable();
  await writable.write(content);
  await writable.close();
}

// 追加写入
async function appendToFile(handle, content) {
  const file = await handle.getFile();
  const existing = await file.text();
  const writable = await handle.createWritable();
  await writable.write(existing + content);
  await writable.close();
}
```

#### FileSystemHandle 持久化

```js
// 将 handle 存入 IndexedDB，下次打开可直接使用
async function saveHandleToDB(key, handle) {
  const db = await openDB('fileHandles', 1, {
    upgrade(db) {
      db.createObjectStore('handles');
    }
  });
  await db.put('handles', handle, key);
}

async function getHandleFromDB(key) {
  const db = await openDB('fileHandles', 1);
  return await db.get('handles', key);
}

// 验证权限
async function verifyPermission(handle, readWrite = false) {
  const options = { mode: readWrite ? 'readwrite' : 'read' };

  // 已有权限
  if ((await handle.queryPermission(options)) === 'granted') {
    return true;
  }

  // 请求权限
  if ((await handle.requestPermission(options)) === 'granted') {
    return true;
  }

  return false;
}

// 使用：打开上次编辑的文件
async function reopenLastFile() {
  const handle = await getHandleFromDB('lastFile');
  if (handle) {
    const hasPermission = await verifyPermission(handle, true);
    if (hasPermission) {
      const content = await readFile(handle);
      return { handle, content };
    }
  }
  // 无权限或无记录，重新选择
  return openFile();
}
```

#### 创建新文件

```js
// 保存新文件
async function saveNewFile(content, suggestedName = 'untitled.txt') {
  const handle = await window.showSaveFilePicker({
    suggestedName,
    types: [
      {
        description: '文本文件',
        accept: { 'text/plain': ['.txt'] }
      },
      {
        description: 'JSON 文件',
        accept: { 'application/json': ['.json'] }
      }
    ]
  });

  const writable = await handle.createWritable();
  await writable.write(content);
  await writable.close();

  return handle;
}
```

#### 编辑器场景完整示例

```js
class LocalFileEditor {
  constructor() {
    this.currentHandle = null;
    this.isModified = false;
  }

  async open() {
    const [handle] = await window.showOpenFilePicker({
      types: [{
        description: '文本文件',
        accept: { 'text/plain': ['.txt', '.md', '.js', '.css', '.html'] }
      }]
    });

    const file = await handle.getFile();
    const content = await file.text();

    this.currentHandle = handle;
    this.isModified = false;

    return { name: file.name, content };
  }

  async save(content) {
    if (!this.currentHandle) {
      return this.saveAs(content);
    }

    const writable = await this.currentHandle.createWritable();
    await writable.write(content);
    await writable.close();
    this.isModified = false;
  }

  async saveAs(content) {
    const handle = await window.showSaveFilePicker({
      suggestedName: 'untitled.txt'
    });

    const writable = await handle.createWritable();
    await writable.write(content);
    await writable.close();

    this.currentHandle = handle;
    this.isModified = false;
  }
}

// 使用
const editor = new LocalFileEditor();

document.getElementById('openBtn').addEventListener('click', async () => {
  const { name, content } = await editor.open();
  document.getElementById('editor').value = content;
  document.title = name;
});

document.getElementById('saveBtn').addEventListener('click', async () => {
  const content = document.getElementById('editor').value;
  await editor.save(content);
});
```

---

### 10. Clipboard API 文件操作

#### 复制图片到剪贴板

```js
// 复制 Canvas 图片到剪贴板
async function copyCanvasToClipboard(canvas) {
  const blob = await new Promise(resolve => canvas.toBlob(resolve, 'image/png'));
  await navigator.clipboard.write([
    new ClipboardItem({ 'image/png': blob })
  ]);
  console.log('图片已复制到剪贴板');
}

// 复制文件图片到剪贴板
async function copyImageFileToClipboard(file) {
  // ClipboardItem 要求类型与文件类型匹配
  const blob = new Blob([await file.arrayBuffer()], { type: file.type });
  await navigator.clipboard.write([
    new ClipboardItem({ [file.type]: blob })
  ]);
}

// 从 DOM 图片复制
async function copyDomImageToClipboard(imgElement) {
  const canvas = document.createElement('canvas');
  canvas.width = imgElement.naturalWidth;
  canvas.height = imgElement.naturalHeight;
  canvas.getContext('2d').drawImage(imgElement, 0, 0);
  await copyCanvasToClipboard(canvas);
}
```

#### 从剪贴板读取图片

```js
// 读取剪贴板中的图片
async function readImageFromClipboard() {
  const items = await navigator.clipboard.read();
  for (const item of items) {
    const imageType = item.types.find(t => t.startsWith('image/'));
    if (imageType) {
      const blob = await item.getType(imageType);
      const url = URL.createObjectURL(blob);
      return url;
    }
  }
  return null;
}

// 监听粘贴事件获取图片
document.addEventListener('paste', async (e) => {
  const items = e.clipboardData?.items;
  if (!items) return;

  for (const item of items) {
    if (item.type.startsWith('image/')) {
      const file = item.getAsFile();
      const url = URL.createObjectURL(file);
      imgElement.src = url;
      break;
    }
  }
});
```

#### ClipboardItem 详解

```js
// ClipboardItem 支持的类型
const item = new ClipboardItem({
  'text/plain': new Blob(['Hello'], { type: 'text/plain' }),
  'image/png': pngBlob,
  'text/html': new Blob(['<b>Bold</b>'], { type: 'text/html' })
});

// 注意事项：
// 1. ClipboardItem 的值必须是 Blob
// 2. 写入剪贴板时，至少需要一种 MIME 类型
// 3. 图片类型必须是浏览器支持的格式（PNG 最安全）
// 4. 需要用户交互触发（如点击事件）
// 5. 需要HTTPS或localhost环境
```

---

### 11. 文件拖拽

#### DataTransfer.files

```js
const dropZone = document.getElementById('drop-zone');

// dragover 必须阻止默认行为
dropZone.addEventListener('dragover', (e) => {
  e.preventDefault();
  e.dataTransfer.dropEffect = 'copy';
});

dropZone.addEventListener('drop', (e) => {
  e.preventDefault();

  // e.dataTransfer.files —— FileList 对象
  const files = e.dataTransfer.files;
  for (const file of files) {
    console.log(`${file.name} (${file.type}, ${file.size}B)`);
  }
});
```

#### DataTransferItem 与 getAsFile

```js
dropZone.addEventListener('drop', (e) => {
  e.preventDefault();

  // 方式1：直接用 files（简单场景）
  const files = e.dataTransfer.files;

  // 方式2：用 items + getAsFile（高级场景）
  const items = e.dataTransfer.items;
  for (const item of items) {
    if (item.kind === 'file') {
      const file = item.getAsFile();
      console.log(file.name);
    }

    // DataTransferItem 还支持字符串类型
    if (item.kind === 'string' && item.type === 'text/plain') {
      item.getAsString((text) => {
        console.log('拖拽文本:', text);
      });
    }
  }
});
```

#### 拖拽上传文件夹

```js
dropZone.addEventListener('drop', async (e) => {
  e.preventDefault();

  const items = e.dataTransfer.items;
  const allFiles = [];

  for (const item of items) {
    // webkitGetAsEntry —— 获取文件系统入口（支持目录）
    const entry = item.webkitGetAsEntry();
    if (entry) {
      await traverseEntry(entry, allFiles, '');
    }
  }

  console.log('所有文件:', allFiles);
});

// 递归遍历目录
async function traverseEntry(entry, results, path) {
  if (entry.isFile) {
    const file = await new Promise(resolve => entry.file(resolve));
    results.push({ file, path: path + file.name });
  } else if (entry.isDirectory) {
    const reader = entry.createReader();
    const entries = await new Promise(resolve => {
      const allEntries = [];
      const readBatch = () => {
        reader.readEntries((batch) => {
          if (batch.length === 0) {
            resolve(allEntries);
          } else {
            allEntries.push(...batch);
            readBatch(); // readEntries 一次最多100条，需循环读取
          }
        });
      };
      readBatch();
    });

    for (const subEntry of entries) {
      await traverseEntry(subEntry, results, path + entry.name + '/');
    }
  }
}
```

#### 拖拽排序 + 文件上传组合

```js
// 拖拽文件上传 + 拖拽排序文件列表
const dropZone = document.getElementById('drop-zone');
const fileList = document.getElementById('file-list');
let uploadedFiles = [];

// 拖入文件上传
dropZone.addEventListener('drop', async (e) => {
  e.preventDefault();
  const files = Array.from(e.dataTransfer.files);
  uploadedFiles.push(...files);
  renderFileList();
});

// 拖拽排序文件列表
function renderFileList() {
  fileList.innerHTML = '';
  uploadedFiles.forEach((file, index) => {
    const item = document.createElement('div');
    item.draggable = true;
    item.dataset.index = index;
    item.textContent = file.name;

    item.addEventListener('dragstart', (e) => {
      e.dataTransfer.setData('text/plain', index);
      e.dataTransfer.effectAllowed = 'move';
    });

    item.addEventListener('dragover', (e) => {
      e.preventDefault();
      e.dataTransfer.dropEffect = 'move';
    });

    item.addEventListener('drop', (e) => {
      e.preventDefault();
      const fromIndex = +e.dataTransfer.getData('text/plain');
      const toIndex = index;
      // 交换位置
      const [moved] = uploadedFiles.splice(fromIndex, 1);
      uploadedFiles.splice(toIndex, 0, moved);
      renderFileList();
    });

    fileList.appendChild(item);
  });
}
```

#### 页面外拖入文件与内部拖拽区分

```js
let dragCounter = 0;

dropZone.addEventListener('dragenter', (e) => {
  e.preventDefault();
  dragCounter++;

  // 判断是外部文件拖入还是内部元素拖拽
  if (e.dataTransfer.types.includes('Files')) {
    dropZone.classList.add('file-drag-over');
  }
});

dropZone.addEventListener('dragleave', (e) => {
  dragCounter--;
  if (dragCounter === 0) {
    dropZone.classList.remove('file-drag-over');
  }
});

dropZone.addEventListener('drop', (e) => {
  e.preventDefault();
  dragCounter = 0;
  dropZone.classList.remove('file-drag-over');

  if (e.dataTransfer.files.length > 0) {
    // 外部文件拖入 —— 上传
    handleFiles(e.dataTransfer.files);
  } else if (e.dataTransfer.getData('text/plain')) {
    // 内部元素拖拽 —— 排序等
    handleInternalDrop(e);
  }
});
```

---

## 常见问题

### 1. createObjectURL 内存泄漏

`URL.createObjectURL` 创建的 URL 会持有对 Blob 的引用，即使 Blob 变量已离开作用域，Blob 数据也不会被 GC 回收，直到调用 `revokeObjectURL` 或页面卸载。

```js
// 典型泄漏场景
function leakyPreview(files) {
  files.forEach(file => {
    const url = URL.createObjectURL(file);
    const img = document.createElement('img');
    img.src = url;
    gallery.appendChild(img);
    // 忘记 revokeObjectURL，每次调用累积泄漏
  });
}

// 修复方案
function safePreview(files) {
  files.forEach(file => {
    const url = URL.createObjectURL(file);
    const img = document.createElement('img');
    img.src = url;
    img.onload = () => URL.revokeObjectURL(url); // 加载完立即释放
    gallery.appendChild(img);
  });
}

// React 组件卸载时统一释放
useEffect(() => {
  const urls = files.map(f => URL.createObjectURL(f));
  setPreviewUrls(urls);
  return () => urls.forEach(url => URL.revokeObjectURL(url));
}, [files]);
```

### 2. FileReader vs Blob 方法选择

| 场景 | 选择 | 理由 |
|------|------|------|
| 简单读取文本 | `file.text()` | 代码更简洁，Promise 风格 |
| 简单读取二进制 | `file.arrayBuffer()` | 代码更简洁 |
| 需要进度监听 | `FileReader` | 唯一支持 `onprogress` |
| 需要读取为 Data URL | `FileReader.readAsDataURL()` | Blob 方法无直接替代 |
| 需要指定编码 | `FileReader.readAsText(file, 'GBK')` | `file.text()` 仅支持 UTF-8 |
| 流式处理 | `file.stream()` | 内存友好 |
| 兼容 IE | `FileReader` | Blob 方法 IE 不支持 |

### 3. iOS 文件选择限制

iOS Safari 对文件选择有特殊限制：

- `<input type="file">` 只能选择照片/视频，无法选择任意文件（iOS 15+ 改善）
- `capture` 属性在 iOS 上行为可能不一致
- 多文件选择 `multiple` 在部分 iOS 版本不稳定
- File System Access API 在 Safari 上不支持
- 拖拽上传在移动端不可用（无拖拽操作）
- 文件大小限制：部分 iOS 版本对单次选择的文件有大小上限

```js
// iOS 兼容处理
const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);

if (isIOS) {
  // iOS 上使用更宽松的 accept
  input.accept = 'image/*,video/*,.pdf,.txt,.doc,.docx';
  // 避免 multiple 在 iOS 上的 bug
  // input.removeAttribute('multiple');
}
```

### 4. File System Access API 兼容性

| 浏览器 | 支持情况 |
|--------|---------|
| Chrome 86+ | 完全支持 |
| Edge 86+ | 完全支持 |
| Opera 72+ | 完全支持 |
| Firefox | 不支持 |
| Safari | 不支持（部分 Origin Private FS） |
| 移动端 | 几乎不支持 |

```js
// 兼容性检测
function isFileSystemAccessSupported() {
  return 'showOpenFilePicker' in window;
}

// 降级方案
async function openFileSafe() {
  if (isFileSystemAccessSupported()) {
    // 使用 File System Access API
    const [handle] = await window.showOpenFilePicker();
    return handle.getFile();
  } else {
    // 降级到 input[type=file]
    return new Promise((resolve) => {
      const input = document.createElement('input');
      input.type = 'file';
      input.onchange = () => resolve(input.files[0]);
      input.click();
    });
  }
}
```

### 5. 大文件读取导致页面卡顿

直接对大文件调用 `file.text()` 或 `file.arrayBuffer()` 会一次性将全部内容加载到内存，可能导致页面卡顿甚至崩溃。

```js
// 错误：直接读取大文件
const text = await largeFile.text(); // 1GB 文件 -> 1GB 内存

// 正确：流式读取
const reader = largeFile.stream().getReader();
while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  processChunk(value); // 逐块处理，内存占用恒定
}

// 正确：分片读取
const CHUNK = 1024 * 1024; // 1MB
for (let offset = 0; offset < largeFile.size; offset += CHUNK) {
  const chunk = largeFile.slice(offset, Math.min(offset + CHUNK, largeFile.size));
  const buffer = await chunk.arrayBuffer();
  processChunk(buffer);
}
```

---

## 面试题

### 1. File 和 Blob 有什么区别？

**答：**

File 继承自 Blob，是带有文件元信息的 Blob。两者的核心区别：

| 维度 | Blob | File |
|------|------|------|
| 继承关系 | 基类 | 继承 Blob |
| name 属性 | 无 | 有（文件名） |
| lastModified 属性 | 无 | 有（最后修改时间戳） |
| 创建方式 | `new Blob([data], {type})` | `<input>` / 拖拽 / `new File([data], name, options)` |
| 共有属性 | size, type | size, type（继承） |
| 共有方法 | slice(), text(), arrayBuffer(), stream() | 同左（继承） |

File 是 Blob 的特例，任何接受 Blob 的 API 都能接受 File。File 通常来源于用户操作（input 选择、拖拽），而 Blob 可由代码手动创建。需要文件名等元信息时用 File，纯二进制数据处理用 Blob。

---

### 2. FileReader 的事件模型是怎样的？

**答：**

FileReader 基于事件回调模型，生命周期事件依次为：

```
onloadstart -> onprogress(多次) -> onload / onerror -> onloadend
```

| 事件 | 触发时机 | 用途 |
|------|---------|------|
| `loadstart` | 开始读取 | 初始化进度条 |
| `progress` | 读取过程中多次触发 | 更新进度（`e.loaded / e.total`） |
| `load` | 读取成功 | 获取结果 `reader.result` |
| `error` | 读取失败 | 获取错误 `reader.error` |
| `abort` | 读取中止 | 处理中止逻辑 |
| `loadend` | 读取结束（无论成败） | 清理工作 |

关键点：
- `onload` 和 `onerror` 互斥，只会触发一个
- `onloadend` 必定触发，类似 `finally`
- `onprogress` 的 `e.lengthComputable` 为 true 时才能计算百分比
- FileReader 不能复用，读取第二个文件需创建新实例
- 可调用 `reader.abort()` 中止读取，触发 `onabort`

现代替代方案：`blob.text()` / `blob.arrayBuffer()` 更简洁，但缺少进度监听和 Data URL 读取。

---

### 3. createObjectURL 有哪些注意事项？

**答：**

三个核心注意事项：

1. **内存泄漏**：`createObjectURL` 创建的 URL 持有 Blob 引用，Blob 数据不会被 GC 回收，直到调用 `revokeObjectURL` 或页面卸载。大量创建不释放会导致内存持续增长。

2. **同源策略**：`blob:` URL 与创建它的页面同源，可用于 `<img>`、`<video>`、`<iframe>` 等标签，但不能在跨域 iframe 中直接使用。

3. **生命周期**：`blob:` URL 在页面关闭前一直有效，即使原始 Blob 变量已被回收。最佳实践是在资源加载完成后立即调用 `revokeObjectURL`：

```js
const url = URL.createObjectURL(file);
img.src = url;
img.onload = () => URL.revokeObjectURL(url); // 加载完即释放
```

额外注意：每个 `blob:` URL 是唯一的，不能跨 Document 使用（如 `window.open` 的新窗口）。`revokeObjectURL` 后，所有引用该 URL 的元素都会失效。

---

### 4. 大文件上传方案如何设计？

**答：**

大文件上传核心方案：**分片上传 + 断点续传 + 秒传**。

1. **分片上传**：将文件按固定大小（如 5MB）切割为多个分片，逐个上传。使用 `file.slice(start, end)` 切片，配合并发控制（3-5 个并发）提高上传速度。

2. **断点续传**：上传前计算文件哈希（SHA-256），向服务器查询已上传的分片列表，跳过已上传的分片继续上传。页面刷新或网络中断后可恢复。

3. **秒传**：根据文件哈希判断服务器是否已存在相同文件，如存在则直接返回成功，无需重复上传。

4. **合并**：所有分片上传完成后，通知服务器按序合并为完整文件。

5. **哈希优化**：大文件全量计算 SHA-256 耗时，采用抽样哈希（每段取头尾各 2KB）或 Web Worker 后台计算避免阻塞主线程。

关键实现细节：
- 分片大小根据网络情况动态调整
- 失败分片自动重试（指数退避）
- 并发数控制（避免过多连接）
- 上传进度实时反馈
- 暂停/恢复功能

---

### 5. File System Access API 的用途和限制？

**答：**

**用途：**

- 本地文件编辑器（如 VS Code for Web）
- 本地文件管理器
- 直接读写本地项目文件（无需上传/下载）
- 持久化文件句柄，下次打开自动恢复

**核心 API：**

| API | 功能 |
|-----|------|
| `showOpenFilePicker()` | 选择文件 |
| `showSaveFilePicker()` | 保存新文件 |
| `showDirectoryPicker()` | 选择目录 |
| `FileSystemFileHandle.getFile()` | 读取文件 |
| `FileSystemFileHandle.createWritable()` | 写入文件 |
| `FileSystemDirectoryHandle.values()` | 遍历目录 |

**限制：**

1. **兼容性差**：仅 Chromium 浏览器支持，Firefox/Safari 不支持
2. **安全限制**：必须由用户手势触发（点击事件），不能自动弹出文件选择器
3. **权限模型**：每次页面加载后首次使用需重新授权，但可存入 IndexedDB 后请求权限
4. **不能访问任意路径**：只能访问用户主动选择的文件/目录
5. **无法静默读写**：所有操作都需用户确认

---

### 6. 拖拽上传如何实现？

**答：**

拖拽上传涉及四个拖拽事件：

1. **dragenter**：文件拖入目标区域，添加视觉反馈样式
2. **dragover**：文件在目标区域上方悬停，**必须 `e.preventDefault()`** 阻止默认行为（否则浏览器会打开文件）
3. **dragleave**：文件离开目标区域，移除视觉反馈
4. **drop**：文件放下，从 `e.dataTransfer.files` 获取文件列表

关键实现要点：

```js
dropZone.addEventListener('dragover', (e) => {
  e.preventDefault();          // 必须！否则 drop 不会触发
  e.dataTransfer.dropEffect = 'copy';
});

dropZone.addEventListener('drop', (e) => {
  e.preventDefault();          // 必须！阻止浏览器打开文件
  const files = e.dataTransfer.files;
  // 上传处理...
});
```

- `dragenter`/`dragleave` 在拖入子元素时会反复触发，需用计数器或 `contains` 判断处理
- 移动端无拖拽操作，需同时提供 `<input type="file">` 作为替代
- 拖拽文件夹需使用 `item.webkitGetAsEntry()` 获取 `FileSystemEntry`，递归遍历目录

---

### 7. Blob.slice 的用途有哪些？

**答：**

`Blob.slice(start, end, contentType)` 用于切割 Blob，返回新的 Blob，不复制原始数据（写时复制）。主要用途：

1. **大文件分片上传**：将大文件按固定大小切片，逐片上传，支持断点续传
2. **分片读取**：避免一次性加载大文件到内存，逐片 `arrayBuffer()` 读取
3. **文件截取**：只取文件的部分内容（如读取文件头判断类型）
4. **修改 MIME 类型**：`blob.slice(0, blob.size, 'application/json')` 创建同内容但不同类型的 Blob

```js
// 读取文件头判断真实类型
async function detectFileType(file) {
  const header = file.slice(0, 8);
  const buffer = await header.arrayBuffer();
  const view = new Uint8Array(buffer);

  // PNG: 89 50 4E 47
  if (view[0] === 0x89 && view[1] === 0x50) return 'image/png';
  // JPEG: FF D8 FF
  if (view[0] === 0xFF && view[1] === 0xD8) return 'image/jpeg';
  // PDF: 25 50 44 46
  if (view[0] === 0x25 && view[1] === 0x50) return 'application/pdf';

  return file.type || 'unknown';
}
```

注意：`slice` 是浅拷贝，不复制底层数据，性能开销极小。

---

### 8. 前端文件预览有哪些方案？

**答：**

| 文件类型 | 预览方案 | 说明 |
|---------|---------|------|
| 图片 | `URL.createObjectURL` + `<img>` | 最简单，支持缩略图、压缩 |
| 图片 | Canvas 绘制 + 压缩 | 可裁剪、旋转、压缩 |
| PDF | `<iframe src="blob:...">` | 浏览器内置 PDF 查看器 |
| PDF | `<object data="blob:...">` | 类似 iframe |
| PDF | pdf.js 渲染到 Canvas | 精确控制、自定义 UI |
| 视频 | `URL.createObjectURL` + `<video>` | 浏览器原生播放 |
| 音频 | `URL.createObjectURL` + `<audio>` | 浏览器原生播放 |
| 文本 | `file.text()` + `<pre>` | 直接展示 |
| CSV | 解析为 HTML 表格 | 简单方案 |
| CSV | PapaParse 库 | 专业 CSV 解析 |
| Excel | SheetJS (XLSX) | 解析为 JSON 或 HTML |
| Word | mammoth.js | docx 转 HTML |
| Markdown | marked.js | 转为 HTML 渲染 |
| 代码 | highlight.js / Prism.js | 语法高亮 |

核心原则：
- 优先用 `createObjectURL` 做预览（性能好、内存占用低）
- 用完立即 `revokeObjectURL` 防止内存泄漏
- 大文件用流式读取，避免一次性加载
- 复杂格式（Excel/Word/PDF）需引入专业库

相关链接：[[Fetch API与请求模式]] [[Canvas与SVG]] [[WebGL与WebGPU]]
