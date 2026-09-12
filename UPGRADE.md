# nodejs-mobile 升级适配完整指南（v18.20.4 → v22.23.2）

**适用环境**：Windows 11 + WSL2 Ubuntu 24.04，项目位于 `/mnt/d/nodejs-mobile-build`
**目标平台**：Android ARM64 / x86_64，API 30（Android 11）
**编译工具链**：NDK r27，Python 3.14.7，Rust 1.97.1

---

## 一、目录结构（成功后的最终状态，含修改备注）

### 1.1 WSL 端（`/mnt/d/nodejs-mobile-build/`）

```
/mnt/d/nodejs-mobile-build/
├── android-ndk-r27/                   # NDK 安装目录（无需修改）
│   └── sources/android/cpufeatures/
│       ├── cpu-features.c             # 源文件：复制到 nodejs-mobile/deps/zlib/
│       └── cpu-features.h             # 源文件：复制到 nodejs-mobile/deps/zlib/
├── node-v18/                          # 旧版本源码（保留备用，无需修改）
├── node-v22.23.2.tar.gz               # 官方源码包（解压后重命名为 nodejs-mobile）
├── nodejs-mobile/
│   ├── android-configure              # 修改：支持 Python 3.14
│   ├── android_configure.py           # 修改：添加 android_ndk_path
│   ├── android-patches/               # 保留：nodejs-mobile 特有补丁
│   ├── doc_mobile/                    # 保留：nodejs-mobile 特有文档
│   ├── node.gyp                       # 修改：添加 deps/zlib、cpu-features.c、符号导出选项
│   ├── common.gypi                    # 确认：不含 ANDROID_CPU_FEATURES
│   ├── deps/zlib/
│   │   ├── cpu_features.c             # 原 zlib 文件（保留，不修改）
│   │   ├── cpu-features.c             # 新增：从 NDK 复制
│   │   └── cpu-features.h             # 新增：从 NDK 复制
│   ├── deps/v8/src/handles/handles.h  # 修改：注释静态断言
│   ├── deps/v8/src/trap-handler/trap-handler.h          # 修改：强制 V8_TRAP_HANDLER_SUPPORTED false
│   ├── deps/v8/src/trap-handler/handler-outside.cc      # 修改：添加 TryHandleSignal 桩函数
│   ├── deps/v8/src/execution/arm64/simulator-arm64.cc   # 修改：添加 ProbeMemory 桩函数
│   ├── lib/internal/modules/cjs/loader.js               # 修改：支持 require('rn-bridge') 加载 JS 包装文件
│   ├── src/node_version.h             # 自动生成/无需手动修改
│   ├── out/Release/
│   │   ├── libnode.so                 # 编译产物（每次编译会覆盖）
│   │   └── node                       # 编译产物软链接
│   └── ... (其他官方源码)
├── libnode-android/                    # 最终产物集中保存目录（推荐）
│   ├── arm64-v8a/
│   │   └── libnode.so                 # 75M（剥离后）
│   └── x86_64/
│       └── libnode.so                 # 80M（剥离后）
├── libnode-x86_64.so                  # x86_64 未剥离产物（99M）
├── libnode-x86_64-stripped.so         # x86_64 剥离后产物（80M）
└── nodejs-mobile-build-backup/        # 备份目录（可选）
```

### 1.2 Android 项目端（`D:\Extend_npm\node-mobile-app\`）

```
D:\Extend_npm\node-mobile-app\
├── node_modules\nodejs-mobile-react-native\android\libnode\
│   ├── bin\arm64-v8a\libnode.so       # 替换：编译产物
│   └── include\node\
│       ├── v8-exception.h             # 修改：ABI 匹配（Error/TypeError 添加第二参数）
│       └── v8-persistent-handle.h     # 修改：ABI 匹配（GlobalizeReference 第二参数改为值传递）
├── node_modules\nodejs-mobile-react-native\android\
│   ├── build.gradle                   # 修改：Windows 支持、Gradle 9.0、ABI 限制
│   └── src\main\cpp\rn-bridge.cpp     # 修改：Error 替代 TypeError、Global 替代 Persistent；保持 NODE_MODULE_LINKED
├── android\gradle.properties          # 修改：reactNativeArchitectures=arm64-v8a
└── nodejs-assets\nodejs-project\      # 无需修改（业务代码），但需重新安装原生模块
```

---

## 二、环境准备（首次搭建）

### 2.1 安装 WSL2 和 Ubuntu 24.04
在 Windows 中启用 WSL2，安装 Ubuntu 24.04 LTS。

### 2.2 安装编译依赖
```bash
sudo apt update
sudo apt install -y cmake ninja-build bison flex gperf libssl-dev git wget
```

### 2.3 安装 Python 3.14.7
```bash
wget https://www.python.org/ftp/python/3.14.7/Python-3.14.7.tgz
tar -xzf Python-3.14.7.tgz
cd Python-3.14.7
./configure --enable-optimizations
make -j$(nproc)
sudo make altinstall
python3.14 --version
```

### 2.4 安装 Rust 1.97.1+
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustc --version
```

### 2.5 下载 Android NDK r27
```bash
cd /mnt/d/nodejs-mobile-build
wget https://dl.google.com/android/repository/android-ndk-r27-linux.zip
unzip android-ndk-r27-linux.zip
```

---

## 三、编译 v18.20.4（基线验证，可选）

```bash
cd /mnt/d/nodejs-mobile-build/nodejs-mobile
./android-configure /mnt/d/nodejs-mobile-build/android-ndk-r27 30 arm64
make -j2
```

成功后 `out/Release/libnode.so` 存在，且 `strings out/Release/libnode.so | grep "v18.20.4"` 显示版本。

---

## 四、升级到 v22.23.2（完整步骤）

> **逻辑链提醒**：本章所有修改均在**编译 `libnode.so` 之前完成**，编译完成后产物直接用于第五章替换。

### 4.1 下载并解压 v22.23.2
```bash
cd /mnt/d/nodejs-mobile-build
wget https://nodejs.org/dist/v22.23.2/node-v22.23.2.tar.gz
tar -xzf node-v22.23.2.tar.gz
```

### 4.2 重命名旧项目并创建新目录
```bash
mv nodejs-mobile nodejs-mobile-v18
mv node-v22.23.2 nodejs-mobile
cd nodejs-mobile
```

### 4.3 复制 nodejs-mobile 特有文件
```bash
cp -r ../nodejs-mobile-v18/android-configure .
cp -r ../nodejs-mobile-v18/android_configure.py .
cp -r ../nodejs-mobile-v18/android-patches .
cp -r ../nodejs-mobile-v18/doc_mobile .
```

### 4.4 修改 `android-configure` 支持 Python 3.14
```bash
sed -i 's/acceptable_pythons = ((3, 11),/acceptable_pythons = ((3, 14), (3, 11),/' android-configure
```

### 4.5 修改 `android_configure.py` 添加 `android_ndk_path`
```bash
sed -i '/GYP_DEFINES += " ANDROID_NDK_SYSROOT=" + toolchain_path + "\/sysroot"/a\    GYP_DEFINES += " android_ndk_path=" + android_ndk_path' android_configure.py
```

### 4.6 复制 NDK CPU 特性源文件
```bash
cp ../android-ndk-r27/sources/android/cpufeatures/cpu-features.c deps/zlib/
cp ../android-ndk-r27/sources/android/cpufeatures/cpu-features.h deps/zlib/
```

### 4.7 修改 `node.gyp`
在 `'target_name': '<(node_lib_target_name)'` 部分：
- `include_dirs` 末尾添加 `'deps/zlib'`
- `sources` 列表中添加 `'deps/zlib/cpu-features.c'`
- 在 `OS=="android"` 条件块中添加：
```python
[ "OS==\"android\"", {
  "cflags": [ "-fvisibility=default" ],
  "ldflags": [ "-Wl,--export-dynamic", "-Wl,--whole-archive", "-Wl,--no-whole-archive" ],
  "defines": [ "BUILDING_V8_SHARED=1", "V8_SHARED=1", "V8_TRAP_HANDLER_SUPPORTED=0", "V8_USE_SIMULATOR=0" ],
}],
```

### 4.8 应用 `trap-handler.h` 补丁
```bash
patch -p1 < android-patches/trap-handler.h.patch
```
若失败，手动将 `deps/v8/src/trap-handler/trap-handler.h` 中 `#if V8_HOST_ARCH_X64 ... #endif` 替换为：
```c
#define V8_TRAP_HANDLER_SUPPORTED false
```

### 4.9 注释 `handles.h` 静态断言
```bash
sed -i '/^#if defined(__clang__) && __clang_major__ >= 17$/,/^#endif$/c\/* block commented out for Android build *\/' deps/v8/src/handles/handles.h
```

### 4.10 确认 `common.gypi` 不含 `ANDROID_CPU_FEATURES`
确保 `common.gypi` 的 `OS=="android"` 块中 `defines` 不包含 `ANDROID_CPU_FEATURES`。
（v20 曾需要此宏，v22 中已弃用，添加会导致 V8 编译冲突。）

### 4.11 直接修改 V8 源文件禁用 trap handler 和模拟器
由于 v22 的 V8 构建已迁移到 GN，通过 GYP 变量设置无效，必须直接修改源文件。

**4.11.1 修改 `deps/v8/src/trap-handler/handler-outside.cc`**
```bash
cp deps/v8/src/trap-handler/handler-outside.cc deps/v8/src/trap-handler/handler-outside.cc.bak
sed -i '1i #include <signal.h>' deps/v8/src/trap-handler/handler-outside.cc
sed -i 's/g_is_trap_handler_enabled = RegisterDefaultTrapHandler();/g_is_trap_handler_enabled = false;/' deps/v8/src/trap-handler/handler-outside.cc
sed -i '/^}  \/\/ namespace trap_handler/i bool TryHandleSignal(int, siginfo_t*, void*) { return false; }' deps/v8/src/trap-handler/handler-outside.cc
```

**4.11.2 修改 `deps/v8/src/execution/arm64/simulator-arm64.cc`**
```bash
cp deps/v8/src/execution/arm64/simulator-arm64.cc deps/v8/src/execution/arm64/simulator-arm64.cc.bak
echo -e '\nextern "C" bool v8_internal_simulator_ProbeMemory(uintptr_t, uintptr_t) { return false; }' >> deps/v8/src/execution/arm64/simulator-arm64.cc
```

### 4.12 修改 Node.js CJS loader 使 `require('rn-bridge')` 可用（编译前必须完成）

#### 4.12.1 背景与原理

`rn-bridge` 在 JNI 库中通过 `NODE_MODULE_LINKED(rn_bridge, Init)` 注册为**链接绑定（Linked Binding）**。在 Node.js 18 中，`require('rn-bridge')` 会通过全局注册表回退找到它并加载 `builtin_modules/rn-bridge/index.js`（一个 JS 包装文件，负责把原生绑定封装成 `channel`、`app` 等用户 API）。

但 Node.js 20+ 收紧了模块解析机制：
- 链接绑定不再暴露给 `require()`，只能通过 `process._linkedBinding('rn_bridge')` 访问。
- 直接 `process._linkedBinding('rn_bridge')` 返回的**仅是原生绑定对象**（只有 `sendMessage`、`registerChannel`、`getDataDir` 三个方法），**没有 `channel` 属性**。
- 用户业务代码（`main.js`）依赖 `rn_bridge.channel.send(...)`，此 API 由 JS 包装文件 `builtin_modules/rn-bridge/index.js` 提供。
- 该 JS 包装文件由 `nodejs-mobile-react-native` 的 Java 层通过 `copyAssetFolder("builtin_modules", builtinModulesPath)` 复制到设备目录，并在启动时通过 `setenv("NODE_PATH", modulesPath, 1)` 将该目录加入 `NODE_PATH`。

因此，正确的修复方式不是让 `require('rn-bridge')` 返回原生绑定，而是让它**加载 JS 包装文件**。

#### 4.12.2 修改 `lib/internal/modules/cjs/loader.js`

**修改位置**：`Module._load` 函数开头（约第 1193 行）

**操作命令**：
```bash
cp lib/internal/modules/cjs/loader.js lib/internal/modules/cjs/loader.js.bak
perl -0777 -pi -e 's/(Module\._load = function\(request, parent, isMain, options = kEmptyObject\) \{\n)/$1  \/\/ Nodejs-mobile: load rn-bridge JS wrapper from NODE_PATH\n  if (request === "rn-bridge") {\n    const path = require("path");\n    const fs = require("fs");\n    const nodePath = process.env.NODE_PATH || "";\n    const paths = nodePath.split(path.delimiter);\n    for (let i = 0; i < paths.length; i++) {\n      const p = paths[i];\n      if (p) {\n        const file = path.join(p, "rn-bridge", "index.js");\n        if (fs.existsSync(file)) {\n          return Module._load(file, parent, isMain, options);\n        }\n      }\n    }\n  }\n/' lib/internal/modules/cjs/loader.js
```

**验证修改**：
```bash
node --check lib/internal/modules/cjs/loader.js && sed -n '1193,1212p' lib/internal/modules/cjs/loader.js
```
预期输出包含新增的 `if (request === "rn-bridge") { ... }` 代码块，且 `node --check` 无语法错误。

**同时确保**：`rn-bridge.cpp` 中注册宏保持原样（不要改成 `NODE_MODULE_CONTEXT_AWARE`）：
```cpp
NODE_MODULE_LINKED(rn_bridge, Init);
```

#### 4.12.3 JS 包装文件的位置与作用（供参考）

- **构建产物位置**（Android APK 内部 assets）：
  ```
  nodejs-mobile-react-native/android/build/intermediates/assets/debug/mergeDebugAssets/builtin_modules/rn-bridge/index.js
  ```
- **设备上的路径**（运行时由 `NODE_PATH` 指向）：
  ```
  /data/data/<package>/files/nodejs-builtin_modules/rn-bridge/index.js
  ```
- **作用**：定义 `EventChannel`、`SystemChannel`、`MessageCodec` 等类，导出 `{ app, channel }`。内部通过 `process._linkedBinding('rn_bridge')` 获取原生绑定。

**核心代码片段**：
```js
const NativeBridge = process._linkedBinding('rn_bridge');
// ... EventChannel、SystemChannel 定义 ...
module.exports = exports = {
  app: systemChannel,
  channel: eventChannel
};
```

### 4.13 清理并编译（arm64-v8a）
```bash
rm -rf out config.gypi config.mk config.status
./android-configure /mnt/d/nodejs-mobile-build/android-ndk-r27 30 arm64
make -j4
```

### 4.14 验证产物
```bash
ls -lh out/Release/libnode.so
strings out/Release/libnode.so | grep "v22.23.2"
strings out/Release/libnode.so | grep NODE_MODULE_VERSION | head -1
```

预期应显示 `v22.23.2` 和 `NODE_MODULE_VERSION 127`。

### 4.15 剥离调试符号（推荐）
```bash
cp out/Release/libnode.so out/Release/libnode.so.bak
/mnt/d/nodejs-mobile-build/android-ndk-r27/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip out/Release/libnode.so
ls -lh out/Release/libnode.so out/Release/libnode.so.bak    # 剥离前约 97M，剥离后约 75M
```

验证剥离前后动态符号表是否一致（确保剥离安全）：
```bash
diff <(nm -D out/Release/libnode.so.bak | awk '{print $2, $3}' | sort) <(nm -D out/Release/libnode.so | awk '{print $2, $3}' | sort) && echo "符号表完全一致"
```
若输出“符号表完全一致”，则剥离安全。

---

## 五、多架构编译（arm64-v8a / x86_64 / armeabi-v7a）

### 5.1 支持的架构与现状

| 架构            | 支持状态   | 说明                                                           |
| --------------- | ---------- | -------------------------------------------------------------- |
| **arm64-v8a**   | ✅ 完全支持 | 主目标，现代 Android 设备全部支持                              |
| **x86_64**      | ✅ 完全支持 | 模拟器、部分平板；x64 host 上原生编译，无交叉编译问题          |
| **armeabi-v7a** | ❌ 不支持   | 见 5.4 节说明；V8 v22 官方不支持 x64 host 上交叉编译 32 位 arm |

### 5.2 编译 x86_64

```bash
cd /mnt/d/nodejs-mobile-build/nodejs-mobile
rm -rf out config.gypi config.mk config.status
./android-configure /mnt/d/nodejs-mobile-build/android-ndk-r27 30 x86_64
make -j4
```

成功后（末尾出现 `ln -fs out/Release/node node`），剥离符号并保存：

```bash
cp out/Release/libnode.so /mnt/d/nodejs-mobile-build/libnode-x86_64.so
/mnt/d/nodejs-mobile-build/android-ndk-r27/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip out/Release/libnode.so
cp out/Release/libnode.so /mnt/d/nodejs-mobile-build/libnode-x86_64-stripped.so
```

产物：
- 未剥离：`/mnt/d/nodejs-mobile-build/libnode-x86_64.so`（约 99M）
- 剥离后：`/mnt/d/nodejs-mobile-build/libnode-x86_64-stripped.so`（约 80M）

### 5.3 集中保存最终产物

切换架构编译时，`rm -rf out` 会清空 `out` 目录，导致上一架构的产物丢失。因此**编译完一个架构后，必须立即把 `libnode.so` 复制到独立目录**：

```bash
mkdir -p /mnt/d/nodejs-mobile-build/libnode-android/{arm64-v8a,x86_64}
cp /mnt/d/Extend_npm/node-mobile-app/node_modules/nodejs-mobile-react-native/android/libnode/bin/arm64-v8a/libnode.so /mnt/d/nodejs-mobile-build/libnode-android/arm64-v8a/libnode.so
cp /mnt/d/nodejs-mobile-build/libnode-x86_64-stripped.so /mnt/d/nodejs-mobile-build/libnode-android/x86_64/libnode.so
```

### 5.4 为什么放弃 armeabi-v7a

在 x64 host 上交叉编译 32 位 arm 目标时，会依次遇到以下不可逾越的障碍：

1. **`v8config.h:914` 硬性拒绝**：
   ```
   #error Target architecture arm is only supported on arm and ia32 host
   ```
   V8 官方只允许在 arm host 或 ia32 host 上构建 arm target。即使强行 `#if 0` 绕过，后续仍会失败。

2. **Torque 工具编译失败**：Torque 是 host 工具（x64 原生），但被错误地赋予了 `V8_TARGET_ARCH_ARM` 宏，导致 8 字节对齐检查失败：
   ```
   ../deps/v8/src/objects/ordered-hash-table.tq:58:3: Torque Error: field value at offset [4 mod 2^4] is not 8-byte aligned.
   ```
   这是架构级不兼容，不是简单补丁能修复的。

3. **实际意义有限**：Android 从 2019 年起要求新应用提供 64 位版本，现代设备几乎全部支持 arm64-v8a。Node.js v22 的运行时体量已远超 2015 年前的老设备能力，arm32 兼容意义不大。

**结论**：arm64-v8a + x86_64 已覆盖全部主流 Android 设备，无需 arm32。

### 5.5 每次切换架构编译的推荐流程

```bash
# 1. 切换到目标架构
cd /mnt/d/nodejs-mobile-build/nodejs-mobile
rm -rf out config.gypi config.mk config.status
./android-configure /mnt/d/nodejs-mobile-build/android-ndk-r27 30 <arch>   # <arch>: arm64 / x86_64

# 2. 编译
make -j4

# 3. 剥离符号
cp out/Release/libnode.so out/Release/libnode.so.bak
/mnt/d/nodejs-mobile-build/android-ndk-r27/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip out/Release/libnode.so

# 4. 立即保存产物到独立目录
mkdir -p /mnt/d/nodejs-mobile-build/libnode-android/<arch>
cp out/Release/libnode.so /mnt/d/nodejs-mobile-build/libnode-android/<arch>/libnode.so
```

---

## 六、替换到 Android 项目并适配 JNI

> **逻辑链提醒**：本章所有修改均在 **`libnode.so` 编译完成之后**、**重新编译 Android 项目之前**完成。

### 6.1 替换 `libnode.so`
```bash
cp /mnt/d/nodejs-mobile-build/libnode-android/arm64-v8a/libnode.so /mnt/d/Extend_npm/node-mobile-app/node_modules/nodejs-mobile-react-native/android/libnode/bin/arm64-v8a/libnode.so
```

若需支持 x86_64 模拟器：
```bash
cp /mnt/d/nodejs-mobile-build/libnode-android/x86_64/libnode.so /mnt/d/Extend_npm/node-mobile-app/node_modules/nodejs-mobile-react-native/android/libnode/bin/x86_64/libnode.so
```

### 6.2 修改 JNI 头文件使其与 v22 ABI 匹配

**原因**：`nodejs-mobile-react-native` 自带的旧头文件（v18 时代）与 v22 的 `libnode.so` ABI 不匹配，导致 JNI 编译时链接错误：
```
ld.lld: error: undefined symbol: v8::Exception::Error(v8::Local<v8::String>)
ld.lld: error: undefined symbol: v8::api_internal::GlobalizeReference(v8::internal::Isolate*, unsigned long*)
```

**修改 `v8-exception.h`**：
```bash
cd /mnt/d/Extend_npm/node-mobile-app/node_modules/nodejs-mobile-react-native/android/libnode/include/node
cp v8-exception.h v8-exception.h.bak
sed -i 's/static Local<Value> Error(Local<String> message);/static Local<Value> Error(Local<String> message, Local<Value> options = {});/' v8-exception.h
sed -i 's/static Local<Value> TypeError(Local<String> message);/static Local<Value> TypeError(Local<String> message, Local<Value> options = {});/' v8-exception.h
```

**修改 `v8-persistent-handle.h`**：
```bash
cp v8-persistent-handle.h v8-persistent-handle.h.bak
sed -i '/GlobalizeReference/,/handle);/ s/internal::Address\* handle);/internal::Address handle);/' v8-persistent-handle.h
sed -i 's/reinterpret_cast<internal::Isolate\*>(isolate), p));/reinterpret_cast<internal::Isolate*>(isolate), reinterpret_cast<internal::Address>(p)));/' v8-persistent-handle.h
```

### 6.3 修改 JNI 源码 `rn-bridge.cpp`

```bash
cd /mnt/d/Extend_npm/node-mobile-app/node_modules/nodejs-mobile-react-native/android/src/main/cpp
cp rn-bridge.cpp rn-bridge.cpp.bak
sed -i 's/v8::Exception::TypeError/v8::Exception::Error/g' rn-bridge.cpp
sed -i 's/v8::Persistent<v8::Function>/v8::Global<v8::Function>/g' rn-bridge.cpp
```
确保 `Error` 调用为单参数，删除多余的第二个参数。

**注意**：注册宏**保持** `NODE_MODULE_LINKED(rn_bridge, Init);` 不变（不要改为 `NODE_MODULE_CONTEXT_AWARE`），否则 JS 包装文件无法通过 `process._linkedBinding('rn_bridge')` 获取原生绑定。

### 6.4 修改 `build.gradle` 支持 Windows 平台

**原因**：`nodejs-mobile-react-native` 的 `build.gradle` 硬编码只支持 macOS 和 Linux，Windows 下会抛出 `Unsupported operating system`。

```powershell
$file = "D:\Extend_npm\node-mobile-app\node_modules\nodejs-mobile-react-native\android\build.gradle"
Copy-Item $file "$file.bak"
(Get-Content $file) | ForEach-Object {
    if ($_ -match 'Unsupported operating system for nodejs-mobile native builds') {
        "            temp_host_tag = 'windows-x86_64'"
    } elseif ($_ -match 'Unsupported opperating system for nodejs-mobile native builds') {
        '            npm_gyp_defines += " host_os=win32 OS=android"'
    } else {
        $_
    }
} | Set-Content $file
```

### 6.5 修改 `build.gradle` 修复 Gradle 9.0 `exec()` 缺失

**原因**：Gradle 9.0 移除了 `exec()` 方法，需改用 `providers.exec()`。

```powershell
$file = "D:\Extend_npm\node-mobile-app\node_modules\nodejs-mobile-react-native\android\build.gradle"
Copy-Item $file "$file.bak2"
(Get-Content $file) | ForEach-Object {
    if ($_ -match "commandLine 'node', '-p'") {
        '                commandLine ''node'', ''-p'', "process.versions.node.split(''.'')[0]"'
    } else {
        $_
    }
} | Set-Content $file
```

### 6.6 限制编译架构

**原因**：若只编译了 arm64-v8a 的 `libnode.so`，需避免 Gradle 构建 armeabi-v7a 导致链接失败。

**修改 `android/gradle.properties`**：
```properties
reactNativeArchitectures=arm64-v8a
```

若同时支持 x86_64 模拟器：
```properties
reactNativeArchitectures=arm64-v8a,x86_64
```

**修改 `nodejs-mobile-react-native/android/build.gradle`**（仅 arm64 时）：
```powershell
$file = "D:\Extend_npm\node-mobile-app\node_modules\nodejs-mobile-react-native\android\build.gradle"
Copy-Item $file "$file.bak5"
(Get-Content $file) | ForEach-Object {
    if ($_ -match 'abiFilters = project\(":app"\)') {
        '            abiFilters = ["arm64-v8a"]'
    } elseif ($_ -match 'nativeModulesABIs = \["armeabi-v7a", "arm64-v8a", "x86_64"\] as Set<String>;') {
        '        nativeModulesABIs = ["arm64-v8a"] as Set<String>;'
    } else {
        $_
    }
} | Set-Content $file
```

### 6.7 清理缓存并重新编译原生模块
```powershell
cd D:\Extend_npm\node-mobile-app\android
.\gradlew clean
cd ..
Remove-Item -Recurse -Force D:\Extend_npm\node-mobile-app\android\app\build -ErrorAction SilentlyContinue

cd D:\Extend_npm\node-mobile-app\nodejs-assets\nodejs-project
$env:NODEJS_MOBILE_BUILD_NATIVE_MODULES = "1"
Remove-Item -Recurse -Force node_modules, package-lock.json -ErrorAction SilentlyContinue
npm install
```

### 6.8 重新编译 Android 项目
```powershell
cd D:\Extend_npm\node-mobile-app
Remove-Item -Recurse -Force node_modules\nodejs-mobile-react-native\android\.cxx -ErrorAction SilentlyContinue
npx react-native run-android
```

---

## 七、验证升级成功

### 7.1 检查产物
```bash
ls -lh out/Release/libnode.so
strings out/Release/libnode.so | grep "v22.23.2"
strings out/Release/libnode.so | grep NODE_MODULE_VERSION | head -1
```

### 7.2 手机日志验证
```powershell
& "C:\Users\$env:USERNAME\AppData\Local\Android\Sdk\platform-tools\adb.exe" logcat -c
& "C:\Users\$env:USERNAME\AppData\Local\Android\Sdk\platform-tools\adb.exe" logcat | Select-String -Pattern "FATAL|AndroidRuntime|nodejs|NODEJS-MOBILE"
```

预期应看到 `Node.js v22.23.2` 启动成功，无以下错误：
- `Cannot find module 'rn-bridge'`
- `TypeError: Cannot read properties of undefined (reading 'send')`

若出现 `TypeError: Cannot read properties of undefined (reading 'send')`，说明加载的是原生绑定而非 JS 包装文件，需检查 4.12 节的 `loader.js` 补丁是否生效，以及设备上 `NODE_PATH` 目录下是否存在 `rn-bridge/index.js`。

### 7.3 环境变量补充（可选，便于自动启动）
将 adb 所在目录加入 Windows 用户 PATH，避免 `run-android` 最后一步因找不到 adb 而报错（不影响应用安装）：
```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Users\flun\AppData\Local\Android\Sdk\platform-tools", "User")
```
设置后需**重启 PowerShell** 才生效。

---

## 八、常见问题与解决方案

### 8.1 Python 版本不被接受
修改 `android-configure`，将 `(3, 14)` 添加到 `acceptable_pythons` 列表首位。

### 8.2 `android_getCpuFeatures` 未定义
复制 NDK 源文件并添加进 `libnode` 编译（见 4.6、4.7）。

### 8.3 重复符号错误（如 `arm_cpu_enable_pmull`）
确保只添加 NDK 的 `cpu-features.c`，不修改 zlib 自己的编译。

### 8.4 路径错误导致 .o 文件找不到
在 `sources` 中使用相对路径。

### 8.5 `common.gypi` 语法错误
确保括号、引号、逗号正确。

### 8.6 `handles.h` 静态断言失败
注释掉相关代码块（见 4.9）。

### 8.7 `trap-handler.h` 补丁应用失败
手动强制 `V8_TRAP_HANDLER_SUPPORTED false`（见 4.8）。

### 8.8 编译进程被 `Terminated`
降低并行任务数，使用 `make -j2`。`nproc` 为 8、内存 7.7Gi 时，推荐 `make -j4`。

### 8.9 链接错误 `TryHandleSignal`、`RegisterDefaultTrapHandler`、`v8_internal_simulator_ProbeMemory`
直接修改 V8 源文件提供桩函数（见 4.11）。

### 8.10 JNI 库链接失败（`v8::Exception::Error` 未导出）
修改 JNI 头文件与 `libnode.so` ABI 匹配（见 6.2）。

### 8.11 运行时 `Cannot find module 'rn-bridge'`
修改 Node.js CJS loader，从 `NODE_PATH` 加载 JS 包装文件（见 4.12）。

### 8.12 运行时 `TypeError: Cannot read properties of undefined (reading 'send')`
**原因**：`require('rn-bridge')` 返回的是原生绑定（只有 `sendMessage`/`registerChannel`/`getDataDir`），而不是 JS 包装文件导出的 `{ app, channel }` 对象。
**解决**：确认 4.12 节的 `loader.js` 补丁已正确应用，并重新编译 `libnode.so`。不要将 `rn-bridge.cpp` 的注册宏改为 `NODE_MODULE_CONTEXT_AWARE`。

### 8.13 Windows 平台 `Unsupported operating system`
修改 `build.gradle`（见 6.4）。

### 8.14 Gradle 9.0 `exec()` 缺失
修改 `build.gradle` 使用 `providers.exec`（见 6.5）。

### 8.15 armeabi-v7a 链接失败
见 5.4 节，放弃 arm32，只编译 arm64-v8a 和 x86_64（见 6.6）。

### 8.16 Windows 下 `'adb' is not recognized`
环境变量 PATH 未包含 platform-tools（见 7.3）。不影响应用安装，仅 `run-android` 最后自动启动失败。

### 8.17 armeabi-v7a 编译失败（Torque 对齐错误）
见 5.4 节。V8 v22 官方不支持在 x64 host 上交叉编译 32 位 arm 目标，放弃 arm32。

---

## 九、长期维护策略

### 9.1 补丁文件化
将对 Node.js 源码的所有修改固化为补丁文件，纳入版本控制：
```bash
cd /mnt/d/nodejs-mobile-build/nodejs-mobile
git diff > patches/nodejs-mobile-v22-rn-bridge.patch
```

每次升级 Node.js 大版本时：
```bash
git apply patches/nodejs-mobile-v22-rn-bridge.patch
```
若补丁失败，手动解决冲突后重新生成补丁。

### 9.2 需要固化的补丁列表
| 文件                                             | 修改内容                                                                                                    | 修改阶段        |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- | --------------- |
| `android-configure`                              | 支持 Python 3.14                                                                                            | 编译前          |
| `android_configure.py`                           | 添加 `android_ndk_path`                                                                                     | 编译前          |
| `node.gyp`                                       | 添加 `deps/zlib`、`cpu-features.c`、符号导出选项                                                            | 编译前          |
| `deps/v8/src/trap-handler/trap-handler.h`        | 强制 `V8_TRAP_HANDLER_SUPPORTED false`                                                                      | 编译前          |
| `deps/v8/src/handles/handles.h`                  | 注释静态断言                                                                                                | 编译前          |
| `deps/v8/src/trap-handler/handler-outside.cc`    | 禁用 trap handler 桩函数                                                                                    | 编译前          |
| `deps/v8/src/execution/arm64/simulator-arm64.cc` | 模拟器桩函数（仅 arm64）                                                                                    | 编译前          |
| `lib/internal/modules/cjs/loader.js`             | 从 `NODE_PATH` 加载 `rn-bridge` JS 包装文件                                                                 | 编译前          |
| `rn-bridge.cpp`                                  | 修改 API 调用（`Error` 替代 `TypeError`、`Global` 替代 `Persistent`），**保持 `NODE_MODULE_LINKED` 注册宏** | 替换 libnode 后 |
| `v8-exception.h`                                 | ABI 匹配（`Error`/`TypeError` 添加第二参数）                                                                | 替换 libnode 后 |
| `v8-persistent-handle.h`                         | ABI 匹配（`GlobalizeReference` 第二参数改为值传递）                                                         | 替换 libnode 后 |
| `build.gradle`                                   | Windows 支持、Gradle 9.0、ABI 限制                                                                          | 替换 libnode 后 |

### 9.3 每次升级 Node.js 大版本的标准流程
1. 下载新版本源码并重命名旧项目。
2. 复制 nodejs-mobile 特有文件。
3. 应用固化补丁（编译前补丁）。
4. 检查补丁是否全部成功；如失败，手动解决并重新生成补丁。
5. 编译 `libnode.so`（arm64 + x86_64）。
6. 替换到 Android 项目，应用 Android 侧补丁（JNI、build.gradle）。
7. 重新编译 Android 项目。
8. 验证运行。

### 9.4 每次升级 Node.js 大版本时对 `rn-bridge` 相关的重点检查清单

升级到 Node.js 24+ 或更高版本时，**必须逐项确认**：

1. **`lib/internal/modules/cjs/loader.js`** 中 `Module._load` 函数是否仍存在于相近位置？
   - 若 Node.js 官方重构了 CJS loader，需重新定位 `Module._load` 并插入补丁。
   - 若 Node.js 官方已支持链接绑定被 `require()` 解析，可移除本补丁。

2. **`nodejs-mobile-react-native` 的 `builtin_modules/rn-bridge/index.js`** 是否仍通过 `process._linkedBinding('rn_bridge')` 获取原生绑定？
   - 若插件升级了此文件，需确认导出的 API（`app`、`channel`）未变。

3. **`NODE_PATH` 机制**是否仍由 `native-lib.cpp` 的 `setenv("NODE_PATH", ...)` 设置？
   - 若插件改为其他机制（如 `BuiltinModule`），需相应调整 `loader.js` 补丁。

4. **`rn-bridge.cpp` 的注册宏**是否仍为 `NODE_MODULE_LINKED`？
   - 若插件升级为 `NODE_MODULE_CONTEXT_AWARE_INTERNAL` 或其他宏，需重新评估 `loader.js` 补丁是否仍适用。

5. **Node.js 官方 `process._linkedBinding`** 是否仍存在？
   - 若被重命名或移除，需同步更新 `rn-bridge/index.js` 和 `loader.js`。

6. **测试运行**：应用启动后，`main.js` 中 `require('rn-bridge')` 应返回包含 `channel` 和 `app` 属性的对象，而非仅含 `sendMessage` 等原生方法。

### 9.5 长期优化方向

**方案 A（当前采用）**：修改 `loader.js`，从 `NODE_PATH` 加载 JS 包装文件。
- 优点：完全保留插件原有的 JS 包装逻辑，业务代码零侵入。
- 缺点：依赖 Node.js 内部 CJS loader 的实现细节。

**方案 B（推荐长期演进）**：将 `rn_bridge` 注册为 Node.js 官方内置模块。
- 使用 `NODE_MODULE_CONTEXT_AWARE_INTERNAL` 注册原生绑定。
- 在 Node.js 源码的内置模块列表（如 `node_builtins.cc`）中注册 `rn_bridge`。
- 将 JS 包装文件作为内置模块的 JS 层实现（类似 `lib/internal/bootstrap/` 中的模块）。
- 优点：走官方机制，跨 Node.js 版本更稳定。
- 缺点：需要深入理解 Node.js 内置模块注册机制，初期工作量较大。

**建议**：先以方案 A 完成当前升级，后续在时间允许时逐步迁移到方案 B。

---

## 十、总结

本指南基于实际成功升级过程编写，涵盖 v18→v22 的完整路径。关键点：
1. 保留 nodejs-mobile 特有文件。
2. 修改构建配置以支持 Python 3.14。
3. 解决 `android_getCpuFeatures` 链接问题。
4. 确认 `common.gypi` 不含 `ANDROID_CPU_FEATURES`（v22 中此宏已弃用）。
5. 精确修改 `node.gyp` 和 `common.gypi`。
6. v22 中额外处理 V8 静态断言、trap handler 和模拟器。
7. 注意编译资源限制，使用 `make -j4`（8 核 / 8G 内存）或 `make -j2`（资源紧张时）。
8. 编译前修改 `loader.js`，从 `NODE_PATH` 加载 `rn-bridge` JS 包装文件（**不要**直接返回原生绑定）。
9. 替换后修改 JNI 头文件和源码，解决链接错误；**保持 `NODE_MODULE_LINKED` 注册宏**。
10. 修改 `build.gradle`，解决 Windows 平台、Gradle 9.0、ABI 限制问题。
11. 多架构支持：arm64-v8a（主目标）+ x86_64（模拟器）；armeabi-v7a 因 V8 v22 官方限制放弃。
12. 将补丁固化，每次升级重新应用，并按 9.4 节的清单逐项检查 `rn-bridge` 相关的适配点。

---

**文档版本**：10.0（新增多架构编译章节：arm64 + x86_64 支持、arm32 放弃原因；产物集中保存机制）
**最后更新**：2026-09-12
**作者**：根据实际升级过程整理