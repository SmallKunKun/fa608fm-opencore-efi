<div align="center">

# FA608FM OpenCore EFI

**An OpenCore EFI suite for FA608FM**

AMD Ryzen 9 9955HX · NVIDIA RTX 5060 Laptop · Intel AX210

[![OpenCore](https://img.shields.io/badge/OpenCore-bootloader-1F6FEB?style=flat-square)](https://github.com/acidanthera/OpenCorePkg)
[![macOS](https://img.shields.io/badge/macOS-Sequoia%2015.8.2-000000?logo=apple&logoColor=white&style=flat-square)](https://www.apple.com/macos)
[![AMD](https://img.shields.io/badge/CPU-Ryzen%209%209955HX-ED1C24?logo=amd&logoColor=white&style=flat-square)](https://www.amd.com)
[![NVIDIA](https://img.shields.io/badge/GPU-RTX%205060%20Laptop-76B900?logo=nvidia&logoColor=white&style=flat-square)](https://www.nvidia.com)
[![Release](https://img.shields.io/badge/release-latest-2ea44f?style=flat-square)](../../releases/latest)

[硬件情况](#-硬件情况) · [工作情况](#-工作情况) · [快速开始](#-快速开始) · [致谢](#-致谢) · [免责声明](#-免责声明)

</div>

---

## 💻 硬件情况

| 项目 | 本机配置 |
| :--- | :--- |
| 机型 | ASUS 天选 6 Pro，主板标识 `FA608FM` |
| CPU | AMD Ryzen 9 9955HX（Fire Range，Zen 5） |
| 显卡 | NVIDIA GeForce RTX 5060 Laptop GPU，NullMoth NVIDIA 2.7.2 |
| 无线网卡 | Intel AX210（替换原装 Realtek RTL8852CE） |
| 内置音频 | Realtek ALC256 |
| 系统 | macOS Sequoia 15.8.2（24H232） |
| SMBIOS 机型 | `MacBookPro16,2` |
| 引导 | OpenCore，`config.plist` 以 [1401](https://github.com/nullmoth/1401) 生成的 EFI 为基线修改 |

最终版以 1401 生成的 U 盘 EFI 为基础逐项修改，并在 U 盘上完成启动验证；验证通过后再同步到内置 NVMe 的 EFI 分区。仓库中的 EFI 使用与内置盘相同的日志设置。记录截至 2026 年 11 月 8 日。

适配结论仅适用于以上组合。固件版本、无线网卡型号或显示器接口变化都可能影响结果。

## ✅ 工作情况

### 已确认可用

| 功能 | 改动 | 验证结果 |
| :--- | :--- | :--- |
| 关机 | 根据本机 [RapidEFI](https://github.com/JeoJay127/RapidEFI-Tool) 硬件报告和 DSDT 定位关机路径。加入 `SSDT-XPME.aml`，将匹配本机 DSDT 的 `_PTS` 重命名为 `ZPTS`，由补丁处理 S5 关机。 | 机器能保持关机，并复测了 USB。 |
| USB | 保留基线的 `USBToolBox.kext`、`UTBDefault.kext`、`SSDT-USB-Reset.aml` 和 `SSDT-USBX.aml`。 | USB 可用；各端口和不同速率设备未逐一记录。 |
| 电池与充电器 | 加入 `SSDT-ACPWR.aml`，在本机 DSDT 上将 `_PSR` 重命名为 `XPSR`，补丁从 EC 的 `ACIN` 读取适配器状态。 | 拔电后报告电池供电，插回后报告交流供电。 |
| 低电量模式 | macOS 电池档设为「低电量」，适配器档设为「自动」（属于 macOS 本机设置）。 | 电池供电时 `lowpowermode=1`，插回后恢复为 `0`。 |
| Wi-Fi | 使用 Sequoia 适配版 `AirportItlwm.kext` 2.4.0，OpenCore 条目限定 Darwin `24.2.0–24.99.99`。 | 可从 macOS 自带 Wi-Fi 菜单连接并正常上网。 |
| 蓝牙 | `IntelBluetoothFirmware.kext` 2.5.0、`IntelBTPatcher.kext` 2.5.0、`BlueToolFixup.kext` 2.7.3。 | 可用。 |
| 扬声器与耳机 | `AppleALC.kext` 1.9.9，ALC256 使用布局 ID 13，清理了重复且无声的输出。 | 内置扬声器与 3.5 mm 耳机可用，插入耳机时模拟输出随插孔状态切换。 |
| 显示器音频 | 启用 `NMHDA.kext` 1.0.0，外接显示器的 3.5 mm 接口使用经 HDMI/DP 传输的数字音频。 | 显示器输出恢复。 |
| 内屏亮度 | 加入 NullMoth 的 `nvrmacpibl=1` 启动参数，使用 ACPI 背光路径。 | 亮度调节恢复。 |
| OpenCore 启动菜单 | 移除 GoldenGate 主题的 `Background.icns`，使用纯黑底。 | 重启确认黑色背景。 |
| Apple 账户弹窗 | 在 macOS 内重新验证 Apple 账户，EFI 的机器身份配置未改动。 | 弹窗已解决。 |
| 内置盘引导 | 最终版同步到本机内置 EFI 分区；`Misc → Debug → Target` 从 `67` 改为 `3`，关闭 `0x40` 文件日志位。 | 已核对落盘文件、配置和旧版备份；内置盘新版的启动和日志状态尚待复测。 |

### 未验证或存在限制

- 内置盘新版 EFI 的启动、重启后日志状态暂无复测记录。
- USB 仅验证「可用」，各端口与不同速率设备未逐一记录。
- AMD 功耗调节尚未验证：ZenMaster 测试时 IOPCIBridge 能连接 9955HX 的 SMU 并读取固件版本，但该后端不提供 PM Table 遥测，功耗限制是否实际生效无法确认。最终 EFI 未启用该测试参数。
- NullMoth 显卡驱动安装在 macOS 内，**仅复制 EFI 不会安装这套驱动**，换机或重装系统后需要单独安装。

## 🚀 快速开始

### 前置条件

| 依赖 | 要求 |
| :--- | :--- |
| 测试介质 | 一个 FAT32 格式的 U 盘，EFI 分区中包含 `EFI` 目录 |
| SMBIOS 信息 | 为 `MacBookPro16,2` 生成 `SystemSerialNumber`、`MLB`、`SystemUUID`、`ROM` |
| 备份 | 修改内置盘前，完整备份原有 EFI 目录 |
| 显卡驱动 | 如需 NVIDIA 加速，需在 macOS 内安装 NullMoth 驱动 |

发布包由 GitHub Actions 自动构建，可从仓库的 **Releases** 页面下载最新版本。

### 1. 获取 EFI

从 **Releases** 下载最新发布包，或克隆本仓库获取完整内容。

### 2. 生成并填写 SMBIOS

发布副本的 `PlatformInfo → Generic` 中以下字段为空：

- `SystemSerialNumber`
- `MLB`
- `SystemUUID`
- `ROM`

使用 GenSMBIOS、OCAT 等工具为 `MacBookPro16,2` 生成一套信息并填入 `EFI/OC/config.plist`。**不要直接使用他人分享的序列号。**

### 3. 写入测试 U 盘

将整个 `EFI` 目录复制到 U 盘的 EFI 分区，保持如下层级：

```text
EFI/
├── BOOT/
│   └── BOOTx64.efi
└── OC/
    ├── config.plist
    └── …
```

### 4. 从 U 盘启动验证

1. 开机按固件启动菜单快捷键，选择 U 盘的 UEFI 启动项。
2. 进入 OpenCore 菜单，选择 macOS 启动。
3. 逐项检查：关机是否保持关闭、拔插充电器、USB、Wi-Fi、蓝牙、扬声器、两个耳机插孔、内屏亮度。
4. 如启动失败，回到 macOS 调整配置后重新写入 U 盘，不影响内置盘。

### 5. 同步到内置盘

1. 完整备份内置盘原有的 `EFI` 目录。本机同步前的备份位于工作区的 `Backups/internal-efi-before-1401-sync-20261108/EFI`。
2. 复制已验证的 `EFI` 目录到内置 EFI 分区。
3. 确认 `Misc → Debug → Target` 为 `3`，避免持续写入文件日志。
4. 重启并再次验证关键功能。

### 回退

如内置盘启动失败，使用已验证的 U 盘进入 macOS 或恢复环境，再把备份的 `EFI` 目录写回内置 EFI 分区。

## 🙏 致谢

- **[OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)**、**[Lilu](https://github.com/acidanthera/Lilu)**、**[VirtualSMC](https://github.com/acidanthera/VirtualSMC)**、**[WhateverGreen](https://github.com/acidanthera/WhateverGreen)**、**[AppleALC](https://github.com/acidanthera/AppleALC)**、**[NVMeFix](https://github.com/acidanthera/NVMeFix)**、**[RestrictEvents](https://github.com/acidanthera/RestrictEvents)**：OpenCore 引导与核心驱动。
- **[OpenIntelWireless](https://github.com/OpenIntelWireless/itlwm)**：`itlwm` / `AirportItlwm` 与 Intel 蓝牙驱动。
- **[USBToolBox](https://github.com/USBToolBox/tool)**：USB 端口定制。
- **[VoodooI2C](https://github.com/VoodooI2C/VoodooI2C)**、**[VoodooPS2](https://github.com/acidanthera/VoodooPS2)**：输入设备驱动。
- **[NullMoth 1401](https://github.com/nullmoth/1401)**：生成初始 OpenCore EFI、制作 macOS 安装盘。
- **[NullMoth NVIDIA 驱动](https://github.com/nullmoth/nvidia-macos-driver)**：RTX 5060 Laptop 的 macOS 驱动与 `NMHDA`。
- **[RapidEFI](https://github.com/JeoJay127/RapidEFI-Tool)**：硬件信息采集与硬件报告。
- **[OpenCanopy](https://github.com/acidanthera/OpenCorePkg)** 与 Acidanthera 图标资源：启动菜单界面。

## 📄 免责声明

本 EFI 仅用于记录个人硬件适配过程与学习交流。使用他人 EFI 存在风险，请先备份数据并在独立 U 盘上验证。因使用本仓库内容造成的设备损坏、数据丢失或账号问题，作者不承担责任。

[1401](https://github.com/nullmoth/1401) 采用 PolyForm Noncommercial License 1.0.0，[RapidEFI](https://github.com/JeoJay127/RapidEFI-Tool) 采用 AGPL-3.0。本仓库只分发配置文件和开源驱动，不包含这两个工具本身；上游驱动与工具的许可归各自作者所有，使用或再分发前请自行核对对应项目的许可证与声明。
