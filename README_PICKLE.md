# SukiSU Ultra GKI 2.0 编译指南 - 一加Ace (PGKM10)

## 设备信息
- **设备名称**: 一加Ace / OnePlus Ace
- **设备代号**: pickle
- **型号**: PGKM10
- **处理器**: MediaTek Dimensity 8100-MAX (MT6895)
- **内核版本**: 5.10.x
- **Android版本**: Android 14

## 编译说明

### 使用GitHub Actions自动编译（推荐）

1. **Fork本仓库到您的GitHub账号**

2. **启用GitHub Actions**
   - 进入仓库的 Settings → Actions → General
   - 允许所有Actions运行

3. **开始编译**
   - 点击 Actions 标签
   - 选择 "Build SukiSU Ultra GKI 2.0 for OnePlus Ace (MT6895)"
   - 点击 "Run workflow"
   - 配置编译参数：
     * 内核版本: 5.10 (默认)
     * Android版本: android14 (默认)
     * KPM支持: true (启用模块支持)
     * 内核名称后缀: 自定义

4. **下载编译产物**
   - 编译完成后在Artifacts中下载
   - 或在Releases页面获取

### 本地编译

#### 环境要求
- Ubuntu 20.04/22.04 或更高版本
- 至少100GB可用磁盘空间
- 16GB RAM (推荐32GB)
- 多核处理器

#### 安装依赖
```bash
sudo apt-get update
sudo apt-get install -y \
  bc bison build-essential ccache curl flex \
  g++-multilib gcc-multilib git gnupg gperf \
  imagemagick lib32ncurses5-dev lib32readline-dev \
  lib32z1-dev liblz4-tool libncurses5 libncurses5-dev \
  libsdl1.2-dev libssl-dev libxml2 libxml2-utils \
  lzop pngcrush rsync schedtool squashfs-tools xsltproc \
  zip zlib1g-dev python3 python-is-python3
```

#### 安装repo工具
```bash
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
export PATH=~/bin:$PATH
```

#### 同步内核源码
```bash
mkdir kernel_workspace && cd kernel_workspace
repo init -u https://android.googlesource.com/kernel/manifest \
  -b common-android14-5.10 --depth=1
repo sync -c -j$(nproc --all)
```

#### 集成SukiSU Ultra
```bash
curl -LSs "https://raw.githubusercontent.com/ShirkNeko/SukiSU-Ultra/main/kernel/setup.sh" | bash -s susfs-dev
```

#### 应用SUSFS补丁
```bash
git clone https://gitlab.com/simonpunk/susfs4ksu.git -b gki-android14-5.10 --depth=1
cp susfs4ksu/kernel_patches/50_add_susfs_in_gki-android14-5.10.patch ./common/
cp -r susfs4ksu/kernel_patches/fs/* ./common/fs/
cp -r susfs4ksu/kernel_patches/include/linux/* ./common/include/linux/
cd common
patch -p1 < 50_add_susfs_in_gki-android14-5.10.patch
```

#### 配置内核
```bash
cd common
cat >> arch/arm64/configs/gki_defconfig << EOF
CONFIG_KSU=y
CONFIG_KPM=y
CONFIG_KSU_SUSFS=y
CONFIG_KSU_SUSFS_HAS_MAGIC_MOUNT=y
CONFIG_KSU_SUSFS_SUS_PATH=y
CONFIG_KSU_SUSFS_SUS_MOUNT=y
CONFIG_KSU_SUSFS_SUS_KSTAT=y
CONFIG_KSU_SUSFS_SPOOF_UNAME=y
CONFIG_KSU_MANUAL_HOOK=y
CONFIG_CRYPTO_LZ4K=y
CONFIG_CRYPTO_LZ4KD=y
EOF
```

#### 编译内核
```bash
cd ..
LTO=thin BUILD_CONFIG=common/build.config.gki.aarch64 build/build.sh -j$(nproc)
```

#### 制作刷机包
```bash
git clone https://github.com/osm0sis/AnyKernel3.git --depth=1
cp out/*/dist/Image AnyKernel3/
cd AnyKernel3
# 编辑anykernel.sh配置文件
zip -r9 ../SukiSU-Ultra-MT6895.zip * -x .git
```

## 功能特性

### ✅ 已集成功能
- **SukiSU Ultra**: 基于KernelSU的增强Root方案
- **SUSFS**: 完整的文件系统隐藏支持
  - Magic Mount 支持
  - 路径隐藏 (SUS_PATH)
  - 挂载点隐藏 (SUS_MOUNT)
  - Kstat欺骗
  - Uname欺骗
  - 命令行/BootConfig欺骗
- **KPM**: 内核模块管理器支持
- **LZ4K/LZ4KD**: 高性能压缩算法
- **GKI 2.0**: 通用内核镜像架构

### 🔧 配置选项说明

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| CONFIG_KSU | 启用KernelSU | y |
| CONFIG_KPM | 启用KPM模块管理 | y |
| CONFIG_KSU_SUSFS | 启用SUSFS核心 | y |
| CONFIG_KSU_SUSFS_HAS_MAGIC_MOUNT | Magic Mount功能 | y |
| CONFIG_KSU_SUSFS_SUS_PATH | 路径隐藏 | y |
| CONFIG_KSU_SUSFS_SUS_MOUNT | 挂载隐藏 | y |
| CONFIG_KSU_SUSFS_SUS_KSTAT | Kstat欺骗 | y |
| CONFIG_KSU_SUSFS_SPOOF_UNAME | 内核版本欺骗 | y |
| CONFIG_KSU_MANUAL_HOOK | 手动Hook模式 | y |

## 刷入指南

### 前置条件
- ✅ 已解锁Bootloader
- ✅ 已安装TWRP/OrangeFox Recovery
- ✅ 已备份原boot分区

### 刷入步骤

#### 方法1: Recovery刷入 (推荐)
1. 下载 `SukiSU-Ultra-MT6895-PGKM10-*.zip`
2. 将ZIP文件传输到手机内部存储
3. 重启到Recovery模式
4. 选择 "Install" 或 "安装"
5. 选择下载的ZIP文件
6. 滑动确认刷入
7. 刷入完成后，选择 "Wipe Dalvik/ART Cache"
8. 重启系统

#### 方法2: Fastboot刷入
```bash
# 解压AnyKernel3包获取boot.img
unzip SukiSU-Ultra-MT6895-PGKM10-*.zip
# 如需手动打包boot.img，请使用magiskboot等工具

# 刷入boot分区
adb reboot bootloader
fastboot flash boot boot.img
fastboot reboot
```

### 安装SukiSU Manager
1. 下载最新版 SukiSU Ultra Manager APK
   - 官方仓库: https://github.com/SukiSU-Ultra/SukiSU-Ultra
2. 安装APK
3. 打开应用，授予必要权限
4. 验证Root状态

## 验证安装

### 检查内核版本
```bash
adb shell uname -a
# 应显示包含 "SukiSU-Ultra-MT6895" 的版本信息
```

### 检查KernelSU状态
```bash
adb shell su -v
# 应显示KernelSU版本号
```

### 检查SUSFS状态
```bash
adb shell "dmesg | grep -i susfs"
# 应显示SUSFS初始化信息
```

## 故障排除

### 无法开机 (Bootloop)
**解决方法:**
1. 重启到Recovery
2. 恢复之前备份的boot.img
3. 或刷入官方boot.img

### Root未生效
**检查项:**
- 确认内核版本正确
- 重新安装SukiSU Manager
- 检查SELinux状态: `adb shell getenforce`

### 模块加载失败
**解决方法:**
1. 确认KPM已启用
2. 检查模块兼容性
3. 查看内核日志: `dmesg | grep -i kpm`

### SafetyNet检测失败
**原因:** SafetyNet需要配合其他模块使用
**解决方法:**
1. 安装Shamiko/LSPosed等隐藏模块
2. 配置SUSFS隐藏规则
3. 使用Zygisk/Riru框架

## 常见问题

**Q: 为什么选择GKI 2.0而不是OFRP/厂商内核?**
A: GKI 2.0提供更好的兼容性和稳定性，更容易获取安全更新。

**Q: 支持OTA更新吗?**
A: 不支持。OTA会覆盖内核，需要重新刷入。

**Q: 对性能有影响吗?**
A: 影响极小，KernelSU设计为低开销运行。

**Q: 可以卸载吗?**
A: 可以，刷回原厂boot.img即可完全卸载。

**Q: 支持哪些模块?**
A: 支持所有KernelSU/Magisk Zygisk模块（需要Zygisk-Next）。

## 技术支持

### 相关链接
- **SukiSU Ultra**: https://github.com/SukiSU-Ultra/SukiSU-Ultra
- **KernelSU**: https://github.com/tiann/KernelSU
- **SUSFS**: https://gitlab.com/simonpunk/susfs4ksu
- **Android GKI**: https://source.android.com/docs/core/architecture/kernel/gki

### 报告问题
如遇到问题，请提供以下信息:
- 设备型号和内核版本
- 详细复现步骤
- logcat和dmesg日志
- 已安装的模块列表

### 捐赠支持
如果本项目对您有帮助，欢迎star⭐本仓库！

## 免责声明

本内核为第三方内核，刷入有风险，可能导致：
- 设备无法启动
- 数据丢失
- 保修失效
- 其他未知问题

**刷入前请务必备份重要数据！**
**使用本内核即表示您了解并接受所有风险。**

## 许可证

本项目遵循以下许可证:
- Linux Kernel: GPL-2.0
- KernelSU: GPL-3.0
- SukiSU Ultra: GPL-3.0

---

**编译时间**: 自动生成  
**维护者**: GitHub Actions  
**最后更新**: 2025-10-24
