---
title: Compare of Tinyml compilers
date: 2024-01-27T09:09:39+08:00
tags:
---



|                    | [TensorFlow Lite for Microcontrollers][tflite-micro] | [PyTorch Edge][executorch] | [deepC][deepC] | [onnx2c][onnx2c]   |
| ------------------ | ---------------------------------------------------- | -------------------------- | -------------- | ------------------ |
| 许可证             | Apache                                               | BSD                        | Apache         | ONNX2C LICENSE[^1] |
| 运行模式           | 基础库+模型                                          | 基础库+模型                | 编译[^2]       | 编译[^2]           |
| 集成语言           | C++17                                                | C++11                      | C              | C                  |
| 模型格式           | `.tflite`                                            | `.pte`                     | onnx           | onnx               |
| 核心大小（RAM,KB） | 16                                                   | 50                         |                | 8                  |
| 项目性质           | 官方                                                 | 官方                       | 社区           | 社区               |
| 活跃度             | 高                                                   | 高                         | 低             | 中                 |



> ℹ️ [TensorFlow Lite for Microcontrollers][tflite-micro] 以及 [PyTorch Edge][executorch] 的 MCU 支持均要求编译器支持编译选项 `-fno-rtti` 和 `-fno-exceptions`，这要求 Keil 使用 `armclang version 6`。而此版本不是keil的默认编译器版本，且厂商提供的 SDK 也往往使用 `armclang version 5`。



[^1]: 想干啥干啥
[^2]: 将模型直接编译为C源文件



[tflite-micro]: https://github.com/tensorflow/tflite-micro.git
[executorch]: https://github.com/pytorch/executorch.git
[deepC]: https://github.com/ai-techsystems/deepC.git
[onnx2c]: https://github.com/kraiskil/onnx2c.git
