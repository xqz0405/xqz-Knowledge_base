---
tags:
  - Web前端
  - Web APIs
  - Web Audio
  - 多媒体
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Web Audio与多媒体API

> 浏览器原生音频处理与多媒体捕获的完整指南：从AudioContext节点图架构到实时音视频采集，涵盖音频合成、可视化、空间音效、录音与Worklet自定义处理。

相关链接：[[WebGL与WebGPU]] [[Canvas与SVG]] [[WebRTC实时通信]]

---

## 1 What — 什么是Web Audio与多媒体API

### Web Audio API

Web Audio API 是浏览器提供的一套**高性能音频处理框架**，基于 `AudioContext` 构建节点图（Node Graph）架构，允许开发者以声明式方式将音频源、处理节点、输出目标连接成处理链路，实现音频播放、合成、分析、空间化等能力。

核心概念：

- **AudioContext**：音频上下文，所有音频节点的容器与管理器
- **音频节点（AudioNode）**：最小处理单元，分为源节点、处理节点、目标节点
- **节点图（Node Graph）**：节点之间通过 `connect()` 方法连接形成有向图
- **精确调度**：基于 `AudioContext.currentTime` 的时间线，支持样本级精度的调度

### Media Capture API

Media Capture API 是浏览器提供的**音视频流采集接口**，通过 `getUserMedia` 获取摄像头与麦克风的实时媒体流，通过 `getDisplayMedia` 获取屏幕共享流，是录音、视频通话、直播等场景的基础。

### 相关API族谱

| API | 职责 | 典型场景 |
|-----|------|----------|
| Web Audio API | 音频处理与合成 | 音效、可视化、空间音效 |
| Media Capture API | 音视频采集 | 录音、视频通话 |
| MediaRecorder API | 媒体录制 | 录音导出、录屏 |
| Picture-in-Picture API | 画中画 | 视频浮动窗口 |
| Media Session API | 媒体会话控制 | 锁屏播放控制 |
| Web MIDI API | MIDI设备交互 | 虚拟乐器、MIDI控制器 |

---

## 2 Why — 为什么需要Web Audio与多媒体API

### 原生能力，无需插件

Flash 时代结束后，浏览器需要原生音频处理能力。Web Audio API 填补了这一空白，提供比 `<audio>` 元素强大得多的音频处理能力：

- **`<audio>` 元素**：仅能播放/暂停/跳转，无法处理音频数据
- **Web Audio API**：可实时处理、分析、合成音频

### 核心优势

1. **浏览器原生**：无需安装任何插件，主流浏览器全面支持
2. **低延迟**：音频处理在独立线程运行，不阻塞主线程
3. **实时处理**：支持实时音频分析与效果处理
4. **模块化架构**：节点图设计，可自由组合效果链
5. **空间音效**：支持3D空间音频定位，适合游戏与VR
6. **精确调度**：样本级精度的音频事件调度
7. **自定义处理**：AudioWorklet 允许编写自定义 DSP 算法

### 典型应用场景

| 场景 | 说明 |
|------|------|
| 游戏音效 | 实时音效合成、3D空间定位、多声道混音 |
| 音频可视化 | 频谱分析、波形绘制、音乐可视化 |
| 音频编辑器 | 在线DAW、波形编辑、效果器链 |
| 语音通信 | 实时音频处理、降噪、回声消除 |
| 录音工具 | 麦克风录制、格式转换、导出 |
| 虚拟乐器 | 振荡器合成、MIDI输入、ADSR包络 |
| 直播/会议 | 摄像头采集、屏幕共享、画中画 |

---

## 3 对比 — Web Audio API vs HTML5 Audio vs Howler.js

| 维度 | Web Audio API | HTML5 `<audio>` | Howler.js |
|------|---------------|-----------------|-----------|
| **能力** | 全功能：合成/处理/分析/空间化 | 基础播放控制 | 播放+基础效果+sprite |
| **延迟** | 极低（~5ms） | 较高（~100ms+） | 中等（~50ms） |
| **复杂度** | 高，需理解节点图 | 极低，一行代码 | 低，API简洁 |
| **兼容性** | 现代浏览器全面支持 | 所有浏览器 | 降级到Flash（旧版） |
| **实时处理** | 支持，节点链实时处理 | 不支持 | 不支持 |
| **音频分析** | AnalyserNode频谱/波形 | 不支持 | 不支持 |
| **空间音效** | PannerNode 3D定位 | 不支持 | 简单立体声 |
| **自定义DSP** | AudioWorklet | 不支持 | 不支持 |
| **适用场景** | 游戏音效、可视化、音频编辑 | 简单音乐播放 | 网页背景音效、简单游戏 |

```javascript
// === HTML5 Audio：最简单的方式 ===
const audio = new Audio('bgm.mp3');
audio.play();

// === Howler.js：便捷的音频库 ===
const sound = new Howl({ src: ['bgm.mp3'] });
sound.play();

// === Web Audio API：完全控制 ===
const ctx = new AudioContext();
const source = ctx.createMediaElementSource(audio);
const gain = ctx.createGain();
const analyser = ctx.createAnalyser();
source.connect(gain).connect(analyser).connect(ctx.destination);
source.start();
```

---

## 4 How — 核心用法详解

### 4.1 AudioContext与音频节点图

AudioContext 是所有音频操作的入口，管理音频节点的创建、连接与生命周期。节点之间通过 `connect()` 方法连接，形成**有向无环图（DAG）**。

```
[源节点] → [处理节点1] → [处理节点2] → [目标节点(AudioContext.destination)]
```

```javascript
// === 创建AudioContext ===
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

// 状态检查
console.log(audioCtx.state); // 'suspended' | 'running' | 'closed'

// 恢复被暂停的上下文（自动播放策略要求）
if (audioCtx.state === 'suspended') {
  await audioCtx.resume();
}

// === 三种节点类型 ===

// 1. 源节点（Source Node）：产生音频信号
const oscillator = audioCtx.createOscillator();      // 振荡器
const bufferSource = audioCtx.createBufferSource();   // 缓冲区源
// MediaElementSourceNode / MediaStreamSourceNode

// 2. 处理节点（Processing Node）：处理音频信号
const gainNode = audioCtx.createGain();               // 增益
const filter = audioCtx.createBiquadFilter();         // 滤波器
const analyser = audioCtx.createAnalyser();           // 分析器

// 3. 目标节点（Destination Node）：输出音频信号
audioCtx.destination; // 默认输出到扬声器

// === 连接节点 ===
oscillator.connect(gainNode);      // 振荡器 → 增益
gainNode.connect(filter);          // 增益 → 滤波器
filter.connect(analyser);          // 滤波器 → 分析器
analyser.connect(audioCtx.destination); // 分析器 → 输出

// === 链式连接（推荐写法）===
oscillator.connect(gainNode)
  .connect(filter)
  .connect(analyser)
  .connect(audioCtx.destination);

// === 多输入/多输出 ===
// 一个节点可以连接到多个目标（分流）
oscillator.connect(gainNode);
oscillator.connect(analyser);

// 多个源可以连接到同一个目标（混流）
const osc1 = audioCtx.createOscillator();
const osc2 = audioCtx.createOscillator();
osc1.connect(gainNode);
osc2.connect(gainNode);

// === 断开连接 ===
oscillator.disconnect();             // 断开所有输出
oscillator.disconnect(gainNode);     // 断开到gainNode的连接

// === 时间线 ===
console.log(audioCtx.currentTime); // 当前时间（秒，高精度）
oscillator.start(audioCtx.currentTime + 1); // 1秒后开始
oscillator.stop(audioCtx.currentTime + 3);  // 3秒后停止

// === 关闭AudioContext ===
audioCtx.close(); // 释放所有资源，不可重用
```

### 4.2 音频源节点

#### OscillatorNode — 振荡器

产生周期性波形，是最基本的音频源，支持4种内置波形。

```javascript
const ctx = new AudioContext();
const osc = ctx.createOscillator();

// === 波形类型 ===
osc.type = 'sine';     // 正弦波（纯音）
osc.type = 'square';   // 方波（8-bit风格）
osc.type = 'sawtooth'; // 锯齿波（明亮）
osc.type = 'triangle'; // 三角波（柔和）

// === 频率 ===
osc.frequency.value = 440;                    // A4音高
osc.frequency.setValueAtTime(440, ctx.currentTime);
osc.frequency.linearRampToValueAtTime(880, ctx.currentTime + 1); // 1秒滑到880Hz
osc.frequency.exponentialRampToValueAtTime(220, ctx.currentTime + 2); // 指数滑到220Hz

// === 自定义波形（PeriodicWave）===
const real = new Float32Array([0, 0.4, 0.4, 0, 0, 0.2, 0, 0]);
const imag = new Float32Array([0, 0, 0.4, 0, 0, 0, 0, 0]);
const wave = ctx.createPeriodicWave(real, imag);
osc.setPeriodicWave(wave);

osc.connect(ctx.destination);
osc.start();
osc.stop(ctx.currentTime + 2); // 2秒后停止

// === 播放简单音阶 ===
function playNote(freq, startTime, duration) {
  const o = ctx.createOscillator();
  const g = ctx.createGain();
  o.frequency.value = freq;
  o.connect(g).connect(ctx.destination);
  o.start(startTime);
  g.gain.setValueAtTime(0.3, startTime);
  g.gain.exponentialRampToValueAtTime(0.001, startTime + duration);
  o.stop(startTime + duration);
}

const now = ctx.currentTime;
// C大调音阶
[261.63, 293.66, 329.63, 349.23, 392.00, 440.00, 493.88, 523.25]
  .forEach((freq, i) => playNote(freq, now + i * 0.4, 0.35));
```

#### AudioBufferSourceNode — 缓冲区源

播放预加载的音频数据，适用于音效、采样器等场景。

```javascript
// === 从URL加载音频文件 ===
async function loadAudio(url) {
  const response = await fetch(url);
  const arrayBuffer = await response.arrayBuffer();
  return await ctx.decodeAudioData(arrayBuffer);
}

const buffer = await loadAudio('/sounds/explosion.mp3');
const source = ctx.createBufferSource();
source.buffer = buffer;

source.connect(ctx.destination);
source.start(0);        // 从头播放
source.start(2);        // 2秒后播放
source.start(0, 1.5);   // 从1.5秒位置开始
source.start(0, 0, 3);  // 播放前3秒

// === 播放速率 ===
source.playbackRate.value = 1.5;  // 1.5倍速
source.playbackRate.value = 0.5;  // 0.5倍速（降调）

// === 循环播放 ===
source.loop = true;
source.loopStart = 0.5; // 循环起点
source.loopEnd = 2.0;   // 循环终点

// === 播放结束事件 ===
source.onended = () => console.log('播放结束');

// === 音效缓存池（避免重复加载）===
class SoundPool {
  constructor() {
    this.ctx = new AudioContext();
    this.buffers = new Map();
  }

  async load(name, url) {
    const response = await fetch(url);
    const arrayBuffer = await response.arrayBuffer();
    const buffer = await this.ctx.decodeAudioData(arrayBuffer);
    this.buffers.set(name, buffer);
  }

  play(name, volume = 1) {
    const source = this.ctx.createBufferSource();
    const gain = this.ctx.createGain();
    source.buffer = this.buffers.get(name);
    gain.gain.value = volume;
    source.connect(gain).connect(this.ctx.destination);
    source.start(0);
    return source;
  }
}

const pool = new SoundPool();
await pool.load('click', '/sounds/click.mp3');
await pool.load('explosion', '/sounds/explosion.mp3');
pool.play('click', 0.5);
```

#### MediaElementSourceNode 与 MediaStreamSourceNode

```javascript
// === MediaElementSourceNode：从<audio>/<video>元素获取音频 ===
const audioElement = document.querySelector('audio');
const mediaSource = ctx.createMediaElementSource(audioElement);
const gain = ctx.createGain();
mediaSource.connect(gain).connect(ctx.destination);

// 仍然可以用audioElement控制播放
audioElement.play();
audioElement.currentTime = 30; // 跳转
// 注意：创建MediaElementSource后，audioElement的音频不再直接输出
// 必须通过Web Audio节点图连接到destination才能听到

// === MediaStreamSourceNode：从MediaStream获取音频 ===
const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
const streamSource = ctx.createMediaStreamSource(stream);
const analyser = ctx.createAnalyser();
streamSource.connect(analyser);
// 注意：不要连接到destination，否则会产生回声！
```

### 4.3 音量与增益

#### GainNode — 增益控制

```javascript
const gain = ctx.createGain();
gain.gain.value = 0.5; // 音量50%

// === 渐变方法 ===
// setValueAtTime：瞬间设置
gain.gain.setValueAtTime(0.5, ctx.currentTime);

// linearRampToValueAtTime：线性渐变
gain.gain.linearRampToValueAtTime(1.0, ctx.currentTime + 2); // 2秒线性增到100%

// exponentialRampToValueAtTime：指数渐变（更自然）
gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 1); // 1秒指数衰减

// setTargetAtTime：指数趋近（指定时间常数）
gain.gain.setTargetAtTime(0.8, ctx.currentTime, 0.5); // 时间常数0.5秒

// setValueCurveAtTime：自定义曲线
const curve = new Float32Array([0, 0.2, 0.5, 0.8, 1.0]);
gain.gain.setValueCurveAtTime(curve, ctx.currentTime, 2); // 2秒内按曲线变化

// === 淡入淡出 ===
function fadeIn(gainNode, duration = 1) {
  gainNode.gain.setValueAtTime(0.001, ctx.currentTime);
  gainNode.gain.exponentialRampToValueAtTime(1, ctx.currentTime + duration);
}

function fadeOut(gainNode, duration = 1) {
  gainNode.gain.setValueAtTime(gainNode.gain.value, ctx.currentTime);
  gainNode.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + duration);
}

// === 交叉淡入（Crossfade）===
function crossfade(sourceA, gainA, sourceB, gainB, duration = 2) {
  const now = ctx.currentTime;
  // A淡出
  gainA.gain.setValueAtTime(1, now);
  gainA.gain.linearRampToValueAtTime(0, now + duration);
  // B淡入
  gainB.gain.setValueAtTime(0, now);
  gainB.gain.linearRampToValueAtTime(1, now + duration);
}

// === 等功率交叉淡入（避免中间音量下降）===
function equalPowerCrossfade(gainA, gainB, mix) {
  // mix: 0 = 全A, 1 = 全B
  gainA.gain.value = Math.cos(mix * Math.PI * 0.5);
  gainB.gain.value = Math.sin(mix * Math.PI * 0.5);
}
```

### 4.4 音频分析可视化

#### AnalyserNode — 频谱与波形分析

```javascript
const analyser = ctx.createAnalyser();

// === 配置参数 ===
analyser.fftSize = 2048;          // FFT大小（必须是2的幂）
analyser.smoothingTimeConstant = 0.8; // 平滑系数（0~1）
analyser.minDecibels = -90;       // 最小分贝
analyser.maxDecibels = -10;       // 最大分贝

const bufferLength = analyser.frequencyBinCount; // fftSize / 2 = 1024
```

#### 频谱绘制（频率域）

```javascript
function drawFrequency(canvas, analyser) {
  const canvasCtx = canvas.getContext('2d');
  const bufferLength = analyser.frequencyBinCount;
  const dataArray = new Uint8Array(bufferLength);

  function draw() {
    requestAnimationFrame(draw);
    analyser.getByteFrequencyData(dataArray); // 0~255 频率数据

    canvasCtx.fillStyle = 'rgb(0, 0, 0)';
    canvasCtx.fillRect(0, 0, canvas.width, canvas.height);

    const barWidth = (canvas.width / bufferLength) * 2.5;
    let x = 0;

    for (let i = 0; i < bufferLength; i++) {
      const barHeight = (dataArray[i] / 255) * canvas.height;

      // 根据频率值着色
      const r = dataArray[i] + 25;
      const g = 250 - dataArray[i];
      const b = 50;
      canvasCtx.fillStyle = `rgb(${r}, ${g}, ${b})`;

      canvasCtx.fillRect(x, canvas.height - barHeight, barWidth, barHeight);
      x += barWidth + 1;
    }
  }

  draw();
}
```

#### 波形绘制（时间域）

```javascript
function drawWaveform(canvas, analyser) {
  const canvasCtx = canvas.getContext('2d');
  const bufferLength = analyser.fftSize;
  const dataArray = new Uint8Array(bufferLength);

  function draw() {
    requestAnimationFrame(draw);
    analyser.getByteTimeDomainData(dataArray); // 0~255 时间域数据

    canvasCtx.fillStyle = 'rgb(0, 0, 0)';
    canvasCtx.fillRect(0, 0, canvas.width, canvas.height);

    canvasCtx.lineWidth = 2;
    canvasCtx.strokeStyle = 'rgb(0, 255, 0)';
    canvasCtx.beginPath();

    const sliceWidth = canvas.width / bufferLength;
    let x = 0;

    for (let i = 0; i < bufferLength; i++) {
      const v = dataArray[i] / 128.0; // 归一化到0~2
      const y = (v * canvas.height) / 2;

      if (i === 0) {
        canvasCtx.moveTo(x, y);
      } else {
        canvasCtx.lineTo(x, y);
      }

      x += sliceWidth;
    }

    canvasCtx.lineTo(canvas.width, canvas.height / 2);
    canvasCtx.stroke();
  }

  draw();
}
```

#### 圆形频谱可视化

```javascript
function drawCircularSpectrum(canvas, analyser) {
  const ctx2d = canvas.getContext('2d');
  const bufferLength = analyser.frequencyBinCount;
  const dataArray = new Uint8Array(bufferLength);
  const centerX = canvas.width / 2;
  const centerY = canvas.height / 2;
  const radius = 100;

  function draw() {
    requestAnimationFrame(draw);
    analyser.getByteFrequencyData(dataArray);

    ctx2d.fillStyle = 'rgba(0, 0, 0, 0.2)';
    ctx2d.fillRect(0, 0, canvas.width, canvas.height);

    const step = Math.floor(bufferLength / 180); // 取180个频率点画圆

    for (let i = 0; i < 180; i++) {
      const dataIndex = i * step;
      const amplitude = dataArray[dataIndex] / 255;
      const barHeight = amplitude * 80;

      const angle = (i / 180) * Math.PI * 2 - Math.PI / 2;
      const x1 = centerX + Math.cos(angle) * radius;
      const y1 = centerY + Math.sin(angle) * radius;
      const x2 = centerX + Math.cos(angle) * (radius + barHeight);
      const y2 = centerY + Math.sin(angle) * (radius + barHeight);

      const hue = (i / 180) * 360;
      ctx2d.strokeStyle = `hsl(${hue}, 100%, 50%)`;
      ctx2d.lineWidth = 2;
      ctx2d.beginPath();
      ctx2d.moveTo(x1, y1);
      ctx2d.lineTo(x2, y2);
      ctx2d.stroke();
    }
  }

  draw();
}
```

### 4.5 音效处理

#### BiquadFilterNode — 滤波器

```javascript
const filter = ctx.createBiquadFilter();

// === 滤波器类型 ===
filter.type = 'lowpass';    // 低通（去高频）
filter.type = 'highpass';   // 高通（去低频）
filter.type = 'bandpass';   // 带通（保留频段）
filter.type = 'lowshelf';   // 低频搁架（增强/衰减低频）
filter.type = 'highshelf';  // 高频搁架（增强/衰减高频）
filter.type = 'peaking';    // 峰值（增强/衰减指定频段）
filter.type = 'notch';      // 陷波（移除指定频率）
filter.type = 'allpass';    // 全通（仅改变相位）

// === 参数 ===
filter.frequency.value = 1000; // 中心频率
filter.Q.value = 1;            // Q值（品质因数）
filter.gain.value = 0;         // 增益（dB，用于shelf和peaking类型）

// === 模拟电话音效 ===
function telephoneEffect(source) {
  const highpass = ctx.createBiquadFilter();
  highpass.type = 'highpass';
  highpass.frequency.value = 300;

  const lowpass = ctx.createBiquadFilter();
  lowpass.type = 'lowpass';
  lowpass.frequency.value = 3400;

  const distortion = ctx.createWaveShaper();
  distortion.curve = makeDistortionCurve(50);

  source.connect(highpass).connect(lowpass).connect(distortion);
  return distortion;
}
```

#### ConvolverNode — 混响

```javascript
// === 加载混响脉冲响应（Impulse Response）===
async function createReverb(url) {
  const response = await fetch(url);
  const arrayBuffer = await response.arrayBuffer();
  const buffer = await ctx.decodeAudioData(arrayBuffer);

  const convolver = ctx.createConvolver();
  convolver.buffer = buffer;
  return convolver;
}

const reverb = await createReverb('/impulses/hall.wav');

// 干湿混合
const dryGain = ctx.createGain();
const wetGain = ctx.createGain();
dryGain.gain.value = 0.7;
wetGain.gain.value = 0.3;

source.connect(dryGain).connect(ctx.destination);
source.connect(reverb).connect(wetGain).connect(ctx.destination);

// === 程序生成简单混响 ===
function generateReverb(duration = 2, decay = 2) {
  const sampleRate = ctx.sampleRate;
  const length = sampleRate * duration;
  const buffer = ctx.createBuffer(2, length, sampleRate);

  for (let channel = 0; channel < 2; channel++) {
    const data = buffer.getChannelData(channel);
    for (let i = 0; i < length; i++) {
      data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / length, decay);
    }
  }

  const convolver = ctx.createConvolver();
  convolver.buffer = buffer;
  return convolver;
}
```

#### DelayNode — 延迟

```javascript
const delay = ctx.createDelay(5.0); // 最大延迟5秒
delay.delayTime.value = 0.5;        // 延迟0.5秒

// === 简单回声 ===
const feedback = ctx.createGain();
feedback.gain.value = 0.4; // 反馈量（控制回声次数）

source.connect(ctx.destination);           // 干声
source.connect(delay);                      // → 延迟
delay.connect(feedback);                    // → 反馈增益
feedback.connect(delay);                    // → 返回延迟（形成循环）
delay.connect(ctx.destination);             // → 湿声输出

// === 立体声延迟 ===
function stereoDelay(source, leftTime = 0.3, rightTime = 0.5) {
  const splitter = ctx.createChannelSplitter(2);
  const merger = ctx.createChannelMerger(2);
  const leftDelay = ctx.createDelay();
  const rightDelay = ctx.createDelay();
  const leftFeedback = ctx.createGain();
  const rightFeedback = ctx.createGain();

  leftDelay.delayTime.value = leftTime;
  rightDelay.delayTime.value = rightTime;
  leftFeedback.gain.value = 0.3;
  rightFeedback.gain.value = 0.3;

  source.connect(splitter);
  splitter.connect(leftDelay, 0);
  splitter.connect(rightDelay, 1);

  leftDelay.connect(leftFeedback).connect(leftDelay);
  rightDelay.connect(rightFeedback).connect(rightDelay);

  leftDelay.connect(merger, 0, 0);
  rightDelay.connect(merger, 0, 1);

  return merger;
}
```

#### DynamicsCompressorNode — 动态压缩

```javascript
const compressor = ctx.createDynamicsCompressor();
compressor.threshold.value = -24;  // 阈值（dB）
compressor.knee.value = 30;        // 膝盖宽度（dB）
compressor.ratio.value = 12;       // 压缩比
compressor.attack.value = 0.003;   // 启动时间（秒）
compressor.release.value = 0.25;   // 释放时间（秒）

// 压缩器通常放在效果链末端，防止削波
source.connect(gainNode).connect(compressor).connect(ctx.destination);

// 实时观察压缩量
const reductionDisplay = document.querySelector('#reduction');
function updateReduction() {
  reductionDisplay.textContent = compressor.reduction; // 压缩量（dB）
  requestAnimationFrame(updateReduction);
}
updateReduction();
```

#### WaveShaperNode — 失真

```javascript
const shaper = ctx.createWaveShaper();

// === 生成失真曲线 ===
function makeDistortionCurve(amount = 50) {
  const samples = 44100;
  const curve = new Float32Array(samples);
  const deg = Math.PI / 180;

  for (let i = 0; i < samples; i++) {
    const x = (i * 2) / samples - 1;
    curve[i] = ((3 + amount) * x * 20 * deg) / (Math.PI + amount * Math.abs(x));
  }

  return curve;
}

shaper.curve = makeDistortionCurve(100);
shaper.oversample = '4x'; // 'none' | '2x' | '4x'（抗混叠）

// === 完整效果链示例：吉他效果器 ===
function guitarEffects(source) {
  // 压缩
  const compressor = ctx.createDynamicsCompressor();
  compressor.threshold.value = -20;

  // 失真
  const distortion = ctx.createWaveShaper();
  distortion.curve = makeDistortionCurve(200);
  distortion.oversample = '4x';

  // 滤波（音色控制）
  const lowpass = ctx.createBiquadFilter();
  lowpass.type = 'lowpass';
  lowpass.frequency.value = 3000;

  // 延迟
  const delay = ctx.createDelay();
  delay.delayTime.value = 0.3;
  const feedback = ctx.createGain();
  feedback.gain.value = 0.3;

  // 混响
  const reverb = generateReverb(1.5, 2);
  const wetGain = ctx.createGain();
  wetGain.gain.value = 0.2;

  // 主音量
  const masterGain = ctx.createGain();
  masterGain.gain.value = 0.6;

  // 连接效果链
  source.connect(compressor)
    .connect(distortion)
    .connect(lowpass)
    .connect(masterGain);

  // 干声
  masterGain.connect(ctx.destination);

  // 延迟回路
  masterGain.connect(delay).connect(feedback).connect(delay);
  delay.connect(ctx.destination);

  // 混响
  masterGain.connect(reverb).connect(wetGain).connect(ctx.destination);

  return masterGain;
}
```

### 4.6 空间音频

#### PannerNode — 3D空间定位

```javascript
const panner = ctx.createPanner();

// === 距离模型 ===
panner.distanceModel = 'inverse';  // 'linear' | 'inverse' | 'exponential'
panner.refDistance = 1;             // 参考距离
panner.maxDistance = 10000;         // 最大距离
panner.rolloffFactor = 1;          // 衰减系数

// === 定位模型 ===
panner.panningModel = 'HRTF';      // 'equalpower' | 'HRTF'（头相关传递函数，更真实）

// === 音源位置 ===
panner.positionX.value = 5;
panner.positionY.value = 0;
panner.positionZ.value = -3;

// === 音源方向 ===
panner.orientationX.value = 1;
panner.orientationY.value = 0;
panner.orientationZ.value = 0;

// === 听者位置（Listener）===
const listener = ctx.listener;
listener.positionX.value = 0;
listener.positionY.value = 0;
listener.positionZ.value = 0;
listener.forwardX.value = 0;
listener.forwardY.value = 0;
listener.forwardZ.value = -1;
listener.upX.value = 0;
listener.upY.value = 1;
listener.upZ.value = 0;
```

#### 3D游戏空间音效

```javascript
class SpatialAudio {
  constructor() {
    this.ctx = new AudioContext();
    this.sounds = new Map();
    this.listener = this.ctx.listener;

    if (this.listener.positionX) {
      this.listener.positionX.value = 0;
      this.listener.positionY.value = 0;
      this.listener.positionZ.value = 0;
    } else {
      // 兼容旧版API
      this.listener.setPosition(0, 0, 0);
    }
  }

  async load(name, url) {
    const response = await fetch(url);
    const buffer = await this.ctx.decodeAudioData(await response.arrayBuffer());
    this.sounds.set(name, buffer);
  }

  // 在指定3D位置播放音效
  playAt(name, x, y, z) {
    const source = this.ctx.createBufferSource();
    const panner = this.ctx.createPanner();
    const gain = this.ctx.createGain();

    source.buffer = this.sounds.get(name);
    panner.panningModel = 'HRTF';
    panner.distanceModel = 'inverse';
    panner.refDistance = 1;
    panner.maxDistance = 100;
    panner.rolloffFactor = 1;

    if (panner.positionX) {
      panner.positionX.value = x;
      panner.positionY.value = y;
      panner.positionZ.value = z;
    } else {
      panner.setPosition(x, y, z);
    }

    gain.gain.value = 1;
    source.connect(gain).connect(panner).connect(this.ctx.destination);
    source.start(0);
    return { source, panner, gain };
  }

  // 更新听者位置与朝向（游戏循环中调用）
  updateListener(position, forward, up) {
    const l = this.listener;
    if (l.positionX) {
      l.positionX.value = position.x;
      l.positionY.value = position.y;
      l.positionZ.value = position.z;
      l.forwardX.value = forward.x;
      l.forwardY.value = forward.y;
      l.forwardZ.value = forward.z;
      l.upX.value = up.x;
      l.upY.value = up.y;
      l.upZ.value = up.z;
    } else {
      l.setPosition(position.x, position.y, position.z);
      l.setOrientation(forward.x, forward.y, forward.z, up.x, up.y, up.z);
    }
  }

  // 移动音源位置
  moveSound(sound, x, y, z) {
    const p = sound.panner;
    if (p.positionX) {
      p.positionX.setValueAtTime(x, this.ctx.currentTime);
      p.positionY.setValueAtTime(y, this.ctx.currentTime);
      p.positionZ.setValueAtTime(z, this.ctx.currentTime);
    } else {
      p.setPosition(x, y, z);
    }
  }
}

// 使用示例
const spatial = new SpatialAudio();
await spatial.load('footstep', '/sounds/footstep.mp3');
await spatial.load('explosion', '/sounds/explosion.mp3');

// 右边5米处播放脚步声
spatial.playAt('footstep', 5, 0, 0);

// 游戏循环中更新听者
spatial.updateListener(
  { x: player.x, y: player.y, z: player.z },
  { x: player.dirX, y: 0, z: player.dirZ },
  { x: 0, y: 1, z: 0 }
);
```

### 4.7 录音功能

#### MediaRecorder API — 媒体录制

```javascript
class AudioRecorder {
  constructor() {
    this.mediaRecorder = null;
    this.chunks = [];
    this.stream = null;
  }

  // === 获取麦克风并开始录音 ===
  async start(options = {}) {
    const constraints = {
      audio: {
        echoCancellation: true,     // 回声消除
        noiseSuppression: true,     // 降噪
        autoGainControl: true,      // 自动增益
        sampleRate: options.sampleRate || 44100,
        channelCount: options.channels || 1,
        ...options.constraints
      }
    };

    this.stream = await navigator.mediaDevices.getUserMedia(constraints);

    // 检查支持的MIME类型
    const mimeType = this._getSupportedMimeType();
    console.log('录制格式:', mimeType);

    this.mediaRecorder = new MediaRecorder(this.stream, {
      mimeType,
      audioBitsPerSecond: options.bitrate || 128000
    });

    this.chunks = [];

    this.mediaRecorder.ondataavailable = (e) => {
      if (e.data.size > 0) {
        this.chunks.push(e.data);
      }
    };

    this.mediaRecorder.onstop = () => {
      // 录音结束，触发回调
      if (this.onComplete) {
        const blob = new Blob(this.chunks, { type: mimeType });
        this.onComplete(blob);
      }
    };

    // 每100ms收集一次数据（确保不丢数据）
    this.mediaRecorder.start(100);
  }

  // === 暂停/恢复 ===
  pause() {
    if (this.mediaRecorder?.state === 'recording') {
      this.mediaRecorder.pause();
    }
  }

  resume() {
    if (this.mediaRecorder?.state === 'paused') {
      this.mediaRecorder.resume();
    }
  }

  // === 停止录音 ===
  stop() {
    if (this.mediaRecorder?.state !== 'inactive') {
      this.mediaRecorder.stop();
    }
    // 释放麦克风
    this.stream?.getTracks().forEach(track => track.stop());
  }

  // === 获取支持的最佳MIME类型 ===
  _getSupportedMimeType() {
    const types = [
      'audio/webm;codecs=opus',
      'audio/webm',
      'audio/ogg;codecs=opus',
      'audio/ogg',
      'audio/mp4',
      '' // 默认
    ];
    return types.find(t => t === '' || MediaRecorder.isTypeSupported(t)) || '';
  }

  // === 实时音频电平监测 ===
  getAudioLevel() {
    const ctx = new AudioContext();
    const source = ctx.createMediaStreamSource(this.stream);
    const analyser = ctx.createAnalyser();
    analyser.fftSize = 256;
    source.connect(analyser);

    const dataArray = new Uint8Array(analyser.frequencyBinCount);

    const getLevel = () => {
      analyser.getByteFrequencyData(dataArray);
      const avg = dataArray.reduce((a, b) => a + b) / dataArray.length;
      return avg / 255; // 0~1
    };

    return { getLevel, ctx };
  }
}

// === 使用示例 ===
const recorder = new AudioRecorder();

// 开始录音
await recorder.start({ bitrate: 192000 });

// 设置完成回调
recorder.onComplete = (blob) => {
  console.log('录音完成:', blob.size, 'bytes');
  const url = URL.createObjectURL(blob);
  const audio = new Audio(url);
  audio.play();

  // 下载录音文件
  const a = document.createElement('a');
  a.href = url;
  a.download = `recording_${Date.now()}.webm`;
  a.click();
};

// 实时音频电平显示
const { getLevel, ctx: audioCtx } = recorder.getAudioLevel();
function updateLevel() {
  const level = getLevel();
  levelMeter.style.width = `${level * 100}%`;
  requestAnimationFrame(updateLevel);
}
updateLevel();

// 停止录音
setTimeout(() => recorder.stop(), 5000);
```

### 4.8 音频合成器

#### 多振荡器叠加与ADSR包络

```javascript
class Synthesizer {
  constructor() {
    this.ctx = new AudioContext();
    this.masterGain = this.ctx.createGain();
    this.masterGain.gain.value = 0.5;

    // 效果链
    this.compressor = this.ctx.createDynamicsCompressor();
    this.masterGain.connect(this.compressor).connect(this.ctx.destination);

    this.activeNotes = new Map(); // 当前按下的音符
  }

  // === ADSR包络 ===
  _applyADSR(param, velocity, startTime) {
    const attackTime = 0.02;   // 起音
    const decayTime = 0.1;     // 衰减
    const sustainLevel = 0.3;  // 持续（音量比例）
    const releaseTime = 0.3;   // 释音

    const peak = velocity; // 最大音量

    // Attack: 0 → peak
    param.setValueAtTime(0.001, startTime);
    param.linearRampToValueAtTime(peak, startTime + attackTime);

    // Decay: peak → sustain
    param.linearRampToValueAtTime(
      peak * sustainLevel,
      startTime + attackTime + decayTime
    );

    // Sustain: 保持直到noteOff
    // Release在noteOff中处理
    return { attackTime, decayTime, sustainLevel, releaseTime };
  }

  // === 按下音符 ===
  noteOn(frequency, velocity = 0.8) {
    if (this.activeNotes.has(frequency)) return;

    const now = this.ctx.currentTime;

    // 主振荡器
    const osc1 = this.ctx.createOscillator();
    osc1.type = 'sawtooth';
    osc1.frequency.value = frequency;

    // 第二振荡器（微调产生厚实感）
    const osc2 = this.ctx.createOscillator();
    osc2.type = 'sawtooth';
    osc2.frequency.value = frequency * 1.005; // 微调5音分

    // 子振荡器（低一个八度）
    const subOsc = this.ctx.createOscillator();
    subOsc.type = 'sine';
    subOsc.frequency.value = frequency / 2;

    // 各振荡器增益
    const osc1Gain = this.ctx.createGain();
    const osc2Gain = this.ctx.createGain();
    const subGain = this.ctx.createGain();

    // ADSR包络
    const envGain = this.ctx.createGain();
    const adsr = this._applyADSR(envGain.gain, velocity, now);

    // 滤波器
    const filter = this.ctx.createBiquadFilter();
    filter.type = 'lowpass';
    filter.frequency.value = 2000;
    filter.Q.value = 2;

    // 滤波器包络
    filter.frequency.setValueAtTime(500, now);
    filter.frequency.linearRampToValueAtTime(4000, now + adsr.attackTime);
    filter.frequency.linearRampToValueAtTime(1500, now + adsr.attackTime + adsr.decayTime);

    // 连接
    osc1.connect(osc1Gain);
    osc2.connect(osc2Gain);
    subOsc.connect(subGain);

    osc1Gain.gain.value = 0.4;
    osc2Gain.gain.value = 0.3;
    subGain.gain.value = 0.2;

    osc1Gain.connect(filter);
    osc2Gain.connect(filter);
    subGain.connect(filter);

    filter.connect(envGain).connect(this.masterGain);

    // 启动振荡器
    osc1.start(now);
    osc2.start(now);
    subOsc.start(now);

    this.activeNotes.set(frequency, {
      oscillators: [osc1, osc2, subOsc],
      envGain,
      filter,
      adsr
    });
  }

  // === 释放音符 ===
  noteOff(frequency) {
    const note = this.activeNotes.get(frequency);
    if (!note) return;

    const now = this.ctx.currentTime;
    const { oscillators, envGain, adsr } = note;

    // Release阶段
    envGain.gain.cancelScheduledValues(now);
    envGain.gain.setValueAtTime(envGain.gain.value, now);
    envGain.gain.exponentialRampToValueAtTime(0.001, now + adsr.releaseTime);

    // 延迟停止振荡器
    oscillators.forEach(osc => osc.stop(now + adsr.releaseTime + 0.1));

    this.activeNotes.delete(frequency);
  }
}

// === 键盘映射 ===
const KEY_MAP = {
  'a': 261.63, 'w': 277.18, 's': 293.66, 'e': 311.13,
  'd': 329.63, 'f': 349.23, 't': 369.99, 'g': 392.00,
  'y': 415.30, 'h': 440.00, 'u': 466.16, 'j': 493.88,
  'k': 523.25
};

const synth = new Synthesizer();

document.addEventListener('keydown', (e) => {
  if (e.repeat) return;
  const freq = KEY_MAP[e.key.toLowerCase()];
  if (freq) synth.noteOn(freq);
});

document.addEventListener('keyup', (e) => {
  const freq = KEY_MAP[e.key.toLowerCase()];
  if (freq) synth.noteOff(freq);
});
```

#### Web MIDI API

```javascript
// === 请求MIDI访问 ===
const midiAccess = await navigator.requestMIDIAccess();

// === 监听MIDI设备连接 ===
midiAccess.onstatechange = (e) => {
  console.log(`MIDI设备: ${e.port.name}, 状态: ${e.port.state}`);
};

// === 列出所有输入设备 ===
for (const input of midiAccess.inputs.values()) {
  console.log(`输入: ${input.name} (${input.manufacturer})`);

  // 监听MIDI消息
  input.onmidimessage = (message) => {
    const [status, note, velocity] = message.data;
    const command = status & 0xf0;

    switch (command) {
      case 0x90: // Note On
        if (velocity > 0) {
          const freq = 440 * Math.pow(2, (note - 69) / 12);
          synth.noteOn(freq, velocity / 127);
        } else {
          const freq = 440 * Math.pow(2, (note - 69) / 12);
          synth.noteOff(freq);
        }
        break;
      case 0x80: // Note Off
        const freq = 440 * Math.pow(2, (note - 69) / 12);
        synth.noteOff(freq);
        break;
      case 0xB0: // 控制变化
        console.log(`CC${note} = ${velocity}`);
        break;
      case 0xE0: // 弯音轮
        const pitchBend = ((velocity << 7) | note) - 8192;
        console.log(`Pitch Bend: ${pitchBend}`);
        break;
    }
  };
}
```

### 4.9 AudioWorklet — 自定义音频处理

AudioWorklet 允许在独立的音频渲染线程中运行自定义 DSP 代码，实现零延迟的自定义音频处理。

```javascript
// === 自定义处理器（white-noise-processor.js）===
// 注意：Worklet代码运行在独立线程，不能访问DOM和主线程API

/*
class WhiteNoiseProcessor extends AudioWorkletProcessor {
  // 静态参数描述符
  static get parameterDescriptors() {
    return [
      {
        name: 'amplitude',
        defaultValue: 0.5,
        minValue: 0,
        maxValue: 1,
        automationRate: 'a-rate' // 'a-rate' | 'k-rate'
      }
    ];
  }

  // 处理函数
  // inputs: 输入音频数据 [inputChannel][sample]
  // outputs: 输出音频数据 [outputChannel][sample]
  // parameters: 自动化参数
  process(inputs, outputs, parameters) {
    const output = outputs[0];
    const amplitude = parameters.amplitude;

    for (let channel = 0; channel < output.length; channel++) {
      const outputChannel = output[channel];
      for (let i = 0; i < outputChannel.length; i++) {
        // 参数可能是a-rate（每样本一个值）或k-rate（一个值）
        const amp = amplitude.length > 1 ? amplitude[i] : amplitude[0];
        outputChannel[i] = (Math.random() * 2 - 1) * amp;
      }
    }

    return true; // 返回true保持处理器存活
  }
}

registerProcessor('white-noise-processor', WhiteNoiseProcessor);
*/
```

```javascript
// === 主线程使用Worklet ===
async function useAudioWorklet() {
  const ctx = new AudioContext();

  // 加载Worklet模块
  await ctx.audioWorklet.addModule('/worklets/white-noise-processor.js');

  // 创建Worklet节点
  const noiseNode = new AudioWorkletNode(ctx, 'white-noise-processor');

  // 获取参数
  const amplitudeParam = noiseNode.parameters.get('amplitude');
  amplitudeParam.value = 0.3;

  // 连接输出
  noiseNode.connect(ctx.destination);
  noiseNode.start?.(); // AudioWorkletNode没有start方法，自动开始

  // 动态调整参数
  amplitudeParam.linearRampToValueAtTime(0.8, ctx.currentTime + 2);

  // 主线程与Worklet通信
  noiseNode.port.onmessage = (e) => {
    console.log('Worklet消息:', e.data);
  };
  noiseNode.port.postMessage({ type: 'config', value: 42 });
}
```

#### 自定义效果器示例：Bit Crusher

```javascript
/*
// bit-crusher-processor.js
class BitCrusherProcessor extends AudioWorkletProcessor {
  static get parameterDescriptors() {
    return [
      { name: 'bits', defaultValue: 8, minValue: 1, maxValue: 16 },
      { name: 'frequency', defaultValue: 44100, minValue: 100, maxValue: 44100 }
    ];
  }

  constructor() {
    super();
    this.phase = 0;
    this.lastSample = 0;
  }

  process(inputs, outputs, parameters) {
    const input = inputs[0];
    const output = outputs[0];
    const bits = parameters.bits;
    const freq = parameters.frequency;

    for (let channel = 0; channel < output.length; channel++) {
      const inputChannel = input[channel];
      const outputChannel = output[channel];

      for (let i = 0; i < outputChannel.length; i++) {
        const currentBits = bits.length > 1 ? bits[i] : bits[0];
        const currentFreq = freq.length > 1 ? freq[i] : freq[0];

        // 降采样
        const step = sampleRate / currentFreq;
        this.phase += 1;
        if (this.phase >= step) {
          this.phase -= step;
          // 降位深
          const maxVal = Math.pow(2, currentBits) - 1;
          this.lastSample = Math.round(inputChannel[i] * maxVal) / maxVal;
        }

        outputChannel[i] = this.lastSample;
      }
    }

    return true;
  }
}

registerProcessor('bit-crusher', BitCrusherProcessor);
*/

// 主线程使用
async function useBitCrusher() {
  const ctx = new AudioContext();
  await ctx.audioWorklet.addModule('/worklets/bit-crusher-processor.js');

  const source = ctx.createBufferSource();
  source.buffer = await loadAudio('/sounds/guitar.mp3');

  const crusher = new AudioWorkletNode(ctx, 'bit-crusher');
  const bitsParam = crusher.parameters.get('bits');
  const freqParam = crusher.parameters.get('frequency');

  bitsParam.value = 4;       // 4-bit效果
  freqParam.value = 4000;    // 4kHz采样率

  source.connect(crusher).connect(ctx.destination);
  source.start();
}
```

### 4.10 Media Capture API

#### getUserMedia — 获取摄像头与麦克风

```javascript
// === 基本用法 ===
async function getMedia() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      audio: true,
      video: true
    });

    // 将视频流绑定到<video>元素
    const video = document.querySelector('video');
    video.srcObject = stream;
    await video.play();

  } catch (err) {
    if (err.name === 'NotAllowedError') {
      console.error('用户拒绝了权限请求');
    } else if (err.name === 'NotFoundError') {
      console.error('未找到可用的媒体设备');
    } else if (err.name === 'NotReadableError') {
      console.error('设备被其他应用占用');
    } else {
      console.error('获取媒体失败:', err);
    }
  }
}

// === 约束条件（Constraints）===
const constraints = {
  audio: {
    echoCancellation: true,        // 回声消除
    noiseSuppression: true,        // 降噪
    autoGainControl: true,         // 自动增益
    sampleRate: { ideal: 48000 },  // 期望采样率
    channelCount: { ideal: 2 },    // 期望声道数
    deviceId: { exact: 'device-id' } // 指定设备
  },
  video: {
    width: { ideal: 1920, max: 3840 },
    height: { ideal: 1080, max: 2160 },
    frameRate: { ideal: 30, max: 60 },
    facingMode: 'user',            // 'user'前置 | 'environment'后置
    deviceId: { exact: 'device-id' }
  }
};

const stream = await navigator.mediaDevices.getUserMedia(constraints);

// === 枚举设备 ===
const devices = await navigator.mediaDevices.enumerateDevices();
devices.forEach(device => {
  console.log(`${device.kind}: ${device.label} (${device.deviceId})`);
});
// kind: 'audioinput' | 'audiooutput' | 'videoinput'

// === 切换摄像头 ===
async function switchCamera(videoElement) {
  const devices = await navigator.mediaDevices.enumerateDevices();
  const videoDevices = devices.filter(d => d.kind === 'videoinput');

  if (videoDevices.length < 2) return;

  // 停止当前视频轨道
  const currentTrack = videoElement.srcObject.getVideoTracks()[0];
  currentTrack.stop();

  // 使用下一个摄像头
  const currentIndex = videoDevices.findIndex(
    d => d.deviceId === currentTrack.getSettings().deviceId
  );
  const nextDevice = videoDevices[(currentIndex + 1) % videoDevices.length];

  const newStream = await navigator.mediaDevices.getUserMedia({
    video: { deviceId: { exact: nextDevice.deviceId } }
  });

  videoElement.srcObject = newStream;
}

// === MediaStreamTrack 控制 ===
const tracks = stream.getTracks();
tracks.forEach(track => {
  console.log(`${track.kind}: ${track.label}`);
  console.log('设置:', track.getSettings());
  console.log('约束:', track.getConstraints());
  console.log('能力:', track.getCapabilities());

  // 暂停/恢复
  track.enabled = false; // 暂停（静音/黑屏）
  track.enabled = true;  // 恢复

  // 停止
  track.stop(); // 永久停止，不可恢复

  // 监听事件
  track.onended = () => console.log('轨道已结束');
  track.onmute = () => console.log('轨道已静音');
  track.onunmute = () => console.log('轨道已取消静音');
});

// === 应用约束到已有轨道 ===
const videoTrack = stream.getVideoTracks()[0];
await videoTrack.applyConstraints({
  width: { exact: 1280 },
  height: { exact: 720 },
  frameRate: { exact: 24 }
});

// === 监听设备变化 ===
navigator.mediaDevices.ondevicechange = async () => {
  console.log('设备发生变化');
  const newDevices = await navigator.mediaDevices.enumerateDevices();
  // 更新设备列表UI
};
```

#### getDisplayMedia — 屏幕共享

```javascript
// === 基本屏幕共享 ===
async function startScreenShare() {
  try {
    const stream = await navigator.mediaDevices.getDisplayMedia({
      video: {
        displaySurface: 'monitor',   // 'monitor' | 'window' | 'application' | 'browser'
        width: { ideal: 1920 },
        height: { ideal: 1080 },
        frameRate: { ideal: 30 }
      },
      audio: true // 包含系统音频
    });

    const video = document.querySelector('video');
    video.srcObject = stream;

    // 用户停止共享时处理
    stream.getVideoTracks()[0].onended = () => {
      console.log('屏幕共享已停止');
      stream.getTracks().forEach(track => track.stop());
    };

    return stream;
  } catch (err) {
    if (err.name === 'NotAllowedError') {
      console.error('用户取消了屏幕共享');
    }
  }
}

// === 屏幕共享 + 摄像头画中画 ===
async function screenShareWithCamera() {
  const screenStream = await navigator.mediaDevices.getDisplayMedia({
    video: true, audio: true
  });

  const cameraStream = await navigator.mediaDevices.getUserMedia({
    video: { facingMode: 'user', width: 320, height: 240 }
  });

  // 合并流（使用Canvas合成）
  const canvas = document.createElement('canvas');
  canvas.width = 1920;
  canvas.height = 1080;
  const ctx2d = canvas.getContext('2d');

  const screenVideo = document.createElement('video');
  screenVideo.srcObject = screenStream;
  screenVideo.play();

  const cameraVideo = document.createElement('video');
  cameraVideo.srcObject = cameraStream;
  cameraVideo.play();

  function render() {
    ctx2d.drawImage(screenVideo, 0, 0, 1920, 1080);
    // 摄像头画中画（右下角）
    ctx2d.drawImage(cameraVideo, 1560, 820, 320, 240);
    requestAnimationFrame(render);
  }
  render();

  const combinedStream = canvas.captureStream(30);
  return combinedStream;
}
```

### 4.11 Picture-in-Picture API

```javascript
// === 基本画中画 ===
const video = document.querySelector('video');

async function togglePiP() {
  try {
    if (document.pictureInPictureElement) {
      // 退出画中画
      await document.exitPictureInPicture();
    } else {
      // 进入画中画
      await video.requestPictureInPicture();
    }
  } catch (err) {
    console.error('画中画失败:', err);
  }
}

// === 监听画中画事件 ===
video.onenterpictureinpicture = (event) => {
  console.log('进入画中画');
  // 可获取画中画窗口大小
  const pipWindow = event.pictureInPictureWindow;
  console.log(`窗口大小: ${pipWindow.width}x${pipWindow.height}`);

  pipWindow.onresize = () => {
    console.log(`窗口大小变化: ${pipWindow.width}x${pipWindow.height}`);
  };
};

video.onleavepictureinpicture = () => {
  console.log('退出画中画');
};

// === 检查画中画支持 ===
const isPiPSupported = 'pictureInPictureEnabled' in document
  && document.pictureInPictureEnabled;

if (!isPiPSupported) {
  console.warn('当前浏览器不支持画中画');
}

// === 自定义画中画按钮 ===
const pipButton = document.querySelector('#pip-btn');
pipButton.disabled = !document.pictureInPictureEnabled;

pipButton.addEventListener('click', async () => {
  // 禁用画中画时跳过
  if (!video.disablePictureInPicture) {
    await togglePiP();
  }
});

// === 在画中画中显示摄像头 ===
async function cameraPiP() {
  const stream = await navigator.mediaDevices.getUserMedia({ video: true });
  const video = document.createElement('video');
  video.srcObject = stream;
  video.muted = true;
  await video.play();
  await video.requestPictureInPicture();
}

// === 使用Canvas作为画中画源 ===
async function canvasPiP() {
  const canvas = document.createElement('canvas');
  canvas.width = 640;
  canvas.height = 360;
  const ctx2d = canvas.getContext('2d');

  // 创建一个虚拟视频元素用于画中画
  const stream = canvas.captureStream(30);
  const video = document.createElement('video');
  video.srcObject = stream;
  video.muted = true;
  await video.play();

  // 在Canvas上绘制动态内容
  let frame = 0;
  function animate() {
    ctx2d.fillStyle = '#1a1a2e';
    ctx2d.fillRect(0, 0, 640, 360);
    ctx2d.fillStyle = '#e94560';
    ctx2d.font = '24px monospace';
    ctx2d.fillText(`Frame: ${frame++}`, 20, 40);
    requestAnimationFrame(animate);
  }
  animate();

  await video.requestPictureInPicture();
}
```

---

## 5 常见问题

### AudioContext自动播放策略

浏览器出于用户体验考虑，禁止页面自动播放音频。`AudioContext` 创建后默认处于 `suspended` 状态，必须由用户手势触发才能恢复。

```javascript
// === 解决方案：用户交互时恢复 ===
document.addEventListener('click', async () => {
  if (audioCtx.state === 'suspended') {
    await audioCtx.resume();
  }
}, { once: true });

// === 封装安全的AudioContext ===
class SafeAudioContext {
  constructor() {
    this.ctx = null;
    this._ready = false;
  }

  async init() {
    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
    if (this.ctx.state === 'suspended') {
      await this.ctx.resume();
    }
    this._ready = true;
  }

  get ready() { return this._ready; }
  get context() { return this.ctx; }
}

const safeCtx = new SafeAudioContext();
// 绑定到"开始"按钮
startBtn.addEventListener('click', async () => {
  await safeCtx.init();
  // 开始音频操作...
});
```

### 移动端兼容性

```javascript
// === 移动端注意事项 ===

// 1. iOS Safari特殊处理
// iOS需要用户交互才能创建/恢复AudioContext
// 且每次用户交互只能解锁一次

// 2. 触摸事件解锁
document.addEventListener('touchstart', async () => {
  if (audioCtx.state === 'suspended') {
    await audioCtx.resume();
  }
}, { once: true });

// 3. iOS音频路由变化
// 插拔耳机时可能需要重新创建AudioContext
navigator.mediaDevices.addEventListener('devicechange', () => {
  console.log('音频设备变化');
});

// 4. 移动端采样率可能不同
console.log(audioCtx.sampleRate); // iOS通常是48000，Android可能不同

// 5. webkit前缀兼容
const AudioCtx = window.AudioContext || window.webkitAudioContext;
const audioCtx = new AudioCtx();

// 6. 移动端不支持某些API
if (!navigator.mediaDevices?.getUserMedia) {
  console.warn('当前环境不支持getUserMedia');
}
```

### 音频节点释放与内存管理

```javascript
// === 正确释放音频资源 ===

// 1. BufferSourceNode播放完自动释放，但应手动断开
source.onended = () => {
  source.disconnect();
};

// 2. 停止的节点不可重用，需重新创建
source.stop();
// source.start(); // Error! 不能重新启动

// 3. 关闭AudioContext释放所有资源
await audioCtx.close();

// 4. 释放MediaStream
stream.getTracks().forEach(track => track.stop());

// === 音频缓冲区管理 ===
class AudioManager {
  constructor() {
    this.ctx = new AudioContext();
    this.bufferCache = new Map();
    this.maxCacheSize = 50 * 1024 * 1024; // 50MB上限
    this.currentCacheSize = 0;
  }

  async loadBuffer(url) {
    // 缓存命中
    if (this.bufferCache.has(url)) {
      return this.bufferCache.get(url);
    }

    const response = await fetch(url);
    const arrayBuffer = await response.arrayBuffer();
    const buffer = await this.ctx.decodeAudioData(arrayBuffer);

    const bufferSize = buffer.length * buffer.numberOfChannels * 4;
    this.currentCacheSize += bufferSize;

    // 超出缓存上限时清理最旧的
    if (this.currentCacheSize > this.maxCacheSize) {
      const oldest = this.bufferCache.keys().next().value;
      const oldBuffer = this.bufferCache.get(oldest);
      this.currentCacheSize -= oldBuffer.length * oldBuffer.numberOfChannels * 4;
      this.bufferCache.delete(oldest);
    }

    this.bufferCache.set(url, buffer);
    return buffer;
  }

  dispose() {
    this.bufferCache.clear();
    this.ctx.close();
  }
}
```

### 录音格式支持

```javascript
// === 格式兼容性检查 ===
function checkRecordingSupport() {
  const formats = {
    'audio/webm;codecs=opus': 'Chrome/Firefox/Edge',
    'audio/ogg;codecs=opus': 'Firefox',
    'audio/mp4': 'Safari',
    'audio/wav': '有限支持',
    'audio/pcm': '有限支持'
  };

  const supported = {};
  for (const [mimeType, browser] of Object.entries(formats)) {
    supported[mimeType] = {
      supported: MediaRecorder.isTypeSupported(mimeType),
      browser
    };
  }

  console.table(supported);
  return supported;
}

// === 跨浏览器录音方案 ===
async function recordAudio() {
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

  let recorder;
  let mimeType;

  // 按优先级选择格式
  const preferredTypes = [
    'audio/webm;codecs=opus',
    'audio/webm',
    'audio/ogg;codecs=opus',
    'audio/mp4'
  ];

  for (const type of preferredTypes) {
    if (MediaRecorder.isTypeSupported(type)) {
      mimeType = type;
      break;
    }
  }

  recorder = new MediaRecorder(stream, { mimeType });
  // ... 继续录音逻辑
}
```

---

## 6 面试题

### 题目1：Web Audio API的节点图架构是怎样的？如何实现多路音频混流与分流？

**答：**

Web Audio API 采用**有向无环图（DAG）**架构，由三种节点组成：

- **源节点**（OscillatorNode、AudioBufferSourceNode等）：产生音频信号
- **处理节点**（GainNode、BiquadFilterNode等）：处理音频信号
- **目标节点**（AudioContext.destination）：输出音频信号

节点通过 `connect()` 方法连接。核心特性是**一个输出可以连接多个输入（分流），一个输入可以接收多个输出（混流）**：

```javascript
// 分流：一个源 → 多个处理路径
source.connect(analyser);   // 分支1：分析
source.connect(gainNode);   // 分支2：增益

// 混流：多个源 → 一个目标（自动求和）
osc1.connect(masterGain);   // 轨道1
osc2.connect(masterGain);   // 轨道2
bass.connect(masterGain);   // 低音
masterGain.connect(ctx.destination); // 混流输出
```

混流原理：当多个节点连接到同一目标时，Web Audio引擎自动对信号进行**逐样本相加**。这也是为什么需要 DynamicsCompressorNode 防止多路叠加后信号削波。

### 题目2：AnalyserNode实现音频可视化的原理是什么？getByteFrequencyData和getByteTimeDomainData有何区别？

**答：**

AnalyserNode 内部维护一个 **FFT（快速傅里叶变换）** 分析器，将时域信号转换为频域信号：

- **`getByteFrequencyData()`**：返回**频率域**数据，每个元素对应一个频率区间的能量值（0~255），用于绘制**频谱柱状图**
- **`getByteTimeDomainData()`**：返回**时间域**数据，每个元素对应一个采样点的振幅值（128为中心线，0~255），用于绘制**波形图**

```javascript
const analyser = ctx.createAnalyser();
analyser.fftSize = 2048; // FFT大小，决定频率分辨率

// 频率数据：频谱图
const freqData = new Uint8Array(analyser.frequencyBinCount); // fftSize/2 = 1024
analyser.getByteFrequencyData(freqData);
// freqData[0] = 低频能量, freqData[1023] = 高频能量

// 时间数据：波形图
const timeData = new Uint8Array(analyser.fftSize); // 2048
analyser.getByteTimeDomainData(timeData);
// timeData[i] ≈ 128 ± 振幅偏移
```

关键参数：
- `fftSize`：越大频率分辨率越高，但时间分辨率越低
- `smoothingTimeConstant`：平滑系数（0~1），值越大频谱变化越平滑
- `frequencyBinCount` = `fftSize / 2`，频率区间的数量

### 题目3：AudioContext的自动播放限制是什么？如何正确处理？

**答：**

浏览器出于**防止网页自动播放声音干扰用户**的安全策略，要求 `AudioContext` 必须由**用户手势**（点击、触摸等）触发才能进入 `running` 状态。创建后的 `AudioContext` 默认处于 `suspended` 状态。

处理方式：

```javascript
// 1. 在用户交互事件中创建/恢复
button.addEventListener('click', async () => {
  if (!audioCtx) {
    audioCtx = new AudioContext();
  }
  if (audioCtx.state === 'suspended') {
    await audioCtx.resume();
  }
  // 开始音频操作
});

// 2. 全局解锁（首次交互）
document.addEventListener('click', async () => {
  if (audioCtx?.state === 'suspended') {
    await audioCtx.resume();
  }
}, { once: true });

// 3. 监听状态变化
audioCtx.onstatechange = () => {
  console.log('状态:', audioCtx.state);
  // 'suspended' | 'running' | 'closed'
};
```

注意事项：
- 每次 `suspend()` 后都需要用户手势才能 `resume()`
- iOS Safari限制更严格，每次新交互可能需要重新解锁
- `<audio>` 元素的 `autoplay` 同样受限
- 可以设置 `muted` 属性绕过限制（静音自动播放通常被允许）

### 题目4：如何使用MediaRecorder API实现录音功能？如何处理不同浏览器的格式兼容性？

**答：**

```javascript
async function recordAudio() {
  // 1. 获取麦克风
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

  // 2. 选择兼容的MIME类型
  const mimeType = [
    'audio/webm;codecs=opus',  // Chrome/Firefox/Edge
    'audio/ogg;codecs=opus',   // Firefox
    'audio/mp4',               // Safari
  ].find(t => MediaRecorder.isTypeSupported(t)) || '';

  // 3. 创建录制器
  const recorder = new MediaRecorder(stream, {
    mimeType,
    audioBitsPerSecond: 128000
  });

  const chunks = [];
  recorder.ondataavailable = (e) => {
    if (e.data.size > 0) chunks.push(e.data);
  };

  recorder.onstop = () => {
    const blob = new Blob(chunks, { type: mimeType });
    const url = URL.createObjectURL(blob);
    // 下载或播放
  };

  // 4. 开始录制（带时间片参数确保数据不丢失）
  recorder.start(100); // 每100ms触发一次ondataavailable

  // 5. 停止
  setTimeout(() => {
    recorder.stop();
    stream.getTracks().forEach(t => t.stop()); // 释放麦克风
  }, 5000);
}
```

格式兼容性要点：
- Chrome/Edge/Firefox 支持 `audio/webm;codecs=opus`
- Safari 仅支持 `audio/mp4`
- 使用 `MediaRecorder.isTypeSupported()` 检测
- 不指定 mimeType 让浏览器自动选择是最安全的做法
- 录制过程中可暂停/恢复（`pause()/resume()`）

### 题目5：AudioWorklet的作用是什么？与ScriptProcessorNode有何区别？

**答：**

AudioWorklet 是 Web Audio API 提供的**自定义音频处理机制**，允许开发者在**独立的音频渲染线程**中运行自定义 DSP 代码。

与已废弃的 ScriptProcessorNode 的核心区别：

| 维度 | AudioWorklet | ScriptProcessorNode（已废弃） |
|------|-------------|------------------------------|
| **运行线程** | 独立音频渲染线程 | 主线程 |
| **延迟** | 极低（无主线程阻塞） | 高（受主线程影响） |
| **卡顿** | 不会因JS阻塞产生卡顿 | 主线程繁忙时产生卡顿 |
| **通信** | 通过MessagePort | 通过事件回调 |
| **生命周期** | 持续运行直到返回false | 每次事件触发回调 |

```javascript
// Worklet处理器（独立文件）
class MyProcessor extends AudioWorkletProcessor {
  static get parameterDescriptors() {
    return [{ name: 'gain', defaultValue: 1 }];
  }

  process(inputs, outputs, parameters) {
    const input = inputs[0];
    const output = outputs[0];

    for (let ch = 0; ch < output.length; ch++) {
      for (let i = 0; i < output[ch].length; i++) {
        output[ch][i] = input[ch][i] * parameters.gain[0];
      }
    }
    return true; // 保持存活
  }
}
registerProcessor('my-processor', MyProcessor);

// 主线程使用
await ctx.audioWorklet.addModule('/worklets/my-processor.js');
const node = new AudioWorkletNode(ctx, 'my-processor');
```

适用场景：自定义滤波器、物理建模合成、实时音频效果器、音频分析算法等需要低延迟的DSP场景。

### 题目6：如何实现3D空间音频效果？PannerNode的关键参数有哪些？

**答：**

3D空间音频通过 `PannerNode` 和 `AudioListener` 配合实现，核心原理是模拟**声源位置**与**听者位置**的相对关系，通过 HRTF（头相关传递函数）计算双耳信号差异：

```javascript
const panner = ctx.createPanner();

// 关键参数：
// 1. panningModel — 声像模型
panner.panningModel = 'HRTF'; // 头相关传递函数，最真实的3D效果
// 'equalpower' — 简单等功率声像，性能更好但不够真实

// 2. distanceModel — 距离衰减模型
panner.distanceModel = 'inverse'; // 反比衰减（最常用，符合物理）
// 'linear' — 线性衰减
// 'exponential' — 指数衰减

// 3. 距离参数
panner.refDistance = 1;      // 参考距离（不衰减的距离）
panner.maxDistance = 10000;  // 最大距离
panner.rolloffFactor = 1;   // 衰减速度

// 4. 音源位置
panner.positionX.value = 5;
panner.positionY.value = 0;
panner.positionZ.value = -3;

// 5. 音源方向（锥形指向）
panner.orientationX.value = 1;
panner.coneInnerAngle = 360;  // 内锥角度（全音量）
panner.coneOuterAngle = 360;  // 外锥角度
panner.coneOuterGain = 0;     // 外锥音量

// 听者位置
const listener = ctx.listener;
listener.positionX.value = 0;
listener.positionY.value = 0;
listener.positionZ.value = 0;
```

游戏中的使用：每帧更新音源位置和听者位置/朝向，PannerNode自动计算正确的空间声像。

### 题目7：getUserMedia的约束条件如何工作？ideal、exact、min、max分别代表什么？

**答：**

getUserMedia 的约束条件分两类：**强制约束**和**理想约束**：

```javascript
const constraints = {
  video: {
    // ideal — 优先选择最接近的值，不满足也不会报错
    width: { ideal: 1920 },
    height: { ideal: 1080 },

    // exact — 必须精确匹配，不满足则抛出OverconstrainedError
    facingMode: { exact: 'environment' }, // 必须用后置摄像头

    // min/max — 范围约束
    frameRate: { min: 24, max: 60 }
  },
  audio: {
    // 布尔值简写
    echoCancellation: true  // 等价于 { exact: true }
  }
};
```

关键规则：
- `ideal`：浏览器尽量满足，不满足时选择最接近的值（不会报错）
- `exact`：必须精确匹配，否则抛出 `OverconstrainedError`
- `min`/`max`：定义可接受范围
- 同时指定 `ideal` 和 `exact` 无意义（exact优先）
- 约束冲突时（多个exact互相矛盾），抛出 `OverconstrainedError`

```javascript
// 处理约束错误
try {
  const stream = await navigator.mediaDevices.getUserMedia(constraints);
} catch (err) {
  if (err.name === 'OverconstrainedError') {
    console.error('约束无法满足:', err.constraint);
    // 降级：使用宽松约束重试
    const fallbackStream = await navigator.mediaDevices.getUserMedia({
      video: true // 无约束
    });
  }
}
```

### 题目8：如何优化Web Audio API的音频延迟？有哪些最佳实践？

**答：**

音频延迟优化从多个层面入手：

**1. AudioContext配置优化**

```javascript
// 创建低延迟AudioContext
const ctx = new AudioContext({
  sampleRate: 44100,       // 固定采样率
  latencyHint: 'interactive' // 'balanced' | 'interactive' | 'playback'
});
// 'interactive'：最低延迟，适合游戏/乐器
// 'playback'：最高延迟，适合音乐播放（更省电）
```

**2. AudioWorklet替代ScriptProcessorNode**

ScriptProcessorNode在主线程运行，受UI渲染等影响产生卡顿。AudioWorklet在独立线程运行，保证稳定低延迟。

**3. 缓冲区管理**

```javascript
// 预加载音频缓冲区，避免播放时解码
const buffer = await ctx.decodeAudioData(arrayBuffer);
// 使用时直接创建BufferSource，无需等待
const source = ctx.createBufferSource();
source.buffer = buffer; // 已解码，即时可用
```

**4. 使用精确调度**

```javascript
// 错误：setTimeout调度（不准确）
setTimeout(() => source.start(), 100); // 延迟不确定

// 正确：基于AudioContext时间线调度
source.start(ctx.currentTime + 0.1); // 精确100ms后
```

**5. 节点图优化**

- 减少不必要的处理节点，缩短信号链
- 对不活动的节点及时 `disconnect()`
- 避免在动画帧中创建新节点

**6. 移动端特殊处理**

- iOS的Web Audio延迟通常比桌面高
- 使用 `webkitAudioContext` 兼容旧版iOS
- 避免在触摸事件中创建大量节点

**7. 监控延迟**

```javascript
// 测量输出延迟
const baseLatency = ctx.baseLatency;   // AudioContext到OS的延迟
const outputLatency = ctx.outputLatency; // OS到扬声器的延迟
console.log(`总延迟: ${((baseLatency + outputLatency) * 1000).toFixed(1)}ms`);
```

最佳实践总结：选择 `interactive` 延迟提示、使用 AudioWorklet、预加载缓冲区、基于 `currentTime` 调度、精简节点图、及时释放资源。
