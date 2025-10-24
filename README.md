# SukiSU Ultra GKI 2.0 - OnePlus Ace (MT6895)

<div align="center">

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Kernel](https://img.shields.io/badge/kernel-5.10-blue)
![Android](https://img.shields.io/badge/android-14-green)
![Platform](https://img.shields.io/badge/platform-MT6895-orange)

**为一加Ace (PGKM10) 构建的 SukiSU Ultra GKI 2.0 内核**

[功能特性](#-功能特性) • [快速开始](#-快速开始) • [编译说明](#-编译说明) • [刷入指南](#-刷入指南)

</div>

---

## 📱 设备信息

| 项目 | 信息 |
|------|------|
| 设备名称 | OnePlus Ace (一加Ace) |
| 设备代号 | pickle |
| 型号 | PGKM10 |
| 处理器 | MediaTek Dimensity 8100-MAX (MT6895) |
| 内核版本 | 5.10.x |
| Android版本 | Android 13/14 (使用Android 13 GKI内核) |
| 架构 | GKI 2.0 |

## ✨ 功能特性

- 🔐 **SukiSU Ultra** - 基于KernelSU的高级Root解决方案
- 👻 **SUSFS完整支持** - 强大的文件系统隐藏能力
  - ✅ Magic Mount - 模块化挂载系统
  - ✅ 路径隐藏 (SUS_PATH) - 隐藏敏感路径
  - ✅ 挂载隐藏 (SUS_MOUNT) - 隐藏挂载点
  - ✅ Kstat欺骗 - 文件状态伪装
  - ✅ Uname欺骗 - 内核信息伪装
  - ✅ BootConfig欺骗 - 启动配置隐藏
- 🧩 **KPM模块管理** - 内核模块动态加载
- ⚡ **LZ4K/LZ4KD压缩** - 高性能内核压缩
- 🎯 **MTK优化** - 针对天玑8100-MAX专门优化
- 🛡️ **SELinux兼容** - 完整的SELinux策略支持

## 🚀 快速开始

### 使用GitHub Actions自动编译（推荐）

1. **Fork本仓库**
   ```bash
   # 点击页面右上角的 "Fork" 按钮
   ```

2. **启用GitHub Actions**
   - 进入你Fork的仓库
   - 点击 `Settings` → `Actions` → `General`
   - 允许 "All actions and reusable workflows"

3. **开始编译**
   - 点击 `Actions` 标签
   - 选择 "Build SukiSU Ultra GKI 2.0 for OnePlus Ace (MT6895)"
   - 点击 `Run workflow`
   - 配置编译参数：
     - 内核版本: `5.10` (默认)
     - Android版本: `android13` (默认，适用于5.10内核)
     - KPM支持: `true` ✅
     - 自定义名称后缀: `-SukiSU-Ultra-MT6895`

4. **下载编译产物**
   - 编译完成后（约30-60分钟）
   - 在 `Actions` 页面点击对应的workflow运行
   - 在 `Artifacts` 部分下载ZIP包
   - 或在 `Releases` 页面获取发布版本

## 📖 编译说明

### 环境要求

- **操作系统**: Ubuntu 20.04/22.04 或更高版本
- **磁盘空间**: 至少 100GB 可用空间
- **内存**: 16GB RAM（推荐 32GB）
- **处理器**: 多核处理器（推荐8核以上）
- **网络**: 稳定的网络连接（首次需下载约20GB数据）

### 本地编译步骤

详细的本地编译说明请查看 [README_PICKLE.md](README_PICKLE.md)

快速命令：

```bash
# 1. 安装依赖
sudo apt-get update
sudo apt-get install -y bc bison build-essential ccache curl flex \
  g++-multilib gcc-multilib git gnupg gperf imagemagick \
  lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool \
  libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev \
  libxml2 libxml2-utils lzop pngcrush rsync schedtool \
  squashfs-tools xsltproc zip zlib1g-dev python3 python-is-python3

# 2. 同步内核源码
mkdir kernel_workspace && cd kernel_workspace
repo init -u https://android.googlesource.com/kernel/manifest \
  -b common-android14-5.10 --depth=1
repo sync -c -j$(nproc --all)

# 3. 集成SukiSU Ultra
curl -LSs "https://raw.githubusercontent.com/ShirkNeko/SukiSU-Ultra/main/kernel/setup.sh" | bash -s susfs-dev

# 4. 编译内核
LTO=thin BUILD_CONFIG=common/build.config.gki.aarch64 build/build.sh -j$(nproc)
```

## 📲 刷入指南

### 前置条件

- ✅ 已解锁Bootloader
- ✅ 已安装自定义Recovery (TWRP/OrangeFox)
- ✅ **已备份原boot分区（重要！）**

### 方法1: Recovery刷入（推荐）

1. 下载编译好的 `SukiSU-Ultra-MT6895-PGKM10-*.zip`
2. 将ZIP文件传输到手机内部存储
3. 重启到Recovery模式：
   ```bash
   adb reboot recovery
   ```
4. 在Recovery中选择 "Install" / "安装"
5. 选择下载的ZIP文件
6. 滑动确认刷入
7. 刷入完成后，清除Dalvik/ART缓存
8. 重启系统

### 方法2: Fastboot刷入

```bash
# 提取kernel image
unzip SukiSU-Ultra-MT6895-PGKM10-*.zip
# 使用magiskboot等工具打包boot.img

# 刷入
adb reboot bootloader
fastboot flash boot boot.img
fastboot reboot
```

### 安装SukiSU Manager

1. 从 [SukiSU Ultra Releases](https://github.com/SukiSU-Ultra/SukiSU-Ultra/releases) 下载最新的Manager APK
2. 安装APK到手机
3. 打开应用，授予必要权限
4. 验证Root状态和KernelSU版本

## 🔍 验证安装

### 检查内核版本
```bash
adb shell uname -a
# 输出应包含: SukiSU-Ultra-MT6895
```

### 检查KernelSU状态
```bash
adb shell su -v
# 输出: KernelSU版本号
```

### 检查SUSFS状态
```bash
adb shell "dmesg | grep -i susfs"
# 应显示SUSFS初始化日志
```

## 🛠️ 配置选项

内核已启用以下配置：

| 配置项 | 状态 | 说明 |
|--------|------|------|
| CONFIG_KSU | ✅ | KernelSU核心 |
| CONFIG_KPM | ✅ | 内核模块管理 |
| CONFIG_KSU_SUSFS | ✅ | SUSFS文件系统隐藏 |
| CONFIG_KSU_SUSFS_HAS_MAGIC_MOUNT | ✅ | Magic Mount支持 |
| CONFIG_KSU_SUSFS_SUS_PATH | ✅ | 路径隐藏 |
| CONFIG_KSU_SUSFS_SUS_MOUNT | ✅ | 挂载隐藏 |
| CONFIG_KSU_SUSFS_SUS_KSTAT | ✅ | Kstat欺骗 |
| CONFIG_KSU_SUSFS_SPOOF_UNAME | ✅ | Uname欺骗 |
| CONFIG_KSU_MANUAL_HOOK | ✅ | 手动Hook模式 |
| CONFIG_CRYPTO_LZ4K | ✅ | LZ4K压缩 |
| CONFIG_CRYPTO_LZ4KD | ✅ | LZ4K解压 |

## ❓ 常见问题

<details>
<summary><b>Q: 为什么使用GKI 2.0而不是厂商内核？</b></summary>

GKI 2.0提供更好的兼容性、稳定性和可维护性。更容易获取安全更新，且支持更广泛的模块生态。
</details>

<details>
<summary><b>Q: 支持OTA升级吗？</b></summary>

不支持。系统OTA会覆盖内核分区，升级后需要重新刷入。建议在OTA前备份当前内核。
</details>

<details>
<summary><b>Q: 对性能和续航有影响吗？</b></summary>

影响极小。KernelSU设计为低开销运行，日常使用几乎感觉不到差异。
</details>

<details>
<summary><b>Q: 如何卸载？</b></summary>

刷回原厂boot.img即可完全卸载，或使用以下命令：
```bash
fastboot flash boot stock_boot.img
```
</details>

<details>
<summary><b>Q: 支持哪些模块？</b></summary>

支持所有KernelSU模块和Magisk Zygisk模块（需配合Zygisk-Next）。
</details>

<details>
<summary><b>Q: SafetyNet/Play Integrity检测？</b></summary>

需要配合Shamiko、LSPosed等隐藏模块使用，并正确配置SUSFS规则。
</details>

## 🐛 故障排除

### 无法开机（Bootloop）
1. 重启到Recovery
2. 恢复之前备份的boot.img
3. 或从官方包提取boot.img刷入

### Root权限未生效
1. 确认内核版本是否正确
2. 重新安装SukiSU Manager
3. 检查SELinux状态：`adb shell getenforce`

### 模块无法加载
1. 确认KPM已启用
2. 检查模块是否兼容当前内核版本
3. 查看日志：`dmesg | grep -i kpm`

更多问题请查看 [README_PICKLE.md](README_PICKLE.md) 的详细说明。

## 📚 相关资源

- 🌟 [SukiSU Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra) - 官方仓库
- 🔧 [KernelSU](https://github.com/tiann/KernelSU) - KernelSU项目
- 👻 [SUSFS](https://gitlab.com/simonpunk/susfs4ksu) - SUSFS for KernelSU
- 📱 [Android GKI](https://source.android.com/docs/core/architecture/kernel/gki) - GKI官方文档
- 🛠️ [AnyKernel3](https://github.com/osm0sis/AnyKernel3) - 刷机包工具

## 🤝 贡献

欢迎提交Issue和Pull Request！

如果您：
- 发现了Bug
- 有功能建议
- 想要改进文档
- 想要适配其他设备

请随时提交Issue或PR！

## ⚠️ 免责声明

- ⚠️ 本内核为第三方内核，刷入有风险
- ⚠️ 可能导致设备无法启动、数据丢失、保修失效
- ⚠️ **刷入前请务必备份重要数据**
- ⚠️ 使用本内核即表示您了解并接受所有风险
- ⚠️ 作者不对任何损失负责

## 📄 许可证

- Linux Kernel: GPL-2.0
- KernelSU: GPL-3.0
- SukiSU Ultra: GPL-3.0
- 本项目: GPL-3.0

## 💖 致谢

感谢以下项目和开发者：

- [ShirkNeko](https://github.com/ShirkNeko) - SukiSU Ultra
- [tiann](https://github.com/tiann) - KernelSU
- [simonpunk](https://gitlab.com/simonpunk) - SUSFS
- [osm0sis](https://github.com/osm0sis) - AnyKernel3
- Google - Android GKI
- 所有贡献者和测试者

---

<div align="center">

**如果这个项目对您有帮助，请给个⭐Star支持一下！**

Made with ❤️ for OnePlus Ace Users

</div>
