---
title: "TresJS：用 Vue 组件声明式构建 Three.js 3D 场景"
date: 2026-02-27T20:30:00+08:00
draft: false
tags: ["Vue", "Three.js", "TresJS", "WebGL", "前端", "3D"]
categories: ["前端开发"]
description: "告别命令式 Three.js 代码，用 Vue 组件的方式声明式构建 3D 场景，享受响应式数据驱动的开发体验。"
---

如果你用过 Three.js，一定写过这样的代码：

```js
const scene = new THREE.Scene()
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
const renderer = new THREE.WebGLRenderer()

const geometry = new THREE.BoxGeometry()
const material = new THREE.MeshBasicMaterial({ color: 0x00FF00 })
const cube = new THREE.Mesh(geometry, material)
scene.add(cube)

function animate() {
  requestAnimationFrame(animate)
  cube.rotation.x += 0.01
  cube.rotation.y += 0.01
  renderer.render(scene, camera)
}
animate()
```

命令式、冗长、难以维护。每次要调整一个物体，都得翻文档找 API。

<!--more-->

现在，用 **TresJS**，同样的场景可以这样写：

```vue
<template>
  <TresCanvas>
    <TresPerspectiveCamera :position="[0, 0, 5]" />
    <TresMesh>
      <TresBoxGeometry />
      <TresMeshBasicMaterial :color="0x00FF00" />
    </TresMesh>
  </TresCanvas>
</template>
```

干净、声明式、符合 Vue 开发直觉。

## TresJS 是什么？

TresJS（西班牙语"三"，读作 /tres/）是一个让 Vue 开发者用组件方式构建 Three.js 3D 场景的库。

### 核心价值

**1. 声明式语法**

用 Vue 组件描述 3D 场景，而不是写一堆 `new THREE.XXX()`。

**2. 响应式绑定**

直接用 Vue 的响应式数据驱动 3D 物体属性变化，动画状态管理变得简单。

**3. 始终兼容最新 Three.js**

基于 Vue Custom Renderer，Three.js 更新时 TresJS 自动跟进，无需等待封装层更新。

**4. TypeScript 全量支持**

完整的类型定义，IDE 智能提示和类型检查。

**5. Nuxt 集成**

官方提供 Nuxt 模块，SSR 和 SEO 友好。

## 快速开始

### 安装

```bash
# 使用 npm
npm install @tresjs/core three

# 或使用 pnpm
pnpm add @tresjs/core three
```

### 最小示例

```vue
<script setup lang="ts">
import { TresCanvas } from '@tresjs/core'
</script>

<template>
  <TresCanvas clear-color="#82DBC5">
    <TresPerspectiveCamera :position="[3, 3, 3]" :look-at="[0, 0, 0]" />
    <TresMesh>
      <TresBoxGeometry :args="[1, 1, 1]" />
      <TresMeshStandardMaterial color="orange" />
    </TresMesh>
    <TresDirectionalLight :position="[0, 2, 4]" :intensity="2" />
    <TresAmbientLight :intensity="0.5" />
  </TresCanvas>
</template>
```

运行后你会看到一个橙色的立方体。

## 核心概念

### 1. TresCanvas — 场景容器

`<TresCanvas>` 是所有 3D 内容的根容器，自动创建 Scene、Renderer、Render Loop。

```vue
<TresCanvas 
  :shadows="true"
  :shadow-map-type="PCFSoftShadowMap"
  clear-color="#111"
>
  <!-- 3D 内容 -->
</TresCanvas>
```

### 2. 组件命名规则

Three.js 的类名前缀去掉，加上 `Tres`：

| Three.js | TresJS 组件 |
|----------|-------------|
| `THREE.PerspectiveCamera` | `<TresPerspectiveCamera>` |
| `THREE.Mesh` | `<TresMesh>` |
| `THREE.BoxGeometry` | `<TresBoxGeometry>` |
| `THREE.MeshStandardMaterial` | `<TresMeshStandardMaterial>` |

### 3. 属性绑定

Three.js 构造函数参数通过 `args` 传递，属性直接绑定：

```vue
<TresBoxGeometry :args="[2, 2, 2]" />
<TresMeshStandardMaterial :color="materialColor" :metalness="0.5" />
```

### 4. 响应式更新

Vue 的响应式数据变化会自动反映到 3D 场景：

```vue
<script setup lang="ts">
import { ref } from 'vue'

const cubeScale = ref(1)
const color = ref('orange')

// 2秒后放大并变色
setTimeout(() => {
  cubeScale.value = 2
  color.value = 'blue'
}, 2000)
</script>

<template>
  <TresMesh :scale="cubeScale">
    <TresBoxGeometry />
    <TresMeshStandardMaterial :color="color" />
  </TresMesh>
</template>
```

## 实用技巧

### 使用 Composables

TresJS 提供了实用的 composables：

```vue
<script setup lang="ts">
import { useRenderLoop } from '@tresjs/core'

const { onLoop } = useRenderLoop()

onLoop(({ delta }) => {
  // 每帧执行，delta 是时间差
  mesh.value.rotation.y += delta
})
</script>
```

### 加载 3D 模型

配合 `@tresjs/cientos` 包可以轻松加载 GLTF 模型：

```bash
pnpm add @tresjs/cientos
```

```vue
<script setup lang="ts">
import { useGLTF } from '@tresjs/cientos'

const { scene } = await useGLTF('/models/robot.gltf')
</script>

<template>
  <primitive :object="scene" />
</template>
```

### 鼠标交互

```vue
<script setup lang="ts">
const handleClick = (event) => {
  console.log('点击了物体', event.object)
}

const handlePointerEnter = () => {
  document.body.style.cursor = 'pointer'
}

const handlePointerLeave = () => {
  document.body.style.cursor = 'default'
}
</script>

<template>
  <TresMesh 
    @click="handleClick"
    @pointer-enter="handlePointerEnter"
    @pointer-leave="handlePointerLeave"
  >
    <TresBoxGeometry />
    <TresMeshStandardMaterial />
  </TresMesh>
</template>
```

## 生态系统

TresJS 有丰富的官方扩展包：

| 包名 | 用途 |
|------|------|
| `@tresjs/core` | 核心库 |
| `@tresjs/cientos` | 常用工具集合（轨道控制、加载器等） |
| `@tresjs/post-processing` | 后处理效果 |
| `@tresjs/nuxt` | Nuxt 集成模块 |
| `@tresjs/leches` | Storybook 风格的组件展示 |

## 对比其他方案

| 特性 | TresJS | TroisJS | react-three-fiber |
|------|--------|---------|-------------------|
| 框架 | Vue 3 | Vue 3 | React |
| Three.js 更新 | 即时 | 滞后 | 即时 |
| TypeScript | 完整 | 部分 | 完整 |
| 生态 | 成长中 | 较少 | 丰富 |
| 学习曲线 | 低 | 低 | 中 |

TroisJS 曾是 Vue + Three.js 的首选，但维护不够活跃。TresJS 是新一代替代方案，设计更现代，更新更及时。

## 适用场景

**推荐使用：**
- 产品 3D 展示页
- 交互式数据可视化
- 轻量级 WebGL 游戏
- 建筑/室内设计展示
- 品牌官网 3D 特效

**不推荐：**
- 复杂 3D 游戏（用 Unity/Unreal）
- 大规模场景（需要 LOD、遮挡剔除等高级优化）
- 需要物理引擎的项目（可配合 Cannon.js，但需要更多配置）

## 总结

TresJS 让 Vue 开发者可以用熟悉的方式进入 3D 开发领域。如果你是 Vue 开发者，想要快速给项目添加 3D 效果、用声明式思维构建 3D 场景、享受 Vue 生态的响应式和组件化，TresJS 值得一试。

---

**相关链接：**

- 官网：https://tresjs.org
- GitHub：https://github.com/Tresjs/tres
- 在线 Playground：https://play.tresjs.org
- Discord 社区：https://tresjs.org/discord
