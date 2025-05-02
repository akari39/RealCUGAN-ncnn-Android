# RealCUGAN-ncnn-Android (RealCUGAN Android Wrapper)

## 项目简介
本项目基于 [nihui/realcugan-ncnn-vulkan](https://github.com/nihui/realcugan-ncnn-vulkan) 实现，封装了 Real-CUGAN 超分辨率模型，专门针对 Android 平台进行优化。它沿用了原项目在 Vulkan + ncnn 上“免 CUDA/PyTorch 运行环境、可跨 Intel/AMD/NVIDIA/Apple-Silicon GPU 快速推理”的设计理念，同时加入了对手机 CPU 和 Qualcomm GPU 的适配与兼容处理，更加贴合移动端开发者的需求。

## 特性
- **Android & 移动端优化**
  - 调整了 `tilesize` 在CPU模式下的大小以兼容移动设备
  - 本项目在 JNI 层自动检测设备厂商（vendorID/deviceName），针对 Qualcomm GPU 调整 `tilesize` 避免黑屏问题。
- **参数安全校验**
  - 对 `noise`、`scale`、`syncgap`、`gpuId` 等输入做了严格范围检查，非法值立刻抛出异常，便于快速定位错误。
  - 默认参数贴合 Android 开发者习惯，一行代码即可完成模型拷贝、初始化和推理调用。
- **内置多线程与协程支持**
  - 利用 Kotlin 协程和自定义线程池，将 GPU 推理与 IO 解码/拼装分离，保证主线程流畅不卡顿。

## 原项目注意事项（摘要）
（本节内容来源于 [nihui/realcugan-ncnn-vulkan](https://github.com/nihui/realcugan-ncnn-vulkan) 仓库 README）
> **⚠️ 本软件仍处于早期开发阶段**
>
> - **跨平台运行**：无需 CUDA/PyTorch，Windows/Linux/macOS 均可执行；  
> - **模型结构**：包含 `models-nose`、`models-pro`、`models-se` 等多种放大 + 去噪组合；  
> - **依赖库**：ncnn、libwebp、stb_image、dirent 等  

## 快速开始
1. **添加 JitPack 仓库**  
   在项目根的 `settings.gradle.kts` 或根 `build.gradle` 中添加：  
   ```groovy
   allprojects {
     repositories {
       google()
       mavenCentral()
       maven { url 'https://jitpack.io' }
     }
   }
   ```
2. 添加依赖
在你的模块（app/build.gradle）中写：
  ```groovy
  dependencies {
    implementation 'com.github.akari39:RealCUGAN-ncnn-Android:1.0.1'
  }
  ```
3. **创建模型**
  > **⚠️ 注意**
  > - 非必要请只创建一个实例
  > - 尽管作者在高通骁龙730 8GB设备上测试了3个实例可以安全运行，但请避免创建过多实例，以免导致堆栈溢出
  ```kotlin
  val options = RealCUGANOption(context, noise = -1, scale = 2, syncgap = 3, gpuId = 0)
  val engine  = RealCUGAN.create(options) // 请只创建一个
  ```
4. **执行推理**
   ```kotlin
   val outputBitmap = engine.process(inputImageByteArray)
   imageView.setImageBitmap(outputBitmap)
   ```
5. **释放资源**
   ```kotlin
   engine.release()
   ```

```
MIT License
Copyright (c) 2025 Akari
```
