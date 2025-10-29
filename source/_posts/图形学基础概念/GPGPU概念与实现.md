---
title: GPGPU概念与实现
date: 2025-10-29 16:57:00
tags:
  - Rendering
  - Basic Concept
  - Computer Shader
  - BBSO
categories: Basic Concept
---

# Computer Shader
## 概念区分
GPGPU (General-Purpose computing on Graphics Processing Units)：这是一个概念或领域。它指的是“使用 GPU 强大的并行处理能力来执行传统上由 CPU 负责的通用计算任务”（如图形渲染之外的科学计算、物理模拟、AI 训练等）。

Compute Shader (计算着色器)：这是一种具体的技术实现。它是一种程序（ HLSL, GLSL, MSL 等语言编写），它允许开发者在 GPU 上执行通用的计算。它最大的特点是它独立于传统的“渲染管线”（即独立于顶点着色器和片段着色器）。

## 为什么说“它独立于传统的渲染管线”
Computer Shader与VS，FS都叫“着色器”(Shader)，都使用 HLSL 或 GLSL 这样的语言编写，并且都运行在 GPU 内部完全相同的“统一着色器核心”上。

而“独立于顶点着色器和片段着色器”是在说：传统的渲染管线工序严格定义为顶点处理(VS) -> 光栅化 -> 像素处理(FS) -> 输出图像，但是Computer Shader并不直接在这条管线里面工作，不关心什么顶点和像素，它的输入可以是任何数据，输出也可以是任何数据。在应用上，它的逻辑其实就类似在GPU上运行的代码，只不过是这一块代码是在GPU上运行的，并且由GPU统一管控。

## 使用例子
举个例子，让一张 100 万像素的图片全部变亮。

CPU 逻辑：
```
for (int i = 0; i < 1000000; ++i) {
    pixels[i] = pixels[i] * 1.1; // 串行执行 100 万次
}
```

GPGPU 逻辑 (伪代码)：
```
// 只写这个函数，它只处理一个像素
void BrightenKernel(int i) {
    pixels[i] = pixels[i] * 1.1;
}

// (在 CPU 端) 调用：
// “GPU，请在 100 万个核心上同时运行 BrightenKernel 函数”
Dispatch(1000000);
```

## Compute Shader如何助力渲染过程
Compute Shader助力渲染过程不是直接插入渲染管线，而是使用多步骤，也即Pass。

第一步，先运行一个 Compute Pass，让 Compute Shader 这个“通用车间”去完成所有繁重的计算（比如模拟成千上万个粒子的运动位置），并把计算结果存放到一块“仓库”（如 Buffer 或 Texture）里。

第二步，再运行一个 Render Pass，启动那条“渲染管线”，此时你的 VS 和 FS 程序不再需要自己去苦苦计算，它们可以直接去“仓库”里读取 Compute Shader 算好的现成数据，然后轻松地把它们画在屏幕上。


# SSBO
## 概念
SSBO (Shader Storage Buffer Object，着色器存储缓冲对象) 是一种在 GPU 上分配的**通用内存缓冲区**。

你可以把它想象成一块“大容量、可读写的 GPU 内存”，所有类型的着色器（尤其是 Compute Shader）都可以自由地从中读取和写入数据。

SSBO 是现代图形 API (如 OpenGL 4.3+, Vulkan, Metal, WebGPU) 中的一个核心功能，它的出现**是 GPGPU 和 Compute Shader 得以普及的关键基石**。

---

## SSBO 的核心特性

1.  **读写能力 (Read/Write Access)**
    * 这是 SSBO **最重要**的特性。在 SSBO 出现之前，GPU 上的大多数数据缓冲区（比如 UBO）对 Shader 来说都是“只读”的。
    * 有了 SSBO，你的 Compute Shader **不仅可以读取数据，还可以把计算结果写回去**。
    * 它甚至支持**原子操作**（Atomics），这允许多个 GPU 线程同时读写同一块内存区域而不会导致数据错乱，这对于并行计数、构建列表等高级算法至关重要。

2.  **超大容量 (Very Large Size)**
    * 在 SSBO 之前，我们有 **UBO (Uniform Buffer Object)**，它用于向 Shader 传递常量（比如摄像机矩阵）。但 UBO 的容量**非常小**（例如，通常限制在 64KB）。
    * SSBO 的容量限制**非常大**，通常只受限于你显卡的**总显存大小**（可能是几百 MB 甚至几个 GB）。

3.  **灵活的结构 (Flexible Structure)**
    * SSBO 就是一块原始内存。你可以在其中定义**任意的数据结构**，最常见的就是定义一个 `struct`（结构体），然后创建一个由这个结构体组成的**巨大数组**。

4.  **通用性 (General Purpose)**
    * SSBO 不仅可以被 **Compute Shader** 访问，也可以在**顶点着色器 (VS)** 和**片段着色器 (FS)** 中访问。这使得在不同管线阶段共享海量数据成为可能。

---

## 为什么 SSBO 如此重要？

**一句话：没有 SSBO，Compute Shader 就几乎毫无用处。**

Compute Shader (GPGPU) 的目标是处理大规模并行计算。这些计算需要满足两个条件：
1.  需要**输入**海量数据。
2.  需要**输出**海量数据。

UBO 太小了，根本无法承载这些数据。而 SSBO 完美地解决了这两个问题。

---

## 一个生动的例子：100 万个粒子的物理模拟

假设你想在 GPU 上模拟 100 万个粒子的运动。

* **数据：** 你需要存储每个粒子的 `位置 (vec3)`、`速度 (vec3)`、`生命周期 (float)` 等。这 100 万个粒子的数据量会非常大（例如 `(12+12+4) * 1,000,000 = 28MB`），UBO 根本放不下。

* **SSBO 的解决方案：**

    1.  **创建 (CPU端)：** 在 CPU 上创建一个 SSBO，其大小足以容纳 100 万个 `Particle` 结构体。
    2.  **计算 (GPU - Compute Pass)：**
        * 启动一个 **Compute Shader**，并让 GPU 开启 100 万个线程。
        * **绑定 SSBO**，让 Compute Shader 可以**读写**它。
        * **每个线程**负责**读取**一个粒子的当前位置和速度，计算它下一帧的新位置和新速度，然后**写回** SSBO 中**同一个粒子**的数据。
    3.  **渲染 (GPU - Render Pass)：**
        * 启动**渲染管线** (VS + FS)。
        * **绑定同一个 SSBO**（这次是作为只读）。
        * **顶点着色器 (VS)** 也启动 100 万次，它从 SSBO 中**读取**每个粒子的（已被 Compute Shader 更新过的）位置，然后把它们画在屏幕上。

---

## 总结对比：SSBO vs. UBO

为了帮你更好地区分，这里有一个表格：

| 特性 | **SSBO (Shader Storage Buffer Object)** | **UBO (Uniform Buffer Object)** |
| :--- | :--- | :--- |
| **主要用途** | **通用计算存储 (GPGPU)**、海量数据 | **常量数据** (如变换矩阵、灯光设置) |
| **Shader 访问** | **可读 / 可写** (Read/Write) | **只读** (Read-Only) |
| **大小限制** | **非常大** (GB 级别，受显存限制) | **非常小** (KB 级别，如 64KB) |
| **主要搭档** | **Compute Shader** | Vertex Shader, Fragment Shader |

**一句话总结：**
* **UBO** 是 GPU 的一块“**只读小黑板**”，用于告诉所有线程“今天的规则是什么”（比如摄像机在哪）。
* **SSBO** 是 GPU 的一个“**巨大共享仓库**”，所有线程都可以进去拿货（读取）和放货（写入），是 GPGPU 的核心数据中枢。


# Indirect Draw
## 概念
Indirect Draw指绘制数据不再由CPU告诉GPU，而是GPU自己去读SSBO，获取绘制数据。CPU要做的事情只是发送一条指令，告诉GPU去哪里读取数据，并进行绘制。

同时，Indirect Draw也有一个对应的API，vkCmdDrawIndirect，是GPU用来给CPU发消息的。

## 样例
### 直接绘制 (Direct Draw)
```
// CPU 必须在“调用时”就明确知道要画多少个顶点
int vertexCount = 6000;
int instanceCount = 1000;
int firstVertex = 0;
int firstInstance = 0;

// CPU 将这些“参数”直接打包成命令
// "画 6000 个顶点, 1000 个实例, ..."
vkCmdDraw(commandBuffer, vertexCount, instanceCount, firstVertex, firstInstance);
```

### 间接绘制（Indirect Draw）
```
// CPU 不知道要画多少个。它只知道“绘制参数”被存在了哪个 GPU 缓冲区里。
VkBuffer drawArgumentsBuffer = ...; // 这是一个 SSBO
uint32_t bufferOffset = 0;

// CPU 发送一个“间接”命令
// "去 drawArgumentsBuffer 的 0 偏移处，自己读取参数，然后按那个参数画"
vkCmdDrawIndirect(commandBuffer, drawArgumentsBuffer, bufferOffset, 1, ...);
```