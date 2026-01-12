# 明控创能 IOT-MKC3568 开发板系统移植项目

[![GitHub Issues](https://img.shields.io/github/issues/dutyc/IOT-MKC3568-OS-Porting)](https://github.com/dutyc/IOT-MKC3568-OS-Porting/issues)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

**语言**: 简体中文 | [English](README_en.md) *(计划中)*

## 📌 项目概述

本项目旨在为 **明控创能 IOT-MKC3568-Main-v1.1** 开发板（基于 Rockchip RK3568）移植主线 Ubuntu/Debian 系统。该开发板无官方资料，属于完全的硬件逆向与驱动适配工程。

**项目状态**: 进行中 - **Phase 2: 解决烧录通道**
**最终目标**: 提供可稳定启动和运行的 Linux 系统镜像及完整构建方法。

## 🧩 硬件规格

| 组件     | 规格                               | 状态                  |
| :------- | :--------------------------------- | :-------------------- |
| **SoC**  | Rockchip RK3568                    | 已验证                |
| **RAM**  | 2GB DDR4 (4x512MB Samsung)         | 已验证                |
| **存储** | 32GB eMMC                          | 已验证，已备份        |
| **网络** | 2x GbE (1口异常)                   | 已验证                |
| **USB**  | 1x Host, 1x OTG (故障)             | **OTG为当前主要障碍** |
| **其他** | Wi-Fi 5, HDMI, 2x SATA, 未标注排针 | 待驱动                |

<img src="./assets/IMG_4291.JPG" alt="开发板正面图" style="zoom:25%;" />

> 完整评估记录见: [docs/Analysis report.md](./docs/Analysis%20report.md)

## 🚧 当前进度与挑战

###  已完成
- 串口调试环境搭建
- ADB over TCP 开机自启动配置
- 完整 eMMC 镜像备份 (`dd if=/dev/block/mmcblk2`)
- `boot.img` 解包，确认 DTB 为 `rk3568-evb1-v10`

###  核心挑战（急需帮助！）
我们正面临三个主要瓶颈，这阻碍了项目的进一步推进。**非常需要社区的智慧！**

1.  **烧录通道缺失**: USB-OTG硬件故障，无法进入Maskrom/Loader模式。
2.  **U-Boot不稳定**: 板载U-Boot执行基础命令即崩溃，无法用于引导新系统。
3.  **硬件接口未知**: 板载多组未标注排针，功能不明。

**详细技术瓶颈、已尝试的方案与具体的求助方向，请移步至专门的问题讨论区**：
👉 **[GitHub Issues #1: 核心障碍：刷机通道、U-Boot 与未知接口](https://github.com/dutyc/IOT-MKC3568-OS-Porting/issues/1)**

##  仓库目录结构

```
IOT-MKC3568-OS-Porting/
│  LICENSE
│  README.md
├─assets
│      IMG_4291.JPG
├─links
│      dev_file-links.md
├─docs
│      Analysis report.md
│      RK_EVB1_RK3568_DDR4P216SD6_V10_20200908.pdf
│      start info.txt
│      u-boot-info.txt
├─firmware
│  └─extracted
│          bootimg_contents.md
│          dtb.0.dts
└─hardware
    └─photos
            IMG_4206.JPG
            IMG_4207.JPG
            ......
```

##  快速文件导航

-  **硬件照片**：前往 [`/hardware/photos/`](./hardware/photos/) 查看板卡各角度高清特写图。
-  **U-Boot分析**：查看 [`/docs/u-boot-info.txt`](./docs/u-boot-info.txt) 了解引导程序的初步分析结果。
- **设备树源文件**：查看 [`/firmware/extracted/dtb.0.dts`](./firmware/extracted/dtb.0.dts) 这是反编译出的核心硬件定义文件。
-  **原始镜像**：前往 [`/links/dev_file-links.md`](./links/dev_file-links.md) 获取完整备份文件及解包产物的下载链接（含密码）。

## 🛠 如何使用本仓库

1.  **对于观察者/协作者**：请关注 [Issues](https://github.com/dutyc/IOT-MKC3568-OS-Porting/issues) 以了解最新动态、技术瓶颈和求助问题。
2.  **对于开发者**：欢迎 Fork 本仓库并提交 PR。建议从标注有 `help-wanted` 标签的 Issue 开始入手。

## 如何参与贡献

我们急需以下方向的帮助，您的任何经验都可能成为突破的关键：
- **Rockchip RK3568 启动流程与刷机绕过方案**
- **U-Boot 移植、修复与调试经验**
- **硬件逆向与接口探测（如UART、JTAG）**
- **Linux 内核驱动适配**

参与方式非常灵活：
1.  在 **[Issues](https://github.com/dutyc/IOT-MKC3568-OS-Porting/issues)** 中分享您的想法、线索或解决方案。
2.  直接提交 **Pull Request** 来修复文档、提供分析脚本或补充资料。
3.  如果您有相关经验，欢迎在 Issue 下直接进行深入的技术讨论。



##  联系与外部链接

- **项目维护者**: [dutyc (LECREATE)](https://github.com/dutyc)
- **详细技术博客记录**: [《明控创能 IOT-MKC3568 开发板系统移植全记录》](https://blog.dutyc.top/2026/01/11/2026011101/)
- **相关社区提问**:
    - [寻求 IOT-MKC3568-Main-v1.1 (基于 EVB1-V10) 开发板的移植帮助 - Neardi 开源论坛](https://forum.neardi.com/d/196-xun-qiu-iot-mkc3568-main-v11-ji-yu-evb1-v10-kai-fa-ban-de-yi-zhi-bang-zhu)
