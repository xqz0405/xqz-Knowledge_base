---
tags:
  - Web前端
  - WebRTC
  - 实时通信
  - 音视频
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# WebRTC 实时通信

## What — 是什么

**WebRTC（Web Real-Time Communication）** 是一组开源的浏览器 API 与协议标准，允许网页应用在无需安装插件的前提下，直接进行**实时音视频通信**和**点对点数据传输**。

### 核心架构

```
┌──────────────────────────────────────────────────────────────┐
│                      Web Browser A                           │
│  ┌────────────┐  ┌───────────────────┐  ┌────────────────┐  │
│  │ MediaStream│──▶│ RTCPeerConnection │──▶│  DataChannel   │  │
│  │ (摄像头/麦克)│  │  (P2P 连接管理)    │  │  (数据通道)     │  │
│  └────────────┘  └─────────┬─────────┘  └────────────────┘  │
│                            │                                 │
└────────────────────────────┼─────────────────────────────────┘
                             │ ICE / STUN / TURN
                     ┌───────▼───────┐
                     │  Signaling    │
                     │  Server       │  ◀── SDP Offer/Answer
                     │  (信令服务器)  │      ICE Candidate 交换
                     └───────┬───────┘
                             │
┌────────────────────────────┼─────────────────────────────────┐
│                      Web Browser B                           │
│  ┌────────────┐  ┌─────────▼─────────┐  ┌────────────────┐  │
│  │ MediaStream│──▶│ RTCPeerConnection │──▶│  DataChannel   │  │
│  │ (摄像头/麦克)│  │  (P2P 连接管理)    │  │  (数据通道)     │  │
│  └────────────┘  └───────────────────┘  └────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 关键组件

| 组件 | 作用 |
|------|------|
| **MediaStream API** | 获取本地音视频流（摄像头、麦克风、屏幕共享） |
| **RTCPeerConnection** | 建立和管理 P2P 连接，处理编解码、NAT 穿透、传输 |
| **RTCDataChannel** | P2P 数据通道，支持文本/二进制数据传输 |
| **SDP（Session Description Protocol）** | 描述会话元信息的文本协议，通过 Offer/Answer 模型交换 |
| **ICE（Interactive Connectivity Establishment）** | 连接候选收集框架，统筹 STUN/TURN 发现可连通路径 |
| **STUN（Session Traversal Utilities for NAT）** | 帮助客户端发现自身公网 IP 和 NAT 映射 |
| **TURN（Traversal Using Relays around NAT）** | 当 P2P 直连失败时，通过中继服务器转发流量 |

### SDP Offer/Answer 流程

```
Caller (A)                              Callee (B)
    │                                       │
    │──── createOffer() ────┐               │
    │◀── SDP Offer ─────────┘               │
    │                                       │
    │──── setLocalDescription(offer) ──▶    │
    │                                       │
    │─────── SDP Offer (via Signaling) ────▶│
    │                                       │
    │              setRemoteDescription(offer)
    │              createAnswer()            │
    │              setLocalDescription(answer)
    │                                       │
    │◀─────── SDP Answer (via Signaling) ───│
    │                                       │
    │  setRemoteDescription(answer)         │
    │                                       │
    │◀═══════ ICE Candidates ══════════════▶│
    │                                       │
    │═══════ P2P Connection Established ════│
```

## Why — 为什么

### 技术方案对比

| 特性 | WebRTC | WebSocket | HTTP Streaming | SSE |
|------|--------|-----------|----------------|-----|
| **通信模式** | P2P 点对点 | C/S 客户端-服务器 | C/S 请求-响应流 | C/S 服务器推送 |
| **传输协议** | UDP（SRTP/SCTP） | TCP | TCP | TCP |
| **延迟** | 极低（< 500ms） | 低（~100ms） | 中 | 中 |
| **音视频支持** | 原生内置 | 需自行编码传输 | 需自行编码传输 | 不支持 |
| **数据传输** | DataChannel（可靠/不可靠） | 文本/二进制帧 | 文本/二进制流 | 仅文本 |
| **NAT 穿透** | ICE/STUN/TURN 内置 | 无（服务器中转） | 无 | 无 |
| **带宽成本** | 低（P2P 不经服务器） | 高（所有流量经服务器） | 高 | 高 |
| **连接规模** | 1对1 / 小组 | 大规模广播 | 大规模下载 | 大规模推送 |
| **浏览器支持** | 现代浏览器原生支持 | 现代浏览器原生支持 | 现代浏览器 | 现代浏览器 |
| **适用场景** | 视频通话、P2P文件传输 | 聊天、实时推送 | 视频点播、下载 | 通知、股票行情 |

### 典型应用场景

1. **视频通话 / 会议** — Zoom Web 版、Google Meet、微信网页版
2. **屏幕共享** — 远程协助、在线演示、录屏工具
3. **P2P 文件传输** — 无需服务器中转的大文件直传（如 WebTorrent）
4. **IoT 设备通信** — 浏览器直连摄像头、传感器等设备
5. **低延迟直播** — 超低延迟互动直播（连麦场景）
6. **云游戏** — 视频流 + 输入通道的低延迟交互

## How — 怎么用

### 1. getUserMedia — 获取摄像头和麦克风

```js
// 获取摄像头和麦克风
async function getLocalStream() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: {
        width: { ideal: 1280 },
        height: { ideal: 720 },
        frameRate: { ideal: 30 }
      },
      audio: {
        echoCancellation: true,   // 回声消除
        noiseSuppression: true,   // 降噪
        autoGainControl: true     // 自动增益
      }
    });

    // 将流绑定到 video 元素
    const localVideo = document.getElementById('localVideo');
    localVideo.srcObject = stream;
    return stream;
  } catch (err) {
    console.error('获取媒体设备失败:', err);
    throw err;
  }
}
```

```js
// 仅获取音频
const audioStream = await navigator.mediaDevices.getUserMedia({ audio: true });

// 枚举可用设备
const devices = await navigator.mediaDevices.enumerateDevices();
devices.forEach(device => {
  console.log(`${device.kind}: ${device.label} (${device.deviceId})`);
});
```

### 2. RTCPeerConnection — 建立连接

```js
const config = {
  iceServers: [
    // Google 公共 STUN 服务器
    { urls: 'stun:stun.l.google.com:19302' },
    // TURN 服务器（P2P 失败时中继）
    {
      urls: 'turn:turn.example.com:3478',
      username: 'user',
      credential: 'pass'
    }
  ]
};

const pc = new RTCPeerConnection(config);

// 添加本地音视频轨道
const localStream = await getLocalStream();
localStream.getTracks().forEach(track => {
  pc.addTrack(track, localStream);
});

// 监听远端轨道
pc.ontrack = (event) => {
  const remoteVideo = document.getElementById('remoteVideo');
  remoteVideo.srcObject = event.streams[0];
};

// 监听 ICE 候选
pc.onicecandidate = (event) => {
  if (event.candidate) {
    // 通过信令服务器发送候选给对端
    signalingServer.emit('ice-candidate', event.candidate);
  }
};

// 监听连接状态
pc.onconnectionstatechange = () => {
  console.log('连接状态:', pc.connectionState);
  // possible: 'connected' | 'disconnected' | 'failed' | 'closed'
};
```

### 3. ICE 候选交换

```js
// Caller 端：创建 Offer
async function createOffer() {
  const offer = await pc.createOffer();
  await pc.setLocalDescription(offer);
  signalingServer.emit('offer', offer);
  // 此后 onicecandidate 会触发，将候选发送给对端
}

// Callee 端：接收 Offer，创建 Answer
signalingServer.on('offer', async (offer) => {
  await pc.setRemoteDescription(new RTCSessionDescription(offer));
  const answer = await pc.createAnswer();
  await pc.setLocalDescription(answer);
  signalingServer.emit('answer', answer);
});

// 双方：接收对端的 ICE 候选
signalingServer.on('ice-candidate', async (candidate) => {
  try {
    await pc.addIceCandidate(new RTCIceCandidate(candidate));
  } catch (err) {
    console.error('添加 ICE 候选失败:', err);
  }
});

// Caller 端：接收 Answer
signalingServer.on('answer', async (answer) => {
  await pc.setRemoteDescription(new RTCSessionDescription(answer));
});
```

### 4. 完整 1 对 1 视频通话示例

```js
// ============================================
// signaling.js — 基于 Socket.IO 的信令客户端
// ============================================
class SignalingClient {
  constructor(roomId) {
    this.socket = io('/', { query: { roomId } });
    this.socket.on('connect', () => {
      console.log('信令服务器已连接, ID:', this.socket.id);
    });
  }

  on(event, callback) {
    this.socket.on(event, callback);
  }

  emit(event, data) {
    this.socket.emit(event, data);
  }
}

// ============================================
// webrtc-call.js — 1 对 1 视频通话核心逻辑
// ============================================
class VideoCall {
  constructor(localVideoEl, remoteVideoEl, roomId) {
    this.localVideo = localVideoEl;
    this.remoteVideo = remoteVideoEl;
    this.pc = null;
    this.localStream = null;
    this.signaling = new SignalingClient(roomId);
    this._setupSignaling();
  }

  // 初始化：获取本地流并创建 PeerConnection
  async init() {
    this.localStream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true
    });
    this.localVideo.srcObject = this.localStream;

    this.pc = new RTCPeerConnection({
      iceServers: [
        { urls: 'stun:stun.l.google.com:19302' },
        { urls: 'turn:turn.example.com:3478', username: 'user', credential: 'pass' }
      ]
    });

    // 添加本地轨道
    this.localStream.getTracks().forEach(track => {
      this.pc.addTrack(track, this.localStream);
    });

    // 监听远端轨道
    this.pc.ontrack = (event) => {
      this.remoteVideo.srcObject = event.streams[0];
    };

    // 转发 ICE 候选
    this.pc.onicecandidate = (event) => {
      if (event.candidate) {
        this.signaling.emit('ice-candidate', event.candidate);
      }
    };

    this.pc.onconnectionstatechange = () => {
      console.log('连接状态:', this.pc.connectionState);
    };
  }

  // 作为呼叫方发起通话
  async call() {
    await this.init();
    const offer = await this.pc.createOffer();
    await this.pc.setLocalDescription(offer);
    this.signaling.emit('offer', offer);
  }

  // 信令事件绑定
  _setupSignaling() {
    // 接收 Offer（被呼叫方）
    this.signaling.on('offer', async (offer) => {
      await this.init();
      await this.pc.setRemoteDescription(new RTCSessionDescription(offer));
      const answer = await this.pc.createAnswer();
      await this.pc.setLocalDescription(answer);
      this.signaling.emit('answer', answer);
    });

    // 接收 Answer（呼叫方）
    this.signaling.on('answer', async (answer) => {
      await this.pc.setRemoteDescription(new RTCSessionDescription(answer));
    });

    // 接收 ICE 候选
    this.signaling.on('ice-candidate', async (candidate) => {
      if (this.pc) {
        await this.pc.addIceCandidate(new RTCIceCandidate(candidate));
      }
    });
  }

  // 挂断
  hangUp() {
    if (this.pc) {
      this.pc.close();
      this.pc = null;
    }
    if (this.localStream) {
      this.localStream.getTracks().forEach(track => track.stop());
      this.localStream = null;
    }
    this.localVideo.srcObject = null;
    this.remoteVideo.srcObject = null;
    this.signaling.emit('hang-up');
  }
}

// 使用
// const call = new VideoCall(localVideoEl, remoteVideoEl, 'room-123');
// call.call();   // 发起
// call.hangUp(); // 挂断
```

### 5. DataChannel — P2P 数据传输

```js
// 创建 DataChannel（由发起方创建）
const dc = pc.createDataChannel('chat', {
  ordered: true,           // 保证顺序
  maxRetransmits: 3        // 最多重传 3 次
});

// 接收方监听
pc.ondatachannel = (event) => {
  const receiveChannel = event.channel;
  receiveChannel.onmessage = (e) => {
    console.log('收到消息:', e.data);
  };
  receiveChannel.onopen = () => console.log('数据通道已打开');
  receiveChannel.onclose = () => console.log('数据通道已关闭');
};

// 发送文本消息
dc.onopen = () => {
  dc.send('Hello via DataChannel!');
};

// 发送二进制文件
async function sendFile(file) {
  const arrayBuffer = await file.arrayBuffer();
  // 先发送文件元信息
  dc.send(JSON.stringify({
    type: 'file-meta',
    name: file.name,
    size: file.size,
    mimeType: file.type
  }));
  // 分片发送（DataChannel 单条消息限制约 64KB-256KB）
  const CHUNK_SIZE = 16384; // 16KB
  let offset = 0;
  while (offset < arrayBuffer.byteLength) {
    const chunk = arrayBuffer.slice(offset, offset + CHUNK_SIZE);
    dc.send(chunk);
    offset += CHUNK_SIZE;
  }
  dc.send(JSON.stringify({ type: 'file-end' }));
}

// 接收方组装文件
let fileBuffer = [];
let fileMeta = null;

dc.onmessage = (event) => {
  if (typeof event.data === 'string') {
    const msg = JSON.parse(event.data);
    if (msg.type === 'file-meta') fileMeta = msg;
    if (msg.type === 'file-end') {
      const blob = new Blob(fileBuffer, { type: fileMeta.mimeType });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = fileMeta.name;
      a.click();
      fileBuffer = [];
      fileMeta = null;
    }
  } else {
    // ArrayBuffer 块
    fileBuffer.push(event.data);
  }
};
```

### 6. 屏幕共享 — getDisplayMedia

```js
async function startScreenShare(pc, localStream) {
  // 先停止摄像头轨道
  localStream.getVideoTracks().forEach(track => track.stop());

  // 获取屏幕共享流
  const screenStream = await navigator.mediaDevices.getDisplayMedia({
    video: {
      width: { ideal: 1920 },
      height: { ideal: 1080 },
      frameRate: { ideal: 30 }
    },
    audio: true  // 可选：包含系统音频
  });

  // 替换 PeerConnection 中的视频轨道
  const screenTrack = screenStream.getVideoTracks()[0];
  const sender = pc.getSenders().find(s => s.track?.kind === 'video');
  await sender.replaceTrack(screenTrack);

  // 监听用户手动停止共享
  screenTrack.onended = async () => {
    // 恢复摄像头
    const camStream = await navigator.mediaDevices.getUserMedia({ video: true });
    const camTrack = camStream.getVideoTracks()[0];
    await sender.replaceTrack(camTrack);
  };

  return screenStream;
}
```

### 7. 信令服务器（Socket.IO 示例）

```js
// server.js — Node.js 信令服务器
const { Server } = require('socket.io');

const io = new Server(3000, {
  cors: { origin: '*' }
});

const rooms = new Map(); // roomId -> Set<socketId>

io.on('connection', (socket) => {
  const { roomId } = socket.handshake.query;
  socket.join(roomId);

  // 通知房间内其他人
  socket.to(roomId).emit('peer-joined', { peerId: socket.id });

  socket.on('offer', (offer) => {
    socket.to(roomId).emit('offer', offer);
  });

  socket.on('answer', (answer) => {
    socket.to(roomId).emit('answer', answer);
  });

  socket.on('ice-candidate', (candidate) => {
    socket.to(roomId).emit('ice-candidate', candidate);
  });

  socket.on('hang-up', () => {
    socket.to(roomId).emit('peer-left', { peerId: socket.id });
  });

  socket.on('disconnect', () => {
    socket.to(roomId).emit('peer-left', { peerId: socket.id });
  });
});

console.log('信令服务器运行在 http://localhost:3000');
```

### 8. TURN 服务器配置 — NAT 穿透

当双方均处于对称型 NAT（Symmetric NAT）时，STUN 无法完成穿透，必须依赖 TURN 中继：

```js
// 生产环境 TURN 配置示例
const peerConfig = {
  iceServers: [
    // STUN — 发现公网 IP
    { urls: 'stun:stun.l.google.com:19302' },

    // TURN — 中继兜底
    {
      urls: [
        'turn:turn.example.com:3478?transport=udp',
        'turn:turn.example.com:3478?transport=tcp',
        'turns:turn.example.com:5349?transport=tcp'  // TLS 加密中继
      ],
      username: '1683825600:user1',      // 时限型凭证
      credential: 'generated-credential'  // 由 TURN 服务端算法生成
    }
  ],
  iceTransportPolicy: 'all'  // 'all' 使用所有候选 | 'relay' 仅用 TURN
};
```

```bash
# 使用 coturn 搭建 TURN 服务器
sudo apt install coturn

# /etc/turnserver.conf 关键配置
listening-port=3478
tls-listening-port=5349
realm=example.com
server-name=turn.example.com
lt-cred-mech                    # 长期凭证机制
user=user1:pass1                # 用户凭证
fingerprint
cert=/etc/ssl/certs/cert.pem
pkey=/etc/ssl/private/key.pem
log-file=/var/log/turnserver.log
```

### 9. 常见问题与踩坑

| # | 问题 | 原因 | 解决方案 |
|---|------|------|----------|
| 1 | `getUserMedia` 报 `NotAllowedError` | 用户拒绝授权或页面非 HTTPS | 使用 HTTPS；先检查权限状态 `navigator.permissions.query` |
| 2 | `NotFoundError` 找不到设备 | 无摄像头/麦克风或设备被占用 | 先 `enumerateDevices()` 检查可用设备；提示用户检查硬件 |
| 3 | 远端无画面 | `addTrack` 在 `setRemoteDescription` 之前/之后时机不对 | Offer 方先 addTrack 再 createOffer；Answer 方在 ontrack 回调中获取流 |
| 4 | ICE 连接 `failed` | NAT 类型不兼容（对称型 NAT） | 配置 TURN 中继服务器；`iceTransportPolicy: 'relay'` 强制走中继 |
| 5 | 连接建立后一段时间 `disconnected` | 网络波动导致 ICE 连接断开 | 监听 `oniceconnectionstatechange`，实现 ICE Restart：重新 createOffer |
| 6 | DataChannel 单条消息过大被截断 | SCTP 消息大小受限（通常 ~256KB） | 分片发送，接收方组装；使用 `dc.bufferedAmount` 控制发送速率 |
| 7 | 多人通话中音频回声 | 未开启回声消除或麦克风收录扬声器 | `audio: { echoCancellation: true }`；建议用户戴耳机 |
| 8 | iOS Safari 兼容性问题 | iOS 对 WebRTC 有特殊限制 | 视频必须由用户手势触发；使用 `playsinline` 属性；H264 编解码 |
| 9 | 切换前后摄像头失败 | 直接 replaceTrack 未考虑约束 | 调用 `getUserMedia({ video: { facingMode: 'user'/'environment' } })` 重新获取 |
| 10 | 屏幕共享后恢复摄像头黑屏 | 替换 track 后原 stream 被释放 | 维护对摄像头 stream 的引用，恢复时重新 `replaceTrack` |

### 10. 最佳实践

```js
// ✅ 1. 始终使用 HTTPS（WebRTC 强制要求安全上下文）
// 开发环境可用 localhost 或 chrome://flags/#unsafely-treat-insecure-origin-as-secure

// ✅ 2. 妥善释放资源
function cleanup(pc, localStream) {
  if (pc) {
    pc.getSenders().forEach(sender => pc.removeTrack(sender));
    pc.close();
  }
  if (localStream) {
    localStream.getTracks().forEach(track => track.stop());
  }
}

// ✅ 3. 实现重连机制
pc.oniceconnectionstatechange = () => {
  if (pc.iceConnectionState === 'failed') {
    pc.restartIce(); // ICE Restart
  }
};

// ✅ 4. 适配不同网络环境的编解码器
const pc = new RTCPeerConnection({
  iceServers: [...],
  sdpSemantics: 'unified-plan',  // 推荐 unified-plan
  bundlePolicy: 'max-bundle',     // 复用单一传输通道
  rtcpMuxPolicy: 'require'        // RTP/RTCP 复用
});

// ✅ 5. 监控连接质量
setInterval(() => {
  pc.getStats(null).then(stats => {
    stats.forEach(report => {
      if (report.type === 'outbound-rtp' && report.kind === 'video') {
        console.log('视频发送码率:', report.bytesSent);
        console.log('帧率:', report.framesPerSecond);
      }
      if (report.type === 'candidate-pair' && report.state === 'succeeded') {
        console.log('RTT:', report.currentRoundTripTime, 's');
      }
    });
  });
}, 5000);

// ✅ 6. 弱网降级策略
async function adaptToNetwork(pc) {
  const stats = await pc.getStats();
  stats.forEach(report => {
    if (report.type === 'candidate-pair' && report.state === 'succeeded') {
      const rtt = report.currentRoundTripTime * 1000;
      if (rtt > 500) {
        // 高延迟：降低分辨率
        const sender = pc.getSenders().find(s => s.track?.kind === 'video');
        const params = sender.getParameters();
        params.encodings[0].maxBitrate = 300_000;  // 300kbps
        params.encodings[0].scaleResolutionDownBy = 2;
        sender.setParameters(params);
      }
    }
  });
}

// ✅ 7. 多人会议使用 SFU 架构而非 Mesh
// Mesh: 每人与所有人建立 P2P → O(n²) 带宽
// SFU:  每人只与服务器建立连接 → O(n) 带宽（如 mediasoup, Janus, LiveKit）
```

## 面试题

### 1. WebRTC 和 WebSocket 的区别是什么？能否互相替代？

**核心区别：** WebRTC 基于 UDP，面向 P2P 实时音视频；WebSocket 基于 TCP，面向服务器中继的可靠消息。

| 维度 | WebRTC | WebSocket |
|------|--------|-----------|
| 传输层 | UDP（SRTP/SCTP） | TCP |
| 连接模式 | P2P 点对点 | 客户端-服务器 |
| 音视频 | 原生支持编解码 | 不支持，需手动编码 |
| 延迟 | 极低 | 较低但受 TCP 影响 |
| NAT 穿透 | ICE/STUN/TURN 内置 | 无，需服务器中转 |

**不能互相替代：** 实际上它们经常配合使用——WebSocket 充当信令通道交换 SDP 和 ICE 候选，WebRTC 负责媒体和数据传输。用 WebSocket 传音视频可行但延迟高且服务器带宽成本大；用 WebRTC 做信令则过于复杂且无必要。

---

### 2. 解释 ICE、STUN、TURN 三者的关系和作用

**ICE** 是一个框架，它协调 STUN 和 TURN 来找到两端之间可连通的路径：

- **STUN** — "你看到的我是谁？"客户端向 STUN 服务器发送请求，服务器返回客户端的公网 IP:Port。适用于锥型 NAT，让两端互相知道对方的公网地址后直接 P2P 连通。
- **TURN** — "我连不上你，帮我转发。"当 STUN 发现的地址仍无法连通（如对称型 NAT），则通过 TURN 服务器中继所有流量，此时实际上已经不是 P2P 了。
- **ICE** — 统筹以上两者，收集所有候选地址（本地、STUN 反射、TURN 中继），按优先级逐一尝试连通，选择最优路径。

**候选优先级：** 主机候选（本地IP） > 反射候选（STUN） > 中继候选（TURN）。

---

### 3. WebRTC 为什么需要信令服务器？信令服务器传输什么内容？

WebRTC 本身不定义信令协议，需要开发者自行实现信令交换。信令服务器负责在两端之间传递：

1. **SDP Offer/Answer** — 包含媒体编解码能力、传输协议、带宽约束等会话描述信息
2. **ICE Candidates** — 包含 IP:Port 候选地址，用于连通性检测

信令服务器可以用 **WebSocket、Socket.IO、HTTP 轮询** 等任何方式实现，它只在连接建立阶段使用，连接建立后的音视频数据走 P2P 通道，不再经过信令服务器。

---

### 4. 什么是 SDP？WebRTC 中的 SDP 包含哪些关键信息？

**SDP（Session Description Protocol）** 是纯文本格式的会话描述协议，WebRTC 用它来协商双方的能力和参数：

```
v=0                                          // 版本
o=- 123456789 2 IN IP4 127.0.0.1            // 会话来源
s=-                                          // 会话名
t=0 0                                        // 时间
a=group:BUNDLE 0 1                           // 音视频捆绑传输
m=audio 9 UDP/TLS/RTP/SAVPF 111             // 音频媒体行
a=rtpmap:111 opus/48000/2                    // Opus 编解码器
m=video 9 UDP/TLS/RTP/SAVPF 96              // 视频媒体行
a=rtpmap:96 VP8/90000                        // VP8 编解码器
a=fmtp:96 max-fr=30;max-fs=3600             // 帧率/分辨率约束
a=ice-ufrag:abcd                             // ICE 用户名片段
a=ice-pwd:1234567890abcdef                   // ICE 密码
a=fingerprint:sha-256 XX:XX:...              // DTLS 指纹
a=candidate:...                              // ICE 候选地址
```

关键信息包括：编解码器列表、ICE 凭证、DTLS 指纹、候选地址、媒体约束等。注意：不要手动修改 SDP 字符串，应使用 WebRTC API 修改。

---

### 5. MediaStream 中的 Track 和 Stream 是什么关系？如何动态替换 Track？

```js
// 一个 MediaStream 包含多个 MediaStreamTrack
// stream = [VideoTrack, AudioTrack]

// 获取轨道
const videoTrack = stream.getVideoTracks()[0];
const audioTrack = stream.getAudioTracks()[0];

// 轨道状态
videoTrack.enabled;      // true/false 是否静音/黑屏（不断流）
videoTrack.readyState;   // 'live' | 'ended'
videoTrack.muted;        // 是否被系统静音

// 动态替换 Track（如切换摄像头/屏幕共享）
// 不需要重新建立 PeerConnection，使用 replaceTrack 无缝切换
const sender = pc.getSenders().find(s => s.track?.kind === 'video');
await sender.replaceTrack(newVideoTrack);
// replaceTrack 是原子操作，不会中断对端播放
```

**Stream 与 Track 的关系：** Stream 是 Track 的容器，一个 Track 只属于一个 Stream。`enabled = false` 会发送静音/黑帧但保持连接；`stop()` 则彻底终止轨道。

---

### 6. DataChannel 和 WebSocket 有什么区别？什么场景下用 DataChannel？

| 特性 | DataChannel | WebSocket |
|------|-------------|-----------|
| 传输层 | SCTP over DTLS（可配置可靠/不可靠） | TCP（始终可靠） |
| 延迟 | 极低（P2P 直连） | 较低（经服务器中转） |
| 服务器负载 | 无（数据不经服务器） | 所有流量经服务器 |
| 可靠性 | 可配置：可靠有序 / 部分可靠 / 不可靠 | 始终可靠有序 |
| 消息大小 | 受限（~256KB），需分片 | 无硬性限制 |
| 建立依赖 | 需先建立 RTCPeerConnection | 直接连接 WebSocket 服务器 |

**适用场景：**
- **游戏实时输入** — 不可靠低延迟模式，丢包可容忍
- **P2P 文件传输** — 可靠模式，不经服务器节省带宽
- **协同编辑** — 可靠有序模式，低延迟同步操作
- 与 WebRTC 音视频通话配套的控制信令（如远程桌面指令）

---

### 7. 什么是 NAT 穿透？WebRTC 如何处理 NAT 问题？

**NAT（网络地址转换）** 让内网设备共享公网 IP，但也导致外网无法直接访问内网地址。NAT 类型：

| NAT 类型 | 穿透难度 | 说明 |
|----------|----------|------|
| Full Cone | 容易 | 任何外部主机都能通过映射端口访问 |
| Restricted Cone | 中等 | 仅允许曾发过包的 IP 回复 |
| Port Restricted Cone | 较难 | IP + Port 都要匹配 |
| Symmetric | 极难 | 每个目标分配不同映射端口，STUN 无法预知 |

**WebRTC 的 ICE 处理流程：**
1. 收集主机候选（本机 IP）
2. 通过 STUN 获取反射候选（公网 IP:Port）
3. 获取 TURN 中继候选（中继服务器地址）
4. 按优先级对候选对进行连通性检查
5. 选择最优可连通路径，若全部失败则使用 TURN 中继

**关键：** 对称型 NAT 场景下 STUN 失效，必须依赖 TURN 中继。生产环境务必部署 TURN 服务器。

---

### 8. 如何优化 WebRTC 的性能？弱网环境下有哪些策略？

**性能优化方向：**

```js
// 1. 编解码器选择
// 视频：VP8 兼容性好 / VP9 压缩率高 / H264 硬件加速广泛 / AV1 最优压缩
// 音频：Opus 通用标准，支持 6-510kbps 自适应码率

// 2. 码率控制与自适应
const sender = pc.getSenders().find(s => s.track?.kind === 'video');
const params = sender.getParameters();
params.encodings[0] = {
  maxBitrate: 1_000_000,         // 最大码率 1Mbps
  scaleResolutionDownBy: 1,      // 分辨率缩放因子
  maxFramerate: 30               // 最大帧率
};
sender.setParameters(params);

// 3. 弱网降级策略
// RTT > 300ms → 降分辨率 scaleResolutionDownBy = 2
// 丢包率 > 10% → 降帧率 maxFramerate = 15
// 丢包率 > 30% → 仅保留音频，禁用视频

// 4. 使用 getStats 监控连接质量
pc.getStats(null).then(stats => {
  stats.forEach(report => {
    if (report.type === 'outbound-rtp') {
      console.log('发送码率:', report.bytesSent);
      console.log('丢包数:', report.packetsLost);
    }
    if (report.type === 'inbound-rtp') {
      console.log('抖动:', report.jitter);
      console.log('丢包率:', report.packetsLost / report.packetsReceived);
    }
  });
});

// 5. SIMULCAST — 同时发送多分辨率流，SFU 按终端能力转发
// 可通过 mediasoup/LiveKit 等服务端 SFU 实现
params.encodings = [
  { rid: 'high', maxBitrate: 1_500_000 },
  { rid: 'mid',  maxBitrate: 600_000,   scaleResolutionDownBy: 2 },
  { rid: 'low',  maxBitrate: 200_000,   scaleResolutionDownBy: 4 }
];
```

**弱网策略总结：**

| 网络状态 | 策略 |
|----------|------|
| 良好 (RTT < 150ms, 丢包 < 2%) | 720p/1080p, 30fps, 高码率 |
| 一般 (RTT 150-300ms, 丢包 2-10%) | 480p/720p, 24fps, 中码率 |
| 较差 (RTT 300-500ms, 丢包 10-30%) | 360p/480p, 15fps, 低码率 |
| 极差 (丢包 > 30%) | 仅保留音频，关闭视频 |

---

## Related Links

- [MDN - WebRTC API](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API)
- [WebRTC 官方规范 - W3C](https://www.w3.org/TR/webrtc/)
- [RFC 8445 - ICE 协议](https://datatracker.ietf.org/doc/html/rfc8445)
- [RFC 8489 - STUN 协议](https://datatracker.ietf.org/doc/html/rfc8489)
- [coturn - 开源 TURN 服务器](https://github.com/coturn/coturn)
- [mediasoup - Node.js SFU 框架](https://mediasoup.org/)
- [LiveKit - 开源 WebRTC 基础设施](https://livekit.io/)
- [WebRTC Samples - 官方示例集](https://webrtc.github.io/samples/)
