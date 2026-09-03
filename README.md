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

| 文件               | 说明                               | GitHub 下载                                                                                   | Gitee 下载                                                                                   |
| ------------------ | ---------------------------------- | --------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `libnode.so`       | 生产库（已剥离调试符号，约 54 MB） | [下载](https://github.com/OpenFlun/nodejs-mobile/releases/download/v20.18.0/libnode.so)       | [下载](https://gitee.com/OpenFlun/nodejs-mobile/releases/download/v20.18.0/libnode.so)       |
| `libnode.so.debug` | 调试库（含调试信息，约 69 MB）     | [下载](https://github.com/OpenFlun/nodejs-mobile/releases/download/v20.18.0/libnode.so.debug) | [下载](https://gitee.com/OpenFlun/nodejs-mobile/releases/download/v20.18.0/libnode.so.debug) |

> 当前最新版本：**v20.18.0**

---

## 功能特性

- 基于 Node.js **v20.18.0 LTS**
- 目标平台：**Android ARM64 (API 30+)**
- 支持标准 Node.js API（ESM、CJS、N-API 等）
- 提供 Android JNI 绑定和运行时适配

---

## 快速开始

### 1. 下载预编译库

从 [Release 页面](https://gitee.com/OpenFlun/nodejs-mobile/releases/latest) 下载 `libnode.so`，放入 Android 项目的 `app/src/main/jniLibs/arm64-v8a/` 目录。

### 2. 从源码构建

```bash
# 克隆仓库（国内推荐使用 Gitee）
git clone https://gitee.com/OpenFlun/nodejs-mobile.git
cd nodejs-mobile

# 配置（请将 /path/to/android-ndk 替换为实际 NDK 路径）
./android-configure /path/to/android-ndk 30 arm64

# 编译
make -j$(nproc)

# 产物位于 out/Release/libnode.so
```

详细构建指南请参考 [BUILDING.md](BUILDING.md)。

---

## 文档

| 文档                                     | 说明             |
| ---------------------------------------- | ---------------- |
| [BUILDING.md](BUILDING.md)               | Android 构建指南 |
| [UPGRADE.md](UPGRADE.md)                 | 升级操作文档     |
| [CHANGELOG.md](CHANGELOG.md)             | 版本变更记录     |
| [CONTRIBUTING.md](CONTRIBUTING.md)       | 贡献指南         |
| [GOVERNANCE.md](GOVERNANCE.md)           | 项目治理模型     |
| [SECURITY.md](SECURITY.md)               | 安全策略         |
| [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) | 行为准则         |

---

## 集成到 Android 项目

1. 将 `libnode.so` 复制到 `app/src/main/jniLibs/arm64-v8a/`
2. 在 Java/Kotlin 中加载：
   ```java
   System.loadLibrary("node");
   ```
3. 参考 [nodejs-mobile-react-native](https://github.com/nodejs-mobile/nodejs-mobile-react-native) 进行 React Native 集成。

---

## 许可

本项目基于 [Node.js](https://nodejs.org/) 源码，遵循 **ISC License**。

---

## 致谢

感谢 Node.js 官方团队和 nodejs-mobile 原作者的贡献。