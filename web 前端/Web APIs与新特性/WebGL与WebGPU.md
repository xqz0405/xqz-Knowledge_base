---
tags:
  - Web前端
  - WebGL
  - WebGPU
  - 3D
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# WebGL 与 WebGPU

## What — 是什么

> WebGL 与 WebGPU 是浏览器中用于 **GPU 加速图形渲染** 和 **通用计算** 的底层 Web API，让网页直接调用 GPU 硬件能力。

### 核心概念

- **WebGL**：基于 **OpenGL ES 2.0/3.0** 的 Web 图形标准，通过 Canvas 元素提供 **2D/3D 渲染**能力，使用 **GLSL**（OpenGL Shading Language）编写着色器
- **WebGPU**：下一代 Web 图形 API，基于 **Vulkan / Metal / Direct3D 12** 等现代图形 API 设计，支持图形渲染与 **通用计算（Compute Shader）**，使用 **WGSL**（WebGPU Shading Language）编写着色器
- **着色器（Shader）**：运行在 GPU 上的小程序，负责处理顶点变换、片元着色、通用并行计算等任务
- **渲染管线（Render Pipeline）**：GPU 将 3D 数据转化为屏幕像素的一系列处理阶段，包括顶点装配、光栅化、片元处理等
- **命令缓冲（Command Buffer）**：WebGPU 中预先录制 GPU 指令，再一次性提交执行的机制，减少 CPU-GPU 通信开销

### GPU 渲染管线阶段

```
应用层（CPU） → 顶点着色器 → 图元装配 → 几何着色器（可选）→ 光栅化 → 片元着色器 → 逐片元操作 → 帧缓冲
     │                                                                              │
     └── 顶点数据、Uniform、纹理 ──────────────────────────────────────── 输出到屏幕 ──┘
```

1. **顶点着色器（Vertex Shader）**：逐顶点执行，完成坐标变换（模型→世界→观察→裁剪空间）
2. **图元装配（Primitive Assembly）**：将顶点组装为点、线、三角形等图元
3. **光栅化（Rasterization）**：将几何图元转换为片元（候选像素）
4. **片元着色器（Fragment Shader）**：逐片元计算颜色，支持纹理采样与光照计算
5. **逐片元操作**：深度测试、模板测试、混合（Blending），最终写入帧缓冲

### WebGL vs WebGPU 架构对比

| 维度 | WebGL | WebGPU |
|------|-------|--------|
| 底层 API | OpenGL ES | Vulkan / Metal / D3D12 |
| 着色器语言 | GLSL | WGSL |
| 管线模型 | 状态机（全局状态切换） | 对象化（显式管线创建） |
| 命令提交 | 即时模式（调用即执行） | 延迟模式（命令缓冲+提交） |
| 计算着色器 | 不支持 | 原生支持 |
| 错误处理 | 运行时 `gl.getError()` 轮询 | 编译时验证 + 结构化错误信息 |
| 多线程 | 单线程 | 支持 Worker 中录制命令 |

---

## Why — 为什么

### 使用场景

- **3D 可视化**：工业数字孪生、建筑 BIM、科学数据三维展示
- **网页游戏**：3D 射击、RPG、休闲游戏
- **数据可视化**：大规模粒子系统、流场可视化、地理信息三维呈现
- **AR / VR**：WebXR 配合 WebGPU 实现沉浸式体验
- **通用 GPU 计算**：机器学习推理、图像处理、物理模拟、密码学运算
- **视频/图像处理**：GPU 加速滤镜、实时视频特效

### 技术对比

| 特性 | Canvas 2D | SVG | CSS 3D | WebGL | WebGPU |
|------|-----------|-----|--------|-------|--------|
| 渲染方式 | CPU 逐像素 | DOM 矢量 | CPU 合成 | GPU 着色器 | GPU 着色器+计算 |
| 3D 支持 | 否 | 否 | 有限（变换） | 完整 | 完整 |
| 性能 | 中 | 低（DOM多时） | 中 | 高 | 极高 |
| 计算着色器 | 否 | 否 | 否 | 否 | 是 |
| 学习曲线 | 低 | 低 | 低 | 高 | 高 |
| 浏览器支持 | 全部 | 全部 | 全部 | 广泛 | Chrome 113+ / Firefox 实验性 |
| 适用规模 | 简单图形 | 图标/图表 | 简单3D变换 | 中大型3D场景 | 大型3D + GPGPU |
| 生态成熟度 | 高 | 高 | 高 | 高（Three.js等） | 快速增长中 |

### 优势与劣势

**WebGL 优势**
- 浏览器兼容性好，几乎所有现代浏览器支持
- 生态成熟，Three.js / Babylon.js 等引擎完善
- 社区资源丰富，教程和示例海量

**WebGL 劣势**
- 基于过时的 OpenGL ES，无法利用现代 GPU 特性
- 全局状态机模型，易产生状态泄漏 Bug
- 不支持计算着色器，GPGPU 需要黑科技绕路
- 错误检测困难，`gl.getError()` 轮询低效

**WebGPU 优势**
- 现代架构，更贴近 GPU 硬件，性能上限更高
- 显式管线与命令缓冲，减少状态切换开销
- 原生计算着色器，直接支持 GPGPU
- 编译时验证，错误信息清晰
- 支持 Worker 多线程录制命令

**WebGPU 劣势**
- 浏览器支持尚不完善（Safari 2024 才开始支持）
- 学习曲线更陡峭，概念更多
- 生态初期，引擎适配进行中
- WGSL 学习资源相对匮乏

---

## How — 怎么用

### WebGL 基础流程

```
1. 获取 Canvas 元素
2. 获取 WebGL 上下文（webgl / webgl2）
3. 编写顶点着色器与片元着色器（GLSL）
4. 编译着色器 → 链接程序
5. 创建缓冲区，上传顶点数据
6. 设置 Uniform 变量
7. 清屏 → 绘制（drawArrays / drawElements）
```

### WebGL 示例：绘制三角形

```javascript
// 1. 获取 Canvas 与 WebGL 上下文
const canvas = document.getElementById('glCanvas');
const gl = canvas.getContext('webgl2') || canvas.getContext('webgl');

if (!gl) {
  console.error('WebGL 不可用');
}

// 2. 顶点着色器（GLSL）—— 将顶点坐标传递到裁剪空间
const vertexShaderSource = `#version 300 es
  in vec2 aPosition;          // 顶点位置属性
  in vec3 aColor;             // 顶点颜色属性
  out vec3 vColor;            // 传递给片元着色器的插值颜色

  void main() {
    gl_Position = vec4(aPosition, 0.0, 1.0);
    vColor = aColor;
  }
`;

// 3. 片元着色器（GLSL）—— 决定每个像素的颜色
const fragmentShaderSource = `#version 300 es
  precision highp float;
  in vec3 vColor;             // 从顶点着色器插值而来的颜色
  out vec4 fragColor;

  void main() {
    fragColor = vec4(vColor, 1.0);
  }
`;

// 4. 编译着色器工具函数
function compileShader(gl, source, type) {
  const shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    console.error('着色器编译失败:', gl.getShaderInfoLog(shader));
    gl.deleteShader(shader);
    return null;
  }
  return shader;
}

const vertexShader = compileShader(gl, vertexShaderSource, gl.VERTEX_SHADER);
const fragmentShader = compileShader(gl, fragmentShaderSource, gl.FRAGMENT_SHADER);

// 5. 链接着色器程序
const program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
gl.linkProgram(program);

if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
  console.error('程序链接失败:', gl.getProgramInfoLog(program));
}

gl.useProgram(program);

// 6. 创建顶点数据（位置 + 颜色交错存储）
// 三个顶点：左下(红)、右下(绿)、顶部(蓝)
const vertices = new Float32Array([
  // x,    y,    r,   g,   b
   -0.5, -0.5,  1.0, 0.0, 0.0,  // 左下 - 红色
    0.5, -0.5,  0.0, 1.0, 0.0,  // 右下 - 绿色
    0.0,  0.5,  0.0, 0.0, 1.0,  // 顶部 - 蓝色
]);

// 7. 创建 VAO（WebGL2）和 VBO
const vao = gl.createVertexArray();
gl.bindVertexArray(vao);

const vbo = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, vbo);
gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);

// 8. 设置顶点属性指针
const aPosition = gl.getAttribLocation(program, 'aPosition');
gl.enableVertexAttribArray(aPosition);
gl.vertexAttribPointer(aPosition, 2, gl.FLOAT, false, 5 * 4, 0); // stride=20, offset=0

const aColor = gl.getAttribLocation(program, 'aColor');
gl.enableVertexAttribArray(aColor);
gl.vertexAttribPointer(aColor, 3, gl.FLOAT, false, 5 * 4, 2 * 4); // stride=20, offset=8

// 9. 渲染循环
function render() {
  gl.clearColor(0.1, 0.1, 0.15, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.TRIANGLES, 0, 3);
  requestAnimationFrame(render);
}
render();
```

### Three.js 快速示例（最实用的方式）

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// 1. 创建场景
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a2e);

// 2. 创建相机
const camera = new THREE.PerspectiveCamera(
  75,                                        // FOV
  window.innerWidth / window.innerHeight,     // 宽高比
  0.1,                                       // 近裁剪面
  1000                                       // 远裁剪面
);
camera.position.set(0, 2, 5);

// 3. 创建渲染器（内部使用 WebGL）
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

// 4. 添加轨道控制器（鼠标旋转/缩放/平移）
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

// 5. 创建几何体 + 材质 → 网格
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshStandardMaterial({ color: 0x00ff88 });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// 6. 添加光源
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1.0);
directionalLight.position.set(5, 5, 5);
scene.add(directionalLight);

// 7. 动画循环
function animate() {
  requestAnimationFrame(animate);
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  controls.update();
  renderer.render(scene, camera);
}
animate();

// 8. 响应窗口大小变化
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

### WebGPU 基础流程

```
1. 获取 Canvas 元素
2. 请求 GPU Adapter（适配器 = 物理显卡）
3. 请求 GPU Device（设备 = 逻辑显卡实例）
4. 配置 Canvas 上下文
5. 编写着色器（WGSL）
6. 创建渲染管线
7. 每帧：编码命令 → 提交命令缓冲
```

### WebGPU 示例：绘制三角形

```javascript
async function initWebGPU() {
  // 1. 检查浏览器支持
  if (!navigator.gpu) {
    console.error('WebGPU 不受此浏览器支持');
    return;
  }

  const canvas = document.getElementById('gpuCanvas');
  canvas.width = 800;
  canvas.height = 600;

  // 2. 获取 Adapter 和 Device
  const adapter = await navigator.gpu.requestAdapter({
    powerPreference: 'high-performance',
  });
  if (!adapter) {
    console.error('无法获取 GPU 适配器');
    return;
  }

  const device = await adapter.requestDevice();
  device.lost.then((info) => {
    console.error('WebGPU 设备丢失:', info.message);
  });

  // 3. 配置 Canvas 上下文
  const context = canvas.getContext('webgpu');
  const canvasFormat = navigator.gpu.getPreferredCanvasFormat();
  context.configure({
    device,
    format: canvasFormat,
    alphaMode: 'premultiplied',
  });

  // 4. WGSL 着色器
  const shaderCode = `
    // 顶点着色器入口
    @vertex
    fn vs_main(@builtin(vertex_index) vertexIndex: u32) -> @builtin(position) vec4<f32> {
      // 三个顶点的位置
      var pos = array<vec2<f32>, 3>(
        vec2<f32>(-0.5, -0.5),  // 左下
        vec2<f32>( 0.5, -0.5),  // 右下
        vec2<f32>( 0.0,  0.5),  // 顶部
      );
      return vec4<f32>(pos[vertexIndex], 0.0, 1.0);
    }

    // 片元着色器入口
    @fragment
    fn fs_main() -> @location(0) vec4<f32> {
      return vec4<f32>(0.0, 1.0, 0.53, 1.0); // 绿色
    }
  `;

  const shaderModule = device.createShaderModule({ code: shaderCode });

  // 5. 创建渲染管线
  const pipeline = device.createRenderPipeline({
    layout: 'auto',
    vertex: {
      module: shaderModule,
      entryPoint: 'vs_main',
    },
    fragment: {
      module: shaderModule,
      entryPoint: 'fs_main',
      targets: [{ format: canvasFormat }],
    },
    primitive: {
      topology: 'triangle-list',
    },
  });

  // 6. 渲染循环
  function frame() {
    // 编码命令
    const commandEncoder = device.createCommandEncoder();
    const textureView = context.getCurrentTexture().createView();

    const renderPassDescriptor = {
      colorAttachments: [
        {
          view: textureView,
          clearValue: { r: 0.1, g: 0.1, b: 0.15, a: 1.0 },
          loadOp: 'clear',
          storeOp: 'store',
        },
      ],
    };

    const passEncoder = commandEncoder.beginRenderPass(renderPassDescriptor);
    passEncoder.setPipeline(pipeline);
    passEncoder.draw(3, 1, 0, 0); // 3个顶点, 1个实例
    passEncoder.end();

    // 提交命令缓冲
    device.queue.submit([commandEncoder.finish()]);
    requestAnimationFrame(frame);
  }

  requestAnimationFrame(frame);
}

initWebGPU();
```

### WebGPU 计算着色器示例（WebGPU 独有特性）

```javascript
async function computeShaderDemo() {
  if (!navigator.gpu) return;

  const adapter = await navigator.gpu.requestAdapter();
  const device = await adapter.requestDevice();

  // 输入数据
  const input = new Float32Array([1, 2, 3, 4, 5, 6, 7, 8]);
  const bufferSize = input.byteLength;

  // 创建 GPU 缓冲区
  const gpuBuffer = device.createBuffer({
    size: bufferSize,
    usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_SRC,
  });

  // 暂存缓冲区（用于回读结果到 CPU）
  const stagingBuffer = device.createBuffer({
    size: bufferSize,
    usage: GPUBufferUsage.MAP_READ | GPUBufferUsage.COPY_DST,
  });

  // 上传输入数据
  device.queue.writeBuffer(gpuBuffer, 0, input);

  // 计算着色器：对每个元素求平方
  const shaderCode = `
    @group(0) @binding(0) var<storage, read_write> data: array<f32>;

    @compute @workgroup_size(8)
    fn main(@builtin(global_invocation_id) id: vec3<u32>) {
      let i = id.x;
      data[i] = data[i] * data[i]; // 平方运算
    }
  `;

  const shaderModule = device.createShaderModule({ code: shaderCode });

  // 创建计算管线
  const computePipeline = device.createComputePipeline({
    layout: 'auto',
    compute: {
      module: shaderModule,
      entryPoint: 'main',
    },
  });

  // 创建绑定组
  const bindGroup = device.createBindGroup({
    layout: computePipeline.getBindGroupLayout(0),
    entries: [{ binding: 0, resource: { buffer: gpuBuffer } }],
  });

  // 编码并提交命令
  const commandEncoder = device.createCommandEncoder();
  const passEncoder = commandEncoder.beginComputePass();
  passEncoder.setPipeline(computePipeline);
  passEncoder.setBindGroup(0, bindGroup);
  passEncoder.dispatchWorkgroups(1); // 1 个工作组，每组 8 个线程
  passEncoder.end();

  // 复制结果到暂存缓冲区
  commandEncoder.copyBufferToBuffer(gpuBuffer, 0, stagingBuffer, 0, bufferSize);

  device.queue.submit([commandEncoder.finish()]);

  // 回读结果
  await stagingBuffer.mapAsync(GPUMapMode.READ);
  const result = new Float32Array(stagingBuffer.getMappedRange().slice(0));
  stagingBuffer.unmap();

  console.log('输入:', input);   // [1, 2, 3, 4, 5, 6, 7, 8]
  console.log('输出:', result);  // [1, 4, 9, 16, 25, 36, 49, 64]
}

computeShaderDemo();
```

### 常见陷阱

| 陷阱 | 说明 | 解决方案 |
|------|------|----------|
| 着色器编译错误静默失败 | WebGL `gl.compileShader` 不抛异常，仅设状态 | 编译后必须检查 `COMPILE_STATUS`，输出 `getShaderInfoLog` |
| 顶点属性未启用 | 忘记 `gl.enableVertexAttribArray` 导致渲染空白 | 每个 attribute 都需显式 enable |
| WebGL 状态泄漏 | 全局状态机中忘记恢复状态，影响后续绘制 | 使用 VAO 隔离顶点属性状态；每次绘制前重置必要状态 |
| Uniform 未设置或位置错误 | `getUniformLocation` 返回 null 但不报错 | 检查返回值是否为 null；确认着色器中 uniform 被使用（未使用会被优化掉） |
| 纹理未设置翻转 | 图片 Y 轴与 WebGL Y 轴方向相反导致纹理倒置 | WebGL 中调用 `gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true)` |
| Canvas 尺寸与 CSS 尺寸不一致 | 仅设 CSS 宽高，实际绘制分辨率默认 300x150 | 同时设置 `canvas.width/height` 属性和 CSS 尺寸，注意 `devicePixelRatio` |
| WebGPU 异步初始化未 await | `requestAdapter()` / `requestDevice()` 是异步的 | 使用 `await` 或 `.then()` 确保初始化完成后再操作 |
| WebGPU Buffer 用法标记不完整 | 缺少 `COPY_SRC` 导致无法复制，缺少 `MAP_READ` 无法回读 | 创建 Buffer 时仔细规划完整 usage 标志组合 |
| 着色器精度声明缺失 | WebGL1 片元着色器 float 默认低精度，导致渲染瑕疵 | 在片元着色器头部声明 `precision highp float;` |
| 深度测试未启用 | 3D 场景中后绘制的物体总在前面 | 调用 `gl.enable(gl.DEPTH_TEST)` 并在清屏时 `gl.clear(gl.DEPTH_BUFFER_BIT)` |

### 最佳实践

1. **优先使用 Three.js 等引擎**：除非需要极致控制，否则不要直接写 WebGL，Three.js 已处理大量兼容性和优化问题
2. **使用 WebGL2**：`getContext('webgl2')` 支持 VAO、3D 纹理、MRT 等特性，写法更现代
3. **WebGPU 渐进增强**：先检测 `navigator.gpu` 是否存在，不支持时降级到 WebGL
4. **复用管线和缓冲区**：WebGPU 中 `createRenderPipeline` 开销大，应缓存复用
5. **减少状态切换**：WebGL 中按状态分组绘制调用，最小化 `gl.bindTexture` / `gl.useProgram` 次数
6. **合理使用 Instancing**：相同几何体多次绘制时，用 `drawArraysInstanced` / `drawElementsInstanced` 代替循环
7. **纹理压缩**：使用 ASTC / ETC2 / BC 等压缩格式减少显存占用和上传带宽
8. **GPU 数据回读最小化**：`readPixels` / `stagingBuffer.mapAsync` 会触发 GPU-CPU 同步，应异步处理且避免每帧回读
9. **命令缓冲合并**：WebGPU 中尽量在单个 `CommandEncoder` 中编码多段渲染 Pass，减少提交次数
10. **着色器代码管理**：将 WGSL / GLSL 拆分为独立文件，构建时打包注入，便于维护和复用

---

## 面试题

### 1. WebGL 渲染管线的完整流程是什么？

**答**：应用层提供顶点数据 → **顶点着色器**（逐顶点坐标变换）→ **图元装配**（顶点组装为三角形等）→ **光栅化**（几何图元转为片元）→ **片元着色器**（逐片元着色、纹理采样）→ **逐片元操作**（深度/模板测试、混合）→ **帧缓冲**。WebGL1 中几何着色器不可用，WebGL2 可选支持变换反馈。

### 2. WebGL 和 WebGPU 的核心区别是什么？

**答**：① 底层 API 不同——WebGL 基于 OpenGL ES，WebGPU 基于 Vulkan/Metal/D3D12；② 管线模型不同——WebGL 是全局状态机，WebGPU 是显式管线对象；③ 命令提交不同——WebGL 即时调用，WebGPU 通过命令缓冲延迟提交；④ 计算能力——WebGPU 原生支持 Compute Shader，WebGL 不支持；⑤ 错误处理——WebGPU 编译时验证，WebGL 运行时轮询。

### 3. 顶点着色器和片元着色器各自的作用是什么？

**答**：**顶点着色器**对每个顶点执行一次，负责将模型坐标变换到裁剪空间（MVP 矩阵变换），并将颜色、法线、UV 等属性通过 `varying` / `out` 传递给片元阶段。**片元着色器**对每个片元（候选像素）执行一次，决定最终颜色输出，可进行纹理采样、光照计算、Alpha 混合等。两者执行频率差异巨大——顶点数远少于片元数。

### 4. 为什么要用 GPU 而不是 CPU 做图形渲染？

**答**：GPU 拥有数千个精简核心，天然适合**数据并行**任务——图形渲染中每个顶点/片元的计算相互独立，GPU 可同时处理大量顶点和片元。CPU 核心少但单核性能强，适合逻辑复杂的串行任务。对于百万级像素的逐帧计算，GPU 吞吐量可达 CPU 的数十到上百倍。

### 5. Three.js 的整体架构是怎样的？

**答**：Three.js 采用**场景图（Scene Graph）**架构：`Scene` 为根节点，`Mesh`（几何体 + 材质）、`Light`、`Camera` 等为子节点，通过 `Object3D` 的树形层级管理变换关系。渲染器 `WebGLRenderer` 负责将场景图转换为 WebGL 调用。核心三件套：**Scene + Camera + Renderer**，对象模型为 **Geometry + Material → Mesh**。内部封装了着色器编译、缓冲管理、光照计算等底层细节。

### 6. WebGPU 的计算着色器有什么用？与渲染着色器有何不同？

**答**：计算着色器（Compute Shader）运行在 GPU 上但**不参与图形渲染管线**，专门用于通用并行计算（GPGPU）。它通过 `@compute` 入口函数、工作组（workgroup）和存储缓冲区（storage buffer）进行数据并行处理。典型用途：物理模拟、粒子系统更新、图像后处理、机器学习推理。与渲染着色器的区别：没有顶点/片元的概念，输入输出都是自定义缓冲区，不受图形管线阶段约束。

### 7. WebGL/WebGPU 场景的性能优化有哪些常见手段？

**答**：① **Draw Call 合并**——使用 Instancing 批量绘制相同几何体；② **状态排序**——按 shader/texture 分组，减少状态切换；③ **LOD**——远处物体使用低模；④ **视锥剔除**——只渲染相机可见物体；⑤ **纹理压缩**——使用 ASTC/ETC2/BC 格式减少显存和带宽；⑥ **遮挡查询**——跳过被遮挡物体的绘制；⑦ **GPU 剔除**——用 Compute Shader 做视锥/遮挡剔除；⑧ **异步上传**——WebGPU 中用 `writeBuffer` / `writeTexture` 异步传输数据。

### 8. WGSL 与 GLSL 的主要区别是什么？

**答**：① **语法体系**——GLSL 类 C 语言，WGSL 类 Rust 风格，有更严格的类型系统；② **入口标注**——GLSL 用 `void main()`，WGSL 用 `@vertex fn vs_main()` / `@fragment fn fs_main()` 等属性标注；③ **变量声明**——GLSL 用 `attribute`/`varying`/`uniform`，WGSL 用 `@location`/`@binding`/`@group` 等装饰器；④ **类型安全**——WGSL 不允许隐式类型转换（如 `float` 与 `int` 相加），GLSL 允许；⑤ **向量类型**——GLSL 写 `vec3`，WGSL 写 `vec3<f32>`；⑥ **WebGPU 原生**——WGSL 为 WebGPU 设计，支持 compute shader 入口点，GLSL 不支持。

---

## Related

- [MDN WebGL 教程](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/Tutorial)
- [WebGPU 规范](https://www.w3.org/TR/webgpu/)
- [WGSL 规范](https://www.w3.org/TR/WGSL/)
- [Three.js 官方文档](https://threejs.org/docs/)
- [WebGPU Fundamentals](https://webgpufundamentals.org/)
- [WebGL2 Fundamentals](https://webgl2fundamentals.org/)
- [Google WebGPU 示例](https://github.com/webgpu/webgpu-samples)
