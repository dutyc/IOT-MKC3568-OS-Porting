# 明控创能 IOT-MKC3568 开发板系统移植全记录

> **作者：** LECREATE  
> **日期：** 2026年1月11日  
> **状态：** 进行中

---

## 前言

本文档记录了在“明控创能 IOT-MKC3568”开发板上，从出厂 Android 11 系统向 Ubuntu/Debian 移植的完整过程。该开发板无官方资料支持，且存在硬件接口不稳定等问题，本项目旨在通过逆向分析与系统调试，突破其软件限制，实现系统自定义与功能扩展。

---

## 一、硬件规格与初步评估

开发板型号为 **明控创能 IOT-MKC3568-Main-v1.1**，核心为 Rockchip RK3568 SoC。经拆解与测试，硬件配置如下：

| 类别         | 规格                              | 备注                         |
| :----------- | :-------------------------------- | :--------------------------- |
| **SoC**      | Rockchip RK3568                   | 主处理器                     |
| **RAM**      | 2GB DDR4                          | 由4颗三星 512MB 颗粒组成     |
| **无线**     | Wi-Fi 5                           | 可搜索到网络，但无法成功连接 |
| **eMMC**     | 32GB                              | 存储介质                     |
| **网络**     | 2x千兆以太网口                    | 一个端口功能异常             |
| **USB**      | 1x USB-A (Host) <br> 1x USB-A OTG | OTG接口存在严重联机问题      |
| **视频输出** | HDMI, VGA（焊盘预留）             | VGA未焊接物理接口            |
| **其他**     | 2x SATA, SIM卡槽, 未知针脚        | 功能待验证                   |

**初步评估：**  
硬件基础扎实，但存在明显的软件与驱动缺陷，尤其是 USB-OTG 功能异常。出厂预装的 Android 11 `userdebug` 版本为系统调试与逆向工作提供了关键入口。

---

## 二、调试环境搭建与 ADB 自动化

### 2.1 基础调试手段

- **串口调试：** 最稳定的调试途径，用于获取底层信息与执行命令。
- **无线 ADB：** 因 Wi-Fi 无法连接，系统中无法启用该功能。
- **USB-OTG：** 当前不可用，是后续刷机的主要障碍。

### 2.2 手动配置 ADB 网络调试

在设备 ADB shell 中执行：

```bash
su
setprop service.adb.tcp.port 5555
stop adbd
start adbd
```

在宿主机中连接：

```powershell
adb connect <开发板IP>:5555
```

**示例输出：**

```powershell
PS C:\Users\MyPC> adb connect 192.168.1.5:5555
connected to 192.168.1.5:5555
```

### 2.3 实现 ADB 网络调试开机自启动

为避免每次重启后手动配置，通过创建系统初始化脚本实现自动化：

```bash
# 1. 挂载 system 分区为可写
su
mount -o rw,remount /system

# 2. 创建开机自启动脚本
cat > /system/etc/init/99-adb-tcp.rc <<'EOF'
# Automatically enable ADB over TCP on boot
on property:sys.boot_completed=1
    setprop service.adb.tcp.port 5555
    start adbd
EOF

# 3. 设置权限
chown root:root /system/etc/init/99-adb-tcp.rc
chmod 644 /system/etc/init/99-adb-tcp.rc

# 4. 重启生效
sync
reboot
```

> **注意：**  
> - 脚本须位于 `/system/etc/init/`，并以 `.rc` 结尾。  
> - 修改系统分区前建议备份重要数据。  
> - 重启后设备将自动监听 5555 端口。

### 2.4 尝试进入刷机模式

- **Loader 模式：** 尝试按住 `REST` 键上电，未能成功进入。但在 ADB root shell 下执行 `reboot loader` 命令时，屏幕会显示“卡在芯片Logo”的画面，表明系统可能进入了某种底层引导状态，但未完全成功。
- **Maskrom 模式：** 无法找到 CLK 短接点，导致无法进入。

---

## 三、固件提取与逆向分析

### 3.1 全盘镜像备份

使用 `dd` 命令完整备份 eMMC 数据：

```bash
cd /storage/CE3297AB329796D5/backup/
dd if=/dev/block/mmcblk2 of=rk3568_android_full.img bs=4M conv=sync,noerror
```

> **说明：** `mmcblk2` 为 eMMC 设备节点，备份文件约 29GB，耗时较长，但能保证数据完整性。

### 3.2 `boot.img` 解包分析

使用 `Android_boot_image_editor` 解包 `boot.img`，提取以下关键组件：

#### 解包结果目录结构：

```powershell
目录: D:\RK3568\boot_editor_v15_r1\build\unzip_boot

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2026/1/10     20:55                root
-a----         2026/1/10     20:55           1780 boot.json
-a----         2026/1/10     20:55         132390 dtb
-a----         2026/1/10     20:55         132390 dtb.0
-a----         2026/1/10     20:55         161669 dtb.0.dts
-a----         2026/1/10     20:55         161669 dtb.0.dts.txt
-a----         2026/1/10     20:55         181184 dtb.0.dts.yaml
-a----         2026/1/10     20:55       31596560 kernel
-a----         2026/1/10     20:55        1835264 ramdisk.img
-a----         2026/1/10     20:55         822486 ramdisk.img.gz
-a----         2026/1/10     20:55            533 ramdisk.img_filelist.txt
-a----         2026/1/10     20:55              8 role
-a----         2026/1/10     20:55         221184 second
```

- **内核（Kernel）：** 提取成功，可用于后续编译或替换。
- **设备树（DTB）：** 提取成功，确认设备型号为 `RK3568-EVB1-V10`。
- **Ramdisk：** 提取成功，包含系统启动所需的最小根文件系统。

> **价值点：** 设备树中定义的硬件信息（如内存、USB控制器等）是移植新系统的关键依据。

---

## 四、U-Boot 探索与烧录瓶颈

### 4.1 U-Boot 进入方式

在串口启动过程中，于倒计时期间按下 `Ctrl + C` 可进入 U-Boot 命令行。

### 4.2 U-Boot 稳定性测试

尝试在 U-Boot 中使用 `tftp` 命令下载文件，结果导致系统崩溃或重启。

> **结论：** 当前 U-Boot 固件存在严重稳定性问题，无法用于常规固件烧录。

---

## 五、Ubuntu/Debian 移植路线图

基于当前进展，制定如下四阶段计划：

###  Phase 1：准备工作（已完成）
- [x] 硬件规格确认
- [x] ADB 调试环境搭建
- [x] 全盘镜像备份
- [x] `boot.img` 逆向分析

### 🛠 Phase 2：解决烧录通道（当前重点）
**目标：** 绕过不稳定的 U-Boot，建立可靠的固件写入机制。  
**可行方案：** 通过 ADB shell 中的 `dd` 命令直接刷写镜像。

---

## 六、总结与展望

尽管面临无官方资料、USB 接口故障、U-Boot 不稳定等挑战，通过系统的调试与逆向分析，我们已掌握开发板的核心信息，并建立了稳定的调试环境。

当前最大瓶颈在于**如何可靠写入新固件**。下一步将重点探索多种烧录方案，力求突破此限制。

本项目不仅是一次技术实践，也为类似“无资料”设备的系统移植提供了可行思路。最终目标是在该开发板上成功运行 Ubuntu/Debian 系统。

---

## 附：临时用途说明

在系统移植完成前，暂定将该开发板用作电视盒子，以充分利用其现有 Android 系统与多媒体功能。

---
**文档维护说明：**  
本记录将持续更新，直至完成 Ubuntu/Debian 系统移植并稳定运行。