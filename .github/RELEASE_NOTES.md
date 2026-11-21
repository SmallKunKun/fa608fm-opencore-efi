## 下载

- `FA608F-OpenCore-EFI-v*.*.*.zip`：解压后第一层为 `EFI/`，可直接复制到 U 盘的 EFI 分区。

## 适用硬件

- 机型：ASUS 天选 6 Pro（FA608FM）
- CPU：AMD Ryzen 9 9955HX
- 显卡：NVIDIA GeForce RTX 5060 Laptop GPU
- 无线网卡：Intel AX210
- 系统：macOS Sequoia 15.8.2
- SMBIOS：`MacBookPro16,2`

## 包含

- 基于 [1401](https://github.com/nullmoth/1401) 生成并逐项验证的 OpenCore 配置
- OpenCore 引导文件、`config.plist`
- ACPI 补丁、驱动、OpenCanopy 主题资源

## 不包含

- SMBIOS 身份信息（序列号、MLB、UUID、ROM）
- macOS 安装镜像
- NullMoth 显卡驱动（需在 macOS 内安装）

## 使用前必读

1. 使用 GenSMBIOS 或 OCAT 为 `MacBookPro16,2` 生成身份信息，填写到 `EFI/OC/config.plist`。
2. 先在独立 U 盘上验证启动，确认关机、USB、电池、Wi-Fi、蓝牙和音频正常。
3. 同步内置盘前完整备份原有 EFI。
4. 详细说明见仓库 README 的「功能状态」和「快速开始」。
