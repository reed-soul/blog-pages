---
title: "TresJS 入门教程：用 Vue 组件构建 3D 场景"
date: 2026-02-27T20:30:00+08:00
draft: false
tags: ["Vue", "Three.js", "TresJS", "WebGL", "教程"]
categories: ["前端开发"]
description: "本文介绍如何使用 TresJS 库，以 Vue 组件的声明式语法构建 Three.js 3D 场景，大幅降低 WebGL 开发门槛。"
cover: https://images.unsplash.com/photo-1633356122544-f134324a6cee?w=1080&h=864&fit=crop
---

WebGL 是在网页上渲染 3D 图形的核心技术，但直接使用 WebGL API 编程非常繁琐。Three.js 封装了 WebGL，让 3D 开发变得简单一些，但对于习惯声明式编程的前端开发者来说，仍然有不小的学习成本。

TresJS 是一个将 Three.js 与 Vue 3 深度结合的库，它让我们可以用 Vue 组件的方式来构建 3D 场景。本文将介绍 TresJS 的基本用法。

![3D 渲染示例](https://images.unsplash.com/photo-1633356122544-f134324a6cee?w=800&h=400&fit=crop)

<!--more-->

## 一、Three.js 的痛点

使用原生 Three.js 创建一个简单的旋转立方体，代码大致如下：

```javascript
// 1. 创建场景
const scene = new THREE.Scene()

// 2. 创建相机
const camera = new THREE.PerspectiveCamera(
  75, 
  window.innerWidth / window.innerHeight, 
  0.1, 
  1000
)
camera.position.z = 5

// 3. 创建渲染器
const renderer = new THREE.WebGLRenderer()
renderer.setSize(window.innerWidth, window.innerHeight)
document.body.appendChild(renderer.domElement)

// 4. 创建几何体和材质
const geometry = new THREE.BoxGeometry(1, 1, 1)
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
const cube = new THREE.Mesh(geometry, material)
scene.add(cube)

// 5. 动画循环
function animate() {
  requestAnimationFrame(animate)
  cube.rotation.x += 0.01
  cube.rotation.y += 0.01
  renderer.render(scene, camera)
}
animate()
```

这段代码有 20 多行，涉及多个概念：场景（Scene）、相机（Camera）、渲染器（Renderer）、几何体（Geometry）、材质（Material）、网格（Mesh）。每一步都需要手动创建和配置，这就是命令式编程的特点——你需要告诉计算机"怎么做"。

## 二、TresJS 的声明式写法

使用 TresJS，同样的效果可以这样实现：

```vue
<template>
  <TresCanvas>
    <TresPerspectiveCamera :position="[0, 0, 5]" />
    <TresMesh>
      <TresBoxGeometry />
      <TresMeshBasicMaterial color="#00ff00" />
    </TresMesh>
  </TresCanvas>
</template>

<script setup>
import { TresCanvas } from '@tresjs/core'
</script>
```

这就是声明式编程——你只需要描述"是什么"，框架会帮你处理"怎么做"。

![声明式 vs 命令式](https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=800&h=400&fit=crop)

## 三、安装与配置

### 3.1 安装依赖

```bash
npm install @tresjs/core three
# 或
pnpm add @tresjs/core three
```

注意，`three` 是 peer dependency，必须单独安装。

### 3.2 基本用法

```vue
<script setup lang="ts">
import { TresCanvas } from '@tresjs/core'
</script>

<template>
  <TresCanvas clear-color="#1a1a2e">
    <!-- 3D 内容 -->
  </TresCanvas>
</template>

<style scoped>
/* TresCanvas 需要明确的宽高 */
canvas {
  width: 100%;
  height: 100vh;
  display: block;
}
</style>
```

## 四、核心概念

### 4.1 TresCanvas

`TresCanvas` 是所有 3D 内容的根容器。它自动完成三件事：

1. 创建 Three.js 的 Scene（场景）
2. 创建 WebGLRenderer（渲染器）
3. 启动渲染循环（Render Loop）

![TresCanvas 架构](https://images.unsplash.com/photo-1558494949-ef010cbdcc31?w=800&h=400&fit=crop)

常用属性：

| 属性 | 说明 | 默认值 |
|------|------|--------|
| `clear-color` | 背景色 | `#000000` |
| `shadows` | 是否启用阴影 | `false` |
| `shadow-map-type` | 阴影类型 | `PCFSoftShadowMap` |

### 4.2 相机

TresJS 提供了 `TresPerspectiveCamera`（透视相机）和 `TresOrthographicCamera`（正交相机）。

```vue
<TresPerspectiveCamera 
  :position="[4, 4, 4]" 
  :look-at="[0, 0, 0]"
  :fov="75"
/>
```

属性说明：
- `position`：相机位置 [x, y, z]
- `look-at`：相机朝向的目标点
- `fov`：视场角（Field of View）

### 4.3 网格与几何体

在 Three.js 中，Mesh（网格）= Geometry（几何体）+ Material（材质）。TresJS 保持了这个结构：

```vue
<TresMesh>
  <TresBoxGeometry :args="[1, 1, 1]" />
  <TresMeshStandardMaterial color="orange" />
</TresMesh>
```

![Mesh 结构图](https://images.unsplash.com/photo-1635070041078-e363dbe005cb?w=800&h=400&fit=crop)

**组件命名规则**：Three.js 的类名去掉 `THREE.` 前缀，加上 `Tres`：

| Three.js | TresJS |
|----------|--------|
| `THREE.BoxGeometry` | `<TresBoxGeometry>` |
| `THREE.SphereGeometry` | `<TresSphereGeometry>` |
| `THREE.MeshStandardMaterial` | `<TresMeshStandardMaterial>` |

**args 属性**：对应 Three.js 构造函数的参数。例如：

```javascript
// Three.js
new THREE.BoxGeometry(2, 2, 2)

// TresJS
<TresBoxGeometry :args="[2, 2, 2]" />
```

### 4.4 光照

3D 场景需要光照才能看到物体：

```vue
<!-- 环境光：均匀照亮所有物体 -->
<TresAmbientLight :intensity="0.5" />

<!-- 方向光：模拟太阳光 -->
<TresDirectionalLight 
  :position="[5, 5, 5]" 
  :intensity="1" 
  cast-shadow
/>
```

![光照效果对比](https://images.unsplash.com/photo-1519681393784-d120267933ba?w=800&h=400&fit=crop)

## 五、一个完整的示例

下面是一个完整的可运行示例，包含旋转动画：

```vue
<script setup lang="ts">
import { ref, onUnmounted } from 'vue'
import { TresCanvas, useRenderLoop } from '@tresjs/core'

const boxRef = ref(null)
const { onLoop } = useRenderLoop()

// 旋转动画
const unsubscribe = onLoop(({ delta }) => {
  if (boxRef.value) {
    boxRef.value.rotation.y += delta
  }
})

onUnmounted(() => {
  unsubscribe()
})
</script>

<template>
  <TresCanvas clear-color="#1a1a2e" :shadows="true">
    <!-- 相机 -->
    <TresPerspectiveCamera :position="[3, 3, 3]" :look-at="[0, 0, 0]" />
    
    <!-- 立方体 -->
    <TresMesh ref="boxRef" cast-shadow>
      <TresBoxGeometry :args="[1, 1, 1]" />
      <TresMeshStandardMaterial color="#ff6b6b" />
    </TresMesh>
    
    <!-- 地面 -->
    <TresMesh :position="[0, -1, 0]" receive-shadow>
      <TresPlaneGeometry :args="[10, 10]" />
      <TresMeshStandardMaterial color="#4a4a4a" />
    </TresMesh>
    
    <!-- 光照 -->
    <TresAmbientLight :intensity="0.4" />
    <TresDirectionalLight 
      :position="[5, 5, 5]" 
      :intensity="1" 
      cast-shadow
    />
  </TresCanvas>
</template>
```

## 六、响应式数据绑定

TresJS 最大的优势是可以直接使用 Vue 的响应式系统：

```vue
<script setup lang="ts">
import { ref } from 'vue'

const scale = ref(1)
const color = ref('#ff6b6b')

const enlarge = () => {
  scale.value = 2
  color.value = '#4ecdc4'
}
</script>

<template>
  <button @click="enlarge">放大</button>
  
  <TresCanvas>
    <TresMesh :scale="scale">
      <TresBoxGeometry />
      <TresMeshStandardMaterial :color="color" />
    </TresMesh>
  </TresCanvas>
</template>
```

当 `scale` 或 `color` 变化时，3D 场景会自动更新，无需手动调用 `render()`。

## 七、加载 3D 模型

实际项目中，我们通常使用 Blender 等工具建模，然后导出为 GLTF 格式。TresJS 通过 `@tresjs/cientos` 包支持模型加载：

```bash
pnpm add @tresjs/cientos
```

```vue
<script setup lang="ts">
import { useGLTF } from '@tresjs/cientos'

const { scene } = await useGLTF('/models/robot.gltf')
</script>

<template>
  <TresCanvas>
    <primitive :object="scene" />
  </TresCanvas>
</template>
```

![3D 模型示例](https://images.unsplash.com/photo-1617802690992-15d93263d3a9?w=800&h=400&fit=crop)

**注意**：模型文件建议放在 `public` 目录下，使用绝对路径引用。

## 八、交互事件

TresJS 支持常见的鼠标事件：

```vue
<script setup lang="ts">
const handleClick = (event) => {
  console.log('点击位置：', event.point)
  console.log('点击物体：', event.object)
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
    <TresMeshStandardMaterial color="orange" />
  </TresMesh>
</template>
```

## 九、生态系统

TresJS 提供了多个扩展包：

| 包名 | 用途 |
|------|------|
| `@tresjs/core` | 核心库 |
| `@tresjs/cientos` | 常用工具：轨道控制、GLTF 加载器等 |
| `@tresjs/post-processing` | 后处理效果：模糊、发光等 |
| `@tresjs/nuxt` | Nuxt 集成模块 |

## 十、与其他方案对比

| 特性 | TresJS | TroisJS | React Three Fiber |
|------|--------|---------|-------------------|
| 框架 | Vue 3 | Vue 3 | React |
| 维护状态 | 活跃 | 停滞 | 活跃 |
| TypeScript | 完整支持 | 部分支持 | 完整支持 |
| 学习曲线 | 低 | 低 | 中等 |

TroisJS 曾经是 Vue 生态的首选，但目前维护不够活跃。TresJS 是新一代解决方案，设计更加现代。

## 十一、适用场景

**推荐使用：**
- 产品 3D 展示页面
- 数据可视化
- 轻量级 WebGL 交互
- 建筑与室内设计展示

**不推荐使用：**
- 复杂 3D 游戏（建议使用 Unity 或 Unreal）
- 大规模场景（需要 LOD、遮挡剔除等高级优化）
- 需要物理引擎的项目（可配合 Cannon.js，但配置较复杂）

## 十二、小结

TresJS 为 Vue 开发者提供了一条进入 3D Web 开发的捷径。通过声明式组件和响应式数据绑定，我们可以用熟悉的 Vue 语法来构建 3D 场景，大大降低了 Three.js 的学习成本。

如果你正在寻找一个 Vue 友好的 3D 开发方案，TresJS 值得一试。

---

## 参考链接

- [TresJS 官方文档](https://docs.tresjs.org/)
- [TresJS GitHub 仓库](https://github.com/Tresjs/tres)
- [在线 Playground](https://play.tresjs.org/)
- [Three.js 官方文档](https://threejs.org/docs/)
- [Discord 社区](https://tresjs.org/discord)
