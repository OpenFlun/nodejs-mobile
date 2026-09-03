# nodejs-mobile 升级适配完整指南（v18.20.4 → v20.18.0 → v22.14.0）

**适用环境**：Windows 11 + WSL2 Ubuntu 24.04，项目位于 `/mnt/d/nodejs-mobile-build`
**目标平台**：Android ARM64，API 30（Android 11）
**编译工具链**：NDK r26c，Python 3.14.7，Rust 1.97.1

---

## 一、目录结构（成功后的最终状态）

```
/mnt/d/nodejs-mobile-build/
├── android-ndk-r26c/                      # NDK 安装目录
│   └── sources/android/cpufeatures/
│       ├── cpu-features.c
│       └── cpu-features.h
├── node-v18/                              # 原始 v18.20.4 源码（保留备用）
│   ├── node.gyp
│   ├── common.gypi
│   └── ...
├── node-v20.18.0.tar.gz                   # 官方源码包
├── node-v22.14.0.tar.gz                   # 官方源码包（升级目标）
├── nodejs-mobile/                         # 主项目目录（已升级到 v22）
│   ├── android-configure                  # nodejs-mobile 特有
│   ├── android_configure.py               # nodejs-mobile 特有
│   ├── android-patches/                   # nodejs-mobile 特有（含 trap-handler.h.patch）
│   ├── doc_mobile/                        # nodejs-mobile 特有
│   ├── node.gyp                           # 已修改（添加 cpu-features.c 和 include path）
│   ├── common.gypi                        # 保持不变（无需添加 ANDROID_CPU_FEATURES）
│   ├── deps/zlib/
│   │   ├── cpu_features.c                 # 原 zlib 文件
│   │   ├── cpu-features.c                 # 从 NDK 复制（新增）
│   │   └── cpu-features.h                 # 从 NDK 复制（新增）
│   ├── deps/v8/src/handles/handles.h      # 已注释掉 static_assert 块
│   ├── deps/v8/src/trap-handler/trap-handler.h  # 已应用补丁
│   ├── src/node_version.h                 # 版本定义（22.14.0）
│   ├── out/Release/
│   │   ├── libnode.so                     # 最终产物（~74MB，strip 后）
│   │   └── node                           # 可执行软链接
│   └── ... (其他官方源码)
└── nodejs-mobile-build-backup/            # 备份目录（可选）
```

---

## 二、环境准备（首次搭建）

### 2.1 安装 WSL2 和 Ubuntu 24.04
在 Windows 中启用 WSL2，安装 Ubuntu 24.04 LTS，并进入 WSL 终端。

### 2.2 安装编译依赖
```bash
sudo apt update
sudo apt install -y cmake ninja-build bison flex gperf libssl-dev git wget
```

### 2.3 安装 Python 3.14.7（若系统未自带）
```bash
wget https://www.python.org/ftp/python/3.14.7/Python-3.14.7.tgz
tar -xzf Python-3.14.7.tgz
cd Python-3.14.7
./configure --enable-optimizations
make -j$(nproc)
sudo make altinstall
python3.14 --version   # 确认版本
```

### 2.4 安装 Rust 1.97.1+
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustc --version
```

### 2.5 下载 Android NDK r26c
```bash
cd /mnt/d/nodejs-mobile-build
wget https://dl.google.com/android/repository/android-ndk-r26c-linux.zip
unzip android-ndk-r26c-linux.zip
```

---

## 三、编译 v18.20.4（基线验证，可选）

如果已有可工作的 `nodejs-mobile` v18 源码，可先编译验证工具链。

```bash
cd /mnt/d/nodejs-mobile-build/nodejs-mobile
./android-configure /mnt/d/nodejs-mobile-build/android-ndk-r26c 30 arm64
make -j$(nproc)
```

成功后，`out/Release/libnode.so` 应存在，且 `strings out/Release/libnode.so | grep "v18.20.4"` 显示 `v18.20.4`。

---

## 四、升级到 v20.18.0（基础步骤）

### 4.1 下载并解压 v20.18.0 源码
```bash
cd /mnt/d/nodejs-mobile-build
wget https://nodejs.org/dist/v20.18.0/node-v20.18.0.tar.gz
tar -xzf node-v20.18.0.tar.gz
```

### 4.2 备份 nodejs-mobile 特有文件
```bash
cd nodejs-mobile
mkdir -p ../tmp_backup
cp -r android-configure android_configure.py android-patches doc_mobile ../tmp_backup/
```

### 4.3 删除旧文件并解压 v20 覆盖
```bash
rm -rf * .[^.]* 2>/dev/null
tar -xzf ../node-v20.18.0.tar.gz --strip-components=1 --overwrite
cp -r ../tmp_backup/* .
rm -rf ../tmp_backup
```

### 4.4 修改 `android-configure` 以支持 Python 3.14
将 `acceptable_pythons` 元组开头添加 `(3, 14)`：
```python
acceptable_pythons = ((3, 14), (3, 11), (3, 10), (3, 9), (3, 8), (3, 7), (3, 6))
```

### 4.5 修改 `android_configure.py` 添加 `android_ndk_path` 到 GYP_DEFINES
在 `GYP_DEFINES += " ANDROID_NDK_SYSROOT=" + toolchain_path + "/sysroot"` 之后插入：
```python
GYP_DEFINES += " android_ndk_path=" + android_ndk_path
```

### 4.6 复制 NDK 的 CPU 特性源文件到 `deps/zlib`
```bash
cp ../android-ndk-r26c/sources/android/cpufeatures/cpu-features.c deps/zlib/
cp ../android-ndk-r26c/sources/android/cpufeatures/cpu-features.h deps/zlib/
```

### 4.7 修改 `node.gyp`
在 `'target_name': '<(node_lib_target_name)'` 部分：
- `include_dirs` 末尾添加 `'deps/zlib'`
- `sources` 列表中添加 `'deps/zlib/cpu-features.c'`

### 4.8 修改 `common.gypi`（仅 v20 需要，v22 已弃用此步）
在 `OS=="android"` 条件块的 `defines` 中添加 `'ANDROID_CPU_FEATURES'`（**v22 中此宏会导致冲突，请勿添加**，改用第十一章的方法）。

### 4.9 清理并编译 v20
```bash
rm -rf out config.gypi config.mk config.status
./android-configure /path/to/ndk 30 arm64
make -j$(nproc)
```

---

## 五、验证升级是否成功（通用）

### 5.1 检查产物
```bash
ls -lh out/Release/libnode.so
```

### 5.2 检查版本字符串
```bash
strings out/Release/libnode.so | grep "v20.18.0"   # 或 v22.14.0
```

### 5.3 检查 NODE_MODULE_VERSION
```bash
strings out/Release/libnode.so | grep NODE_MODULE_VERSION | head -1
```

### 5.4 检查源码头文件
```bash
grep NODE_MODULE_VERSION src/node_version.h
```

---

## 六、常见问题与解决方案

### 6.1 Python 版本不被接受
修改 `android-configure`，将 `(3, 14)` 添加到 `acceptable_pythons` 列表首位。

### 6.2 `android_getCpuFeatures` 未定义
**原因**：zlib 的 `cpu_features.c` 中调用了该函数，但未提供实现。
**解决**：按照 4.6~4.7 步，复制 NDK 源文件并添加进 `libnode` 编译。

### 6.3 重复符号错误（如 `arm_cpu_enable_pmull`）
**原因**：zlib 自己的 `cpu_features.c` 和 NDK 的 `cpu-features.c` 都定义了相同符号。
**解决**：确保只添加 NDK 的 `cpu-features.c`，不要修改 zlib 自己的编译，并且只将其加入 `libnode` 的 sources，不加入其他目标。

### 6.4 路径错误导致 .o 文件找不到
在 `sources` 中使用相对路径（如 `deps/zlib/cpu-features.c`），不要使用绝对路径或 `<(...)` 变量。

### 6.5 `common.gypi` 语法错误
手动编辑时，确保 `defines` 列表的括号、引号、逗号正确。

### 6.6 v22 特有：`handles.h` 静态断言失败
**原因**：V8 中的 `static_assert` 在 Android 编译时触发，需注释掉相关代码块。
**解决**：参见第十一章 11.2 节。

### 6.7 v22 特有：`trap-handler.h` 补丁应用失败
**原因**：补丁与当前 V8 版本不完全匹配，需手动修改或按 11.1 节处理。

---

## 七、后续操作建议

### 7.1 剥离调试符号（减小体积）
```bash
# 确认 llvm-strip 位置
ls -la /path/to/ndk/toolchains/llvm/prebuilt/linux-x86_64/bin/ | grep llvm-strip

# 执行剥离
/path/to/ndk/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip out/Release/libnode.so

# 验证版本
strings out/Release/libnode.so | grep "v22.14.0"
```

---

## 八、升级到更高版本（通用指南）

本章节提供升级到**更高版本 NDK** 或 **更高版本 Node.js** 的通用方法论。

### 8.1 升级 NDK 版本
- 下载新 NDK，验证工具链。
- 注意 `cpufeatures` 路径变化，及时更新 `node.gyp` 中的引用。

### 8.2 升级 Node.js 版本
- 备份特有文件 → 解压覆盖 → 检查版本兼容性（Python/Rust/ABI）。
- 使用 `diff` 对比 `node.gyp` 和 `common.gypi` 的变化，移植 Android 相关配置。
- 注意：某些宏（如 `ANDROID_CPU_FEATURES`）在不同版本中可能有不同作用域，需根据实际情况调整（例如 v22 中不再需要全局定义，改用补丁和局部注释）。

### 8.3 Node.js 版本与 NODE_MODULE_VERSION 对照表

| Node.js 版本 | NODE_MODULE_VERSION |
| ------------ | ------------------- |
| v18.x        | 108                 |
| v20.x        | 115                 |
| v22.x        | 127                 |
| v24.x        | 136                 |

### 8.4 升级失败时的回退策略
```bash
cd /mnt/d/nodejs-mobile-build
rm -rf nodejs-mobile
cp -r nodejs-mobile-build-backup nodejs-mobile
```

### 8.5 升级操作准则（核心实践）
1. **严谨验证**：每一步修改后，必须运行验证命令（如 `grep`）确认修改正确。
2. **优先自动化命令**：能用 `sed`、`awk` 完成修改的，优先使用脚本化方式。
3. **强制备份机制**：修改任何配置文件前先备份。
4. **快速回退**：若验证失败或编译出错，立即用备份恢复。
5. **积极处理问题**：主动分析根本原因，而非盲目尝试。

---

## 九、总结

本指南基于实际成功升级过程编写，涵盖 v18→v20→v22 的完整路径。关键点：
1. 保留 nodejs-mobile 特有文件。
2. 修改构建配置以支持 Python 3.14。
3. 解决 `android_getCpuFeatures` 链接问题（复制 NDK 源文件并添加到 libnode）。
4. 根据版本差异调整宏定义（v20 需添加 `ANDROID_CPU_FEATURES` 到 `common.gypi`，v22 则改为应用补丁和注释 `handles.h`）。
5. 精确修改 `node.gyp` 和 `common.gypi`，避免语法错误。
6. 在 v22 中，额外处理 V8 的静态断言和 trap-handler 补丁。

---

## 十、下一步计划

- 升级到 Node.js v24 LTS（中期目标）。
- 跟踪 Node.js 和 NDK 的更新，持续维护。
- 优化编译参数，进一步减小 `libnode.so` 体积。

---

## 十一、v22.14.0 LTS 升级实战（补充）

本节专为升级到 v22.14.0 提供额外操作，**必须在完成第四章通用步骤之后执行**。

### 11.1 应用 `trap-handler.h.patch` 补丁
nodejs-mobile 的 `android-patches` 目录包含该补丁，需手动应用到 V8 源码：

```bash
cd /mnt/d/nodejs-mobile-build/nodejs-mobile
patch -p1 < android-patches/trap-handler.h.patch
```

若提示找不到文件，输入完整路径：`deps/v8/src/trap-handler/trap-handler.h`。
若补丁失败（`Hunk FAILED`），则手动编辑该文件，将原本的 `#if V8_HOST_ARCH_X64 ... #endif` 整个条件块替换为：

```c
#define V8_TRAP_HANDLER_SUPPORTED false
```

（即强制关闭 trap handler）。

### 11.2 注释 V8 的 `handles.h` 静态断言
在 `deps/v8/src/handles/handles.h` 中，将如下 `#if defined(__clang__) && __clang_major__ >= 17` 至 `#endif` 的整个块注释掉：

```c
#if defined(__clang__) && __clang_major__ >= 17
      // For non-HeapObjects, there's no on-heap object to dereference, so
      // disallow using operator->.
      //
      // If you got an error here and want to access the Tagged<T>, use
      // operator* -- e.g. for `Tagged<Smi>::value()`, use `(*handle).value()`.
      static_assert(
          false,
          "This handle does not reference a heap object. Use `(*handle).foo`.");
#endif
```

用 `/* ... */` 包围，或直接删除该块。
（可以用 `sed` 一键操作：`sed -i '/^#if defined(__clang__) && __clang_major__ >= 17$/,/^#endif$/c\/* block commented out for Android build *\/' deps/v8/src/handles/handles.h`）

### 11.3 确认 `common.gypi` 中不再添加 `ANDROID_CPU_FEATURES`
在 v22 中，该宏会造成 V8 编译错误，因此请确保 `common.gypi` 的 `OS=="android"` 块中 `defines` 仅包含 `_GLIBCXX_USE_C99_MATH`，不要添加 `ANDROID_CPU_FEATURES`。
（若之前添加过，请从备份恢复或手动移除。）

### 11.4 清理并编译
```bash
rm -rf out config.gypi config.mk config.status
./android-configure /path/to/ndk 30 arm64
make -j$(nproc)
```

### 11.5 验证产物
```bash
strings out/Release/libnode.so | grep "v22.14.0"
strings out/Release/libnode.so | grep NODE_MODULE_VERSION | head -1
```
预期应显示 `v22.14.0` 和 `NODE_MODULE_VERSION 127`。

### 11.6 剥离符号
```bash
/path/to/ndk/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip out/Release/libnode.so
```

---

**文档版本**：3.0（新增 v22 LTS 升级实战）
**最后更新**：2026-09-03
**作者**：根据实际升级过程整理