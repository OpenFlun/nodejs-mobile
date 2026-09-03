# Changelog

所有重要的项目变更都将记录在此文件中。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

---

## [22.14.0] - 2026-09-03

### 升级
- **Node.js 内核**：从 v20.18.0 LTS 升级到 **v22.14.0 LTS**
- **NODE_MODULE_VERSION**：从 115 升级到 127

### 修复
- **V8 编译错误**：注释 `handles.h` 中的 `static_assert` 块，解决 Android 编译时 Clang 17+ 的静态断言失败问题
- **trap-handler 补丁**：应用 `android-patches/trap-handler.h.patch`，强制关闭 V8 trap handler（Android 下不完全支持）
- **宏冲突**：移除 `common.gypi` 中的 `ANDROID_CPU_FEATURES` 宏定义（v22 中不再需要，且会导致 V8 编译错误）

### 构建优化
- **并行数调优**：建议使用 `make -j2` 或 `-j4`，避免 `-j$(nproc)` 导致资源耗尽（v22 V8 更庞大）
- **产物体积**：剥离后 `libnode.so` 约 74 MB

### 文档
- 更新 `UPGRADE.md`，新增 v22 升级实战章节及编译资源调优建议

---

## [20.18.0] - 2026-09-02

### 升级
- **Node.js 内核**：从 v18.20.4 LTS 升级到 **v20.18.0 LTS**
- **Python 支持**：配置脚本现在支持 Python 3.14
- **NDK 版本**：使用 Android NDK r26c（兼容 API 30+）

### 修复
- **链接错误**：修复 `undefined reference to android_getCpuFeatures` 问题，通过引入 NDK 的 `cpu-features.c` 源文件并添加 `ANDROID_CPU_FEATURES` 宏定义
- **路径问题**：将 NDK CPU 特性源文件复制到 `deps/zlib/` 并使用相对路径，避免 GYP 路径拼接错误
- **重复符号**：避免与 zlib 库重复编译 `cpu_features.c`

### 构建优化
- 支持 Python 3.14，更新 `android-configure` 中的 `acceptable_pythons` 列表
- 在 `android_configure.py` 中添加 `android_ndk_path` 到 `GYP_DEFINES`
- 剥离调试符号，减小 `libnode.so` 体积（从 69 MB 降至 54 MB）
- 保留调试版本 `libnode.so.debug` 以供调试

### 文档
- 重写 `README.md`，提供项目概览和快速入门
- 合并 `重要参考.md` 为 `UPGRADE.md`，详细记录升级过程
- 精简文档结构，只保留核心文档（`README`、`BUILDING`、`CHANGELOG`、`UPGRADE`、`CONTRIBUTING`、`GOVERNANCE`、`SECURITY`、`CODE_OF_CONDUCT`）
- 清理冗余的 API 文档、测试文档和第三方库文档

### 打包
- 提供预编译二进制文件 `libnode.so`（剥离版）和 `libnode.so.debug`（调试版）
- 发布压缩包 `nodejs-mobile-v20.18.0-android-arm64.tar.gz`（包含源码、构建脚本、文档和预编译库）

### 已知问题
- 无重大已知问题。

---

## [18.20.4] - 2024-08（历史版本）

初始版本，基于 Node.js v18.20.4 LTS，支持 Android ARM64 平台。