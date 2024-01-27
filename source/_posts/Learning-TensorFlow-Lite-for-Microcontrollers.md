---
title: Learning TensorFlow Lite for Microcontrollers
date: 2024-01-25T11:28:18+08:00
tags:
---



## 概述

[TensorFlow Lite for Microcontrollers](https://www.tensorflow.org/lite/microcontrollers) 是 [TensorFlow](https://www.tensorflow.org/) 为微控制器实现的 [TinyML](https://www.tinyml.org/) 解决方案。其基本架构为 [tflite-micro][tflite-micro] + `tflite`：

+ [tflite-micro][tflite-micro] 是 TensorFlow 的微控制器实现。其使用 C++17 编写。微控制器上需要集成此库用于加载 `tflite` 文件。
+ `tflite` 是针对 `tflite-micro` 的模型格式。可通过 TensorFlow 导出，或 [ONNX to TF-Lite Model Conversion](https://siliconlabs.github.io/mltk/mltk/tutorials/onnx_to_tflite.html) 而来。

> ℹ️ [tflite-micro][tflite-micro]: 324ae1ea, 2024-01-23 02:40

<!-- more -->



## 本机构建 [tflite-micro][tflite-micro]

>  ℹ️ 前置条件如下：
>
>  + 至少具备 16KB RAM （仅核心库占用，模型需单独计算）
>  + 支持 C++17 的编译器
>  + 32位 CPU
>  + `make --version` 版本大于等于 `3.82`

> ⚠️ 官方文档已过时。所有信息应以 [tflite-micro][tflite-micro] 源代码目录中为准。

> ℹ️ 运行环境：
>
> + Ubuntu 22.04
> + Python 3.10.12


1. 下载 [tflite-micro][tflite-micro]：

   ```bash
   git clone https://github.com/tensorflow/tflite-micro.git
   ```

   > ℹ️ 后续所有操作均在 `tflite-micro` 目录下进行。所有相对目录位置均以 `tflite-micro` 为根目录。

2. （可选）设置代理：

   ```bash
   export http_proxy=http://127.0.0.1:7890
   export https_proxy=$http_proxy
   ```

3. 安装依赖

   ```bash
   pip install tensorflow Pillow Wave numpy
   ```

4. 生成项目

   ```bash
   make -f tensorflow/lite/micro/tools/make/Makefile
   ```

   > ⚠️ 此步骤会额外下载一些依赖。

5. 生成的文件存放于

   ```
   gen/linux_x86_64_default
   ```

   

## 构建交叉编译项目（keil）

> ⚠️ 官方文档已过时。所有信息应以 [tflite-micro][tflite-micro] 源代码目录中为准。

> ℹ️ 运行环境：
>
> + Ubuntu 22.04
> + Python 3.10.12

1. 生成独立项目目录

   ```bash
   python3 tensorflow/lite/micro/tools/project_generation/create_tflm_tree.py \
     -e hello_world \
     /tmp/tflm-tree
   ```

2. 列出所有源文件，并编写编译脚本

   ```bash
   find . -type f
   ```

   <details>
   <summary>CMakeLists.txt</summary>
   
   ```cmake
   cmake_minimum_required(VERSION 3.3)
   project(tflm)
   
   add_library(${PROJECT_NAME}
       signal/micro/kernels/delay.cc
       signal/micro/kernels/energy.cc
       signal/micro/kernels/fft_auto_scale_common.cc
       signal/micro/kernels/fft_auto_scale_kernel.cc
       signal/micro/kernels/filter_bank.cc
       signal/micro/kernels/filter_bank_log.cc
       signal/micro/kernels/filter_bank_spectral_subtraction.cc
       signal/micro/kernels/filter_bank_square_root.cc
       signal/micro/kernels/filter_bank_square_root_common.cc
       signal/micro/kernels/framer.cc
       signal/micro/kernels/irfft.cc
       signal/micro/kernels/overlap_add.cc
       signal/micro/kernels/pcan.cc
       signal/micro/kernels/rfft.cc
       signal/micro/kernels/stacker.cc
       signal/micro/kernels/window.cc
       signal/src/circular_buffer.cc
       signal/src/energy.cc
       signal/src/fft_auto_scale.cc
       signal/src/filter_bank.cc
       signal/src/filter_bank_log.cc
       signal/src/filter_bank_spectral_subtraction.cc
       signal/src/filter_bank_square_root.cc
       signal/src/irfft_float.cc
       signal/src/irfft_int16.cc
       signal/src/irfft_int32.cc
       signal/src/kiss_fft_wrappers/kiss_fft_float.cc
       signal/src/kiss_fft_wrappers/kiss_fft_int16.cc
       signal/src/kiss_fft_wrappers/kiss_fft_int32.cc
       signal/src/log.cc
       signal/src/max_abs.cc
       signal/src/msb_32.cc
       signal/src/msb_64.cc
       signal/src/overlap_add.cc
       signal/src/pcan_argc_fixed.cc
       signal/src/rfft_float.cc
       signal/src/rfft_int16.cc
       signal/src/rfft_int32.cc
       signal/src/square_root_32.cc
       signal/src/square_root_64.cc
       signal/src/window.cc
       tensorflow/lite/core/api/error_reporter.cc
       tensorflow/lite/core/api/flatbuffer_conversions.cc
       tensorflow/lite/core/api/tensor_utils.cc
       tensorflow/lite/core/c/common.cc
       tensorflow/lite/kernels/internal/common.cc
       tensorflow/lite/kernels/internal/portable_tensor_utils.cc
       tensorflow/lite/kernels/internal/quantization_util.cc
       tensorflow/lite/kernels/internal/reference/comparisons.cc
       tensorflow/lite/kernels/internal/reference/portable_tensor_utils.cc
       tensorflow/lite/kernels/internal/tensor_ctypes.cc
       tensorflow/lite/kernels/internal/tensor_utils.cc
       tensorflow/lite/kernels/kernel_util.cc
       tensorflow/lite/micro/arena_allocator/non_persistent_arena_buffer_allocator.cc
       tensorflow/lite/micro/arena_allocator/persistent_arena_buffer_allocator.cc
       tensorflow/lite/micro/arena_allocator/recording_single_arena_buffer_allocator.cc
       tensorflow/lite/micro/arena_allocator/single_arena_buffer_allocator.cc
       tensorflow/lite/micro/debug_log.cc
       tensorflow/lite/micro/fake_micro_context.cc
       tensorflow/lite/micro/flatbuffer_utils.cc
       tensorflow/lite/micro/kernels/activations.cc
       tensorflow/lite/micro/kernels/activations_common.cc
       tensorflow/lite/micro/kernels/add.cc
       tensorflow/lite/micro/kernels/add_common.cc
       tensorflow/lite/micro/kernels/add_n.cc
       tensorflow/lite/micro/kernels/arg_min_max.cc
       tensorflow/lite/micro/kernels/assign_variable.cc
       tensorflow/lite/micro/kernels/batch_matmul.cc
       tensorflow/lite/micro/kernels/batch_to_space_nd.cc
       tensorflow/lite/micro/kernels/broadcast_args.cc
       tensorflow/lite/micro/kernels/broadcast_to.cc
       tensorflow/lite/micro/kernels/call_once.cc
       tensorflow/lite/micro/kernels/cast.cc
       tensorflow/lite/micro/kernels/ceil.cc
       tensorflow/lite/micro/kernels/circular_buffer.cc
       tensorflow/lite/micro/kernels/circular_buffer_common.cc
       tensorflow/lite/micro/kernels/comparisons.cc
       tensorflow/lite/micro/kernels/concatenation.cc
       tensorflow/lite/micro/kernels/conv.cc
       tensorflow/lite/micro/kernels/conv_common.cc
       tensorflow/lite/micro/kernels/cumsum.cc
       tensorflow/lite/micro/kernels/depthwise_conv.cc
       tensorflow/lite/micro/kernels/depthwise_conv_common.cc
       tensorflow/lite/micro/kernels/depth_to_space.cc
       tensorflow/lite/micro/kernels/dequantize.cc
       tensorflow/lite/micro/kernels/dequantize_common.cc
       tensorflow/lite/micro/kernels/detection_postprocess.cc
       tensorflow/lite/micro/kernels/div.cc
       tensorflow/lite/micro/kernels/elementwise.cc
       tensorflow/lite/micro/kernels/elu.cc
       tensorflow/lite/micro/kernels/embedding_lookup.cc
       tensorflow/lite/micro/kernels/ethosu.cc
       tensorflow/lite/micro/kernels/exp.cc
       tensorflow/lite/micro/kernels/expand_dims.cc
       tensorflow/lite/micro/kernels/fill.cc
       tensorflow/lite/micro/kernels/floor.cc
       tensorflow/lite/micro/kernels/floor_div.cc
       tensorflow/lite/micro/kernels/floor_mod.cc
       tensorflow/lite/micro/kernels/fully_connected.cc
       tensorflow/lite/micro/kernels/fully_connected_common.cc
       tensorflow/lite/micro/kernels/gather.cc
       tensorflow/lite/micro/kernels/gather_nd.cc
       tensorflow/lite/micro/kernels/hard_swish.cc
       tensorflow/lite/micro/kernels/hard_swish_common.cc
       tensorflow/lite/micro/kernels/if.cc
       tensorflow/lite/micro/kernels/kernel_runner.cc
       tensorflow/lite/micro/kernels/kernel_util.cc
       tensorflow/lite/micro/kernels/l2norm.cc
       tensorflow/lite/micro/kernels/l2_pool_2d.cc
       tensorflow/lite/micro/kernels/leaky_relu.cc
       tensorflow/lite/micro/kernels/leaky_relu_common.cc
       tensorflow/lite/micro/kernels/logical.cc
       tensorflow/lite/micro/kernels/logical_common.cc
       tensorflow/lite/micro/kernels/logistic.cc
       tensorflow/lite/micro/kernels/logistic_common.cc
       tensorflow/lite/micro/kernels/log_softmax.cc
       tensorflow/lite/micro/kernels/lstm_eval.cc
       tensorflow/lite/micro/kernels/lstm_eval_common.cc
       tensorflow/lite/micro/kernels/maximum_minimum.cc
       tensorflow/lite/micro/kernels/micro_tensor_utils.cc
       tensorflow/lite/micro/kernels/mirror_pad.cc
       tensorflow/lite/micro/kernels/mul.cc
       tensorflow/lite/micro/kernels/mul_common.cc
       tensorflow/lite/micro/kernels/neg.cc
       tensorflow/lite/micro/kernels/pack.cc
       tensorflow/lite/micro/kernels/pad.cc
       tensorflow/lite/micro/kernels/pooling.cc
       tensorflow/lite/micro/kernels/pooling_common.cc
       tensorflow/lite/micro/kernels/prelu.cc
       tensorflow/lite/micro/kernels/prelu_common.cc
       tensorflow/lite/micro/kernels/quantize.cc
       tensorflow/lite/micro/kernels/quantize_common.cc
       tensorflow/lite/micro/kernels/read_variable.cc
       tensorflow/lite/micro/kernels/reduce.cc
       tensorflow/lite/micro/kernels/reduce_common.cc
       tensorflow/lite/micro/kernels/reshape.cc
       tensorflow/lite/micro/kernels/reshape_common.cc
       tensorflow/lite/micro/kernels/resize_bilinear.cc
       tensorflow/lite/micro/kernels/resize_nearest_neighbor.cc
       tensorflow/lite/micro/kernels/round.cc
       tensorflow/lite/micro/kernels/select.cc
       tensorflow/lite/micro/kernels/shape.cc
       tensorflow/lite/micro/kernels/slice.cc
       tensorflow/lite/micro/kernels/softmax.cc
       tensorflow/lite/micro/kernels/softmax_common.cc
       tensorflow/lite/micro/kernels/space_to_batch_nd.cc
       tensorflow/lite/micro/kernels/space_to_depth.cc
       tensorflow/lite/micro/kernels/split.cc
       tensorflow/lite/micro/kernels/split_v.cc
       tensorflow/lite/micro/kernels/squared_difference.cc
       tensorflow/lite/micro/kernels/squeeze.cc
       tensorflow/lite/micro/kernels/strided_slice.cc
       tensorflow/lite/micro/kernels/strided_slice_common.cc
       tensorflow/lite/micro/kernels/sub.cc
       tensorflow/lite/micro/kernels/sub_common.cc
       tensorflow/lite/micro/kernels/svdf.cc
       tensorflow/lite/micro/kernels/svdf_common.cc
       tensorflow/lite/micro/kernels/tanh.cc
       tensorflow/lite/micro/kernels/transpose.cc
       tensorflow/lite/micro/kernels/transpose_conv.cc
       tensorflow/lite/micro/kernels/unidirectional_sequence_lstm.cc
       tensorflow/lite/micro/kernels/unpack.cc
       tensorflow/lite/micro/kernels/var_handle.cc
       tensorflow/lite/micro/kernels/while.cc
       tensorflow/lite/micro/kernels/zeros_like.cc
       tensorflow/lite/micro/memory_helpers.cc
       tensorflow/lite/micro/memory_planner/greedy_memory_planner.cc
       tensorflow/lite/micro/memory_planner/linear_memory_planner.cc
       tensorflow/lite/micro/memory_planner/non_persistent_buffer_planner_shim.cc
       tensorflow/lite/micro/micro_allocation_info.cc
       tensorflow/lite/micro/micro_allocator.cc
       tensorflow/lite/micro/micro_context.cc
       tensorflow/lite/micro/micro_interpreter.cc
       tensorflow/lite/micro/micro_interpreter_context.cc
       tensorflow/lite/micro/micro_interpreter_graph.cc
       tensorflow/lite/micro/micro_log.cc
       tensorflow/lite/micro/micro_op_resolver.cc
       tensorflow/lite/micro/micro_profiler.cc
       tensorflow/lite/micro/micro_resource_variable.cc
       tensorflow/lite/micro/micro_time.cc
       tensorflow/lite/micro/micro_utils.cc
       tensorflow/lite/micro/mock_micro_graph.cc
       tensorflow/lite/micro/recording_micro_allocator.cc
       tensorflow/lite/micro/system_setup.cc
       tensorflow/lite/micro/test_helpers.cc
       tensorflow/lite/micro/test_helper_custom_ops.cc
       tensorflow/lite/micro/tflite_bridge/flatbuffer_conversions_bridge.cc
       tensorflow/lite/micro/tflite_bridge/micro_error_reporter.cc
       tensorflow/lite/schema/schema_utils.cc
       third_party/kissfft/kiss_fft.c
       third_party/kissfft/tools/kiss_fftr.c
   )
   
   target_include_directories(${PROJECT_NAME}
       PUBLIC
           $<INSTALL_INTERFACE:include>
           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
       PRIVATE
           ${CMAKE_CURRENT_SOURCE_DIR}
           ${CMAKE_CURRENT_SOURCE_DIR}/third_party/flatbuffers/include
           ${CMAKE_CURRENT_SOURCE_DIR}/third_party/gemmlowp
           ${CMAKE_CURRENT_SOURCE_DIR}/third_party/kissfft
           ${CMAKE_CURRENT_SOURCE_DIR}/third_party/ruy
   )
   
   target_compile_options(${PROJECT_NAME}
       PRIVATE
           # create_tflm_tree.py does not copy array.h
           -DTF_LITE_STATIC_MEMORY
           # fix: 'static void operator delete()' is private within this context
           $<$<COMPILE_LANGUAGE:CXX>:-fno-rtti>
           $<$<COMPILE_LANGUAGE:CXX>:-fno-exceptions>
   )
   ```
   
   </details>
   
   
   <details>
   <summary>toolchain.cmake</summary>
   
   ```cmake
   set(CMAKE_SYSTEM_NAME Generic)
   set(CMAKE_SYSTEM_PROCESSOR cortex-m4)
   
   set(CMAKE_C_COMPILER armclang.exe)
   set(CMAKE_CXX_COMPILER ${CMAKE_C_COMPILER})
   ```
   
   </details>
   
   










[tflite-micro]: https://github.com/tensorflow/tflite-micro.git
