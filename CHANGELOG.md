# Changelog

所有重要的项目变更都将记录在此文件中。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

---


## [22.23.2] - 2026-09-12

### 升级
- 从 Node.js v18.20.4 LTS 升级到 v22.23.2 LTS
- NODE_MODULE_VERSION 从 108 升级到 127

### 新增
- 支持 Unicode 全字符（`--with-intl=full-icu`），可解析 `\p{...}` 等 Unicode 属性转义
- 支持 Android `arm64-v8a` 架构
- 支持 Android `x86_64` 架构（用于模拟器）
- **唯一发布物 `libnode.zip`**，结构与官方 nodejs-mobile 一致：
  ```
  libnode/
  ├── bin/
  │   ├── arm64-v8a/libnode.so
  │   └── x86_64/libnode.so
  └── include/node/        # 官方 v22 headers（含 ABI 适配，无需手动修改）
  ```
- 发布时重命名为 `android-libnode.zip`，为未来的 iOS 等平台扩展预留接口（如 `ios-libnode.zip`）
- 符号剥离：`arm64-v8a` 约 113M，`x86_64` 约 118M（含 full-icu）

### 修改
- **构建配置**
  - `android-configure`：支持 Python 3.14
  - `android_configure.py`：
    - 添加 `android_ndk_path` 到 GYP_DEFINES
    - 将 `--with-intl=none` 改为 `--with-intl=full-icu`
  - `node.gyp`：
    - 添加 `deps/zlib` 到 include_dirs
    - 添加 `deps/zlib/cpu-features.c` 到 sources
    - 在 Android 条件块中添加符号导出选项（`-fvisibility=default`、`-Wl,--export-dynamic`、`-Wl,--whole-archive`）
    - 定义 `V8_SHARED=1`、`BUILDING_V8_SHARED=1`、`V8_TRAP_HANDLER_SUPPORTED=0`、`V8_USE_SIMULATOR=0`
- **V8 源码**
  - `deps/v8/src/handles/handles.h`：注释掉静态断言块（Clang 17+）
  - `deps/v8/src/trap-handler/trap-handler.h`：强制 `V8_TRAP_HANDLER_SUPPORTED false`
  - `deps/v8/src/trap-handler/handler-outside.cc`：添加 `TryHandleSignal` 桩函数、禁用 `RegisterDefaultTrapHandler`
  - `deps/v8/src/execution/arm64/simulator-arm64.cc`：添加 `v8_internal_simulator_ProbeMemory` 桩函数
- **Node.js 源码**
  - `lib/internal/modules/cjs/loader.js`：修改 `Module._load`，使 `require('rn-bridge')` 从 `NODE_PATH` 加载 JS 包装文件（适配 Node.js 20+ 模块解析机制）
- **JNI 适配（Android 项目侧）**
  - `node_modules/nodejs-mobile-react-native/android/src/main/cpp/rn-bridge.cpp`：
    - `v8::Exception::TypeError` → `v8::Exception::Error`
    - `v8::Persistent<v8::Function>` → `v8::Global<v8::Function>`
    - 保持 `NODE_MODULE_LINKED(rn_bridge, Init)` 注册宏不变
  - `node_modules/nodejs-mobile-react-native/android/build.gradle`：
    - 支持 Windows 平台（`windows-x86_64`、`host_os=win32`）
    - 修复 Gradle 9.0 `exec()` 缺失，改用 `providers.exec`
    - 限制 `abiFilters` 为 `["arm64-v8a", "x86_64"]`
  - `android/gradle.properties`：`reactNativeArchitectures=arm64-v8a,x86_64`

### 发布产物
- **`android-libnode.zip`**（约 88M）：完整包，含 `bin/arm64-v8a/libnode.so`、`bin/x86_64/libnode.so`、`include/node/`（官方 v22 headers）。
- 用户下载后直接替换插件 `android/libnode/` 整个目录，**无需再手动修改头文件**。
- 单架构 zip（`arm64-v8a.zip`、`x86_64.zip`）已废弃，不再发布。

### 兼容性说明
- **armeabi-v7a 已放弃**：V8 v22 官方不支持在 x64 主机上交叉编译 32 位 ARM 目标（`v8config.h:914` 硬性拒绝，且 Torque 工具会报 8 字节对齐错误）。如需 32 位 ARM 支持，请使用 Node.js v18 或 v20。
- **V8 ABI 与 v18 不兼容**：使用 `android-libnode.zip` 整目录替换后，插件的 `include/node/` 被替换为官方 v22 headers（已含正确 ABI），但仍需修改 `rn-bridge.cpp` 源码。

### 已知问题
- 应用需重新编译所有原生 Node.js 模块（设置 `NODEJS_MOBILE_BUILD_NATIVE_MODULES=1`），否则会因 ABI 不匹配报 `NODE_MODULE_VERSION` 错误。
- full-icu 是 Express 5 的硬性前提：未启用 ICU 的 `libnode.so` 加载 `path-to-regexp@8`（Express 5 依赖）时会报 `Invalid regular expression` 或 `number 116 is not a function`。

### 文档
- 详见 `UPGRADE.md`：完整升级流程、常见问题、长期维护策略
- 详见 `自定义libnode指南.md`：替换 `libnode.zip` 后的 Android 端适配操作

---

## [18.20.4] - 2024-08（历史版本）

初始版本，基于 Node.js v18.20.4 LTS。