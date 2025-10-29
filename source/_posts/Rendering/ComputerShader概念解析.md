---
title: ComputerShader概念解析
date: 2025-10-29 16:57:00
tags: Rendering, Computer Shader
---

# 概念区分
GPGPU (General-Purpose computing on Graphics Processing Units)：这是一个概念或领域。它指的是“使用 GPU 强大的并行处理能力来执行传统上由 CPU 负责的通用计算任务”（如图形渲染之外的科学计算、物理模拟、AI 训练等）。

Compute Shader (计算着色器)：这是一种具体的技术实现。它是一种程序（ HLSL, GLSL, MSL 等语言编写），它允许开发者在 GPU 上执行通用的计算。它最大的特点是它独立于传统的“渲染管线”（即独立于顶点着色器和片段着色器）。

# 为什么说“它独立于传统的渲染管线”
Computer Shader与VS，FS都叫“着色器”(Shader)，都使用 HLSL 或 GLSL 这样的语言编写，并且都运行在 GPU 内部完全相同的“统一着色器核心”上。

而“独立于顶点着色器和片段着色器”是在说：传统的渲染管线工序严格定义为顶点处理(VS) -> 光栅化 -> 像素处理(FS) -> 输出图像，但是Computer Shader并不直接在这条管线里面工作，不关心什么顶点和像素，它的输入可以是任何数据，输出也可以是任何数据。在应用上，它的逻辑其实就类似在GPU上运行的代码，只不过是这一块代码是在GPU上运行的，并且由GPU统一管控。

# 使用例子
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

# Compute Shader如何助力渲染过程
Compute Shader助力渲染过程不是直接插入渲染管线，而是使用多步骤，也即Pass。

第一步，先运行一个 Compute Pass，让 Compute Shader 这个“通用车间”去完成所有繁重的计算（比如模拟成千上万个粒子的运动位置），并把计算结果存放到一块“仓库”（如 Buffer 或 Texture）里。

第二步，再运行一个 Render Pass，启动那条“渲染管线”，此时你的 VS 和 FS 程序不再需要自己去苦苦计算，它们可以直接去“仓库”里读取 Compute Shader 算好的现成数据，然后轻松地把它们画在屏幕上。