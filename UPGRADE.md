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
├── node-v22.23.2.tar.gz               # 官方源码包
├── node-v22.23.2-headers.tar.gz       # 官方头文件包（打包 libnode.zip 用）
├── node-v22.23.2/                     # 官方源码解压目录
├── node-v22.23.2-headers/             # 官方头文件解压目录
│   └── include/node/                  # 完整的 Node.js 头文件（v22 ABI，直接打包进 libnode.zip）
├── libnode-tmp/                       # 中间暂存目录（各架构 .so，切换架构时不丢失）
│   ├── arm64-v8a/libnode.so
│   └── x86_64/libnode.so
├── nodejs-mobile/
│   ├── android-configure              # 修改：支持 Python 3.14
│   ├── android_configure.py           # 修改：添加 android_ndk_path；启用 full-icu
│   ├── android-patches/               # 保留：nodejs-mobile 特有补丁
│   ├── doc_mobile/                    # 保留：nodejs-mobile 特有文档
│   ├── node.gyp                       # 修改：添加 deps/zlib、cpu-features.c、符号导出选项
│   ├── common.gypi                    # 确认：不含 ANDROID_CPU_FEATURES
│   ├── deps/zlib/
│   │   ├── cpu_features.c             # 原 zlib 文件（保留，不修改）
│   │   ├── cpu-features.c             # 新增：从 NDK 复制
│   │   └── cpu-features.h             # 新增：从 NDK 复制
│   ├── deps/icu-tmp/
│   │   └── icudt78l.dat               # 新增：full-icu 数据文件（约 32M，configure 自动下载）
│   ├── deps/v8/src/handles/handles.h  # 修改：注释静态断言
│   ├── deps/v8/src/trap-handler/trap-handler.h          # 修改：强制 V8_TRAP_HANDLER_SUPPORTED false
│   ├── deps/v8/src/trap-handler/handler-outside.cc      # 修改：添加 TryHandleSignal 桩函数
│   ├── deps/v8/src/execution/arm64/simulator-arm64.cc   # 修改：添加 ProbeMemory 桩函数
│   ├── lib/internal/modules/cjs/loader.js               # 修改：支持 require('rn-bridge') 加载 JS 包装文件
│   ├── src/node_version.h             # 自动生成/无需手动修改
│   ├── out/Release/
│   │   ├── libnode.so                 # 编译产物（每次编译会覆盖）
│   │   └── node                       # 编译产物软链接
│   ├── out_android/                   # 最终产物目录
│   │   └── libnode.zip                # ★ 唯一发布物（完整包：bin + include，约 88M）
│   └── ... (其他官方源码)
└── nodejs-mobile-build-backup/        # 备份目录（可选）
```

### 1.2 你的测试项目端（以使用 `@flun/node-mobile-app`包为例）

```
你的项目/
├── package.json              # 你自己的
├── mobileAppConfig.js        # CLI 生成的配置（可编辑）
├── server.js                 # Node 启动脚本（你写的，或 CLI 生成的示例）
├── node_modules/             # 你的依赖
│   └── @flun/nodejs-mobile-react-native/android/
│       ├── libnode/          # ★ 唯一手动替换点（见第七、八章）
│       │   ├── bin/arm64-v8a/libnode.so
│       │   ├── bin/x86_64/libnode.so
│       │   └── include/node/           # 官方 v22 headers（含正确 ABI）
│       ├── build.gradle                # CLI 自动 patch
│       └── src/main/cpp/rn-bridge.cpp  # CLI 自动 patch
├── build/                    # 你的资源（图标、keystore 等）
│   ├── icon.png
│   └── release.keystore
│── 你的其它文件/目录...
│
└── node-mobile-app-build/    # CLI 生成的 RN 工程（可整体删除重建）
    ├── android/  ios/  App.tsx  index.js  ...
    ├── mobileApp.runtime.ts  # CLI 生成，App.tsx 读
    └── nodejs-assets/
        └── nodejs-project/   # 你的 Node 项目副本
            ├── server.js     # 你的源码
            ├── main.js       # CLI 生成，桥接代码
            ├── package.json  # 只保留 dependencies
            └── node_modules/ # 你的生产依赖
```

> **发布物唯一**：只发布 `libnode.zip`，结构与官方 nodejs-mobile 一致（`bin/<arch>/libnode.so` + `include/node/`）。用户下载后直接替换整个插件 `android/libnode/` 目录，无需再手动改头文件。

---

## 二、环境准备（首次搭建）

### 2.1 安装 WSL2 和 Ubuntu 24.04
在 Windows 中启用 WSL2，安装 Ubuntu 24.04 LTS。

### 2.2 安装编译依赖
```bash
sudo apt update
sudo apt install -y cmake ninja-build bison flex gperf libssl-dev git wget zip
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

### 2.6 下载官方 v22.23.2 头文件包（打包 libnode.zip 用）

```bash
cd /mnt/d/nodejs-mobile-build
wget https://nodejs.org/download/release/v22.23.2/node-v22.23.2-headers.tar.gz
tar -xzf node-v22.23.2-headers.tar.gz
mv node-v22.23.2 node-v22.23.2-headers
ls -d node-v22.23.2-headers && ls node-v22.23.2-headers/include/node/ | head -5
```

> **注意**：官方源码包和头文件包解压后**目录名相同**（均为 `node-v22.23.2/`）。必须先解压并重命名头文件包，或解压后立即 `mv`，否则与源码目录冲突。

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

> **逻辑链提醒**：本章所有修改均在**编译 `libnode.so` 之前完成**。

### 4.1 下载并解压 v22.23.2 源码
```bash
cd /mnt/d/nodejs-mobile-build
wget https://nodejs.org/dist/v22.23.2/node-v22.23.2.tar.gz
tar -xzf node-v22.23.2.tar.gz
```

> 若已在 2.6 节解压过头文件包并重命名为 `node-v22.23.2-headers/`，此处 `node-v22.23.2/` 是干净的源码目录。若解压顺序颠倒（先源码后头文件），先确认 `node-v22.23.2/` 里有 `node.gyp`、`deps/`、`lib/`，否则重新下载源码。

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

### 4.6 修改 `android_configure.py` 启用 full-icu（Unicode 全字符支持）

#### 4.6.1 背景与问题现象

`nodejs-mobile` 的默认构建配置使用 `--with-intl=none`，不包含完整的 ICU（International Components for Unicode）数据。这会导致 V8 引擎在解析含 **Unicode 属性转义**（`\p{...}`）的正则表达式时直接抛出语法错误：

```js
const ID_START = /^[$_\p{ID_Start}]$/u;              // SyntaxError
const ID_CONTINUE = /^[$\u200c\u200d\p{ID_Continue}]$/u;  // SyntaxError
```

典型报错信息：

```
Invalid regular expression: /^[$_\p{ID_Start}]$/u: Invalid property name in character class
```

**影响范围**：许多现代 npm 包在源码中直接使用 `\p{...}` 正则，例如：

- `path-to-regexp@8`（Express 5 的核心依赖 `router` 依赖它）
- `@flun/mailer`
- 其他大量使用 Unicode 感知校验的包

在 `--with-intl=none` 环境下，这些包一旦被 `require`/`import`，就会因为 V8 无法解析正则而失败，进而导致整个 Node.js 入口脚本（如 `server.js`）加载失败。

#### 4.6.2 修改 `android_configure.py`

将 `--with-intl=none` 改为 `--with-intl=full-icu`：

```bash
sed -i 's/--with-intl=none/--with-intl=full-icu/' android_configure.py
grep -n "with-intl" android_configure.py
```

预期输出：

```
84:    # nodejs-mobile patch: added --with-intl=full-icu and --shared
85:    os.system("./configure --dest-cpu=" + DEST_CPU + " --dest-os=android --openssl-no-asm --with-intl=full-icu --cross-compiling --shared")
```

#### 4.6.3 三种 ICU 配置对比

| 配置        | 含义               | 体积            | `\p{...}` 支持         |
| ----------- | ------------------ | --------------- | ---------------------- |
| `none`      | 不含 ICU（原默认） | 最小            | ❌ 完全不支持           |
| `small-icu` | 仅英文区域数据     | 中等            | ⚠️ 部分支持（可能失败） |
| `full-icu`  | 完整 ICU 数据      | 最大（约 +30M） | ✅ 完全支持             |

**推荐直接使用 `full-icu`**。编译一次通常需要 30 分钟至 2 小时，使用 `small-icu` 一旦发现不够用就得重新编译，代价太高。

#### 4.6.4 验证 ICU 已启用

configure 完成后运行以下命令：

```bash
grep -i "intl\|icu" config.gypi | head -20
ls -d deps/icu* 2>/dev/null
ls -lh deps/icu-tmp/icudt*l.dat
```

预期输出应包含：

```
    "icu_small": "false",
    "icu_gyp_path": "tools/icu/icu-generic.gyp",
    "icu_path": "deps/icu-small",
    "icu_ver_major": "78",
    ...
deps/icu-small  deps/icu-tmp
-rwxrwxrwx 1 flun flun 32M ... deps/icu-tmp/icudt78l.dat
```

关键判断：

- `icu_small: false` → 表示使用的是 `full-icu`（若为 `true` 则是 `small-icu`）
- `icudt78l.dat` 存在且约 32M → 完整的 ICU 数据文件已就位

**注意**：`full-icu` 在 configure 阶段会从网络下载 ICU 数据（几百 MB 的源码包，最终生成约 32M 的 `.dat` 文件）。确保 WSL 能访问外网。若下载失败，可手动下载后放入 `deps/icu-tmp/`。

### 4.7 复制 NDK CPU 特性源文件
```bash
cp ../android-ndk-r27/sources/android/cpufeatures/cpu-features.c deps/zlib/
cp ../android-ndk-r27/sources/android/cpufeatures/cpu-features.h deps/zlib/
```

### 4.8 修改 `node.gyp`
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

### 4.9 应用 `trap-handler.h` 补丁
```bash
patch -p1 < android-patches/trap-handler.h.patch
```
若失败，手动将 `deps/v8/src/trap-handler/trap-handler.h` 中 `#if V8_HOST_ARCH_X64 ... #endif` 替换为：
```c
#define V8_TRAP_HANDLER_SUPPORTED false
```

### 4.10 注释 `handles.h` 静态断言
```bash
sed -i '/^#if defined(__clang__) && __clang_major__ >= 17$/,/^#endif$/c\/* block commented out for Android build *\/' deps/v8/src/handles/handles.h
```

### 4.11 确认 `common.gypi` 不含 `ANDROID_CPU_FEATURES`
确保 `common.gypi` 的 `OS=="android"` 块中 `defines` 不包含 `ANDROID_CPU_FEATURES`。
（v20 曾需要此宏，v22 中已弃用，添加会导致 V8 编译冲突。）

### 4.12 直接修改 V8 源文件禁用 trap handler 和模拟器
由于 v22 的 V8 构建已迁移到 GN，通过 GYP 变量设置无效，必须直接修改源文件。

**4.12.1 修改 `deps/v8/src/trap-handler/handler-outside.cc`**
```bash
cp deps/v8/src/trap-handler/handler-outside.cc deps/v8/src/trap-handler/handler-outside.cc.bak
sed -i '1i #include <signal.h>' deps/v8/src/trap-handler/handler-outside.cc
sed -i 's/g_is_trap_handler_enabled = RegisterDefaultTrapHandler();/g_is_trap_handler_enabled = false;/' deps/v8/src/trap-handler/handler-outside.cc
sed -i '/^}  \/\/ namespace trap_handler/i bool TryHandleSignal(int, siginfo_t*, void*) { return false; }' deps/v8/src/trap-handler/handler-outside.cc
```

**4.12.2 修改 `deps/v8/src/execution/arm64/simulator-arm64.cc`**
```bash
cp deps/v8/src/execution/arm64/simulator-arm64.cc deps/v8/src/execution/arm64/simulator-arm64.cc.bak
echo -e '\nextern "C" bool v8_internal_simulator_ProbeMemory(uintptr_t, uintptr_t) { return false; }' >> deps/v8/src/execution/arm64/simulator-arm64.cc
```

### 4.13 修改 Node.js CJS loader 使 `require('rn-bridge')` 可用（编译前必须完成）

#### 4.13.1 背景与原理

`rn-bridge` 在 JNI 库中通过 `NODE_MODULE_LINKED(rn_bridge, Init)` 注册为**链接绑定（Linked Binding）**。在 Node.js 18 中，`require('rn-bridge')` 会通过全局注册表回退找到它并加载 `builtin_modules/rn-bridge/index.js`（一个 JS 包装文件，负责把原生绑定封装成 `channel`、`app` 等用户 API）。

但 Node.js 20+ 收紧了模块解析机制：
- 链接绑定不再暴露给 `require()`，只能通过 `process._linkedBinding('rn_bridge')` 访问。
- 直接 `process._linkedBinding('rn_bridge')` 返回的**仅是原生绑定对象**（只有 `sendMessage`、`registerChannel`、`getDataDir` 三个方法），**没有 `channel` 属性**。
- 用户业务代码（`main.js`）依赖 `rn_bridge.channel.send(...)`，此 API 由 JS 包装文件 `builtin_modules/rn-bridge/` 目录下的入口提供（由该目录 `package.json` 的 `main` 字段决定）。
- 该 JS 包装文件由 `@flun/nodejs-mobile-react-native` 的 Java 层通过 `copyAssetFolder("builtin_modules", builtinModulesPath)` 复制到设备目录，并在启动时通过 `setenv("NODE_PATH", modulesPath, 1)` 将该目录加入 `NODE_PATH`。

因此，正确的修复方式不是让 `require('rn-bridge')` 返回原生绑定，而是让它**加载 JS 包装文件**。

#### 4.13.2 修改 `lib/internal/modules/cjs/loader.js`

**说明**：补丁只负责定位 `rn-bridge` 目录，入口文件名交给该目录 `package.json` 的 `main` 字段解析。这样无论入口是 `.js` / `.cjs` / `.mjs` 还是未来任何扩展名，都能正确加载，也不再受 `type: module` 改造影响。

**修改位置**：`Module._load` 函数开头（约第 1193 行）

**操作命令**：
```bash
cp lib/internal/modules/cjs/loader.js lib/internal/modules/cjs/loader.js.bak
perl -0777 -pi -e 's/(Module\._load = function\(request, parent, isMain, options = kEmptyObject\) \{\n)/$1  \/\/ Nodejs-mobile: load rn-bridge JS wrapper from NODE_PATH\n  if (request === "rn-bridge") {\n    const path = require("path");\n    const fs = require("fs");\n    const nodePath = process.env.NODE_PATH || "";\n    const paths = nodePath.split(path.delimiter);\n    for (let i = 0; i < paths.length; i++) {\n      const p = paths[i];\n      if (p) {\n        const file = path.join(p, "rn-bridge");\n        if (fs.existsSync(file)) {\n          return Module._load(file, parent, isMain, options);\n        }\n      }\n    }\n  }\n/' lib/internal/modules/cjs/loader.js
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

#### 4.13.3 JS 包装文件的位置与作用（供参考）

- **构建产物位置**（Android APK 内部 assets）：
  ```
  @flun/nodejs-mobile-react-native/android/build/intermediates/assets/debug/mergeDebugAssets/builtin_modules/rn-bridge/
  ```
- **设备上的路径**（运行时由 `NODE_PATH` 指向）：
  ```
  /data/data/<package>/files/nodejs-builtin_modules/rn-bridge/
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

### 4.14 编译 arm64-v8a（第一轮）
```bash
rm -rf out config.gypi config.mk config.status
./android-configure /mnt/d/nodejs-mobile-build/android-ndk-r27 30 arm64
make -j4
```

### 4.15 验证 arm64-v8a 产物
```bash
ls -lh out/Release/libnode.so
strings out/Release/libnode.so | grep "v22.23.2"
strings out/Release/libnode.so | grep NODE_MODULE_VERSION | head -1
strings out/Release/libnode.so | grep -c "icudt78l"    # 应输出 4318 左右
```

预期应显示 `v22.23.2`、`NODE_MODULE_VERSION 127`，ICU 符号数约 4318。

### 4.16 剥离调试符号并暂存到中间目录
```bash
cp out/Release/libnode.so out/Release/libnode.so.bak
/mnt/d/nodejs-mobile-build/android-ndk-r27/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip out/Release/libnode.so

# 验证剥离前后动态符号表一致
diff <(nm -D out/Release/libnode.so.bak | awk '{print $2, $3}' | sort) <(nm -D out/Release/libnode.so | awk '{print $2, $3}' | sort) && echo "符号表完全一致"

# 暂存到中间目录（不要放 out_android，最终只发布 libnode.zip）
mkdir -p /mnt/d/nodejs-mobile-build/libnode-tmp/arm64-v8a
cp out/Release/libnode.so /mnt/d/nodejs-mobile-build/libnode-tmp/arm64-v8a/libnode.so
ls -lh /mnt/d/nodejs-mobile-build/libnode-tmp/arm64-v8a/
```

### 4.17 编译 x86_64（第二轮）

```bash
cd /mnt/d/nodejs-mobile-build/nodejs-mobile
rm -rf out config.gypi config.mk config.status
./android-configure /mnt/d/nodejs-mobile-build/android-ndk-r27 30 x86_64
make -j4
```

成功后（末尾出现 `ln -fs out/Release/node node`），剥离符号并暂存：

```bash
cp out/Release/libnode.so out/Release/libnode.so.bak
/mnt/d/nodejs-mobile-build/android-ndk-r27/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip out/Release/libnode.so
diff <(nm -D out/Release/libnode.so.bak | awk '{print $2, $3}' | sort) <(nm -D out/Release/libnode.so | awk '{print $2, $3}' | sort) && echo "符号表完全一致"

mkdir -p /mnt/d/nodejs-mobile-build/libnode-tmp/x86_64
cp out/Release/libnode.so /mnt/d/nodejs-mobile-build/libnode-tmp/x86_64/libnode.so
ls -lh /mnt/d/nodejs-mobile-build/libnode-tmp/x86_64/
```

### 4.18 打包最终发布物 `libnode.zip`（唯一产物）

```bash
cd /mnt/d/nodejs-mobile-build && rm -rf libnode-package && mkdir -p libnode-package/libnode/bin/arm64-v8a libnode-package/libnode/bin/x86_64 libnode-package/libnode/include && \
cp -r node-v22.23.2-headers/include/node libnode-package/libnode/include/node && \
cp libnode-tmp/arm64-v8a/libnode.so libnode-package/libnode/bin/arm64-v8a/ && \
cp libnode-tmp/x86_64/libnode.so libnode-package/libnode/bin/x86_64/ && \
cd libnode-package && zip -r libnode.zip libnode && \
mkdir -p /mnt/d/nodejs-mobile-build/nodejs-mobile/out_android && \
mv libnode.zip /mnt/d/nodejs-mobile-build/nodejs-mobile/out_android/libnode.zip && \
ls -lh /mnt/d/nodejs-mobile-build/nodejs-mobile/out_android/
```

**最终 `out_android/` 只包含 `libnode.zip`**（约 88M），结构：

```
libnode/
├── bin/
│   ├── arm64-v8a/libnode.so
│   └── x86_64/libnode.so
└── include/node/
    ├── v8-exception.h
    ├── v8-persistent-handle.h
    ├── cppgc/
    ├── libplatform/
    ├── openssl/
    └── uv/
```

验证：

```bash
cd /mnt/d/nodejs-mobile-build/nodejs-mobile/out_android && unzip -l libnode.zip | head -20
unzip -p libnode.zip libnode/bin/arm64-v8a/libnode.so | strings | grep -c "icudt78l"   # 应约 4318
unzip -p libnode.zip libnode/bin/x86_64/libnode.so | strings | grep -c "icudt78l"     # 应约 4318
```

---

## 五、为什么放弃 armeabi-v7a

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

---

## 六、一键编译脚本（可选）

创建 `/mnt/d/nodejs-mobile-build/build-all-archs.sh`：

```bash
#!/bin/bash
set -e
NDK=/mnt/d/nodejs-mobile-build/android-ndk-r27
SRC=/mnt/d/nodejs-mobile-build/nodejs-mobile
TMP=/mnt/d/nodejs-mobile-build/libnode-tmp
OUT=$SRC/out_android
STRIP=$NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip
HEADERS=/mnt/d/nodejs-mobile-build/node-v22.23.2-headers/include/node

# 编译前检查：确认 full-icu 已启用
if ! grep -q "with-intl=full-icu" $SRC/android_configure.py; then
    echo "❌ 错误：android_configure.py 未启用 --with-intl=full-icu"
    exit 1
fi
echo "✅ full-icu 已启用"

mkdir -p $TMP $OUT

# 1. 编译两个架构，暂存 .so
for ARCH in arm64 x86_64; do
    echo "=== 编译 $ARCH ==="
    cd $SRC
    rm -rf out config.gypi config.mk config.status
    ./android-configure $NDK 30 $ARCH
    make -j4
    $STRIP out/Release/libnode.so
    if [ "$ARCH" = "arm64" ]; then
        DIR_NAME="arm64-v8a"
    else
        DIR_NAME="$ARCH"
    fi
    mkdir -p $TMP/$DIR_NAME
    cp out/Release/libnode.so $TMP/$DIR_NAME/libnode.so
    echo "--- $DIR_NAME ICU 符号数 ---"
    strings $TMP/$DIR_NAME/libnode.so | grep -c "icudt78l" || echo "0"
done

# 2. 打包完整 libnode.zip
echo "=== 打包 libnode.zip ==="
cd /mnt/d/nodejs-mobile-build && rm -rf libnode-package
mkdir -p libnode-package/libnode/bin/arm64-v8a libnode-package/libnode/bin/x86_64 libnode-package/libnode/include
cp -r $HEADERS libnode-package/libnode/include/node
cp $TMP/arm64-v8a/libnode.so libnode-package/libnode/bin/arm64-v8a/
cp $TMP/x86_64/libnode.so libnode-package/libnode/bin/x86_64/
cd libnode-package && zip -r libnode.zip libnode
mv libnode.zip $OUT/libnode.zip

echo "=== 完成，产物位于 $OUT ==="
ls -lh $OUT
```

执行：

```bash
chmod +x /mnt/d/nodejs-mobile-build/build-all-archs.sh
/mnt/d/nodejs-mobile-build/build-all-archs.sh
```

---

## 七、替换到 Android 项目

> 本章 `<你的项目>` 指用 `@flun/node-mobile-app` 测试的项目根目录（含 `mobileAppConfig.js` 那一层）。
> - WSL 下示例：`/mnt/d/my-project`
> - Windows 下示例：`D:\my-project`

### 7.1 备份原有 libnode 目录（首次升级时必做）

```bash
cd <你的项目>/node_modules/@flun/nodejs-mobile-react-native/android
cp -r libnode libnode.bak-v18
echo "备份完成"
```

### 7.2 替换整个 libnode 目录

```bash
cd <你的项目>/node_modules/@flun/nodejs-mobile-react-native/android
rm -rf libnode
unzip -o /mnt/d/nodejs-mobile-build/nodejs-mobile/out_android/libnode.zip -d /tmp/libnode-new
mv /tmp/libnode-new/libnode ./libnode
find libnode -maxdepth 3 -type d
ls -lh libnode/bin/arm64-v8a/ libnode/bin/x86_64/
```

**整目录替换包含 `include/node/`**（官方 v22 headers），`rn-bridge.cpp` 编译时会自动用新头文件，**无需手动改 C++ 源码**。

### 7.3 声明 ABI（CLI 自动 patch）

在 `mobileAppConfig.js` 里声明你要打包的架构：

```js
android: {
  abiFilters: ['arm64-v8a', 'x86_64'],   // 只编了一个架构就只写一个
}
```

CLI 跑 `test` / `build` 时自动 patch 三处：

| 目标文件                                                                        | 内容                           |
| ------------------------------------------------------------------------------- | ------------------------------ |
| `<buildDir>/android/gradle.properties`                                          | `reactNativeArchitectures`     |
| `<buildDir>/android/app/build.gradle`                                           | `defaultConfig.ndk.abiFilters` |
| `<你的项目>/node_modules/@flun/nodejs-mobile-react-native/android/build.gradle` | `abiFilters`                   |

**无需手动操作。**

### 7.4 重编原生模块（CLI 自动）

如果 `nodejs-project` 里有原生模块（`bcrypt`、`sqlite3` 等），在 `mobileAppConfig.js` 里设：

```js
android: {
  buildNativeModules: true,
}
```

跑 `npx node-mobile-app test` 时 CLI 自动：

- 写 `<buildDir>/nodejs-assets/BUILD_NATIVE_MODULES.txt = 1`
- 设环境变量 `NODEJS_MOBILE_BUILD_NATIVE_MODULES=1`
- 重装依赖时重编原生模块

编完后**改回 `false`**（避免每次构建都重编）。

**没有原生模块（纯 JS 依赖）**：跳过这步。

### 7.5 跑 CLI 完成构建

```bash
cd <你的项目>
npx node-mobile-app test
```

CLI 的指纹机制会检测 `libnode.so` / `abiFilters` 变化，**自动清 `.cxx` / `android/build` / `app/build`**，无需手动清任何缓存。

**CLI 还会自动**：
- Patch `@flun/nodejs-mobile-react-native/android/build.gradle`（Windows 平台支持、Gradle 9 `exec()` 兼容）
- Patch `<buildDir>/android/gradle.properties` 的 `reactNativeArchitectures`
- Patch 插件的 `abiFilters`

---

## 八、验证升级成功

### 8.1 检查产物

```bash
ls -lh /mnt/d/nodejs-mobile-build/nodejs-mobile/out_android/
unzip -l /mnt/d/nodejs-mobile-build/nodejs-mobile/out_android/libnode.zip | head -20
```

### 8.2 手机日志验证

```powershell
# adb 路径用环境变量，避免硬编码
$adb = "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe"

# <你的 appId> 是 mobileAppConfig.js 里 appId 的值
$appId = "<你的 appId>"

& $adb logcat -c
& $adb shell am force-stop $appId
& $adb shell am start -n "$appId/.MainActivity"
Start-Sleep -Seconds 5
& $adb logcat -d | Select-String -Pattern "FATAL|AndroidRuntime|nodejs|NODEJS-MOBILE"
```

预期应看到 `Node.js v22.23.2` 启动成功，无以下错误：

- `Cannot find module 'rn-bridge'`
- `TypeError: Cannot read properties of undefined (reading 'send')`
- `Invalid regular expression: /^[$_\p{ID_Start}]$/u`

### 8.3 验证 ICU（Unicode 属性转义）

**在 `server.js` 顶部临时加入**（重构后用户入口是 `server.js`，不是 `main.js`）：

```js
try {
  /^[$_\p{ID_Start}]$/u;
  console.log('=== ICU check: \\p{ID_Start} OK');
} catch (e) {
  console.log('=== ICU check failed:', e.message);
}
```

抓日志：

```powershell
$adb = "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe"
$appId = "<你的 appId>"

& $adb logcat -c
& $adb shell am start -n "$appId/.MainActivity"
Start-Sleep -Seconds 5
& $adb logcat -d | Select-String 'ICU check'
```

预期输出：

```
I NODEJS-MOBILE: === ICU check: \p{ID_Start} OK
```

### 8.4 环境变量补充（可选，便于自动启动）

```powershell
[Environment]::SetEnvironmentVariable(
  "Path",
  $env:Path + ";$env:LOCALAPPDATA\Android\Sdk\platform-tools",
  "User"
)
```

设置后需**重启 PowerShell** 才生效。

---

## 九、常见问题与解决方案

### 9.1 Python 版本不被接受
修改 `android-configure`，将 `(3, 14)` 添加到 `acceptable_pythons` 列表首位。

### 9.2 `android_getCpuFeatures` 未定义
复制 NDK 源文件并添加进 `libnode` 编译（见 4.7、4.8）。

### 9.3 重复符号错误（如 `arm_cpu_enable_pmull`）
确保只添加 NDK 的 `cpu-features.c`，不修改 zlib 自己的编译。

### 9.4 路径错误导致 .o 文件找不到
在 `sources` 中使用相对路径。

### 9.5 `common.gypi` 语法错误
确保括号、引号、逗号正确。

### 9.6 `handles.h` 静态断言失败
注释掉相关代码块（见 4.10）。

### 9.7 `trap-handler.h` 补丁应用失败
手动强制 `V8_TRAP_HANDLER_SUPPORTED false`（见 4.9）。

### 9.8 编译进程被 `Terminated`
降低并行任务数，使用 `make -j2`。`nproc` 为 8、内存 7.7Gi 时，推荐 `make -j4`。

### 9.9 链接错误 `TryHandleSignal`、`RegisterDefaultTrapHandler`、`v8_internal_simulator_ProbeMemory`
直接修改 V8 源文件提供桩函数（见 4.12）。

### 9.10 JNI 库链接失败（`v8::Exception::Error` 未导出）
使用 `libnode.zip` 替换整个 `libnode/` 目录（含官方 v22 头文件，无需手动改头文件）。

### 9.11 运行时 `Cannot find module 'rn-bridge'`
修改 Node.js CJS loader，从 `NODE_PATH` 加载 JS 包装文件（见 4.13）。

### 9.12 运行时 `TypeError: Cannot read properties of undefined (reading 'send')`
**原因**：`require('rn-bridge')` 返回的是原生绑定，而不是 JS 包装文件导出的 `{ app, channel }` 对象。
**解决**：确认 4.13 节的 `loader.js` 补丁已正确应用，并重新编译 `libnode.so`。不要将 `rn-bridge.cpp` 的注册宏改为 `NODE_MODULE_CONTEXT_AWARE`。

### 9.13 运行时 `TypeError: number 116 is not a function`（纯 ESM 加载失败）
**原因**：`path-to-regexp@8`（Express 5 的 `router` 依赖它）是纯 ESM 包，`router/lib/layer.js` 用 `require('path-to-regexp')` 加载它时，若环境中的 `require(esm)` 未生效或包缺少 `default` 导出条件，会返回非对象值。

**排查步骤**：

1. 检查 `require(esm)` 是否支持（在 `main.js` 顶部加一行 `console.log(process.features.require_module)`），预期输出 `true`。
2. 若为 `true` 但仍失败，检查报错包的 `package.json` 中 `exports` 是否有 `default` 条件。若只有 `import`，需要补上 `"default": "./dist/index.js"`。
3. 若补上 `default` 后报 `Invalid regular expression: /^[$_\p{ID_Start}]$/u: Invalid property name`，说明 `libnode.so` 未启用 ICU（见 4.6）。

**根治方案**：启用 `--with-intl=full-icu` 重新编译 `libnode.so`（见 4.6）。

### 9.14 运行时 `Invalid regular expression: /^[$_\p{ID_Start}]$/u: Invalid property name in character class`
**原因**：`libnode.so` 编译时使用了 `--with-intl=none`，V8 缺少完整 ICU 数据，无法识别 Unicode 属性转义 `\p{...}`。

**解决**：修改 `android_configure.py`，将 `--with-intl=none` 改为 `--with-intl=full-icu`，重新编译 `libnode.so`（见 4.6）。

### 9.15 Windows 平台 / Gradle 9 相关构建问题

`@flun/node-mobile-app` 已内置 patch：
- Windows 平台的 `Unsupported operating system for nodejs-mobile native builds`
- Gradle 9 的 `Could not find method exec()`

跑 `npx node-mobile-app test` 时自动处理，无需手动改 `build.gradle`。

**仅当**你不用 CLI、直接跑 `react-native run-android` 时，才需要参考本仓库 `install.js` 的 `patchNodejsMobilePlugin` 逻辑手动 patch。

### 9.16 armeabi-v7a 链接失败
见第五章，放弃 arm32，只编译 arm64-v8a 和 x86_64。

### 9.17 Windows 下 `'adb' is not recognized`
环境变量 PATH 未包含 platform-tools（见 8.4）。不影响应用安装，仅 `run-android` 最后自动启动失败。

### 9.18 `node-v22.23.2/` 目录名冲突（源码包与 headers 包同名）
**现象**：解压 `node-v22.23.2.tar.gz` 后再解压 `node-v22.23.2-headers.tar.gz`，两者都解压到 `node-v22.23.2/`，后者会覆盖或污染前者。

**解决**：解压头文件包后立即改名：
```bash
tar -xzf node-v22.23.2-headers.tar.gz
mv node-v22.23.2 node-v22.23.2-headers
```
或先解压源码包并 `mv node-v22.23.2 nodejs-mobile`，再解压头文件包。

---

## 十、长期维护策略

### 10.1 补丁文件化

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

### 10.2 需要固化的补丁列表

| 文件                                             | 修改内容                                                                                 | 修改阶段        | 谁做                         |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------- | --------------- | ---------------------------- |
| `android-configure`                              | 支持 Python 3.14                                                                         | 编译前          | 编译者                       |
| `android_configure.py`                           | 添加 `android_ndk_path`                                                                  | 编译前          | 编译者                       |
| `android_configure.py`                           | 将 `--with-intl=none` 改为 `--with-intl=full-icu`                                        | 编译前          | 编译者                       |
| `node.gyp`                                       | 添加 `deps/zlib`、`cpu-features.c`、符号导出选项                                         | 编译前          | 编译者                       |
| `deps/v8/src/trap-handler/trap-handler.h`        | 强制 `V8_TRAP_HANDLER_SUPPORTED false`                                                   | 编译前          | 编译者                       |
| `deps/v8/src/handles/handles.h`                  | 注释静态断言                                                                             | 编译前          | 编译者                       |
| `deps/v8/src/trap-handler/handler-outside.cc`    | 禁用 trap handler 桩函数                                                                 | 编译前          | 编译者                       |
| `deps/v8/src/execution/arm64/simulator-arm64.cc` | 模拟器桩函数（仅 arm64）                                                                 | 编译前          | 编译者                       |
| `lib/internal/modules/cjs/loader.js`             | 从 `NODE_PATH` 加载 `rn-bridge` JS 包装文件                                              | 编译前          | 编译者                       |
| `rn-bridge.cpp`                                  | **无需修改**——原版代码在 v22 头文件下直接编译通过（`NODE_MODULE_LINKED` 注册宏保持原样） | —               | 无需处理                     |
| `build.gradle`                                   | Windows 支持、Gradle 9.0、ABI 限制                                                       | 替换 libnode 后 | **CLI 自动**（`install.js`） |

> 采用整目录替换（`libnode.zip`）时，**不再需要对插件头文件（`v8-exception.h`、`v8-persistent-handle.h`）做补丁**，因为官方 v22 headers 本身就是正确的 ABI。

### 10.3 每次升级 Node.js 大版本的标准流程

1. 下载新版本源码并重命名旧项目。
2. 下载官方 headers 包，解压后改名为 `<version>-headers/`（避免与源码目录同名）。
3. 复制 nodejs-mobile 特有文件。
4. 应用固化补丁（编译前补丁）。
5. 检查补丁是否全部成功；如失败，手动解决并重新生成补丁。
6. 编译 arm64 → 剥离 → 暂存到 `libnode-tmp/arm64-v8a/`。
7. 编译 x86_64 → 剥离 → 暂存到 `libnode-tmp/x86_64/`。
8. 用 headers 包打包 `out_android/libnode.zip`。
9. 替换到 Android 项目（整个 `libnode/` 目录），应用 Android 侧补丁（`build.gradle`）。
10. 重新编译 Android 项目。
11. 验证运行。

### 10.4 每次升级 Node.js 大版本时对 `rn-bridge` 相关的重点检查清单

升级到 Node.js 24+ 或更高版本时，**必须逐项确认**：

1. **`lib/internal/modules/cjs/loader.js`** 中 `Module._load` 函数是否仍存在于相近位置？
   - 若 Node.js 官方重构了 CJS loader，需重新定位 `Module._load` 并插入补丁。
   - 若 Node.js 官方已支持链接绑定被 `require()` 解析，可移除本补丁。

2. **`@flun/nodejs-mobile-react-native` 的 `builtin_modules/rn-bridge/` 目录** 是否仍通过 `process._linkedBinding('rn_bridge')` 获取原生绑定？

3. **`NODE_PATH` 机制**是否仍由 `native-lib.cpp` 的 `setenv("NODE_PATH", ...)` 设置？

4. **`rn-bridge.cpp` 的注册宏**是否仍为 `NODE_MODULE_LINKED`？

5. **Node.js 官方 `process._linkedBinding`** 是否仍存在？

6. **`--with-intl=full-icu`** 在新版本 Node.js 中是否仍是有效选项？

7. **官方 headers 包**是否需要重新适配？
   - 检查 `node-vXX/include/node/v8-exception.h` 中 `Error` / `TypeError` 是否为双参数。
   - 检查 `v8-persistent-handle.h` 中 `GlobalizeReference` 是否为值传递。
   - 若官方头文件已符合新版本 ABI，则直接可用。

8. **测试运行**：应用启动后，`main.js` 中 `require('rn-bridge')` 应返回包含 `channel` 和 `app` 属性的对象。

### 10.5 长期优化方向

**当前方案**：发布完整 `libnode.zip`，用户下载后直接替换整个 `libnode/` 目录。
- 优点：与官方 nodejs-mobile 结构一致；官方 v22 头文件已包含 ABI 适配；业务代码零侵入。
- 缺点：依赖 Node.js 内部 CJS loader 的实现细节（`loader.js` 补丁）。

**长期演进**：将 `rn_bridge` 注册为 Node.js 官方内置模块（使用 `NODE_MODULE_CONTEXT_AWARE_INTERNAL` + 内置模块列表），彻底摆脱对 `loader.js` 的修改。

---

## 十一、总结

本指南基于实际成功升级过程编写，涵盖 v18→v22 的完整路径。关键点：

1. 保留 nodejs-mobile 特有文件。
2. 修改构建配置以支持 Python 3.14。
3. 解决 `android_getCpuFeatures` 链接问题。
4. 确认 `common.gypi` 不含 `ANDROID_CPU_FEATURES`。
5. 精确修改 `node.gyp` 和 `common.gypi`。
6. v22 中额外处理 V8 静态断言、trap handler 和模拟器。
7. 注意编译资源限制，使用 `make -j4` 或 `make -j2`。
8. **启用 `--with-intl=full-icu`**。
9. 编译前修改 `loader.js`，从 `NODE_PATH` 加载 `rn-bridge` JS 包装文件。
10. 替换 `libnode/` 目录（无需改 `rn-bridge.cpp`）。
11. 修改 `build.gradle`，解决 Windows 平台、Gradle 9.0、ABI 限制问题。
12. 多架构支持：arm64-v8a + x86_64；armeabi-v7a 放弃。
13. **唯一发布物**：`out_android/libnode.zip`，含 `bin/<arch>/libnode.so` + `include/node/`（官方 v22 headers）。
14. 用户下载后直接替换插件 `android/libnode/` 整个目录，无需再手动改头文件。
15. **官方源码包与 headers 包解压后目录同名**，必须先解压 headers 并改名（`node-v22.23.2-headers/`），避免与源码目录冲突。
16. 将补丁固化，每次升级重新应用，并按 10.4 节清单逐项检查。

---

**文档版本**：16.1（9.15 + 9.16 合并为一条"CLI 已自动"提示；10.2 表格新增"谁做"列，标注 `rn-bridge.cpp` / `build.gradle` 为 CLI 自动）
**最后更新**：2026-09-19
**作者**：根据实际升级过程整理