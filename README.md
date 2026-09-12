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

| 文件            | 说明                                         | GitHub 下载                                                                                | Gitee 下载                                                                                |
| --------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| `arm64-v8a.zip` | 64 位 ARM 真机（已剥离调试符号，约 75 MB）   | [下载](https://github.com/OpenFlun/nodejs-mobile/releases/download/v22.23.2/arm64-v8a.zip) | [下载](https://gitee.com/OpenFlun/nodejs-mobile/releases/download/v22.23.2/arm64-v8a.zip) |
| `x86_64.zip`    | 64 位 x86 模拟器（已剥离调试符号，约 80 MB） | [下载](https://github.com/OpenFlun/nodejs-mobile/releases/download/v22.23.2/x86_64.zip)    | [下载](https://gitee.com/OpenFlun/nodejs-mobile/releases/download/v22.23.2/x86_64.zip)    |

> 当前最新版本：**v22.23.2**
> 每个 zip 包内包含对应的 `libnode.so` 文件。
> **注意**：`armeabi-v7a`（32 位 ARM）在 v22+ 已放弃，原因详见 [UPGRADE.md](UPGRADE.md) 第 5.4 节。

---

## 功能特性

- 基于 Node.js **v22.23.2 LTS**
- 目标平台：**Android ARM64 (arm64-v8a) + x86_64 (API 30+)**
- NODE_MODULE_VERSION：**127**
- 支持标准 Node.js API（ESM、CJS、N-API 等）
- 提供 Android JNI 绑定和运行时适配

---

## 快速开始

### 1. 下载预编译库

从 [Release 页面](https://gitee.com/OpenFlun/nodejs-mobile/releases/latest) 下载对应架构的 zip 包，解压后将 `libnode.so` 放入 Android 项目的对应目录：

```
app/src/main/jniLibs/
├── arm64-v8a/
│   └── libnode.so       # 64 位 ARM 真机
└── x86_64/
    └── libnode.so       # 64 位模拟器（可选）
```

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

详细构建指南请参考 [UPGRADE.md](UPGRADE.md)。

---

## 文档

| 文档                                     | 说明         |
| ---------------------------------------- | ------------ |
| [UPGRADE.md](UPGRADE.md)                 | 升级操作文档 |
| [CHANGELOG.md](CHANGELOG.md)             | 版本变更记录 |
| [CONTRIBUTING.md](CONTRIBUTING.md)       | 贡献指南     |
| [GOVERNANCE.md](GOVERNANCE.md)           | 项目治理模型 |
| [SECURITY.md](SECURITY.md)               | 安全策略     |
| [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) | 行为准则     |

---

## 集成到 Android 项目

1. 将 `libnode.so` 复制到 `app/src/main/jniLibs/<arch>/`（`arm64-v8a` 或 `x86_64`）
2. 在 Java/Kotlin 中加载：
   ```java
   System.loadLibrary("node");
   ```
3. 参考 [nodejs-mobile-react-native](https://github.com/nodejs-mobile/nodejs-mobile-react-native) 进行 React Native 集成。

**v22+ 适配提醒**：升级到 v22 后，`nodejs-mobile-react-native` 的 JNI 头文件和源码需要同步修改（`v8-exception.h`、`v8-persistent-handle.h`、`rn-bridge.cpp`），并修改 Node.js 源码的 `loader.js` 以支持 `require('rn-bridge')`。详见 [自定义libnode指南.md](自定义libnode指南.md)。

---

## 许可

本项目基于 [Node.js](https://nodejs.org/) 源码，遵循 **ISC License**。

---

## 致谢

感谢 Node.js 官方团队和 nodejs-mobile 原作者的贡献。