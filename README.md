# nodejs-mobile

Node.js 运行时针对 Android 平台的移植版本。

本项目将 Node.js 编译为 Android 动态库 (`libnode.so`)，方便在 Android 应用中嵌入 JavaScript 运行时。

---

## 仓库与下载

### 主仓库（GitHub）
- **源码仓库**：[https://github.com/OpenFlun/nodejs-mobile](https://github.com/OpenFlun/nodejs-mobile)
- **Release 页面**：[https://github.com/OpenFlun/nodejs-mobile/releases](https://github.com/OpenFlun/nodejs-mobile/releases)

### 国内镜像（Gitee）
- **源码镜像**：[https://gitee.com/OpenFlun/nodejs-mobile](https://gitee.com/OpenFlun/nodejs-mobile)
- **Release 页面**：[https://gitee.com/OpenFlun/nodejs-mobile/releases](https://gitee.com/OpenFlun/nodejs-mobile/releases)

---

### 预编译二进制文件（直接下载）

| 文件                  | 说明                                                               | GitHub 下载                                                                                      | Gitee 下载                                                                                      |
| --------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| `android-libnode.zip` | 完整包（约 88 MB，含 `bin/` + `include/`，解压后直接替换插件目录） | [下载](https://github.com/OpenFlun/nodejs-mobile/releases/download/v22.23.2/android-libnode.zip) | [下载](https://gitee.com/OpenFlun/nodejs-mobile/releases/download/v22.23.2/android-libnode.zip) |

> 当前最新版本：**v22.23.2**

**`android-libnode.zip` 解压后结构**：

```
libnode/
├── bin/
│   ├── arm64-v8a/
│   │   └── libnode.so          # 64位 ARM 真机（已剥离调试符号，约 113M）
│   └── x86_64/
│       └── libnode.so          # 64位 x86 模拟器（已剥离调试符号，约 118M）
└── include/
    └── node/                   # 官方 v22 headers（含 ABI 适配，无需手动修改）
```

**注意**：
- `armeabi-v7a`（32 位 ARM）在 v22+ 已放弃，原因详见 [UPGRADE.md](UPGRADE.md) 第 5 章。
- 发布名带 `android-` 前缀，为未来 iOS 等平台扩展预留接口（如 `ios-libnode.zip`）。

---

## 功能特性

- 基于 Node.js **v22.23.2 LTS**
- 目标平台：**Android arm64-v8a + x86_64 (API 30+)**
- NODE_MODULE_VERSION：**127**
- **支持 Unicode 全字符**（`--with-intl=full-icu`），可解析 `\p{...}` 等 Unicode 属性转义，兼容 Express 5
- 支持标准 Node.js API（ESM、CJS、N-API 等）
- 提供 Android JNI 绑定和运行时适配

---

## 快速开始

### 1. 下载并替换插件目录

从 [Release 页面](https://gitee.com/OpenFlun/nodejs-mobile/releases/latest) 下载 `android-libnode.zip`，解压后**直接替换** `@flun/nodejs-mobile-react-native` 插件的 `android/libnode/` 整个目录：

```bash
# 备份原有目录（首次升级时）
cd node_modules/@flun/nodejs-mobile-react-native/android
cp -r libnode libnode.bak

# 替换为新的完整目录
rm -rf libnode
unzip /path/to/android-libnode.zip -d /tmp
mv /tmp/libnode ./libnode
```

替换后无需手动修改插件头文件（`include/node/` 已是官方 v22 headers，含正确 ABI）。

### 2. 从源码构建

```bash
# 克隆仓库（国内推荐使用 Gitee）
git clone https://gitee.com/OpenFlun/nodejs-mobile.git
cd nodejs-mobile

# 配置（请将 /path/to/android-ndk 替换为实际 NDK 路径）
# 编译 arm64-v8a
./android-configure /path/to/android-ndk 30 arm64
make -j$(nproc)

# 或者编译 x86_64
./android-configure /path/to/android-ndk 30 x86_64
make -j$(nproc)

# 产物位于 out/Release/libnode.so
```

详细构建与打包指南请参考 [UPGRADE.md](UPGRADE.md)。

---

## 文档

| 文档                                     | 说明                           |
| ---------------------------------------- | ------------------------------ |
| [UPGRADE.md](UPGRADE.md)                 | 完整升级、编译、打包与适配指南 |
| [CHANGELOG.md](CHANGELOG.md)             | 版本变更记录                   |
| [CONTRIBUTING.md](CONTRIBUTING.md)       | 贡献指南                       |
| [GOVERNANCE.md](GOVERNANCE.md)           | 项目治理模型                   |
| [SECURITY.md](SECURITY.md)               | 安全策略                       |
| [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) | 行为准则                       |

---

## 集成到 Android 项目

本项目主要面向 [@flun/nodejs-mobile-react-native](https://github.com/OpenFlun/@flun/nodejs-mobile-react-native) 插件集成：

1. 下载 `android-libnode.zip` 并解压。
2. 替换插件 `node_modules/@flun/nodejs-mobile-react-native/android/libnode/` 整个目录。
3. 修改插件 `android/src/main/cpp/rn-bridge.cpp`（`Error` 替代 `TypeError`、`Global` 替代 `Persistent`）。
4. 修改插件 `android/build.gradle`（Windows 支持、Gradle 9.0 `providers.exec`、ABI 限制）。
5. 修改 `android/gradle.properties`（`reactNativeArchitectures=arm64-v8a,x86_64`）。
6. 清理缓存并重新编译：`npx react-native run-android`。

若需将 `libnode.so` 直接集成到自研 Android 应用：

1. 从 `android-libnode.zip` 中提取 `bin/<arch>/libnode.so`。
2. 复制到 `app/src/main/jniLibs/<arch>/`（`arm64-v8a` 或 `x86_64`）。
3. 在 Java/Kotlin 中加载：
   ```java
   System.loadLibrary("node");
   ```

---

## 许可

本项目基于 [Node.js](https://nodejs.org/) 源码，遵循 **ISC License**。

---

## 致谢

感谢 Node.js 官方团队和 nodejs-mobile 原作者的贡献。