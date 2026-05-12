---
tags:
  - Web前端
  - 3D
  - Three.js
  - WebGL
  - 3D可视化
date: 2026-05-12
status: 已完成
difficulty: 中高
---

# 前端3D开发与Three.js

## What — 是什么

> Three.js 是最流行的 Web 3D 图形库，封装了 WebGL 的底层 API，提供场景图（Scene Graph）、相机、灯光、材质、几何体等高层抽象，让前端开发者无需理解 GPU 着色器即可创建 3D 场景。广泛应用于产品展示、数据可视化、3D 游戏、虚拟展厅、数字孪生等领域。

**核心概念：**

- **场景（Scene）**：3D 世界的容器，包含所有物体、灯光、相机
- **相机（Camera）**：观察场景的视角，透视相机（PerspectiveCamera）模拟人眼，正交相机（OrthographicCamera）无近大远小
- **渲染器（Renderer）**：将场景+相机渲染到 Canvas，WebGLRenderer 使用 GPU 加速
- **几何体（Geometry）**：定义物体形状（顶点、面、法线），如 BoxGeometry、SphereGeometry
- **材质（Material）**：定义物体外观（颜色、纹理、光照响应），如 MeshStandardMaterial、MeshBasicMaterial
- **网格（Mesh）**：几何体 + 材质 = 可渲染物体
- **灯光（Light）**：照亮场景，如 AmbientLight、DirectionalLight、PointLight、SpotLight
- **纹理（Texture）**：贴图，如颜色贴图、法线贴图、粗糙度贴图
- **动画循环（Animation Loop）**：`requestAnimationFrame` 驱动每帧渲染

**核心架构：**

```
┌──────────────────────────────────────────────────┐
│                 Three.js 应用                      │
├──────────────────────────────────────────────────┤
│  Scene (场景图)                                   │
│  ├── Mesh (物体)                                  │
│  │   ├── Geometry (形状)                          │
│  │   └── Material (材质)                          │
│  │       └── Texture (贴图)                       │
│  ├── Light (灯光)                                 │
│  ├── Group (分组)                                 │
│  └── ...                                          │
├──────────────────────────────────────────────────┤
│  Camera (相机)                                    │
│  ├── PerspectiveCamera (透视)                      │
│  └── OrthographicCamera (正交)                    │
├──────────────────────────────────────────────────┤
│  WebGLRenderer (渲染器)                           │
│  ├── render(scene, camera) — 每帧调用            │
│  ├── Shadow (阴影)                                │
│  └── Post-processing (后处理)                     │
├──────────────────────────────────────────────────┤
│  Animation Loop (动画循环)                        │
│  requestAnimationFrame → update → render          │
├──────────────────────────────────────────────────┤
│  WebGL / WebGPU (GPU API)                        │
└──────────────────────────────────────────────────┘
```

**3D 开发生态：**

| 工具/库 | 用途 |
|---------|------|
| Three.js | 通用 3D 渲染库 |
| React Three Fiber | Three.js 的 React 封装 |
| Drei | R3F 常用辅助组件 |
| @react-three/postprocessing | R3F 后处理 |
| Cannon.js / Ammo.js | 物理引擎 |
| GSAP | 动画补间 |
| Blender | 3D 建模（导出 glTF） |
| glTF | 3D 模型标准格式 |
| DRACOLoader | 模型压缩加载 |
| OrbitControls | 轨道控制器（旋转/缩放/平移） |
| Stats.js | 性能监控（FPS） |

## Why — 为什么

**适用场景：**

- 3D 产品展示（电商、汽车、家具、珠宝）
- 数据可视化（3D 地图、城市模型、工业数字孪生）
- 虚拟展厅/博物馆
- 3D 游戏（轻量级，非 AAA）
- AR/VR 体验（WebXR）
- 建筑漫游/BIM 可视化
- 粒子特效（首页 Banner、活动页）
- 医学影像 3D 重建

**3D 方案对比：**

| 方案 | 上手难度 | 性能 | 灵活性 | 适用场景 |
|------|---------|------|--------|---------|
| Three.js | 中 | 高 | 极高 | 通用 3D 开发 |
| React Three Fiber | 低 | 高 | 高 | React 项目 |
| Babylon.js | 中 | 高 | 高 | 游戏/复杂交互 |
| CSS 3D Transform | 低 | 中 | 低 | 简单翻转/卡片 |
| Spline | 极低 | 中 | 中 | 设计师协作，零代码 |
| PlayCanvas | 中 | 高 | 中 | 游戏引擎 |
| 原生 WebGL | 极高 | 极高 | 极高 | 底层定制/极致性能 |

## How — 怎么用

### 原生 Three.js 基础

```bash
npm install three
npm install @types/three -D
```

```ts
// 最简 3D 场景
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

// 1. 创建场景
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xf0f0f0);

// 2. 创建相机
const camera = new THREE.PerspectiveCamera(
  75,                                         // FOV 视场角
  window.innerWidth / window.innerHeight,     // 宽高比
  0.1,                                        // 近裁面
  1000                                        // 远裁面
);
camera.position.set(5, 3, 5);

// 3. 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

// 4. 轨道控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;

// 5. 添加灯光
const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
directionalLight.position.set(5, 10, 5);
directionalLight.castShadow = true;
directionalLight.shadow.mapSize.set(1024, 1024);
scene.add(directionalLight);

// 6. 添加物体
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshStandardMaterial({ color: 0x1677ff });
const cube = new THREE.Mesh(geometry, material);
cube.castShadow = true;
cube.receiveShadow = true;
scene.add(cube);

// 地面
const floor = new THREE.Mesh(
  new THREE.PlaneGeometry(20, 20),
  new THREE.MeshStandardMaterial({ color: 0xcccccc })
);
floor.rotation.x = -Math.PI / 2;
floor.receiveShadow = true;
scene.add(floor);

// 7. 动画循环
function animate() {
  requestAnimationFrame(animate);
  cube.rotation.y += 0.01;
  controls.update();
  renderer.render(scene, camera);
}
animate();

// 8. 窗口自适应
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

### React Three Fiber

```bash
npm install @react-three/fiber @react-three/drei
```

```tsx
import { Canvas } from '@react-three/fiber';
import { OrbitControls, Environment, ContactShadows, Float, Text } from '@react-three/drei';

function Box({ position, color }: { position: [number, number, number]; color: string }) {
  return (
    <Float speed={2} rotationIntensity={1} floatIntensity={0.5}>
      <mesh position={position} castShadow>
        <boxGeometry args={[1, 1, 1]} />
        <meshStandardMaterial color={color} roughness={0.3} metalness={0.5} />
      </mesh>
    </Float>
  );
}

function Scene() {
  return (
    <Canvas
      shadows
      camera={{ position: [5, 3, 5], fov: 75 }}
      dpr={[1, 2]}  // DPR 范围
    >
      {/* 灯光 */}
      <ambientLight intensity={0.6} />
      <directionalLight position={[5, 10, 5]} intensity={0.8} castShadow />

      {/* 物体 */}
      <Box position={[-2, 0.5, 0]} color="#1677ff" />
      <Box position={[0, 0.5, 0]} color="#52c41a" />
      <Box position={[2, 0.5, 0]} color="#ff4d4f" />

      {/* 文字 */}
      <Text position={[0, 2.5, 0]} fontSize={0.5} color="#333">
        Hello 3D
      </Text>

      {/* 地面 */}
      <ContactShadows position={[0, -0.01, 0]} opacity={0.5} scale={10} blur={2} />

      {/* 控制器 */}
      <OrbitControls enableDamping dampingFactor={0.05} />

      {/* 环境贴图 */}
      <Environment preset="city" />
    </Canvas>
  );
}

export default function App() {
  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      <Scene />
    </div>
  );
}
```

### 常用几何体与材质

```tsx
// 几何体
<mesh>
  <boxGeometry args={[1, 1, 1]} />            {/* 立方体 */}
</mesh>
<mesh>
  <sphereGeometry args={[1, 32, 32]} />        {/* 球体 */}
</mesh>
<mesh>
  <cylinderGeometry args={[1, 1, 2, 32]} />   {/* 圆柱体 */}
</mesh>
<mesh>
  <planeGeometry args={[5, 5]} />              {/* 平面 */}
</mesh>
<mesh>
  <torusGeometry args={[1, 0.4, 16, 100]} />  {/* 圆环 */}
</mesh>
<mesh>
  <coneGeometry args={[1, 2, 32]} />           {/* 圆锥 */}
</mesh>

// 材质
<meshBasicMaterial color="#1677ff" />           {/* 基础材质（不受光照） */}
<meshStandardMaterial                          {/* PBR 标准材质 */}
  color="#1677ff"
  roughness={0.3}
  metalness={0.5}
  map={colorTexture}
  normalMap={normalTexture}
/>
<meshPhysicalMaterial                          {/* 物理材质（玻璃/透明） */}
  color="#ffffff"
  transmission={0.9}
  roughness={0}
  metalness={0}
  ior={1.5}
  thickness={0.5}
/>
<meshPhongMaterial                             {/* Phong 光照 */}
  color="#1677ff"
  shininess={100}
  specular="#ffffff"
/>
```

### 加载 3D 模型

```tsx
import { useGLTF } from '@react-three/drei';

// 加载 glTF/glb 模型
function Model({ url }: { url: string }) {
  const { scene } = useGLTF(url);
  return <primitive object={scene} />;
}

// 使用
<Model url="/models/car.glb" />

// 预加载
useGLTF.preload('/models/car.glb');

// 原生 Three.js 加载
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';

const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('/draco/');

const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);

gltfLoader.load(
  '/models/car.glb',
  (gltf) => {
    scene.add(gltf.scene);
  },
  (progress) => {
    console.log(`加载进度: ${(progress.loaded / progress.total * 100).toFixed(0)}%`);
  },
  (error) => {
    console.error('加载失败:', error);
  }
);
```

### 灯光系统

```tsx
// 环境光：均匀照亮所有物体
<ambientLight intensity={0.4} color="#ffffff" />

// 平行光：太阳光效果
<directionalLight
  position={[5, 10, 5]}
  intensity={1}
  color="#ffffff"
  castShadow
  shadow-mapSize-width={1024}
  shadow-mapSize-height={1024}
  shadow-camera-far={50}
  shadow-camera-left={-10}
  shadow-camera-right={10}
  shadow-camera-top={10}
  shadow-camera-bottom={-10}
/>

// 点光源：灯泡效果
<pointLight
  position={[0, 5, 0]}
  intensity={1}
  color="#ff9800"
  distance={10}
  decay={2}
  castShadow
/>

// 聚光灯：手电筒效果
<spotLight
  position={[0, 10, 0]}
  angle={0.3}
  penumbra={0.5}
  intensity={2}
  castShadow
/>

// 半球光：天地双色
<hemisphereLight args={['#87ceeb', '#362907', 0.5]} />

// 环境贴图（PBR 必需）
<Environment preset="sunset" />
// 预设：apartment, city, dawn, forest, lobby, night, park, studio, sunset, warehouse
```

### 动画系统

```tsx
// 1. useFrame 钩子（R3F）
import { useFrame } from '@react-three/fiber';
import { useRef } from 'react';
import * as THREE from 'three';

function RotatingBox() {
  const meshRef = useRef<THREE.Mesh>(null);

  useFrame((state, delta) => {
    if (meshRef.current) {
      meshRef.current.rotation.y += delta;  // 每帧旋转
      meshRef.current.position.y = Math.sin(state.clock.elapsedTime) * 0.5; // 上下浮动
    }
  });

  return (
    <mesh ref={meshRef}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="#1677ff" />
    </mesh>
  );
}

// 2. useSpring 动画（react-spring）
import { useSpring, animated } from '@react-spring/three';

function AnimatedSphere() {
  const [hovered, setHovered] = useState(false);
  const { scale } = useSpring({ scale: hovered ? 1.5 : 1 });

  return (
    <animated.mesh
      scale={scale}
      onPointerOver={() => setHovered(true)}
      onPointerOut={() => setHovered(false)}
    >
      <sphereGeometry args={[1, 32, 32]} />
      <meshStandardMaterial color={hovered ? '#ff4d4f' : '#1677ff'} />
    </animated.mesh>
  );
}

// 3. 关键帧动画（Three.js AnimationMixer）
import { useFrame } from '@react-three/fiber';

function AnimatedModel({ url }: { url: string }) {
  const { scene, animations } = useGLTF(url);
  const mixer = useRef<THREE.AnimationMixer>();

  useEffect(() => {
    if (animations.length > 0) {
      mixer.current = new THREE.AnimationMixer(scene);
      const action = mixer.current.clipAction(animations[0]);
      action.play();
    }
    return () => mixer.current?.stopAllAction();
  }, [animations, scene]);

  useFrame((_, delta) => mixer.current?.update(delta));

  return <primitive object={scene} />;
}
```

### 交互：射线拾取

```tsx
// R3F 内置事件系统
function ClickableObjects() {
  const [selected, setSelected] = useState<string | null>(null);

  return (
    <group>
      <mesh
        onClick={() => setSelected('box')}
        onPointerOver={() => document.body.style.cursor = 'pointer'}
        onPointerOut={() => document.body.style.cursor = 'default'}
      >
        <boxGeometry args={[1, 1, 1]} />
        <meshStandardMaterial color={selected === 'box' ? '#ff4d4f' : '#1677ff'} />
      </mesh>

      <mesh position={[2, 0, 0]} onClick={() => setSelected('sphere')}>
        <sphereGeometry args={[0.5, 32, 32]} />
        <meshStandardMaterial color={selected === 'sphere' ? '#ff4d4f' : '#52c41a'} />
      </mesh>
    </group>
  );
}

// 原生 Three.js Raycaster
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

canvas.addEventListener('click', (event) => {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects(scene.children, true);

  if (intersects.length > 0) {
    const object = intersects[0].object;
    console.log('点击了:', object.name || object.uuid);
  }
});
```

### 粒子系统

```tsx
import { useRef, useMemo } from 'react';
import { useFrame } from '@react-three/fiber';
import * as THREE from 'three';

function Particles({ count = 1000 }) {
  const pointsRef = useRef<THREE.Points>(null);

  const { positions, colors } = useMemo(() => {
    const positions = new Float32Array(count * 3);
    const colors = new Float32Array(count * 3);

    for (let i = 0; i < count; i++) {
      positions[i * 3] = (Math.random() - 0.5) * 20;
      positions[i * 3 + 1] = (Math.random() - 0.5) * 20;
      positions[i * 3 + 2] = (Math.random() - 0.5) * 20;

      colors[i * 3] = Math.random();
      colors[i * 3 + 1] = Math.random();
      colors[i * 3 + 2] = Math.random();
    }

    return { positions, colors };
  }, [count]);

  useFrame((state) => {
    if (pointsRef.current) {
      pointsRef.current.rotation.y = state.clock.elapsedTime * 0.05;
      pointsRef.current.rotation.x = Math.sin(state.clock.elapsedTime * 0.03) * 0.1;
    }
  });

  return (
    <points ref={pointsRef}>
      <bufferGeometry>
        <bufferAttribute
          attach="attributes-position"
          count={count}
          array={positions}
          itemSize={3}
        />
        <bufferAttribute
          attach="attributes-color"
          count={count}
          array={colors}
          itemSize={3}
        />
      </bufferGeometry>
      <pointsMaterial
        size={0.05}
        vertexColors
        transparent
        opacity={0.8}
        sizeAttenuation
      />
    </points>
  );
}
```

### 后处理效果

```bash
npm install @react-three/postprocessing
```

```tsx
import { EffectComposer, Bloom, Vignette, ChromaticAberration, DepthOfField } from '@react-three/postprocessing';

function PostProcessingScene() {
  return (
    <>
      {/* 场景内容 */}
      <mesh>
        <sphereGeometry args={[1, 32, 32]} />
        <meshStandardMaterial color="#1677ff" emissive="#0044ff" emissiveIntensity={2} />
      </mesh>

      {/* 后处理 */}
      <EffectComposer>
        <Bloom
          intensity={1.5}
          luminanceThreshold={0.6}
          luminanceSmoothing={0.9}
        />
        <Vignette offset={0.3} darkness={0.7} />
        <DepthOfField
          focusDistance={0.01}
          focalLength={0.02}
          bokehScale={2}
        />
      </EffectComposer>
    </>
  );
}
```

### 性能优化

```tsx
// 1. 几何体复用
const geometry = useMemo(() => new THREE.BoxGeometry(1, 1, 1), []);
const material = useMemo(() => new THREE.MeshStandardMaterial({ color: '#1677ff' }), []);

function InstancedBoxes({ count = 100 }) {
  // 2. InstancedMesh — 大量相同物体
  const meshRef = useRef<THREE.InstancedMesh>(null);

  useEffect(() => {
    if (!meshRef.current) return;
    const dummy = new THREE.Object3D();
    for (let i = 0; i < count; i++) {
      dummy.position.set(
        (Math.random() - 0.5) * 20,
        (Math.random() - 0.5) * 20,
        (Math.random() - 0.5) * 20,
      );
      dummy.updateMatrix();
      meshRef.current.setMatrixAt(i, dummy.matrix);
    }
    meshRef.current.instanceMatrix.needsUpdate = true;
  }, [count]);

  return (
    <instancedMesh ref={meshRef} args={[undefined, undefined, count]} castShadow>
      <boxGeometry args={[0.5, 0.5, 0.5]} />
      <meshStandardMaterial color="#1677ff" />
    </instancedMesh>
  );
}

// 3. LOD（细节层次）
import { Detailed } from '@react-three/drei';

function LODObject() {
  return (
    <Detailed distances={[0, 10, 30]}>
      {/* 近距离：高精度 */}
      <mesh>
        <sphereGeometry args={[1, 64, 64]} />
        <meshStandardMaterial color="#1677ff" />
      </mesh>
      {/* 中距离：中精度 */}
      <mesh>
        <sphereGeometry args={[1, 16, 16]} />
        <meshStandardMaterial color="#1677ff" />
      </mesh>
      {/* 远距离：低精度 */}
      <mesh>
        <sphereGeometry args={[1, 8, 8]} />
        <meshBasicMaterial color="#1677ff" />
      </mesh>
    </Detailed>
  );
}

// 4. 懒加载 + Suspense
import { Suspense } from 'react';
import { Html } from '@react-three/drei';

function App() {
  return (
    <Canvas>
      <Suspense fallback={<Html><div>加载中...</div></Html>}>
        <Model url="/models/complex.glb" />
      </Suspense>
    </Canvas>
  );
}

// 5. 按需渲染（非连续动画场景）
// renderer.setAnimationLoop(null);  // 停止自动渲染
// 手动触发: renderer.render(scene, camera);

// 6. 纹理优化
const texture = new THREE.TextureLoader().load('/texture.jpg');
texture.generateMipmaps = true;
texture.minFilter = THREE.LinearMipmapLinearFilter;
texture.magFilter = THREE.LinearFilter;
texture.anisotropy = renderer.capabilities.getMaxAnisotropy();
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 模型全黑 | 缺少灯光或环境贴图 | PBR 材质必须搭配灯光/Environment |
| 模型加载后很大/小 | 导出单位不一致 | `model.scale.set(0.01, 0.01, 0.01)` 调整 |
| 性能卡顿 | 多 Draw Call / 高面数 | InstancedMesh / LOD / 减面 |
| 阴影锯齿严重 | Shadow Map 分辨率低 | 增大 `shadow.mapSize`（2048+） |
| 透明物体渲染异常 | 透明排序问题 | `material.depthWrite = false` + `renderOrder` |
| Canvas 模糊 | DPR 未处理 | `renderer.setPixelRatio(window.devicePixelRatio)` |
| 内存泄漏 | Geometry/Material 未 dispose | 组件卸载时 `geometry.dispose(); material.dispose()` |
| glTF 材质不显示 | 缺少环境贴图 | 添加 `<Environment>` |
| 移动端性能差 | GPU 性能弱 + DPR 过高 | 限制 `dpr={[1, 1.5]}`，减少面数 |
| 模型动画不播放 | 未创建 AnimationMixer | 使用 `useAnimations` 钩子 |

### 最佳实践

- PBR 材质（Standard/Physical）必须搭配灯光和 Environment，否则全黑
- 大量相同物体用 `InstancedMesh`，避免创建数千个 Mesh
- 远处物体用 LOD 降低面数，近处高精度
- 模型使用 glTF + DRACO 压缩，减小加载体积
- 纹理使用压缩格式（KTX2），减少 GPU 显存
- 移动端限制 DPR ≤ 1.5，降低阴影分辨率
- 组件卸载时 dispose Geometry、Material、Texture，防止内存泄漏
- 非动画场景使用按需渲染，减少 GPU 占用
- 使用 `Suspense` + 加载进度条处理模型异步加载
- 复杂场景使用 `@react-three/postprocessing` 添加后处理提升视觉

## 面试题

**Q1: Three.js 的渲染管线是什么？一帧渲染经历哪些步骤？**
> 渲染管线：① 场景遍历：遍历 Scene Graph，收集所有可见物体（视锥体裁剪 Frustum Culling）；② 排序：按材质/距离排序，减少状态切换（透明物体从远到近，不透明从近到远）；③ 绘制调用：每个 Mesh 生成一次 Draw Call，将几何体数据 + 材质参数发送到 GPU；④ GPU 渲染：顶点着色器（MVP 变换）→ 光栅化 → 片元着色器（光照计算）→ 深度测试 → 混合输出到帧缓冲。优化核心：减少 Draw Call 数量（合并/实例化）、减少片元数量（LOD/剔除）、减少着色器复杂度。

**Q2: PerspectiveCamera 和 OrthographicCamera 有什么区别？**
> PerspectiveCamera 模拟人眼透视：近大远小，有 FOV（视场角）和宽高比参数，适用于 3D 产品展示、游戏、漫游。OrthographicCamera 无透视变形：物体大小不随距离变化，有 left/right/top/bottom 参数定义可视矩形，适用于 2D 图纸、CAD、等距视角（Isometric）游戏、地图。关键区别：透视相机有灭点（远处的线汇聚），正交相机没有灭点（平行线始终平行）。

**Q3: InstancedMesh 是什么？为什么能提升性能？**
> InstancedMesh 是 Three.js 的实例化渲染方案：用一次 Draw Call 渲染大量相同几何体和材质的物体。原理：GPU 只需接收一份几何体数据，通过 Instance Matrix（每个实例的位置/旋转/缩放矩阵）在着色器中计算每个实例的位置，无需重复传输几何体数据。性能差异：10000 个独立 Mesh = 10000 次 Draw Call（卡顿）；1 个 InstancedMesh(count=10000) = 1 次 Draw Call（流畅）。限制：所有实例必须共享同一个 Geometry 和 Material。

**Q4: 如何优化 Three.js 场景的性能？**
> 7 个层面：① 减少 Draw Call：InstancedMesh、合并静态网格、减少材质种类；② 减少面数：LOD（远处低模）、模型减面、视锥体裁剪（Three.js 自动）；③ 纹理优化：KTX2 压缩格式、合理尺寸（2的幂）、Mipmap；④ 阴影优化：降低 Shadow Map 分辨率、只给关键物体开阴影、CSM（级联阴影）；⑤ 后处理优化：减少 Pass 数量、降低 Bloom 分辨率；⑥ 按需渲染：非动画场景停止 AnimationLoop，事件触发时手动 render；⑦ 内存管理：及时 dispose 不再使用的 Geometry/Material/Texture。

**Q5: PBR 材质（MeshStandardMaterial）需要什么条件才能正确显示？**
> PBR（Physically Based Rendering）基于物理的渲染，需要三个条件：① 灯光：至少一个环境光 + 一个直射光，或使用 Environment 环境贴图；② 环境贴图：PBR 的反射和折射依赖环境贴图，没有 Environment 会全黑或无反射；③ 正确的材质参数：`roughness`（粗糙度 0=镜面 1=漫反射）、`metalness`（金属度 0=非金属 1=金属）。常见错误：只加 ambientLight 不加环境贴图 → 看起来像平面无立体感；metalness=1 但无环境贴图 → 全黑。

**Q6: React Three Fiber 和原生 Three.js 各有什么优劣？**
> R3F 优势：声明式 JSX 语法，组件化开发，React 状态管理自然集成，useFrame/useEffect 生命周期，Drei 组件库开箱即用，代码更简洁。R3F 劣势：额外抽象层有微小性能开销，调试时调用栈更深，某些 Three.js 高级用法需 `useRef` + `useEffect` 原生操作。原生优势：完全控制渲染流程，零抽象开销，Three.js 社区教程直接适用。原生劣势：命令式代码冗长，手动管理生命周期和清理，与 React 状态同步困难。选择：React 项目 → R3F；纯 Canvas/独立项目 → 原生。

**Q7: 如何在 Three.js 中实现鼠标拾取（点击选中 3D 物体）？**
> 射线拾取（Raycasting）：① 将鼠标屏幕坐标转换为 NDC（-1 到 1）：`x = (clientX / width) * 2 - 1; y = -(clientY / height) * 2 + 1`；② 创建射线：`raycaster.setFromCamera(mouse, camera)`，从相机位置沿鼠标方向发射射线；③ 检测相交：`raycaster.intersectObjects(objects, recursive)`，返回相交物体数组（按距离排序）；④ 取最近的相交点 `intersects[0]`，获取物体引用。R3F 简化了此流程：直接在 `<mesh>` 上绑定 `onClick`、`onPointerOver`、`onPointerOut` 事件。

**Q8: glTF 模型格式为什么是 Web 3D 的标准？如何优化加载？**
> glTF 是 Khronos 制定的 3D 资产标准（"3D 的 JPEG"），优势：① JSON 描述场景结构 + 二进制存储几何数据，解析快；② 支持 PBR 材质、动画、蒙皮、形态目标；③ 文件紧凑（.glb 单文件 / .gltf 分文件）；④ 主流 DCC 工具（Blender/Maya）和引擎（Three.js/Babylon/Unity）均支持。加载优化：① DRACO 压缩：几何体压缩 80%+ 体积，`DRACOLoader` 解压；② KTX2 纹理压缩：GPU 直接解码，省显存；③ 预加载：`useGLTF.preload()`；④ Suspense + 进度条；⑤ 模型分块加载，按需加载远处模型。

---

**相关链接：**
- [[WebGL与WebGPU]]
- [[Canvas与SVG]]
- [[可视化与图表库]]
- [[前端性能优化]]
