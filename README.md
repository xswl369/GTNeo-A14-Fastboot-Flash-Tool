# GTNeo A14 Fastboot 线刷工具（realme GT Neo / RMX3031）

> ## ⚠️ 公开声明
> **本项目仅用于技术交流、学习研究与已授权设备的安全测试，不得用于任何商业用途或非法目的。**
> 刷机有风险：可能导致设备变砖、数据全毁、失去保修、触发安全校验失败。**一切后果由使用者自行承担。**
> 请确保你对自己的设备拥有完全所有权，并在动手前完成数据备份。
> 本仓库内容按"现状（AS IS）"提供，不附带任何明示或暗示的担保。

---

## 一、内核版本信息（Kernel Version）

以下信息从包内 `IMG/boot.img` 的 Android boot image 头部与解压后的 kernel 镜像中实际读取，不是估算：

| 项目 | 值 |
|---|---|
| **内核版本** | **Linux 4.14.186+** |
| **架构** | aarch64（ARM64，`ARMd` 魔数，64 位内核） |
| 编译器 | Android Clang **11.0.1**（6443078 based on r383902）+ LLD 11.0.1 |
| 构建标记 | `#1 SMP PREEMPT Thu Apr 1 17:04:47 CST 2021` |
| 内核编译时间 | 2021-04-01 17:04:47 CST |
| boot.img 格式 | Android boot image **header v2**，page size 2048 |
| kernel 大小 | 12,726,882 字节（gzip 压缩，解压后 34,734,080 字节） |
| ramdisk 大小 | 900,320 字节（gzip 压缩，解压后 2,045,696 字节） |
| kernel 加载地址 | `0x40080000` |
| ramdisk 加载地址 | `0x51100000` |
| 内核 cmdline | `bootopt=64S3,32N2,64N2 buildvariant=user` |
| os_version 字段 | `0x16000153` → 11.0.0 / patch 339 |
| recovery.img | 同源 boot image v2，kernel 段与 boot.img 完全一致（同一内核 4.14.186+），ramdisk 32,585,298 字节（recovery ramdisk） |

完整原始字符串：

```
Linux version 4.14.186+ (nobody@android-build) (Android (6443078 based on r383902) clang version 11.0.1
(https://android.googlesource.com/toolchain/llvm-project b397f81060ce6d701042b782172ed13bee898b79),
LLD 11.0.1 (/buildbot/tmp/tmp6_m7QH b397f81060ce6d701042b782172ed13bee898b79))
#1 SMP PREEMPT Thu Apr 1 17:04:47 CST 2021
```

---

## 二、设备与固件平台

| 项目 | 值 |
|---|---|
| 机型 | realme GT Neo（RMX3031 / RMX3031CN） |
| SoC | MediaTek **MT6893**（天玑 1200） |
| 工程号 | 20615 |
| 区域 | China |
| Android 版本 | **11**（`ro.build.version.release=11`，SDK 30） |
| 构建号 | `RP1A.200720.011` / `RMX3031_11_A.01`（OTA：`RMX3031_11.A.01_0010_202104011615`） |
| 安全补丁 | 2021-03-05 |
| Build date | 2021-04-01 |
| 指纹 | `realme/RMX3031/RMX3031CN:11/RP1A.200720.011/1616033393333:user/release-keys` |
| 刷机方式 | Fastboot / MTK 深刷（DA `MT6893`，`Projectconfig.xml`） |

> 注：包名中的 "A14" 是该第三方工具包自带的命名，**并不代表 Android 14**。经实际读取 `super.img` 内 build.prop 与 boot 镜像，其内容为 **Android 11 + 内核 4.14.186**。

---

## 三、目录结构

```
.
├─ RealmeGT Neo线刷工具箱.exe        第三方刷机主程序（v2.0.0.0，R+B玩机乐园）
├─ 1.先点此处安装ADB驱动/
│  ├─ 一键安装安卓驱动.exe            驱动一键安装
│  └─ 打不开安装这个（net4.7框架）.exe  .NET Framework 4.7 运行库
├─ ADB/                             Google platform-tools 33.0.2 + 辅助工具
│  ├─ adb.exe  fastboot.exe
│  ├─ mke2fs.exe  make_f2fs.exe  sqlite3.exe  zstd.exe  7z.exe  aria2c.exe
│  └─ NOTICE.txt（Apache-2.0 等许可证原文）
└─ IMG/                             分区镜像与刷机配置
   ├─ Projectconfig.xml  Efuse.ini  20615-efuse_oppo6893.xml
   ├─ img_list.txt（分区 → 镜像映射）  super_map.csv  hash256_id.txt
   ├─ boot.img  dtbo.img  lk.img  tee.img  md1img.img  super.img …
   └─ auth_sv5.auth  *.sig  等签名文件
```

### 分区镜像一览

| 镜像 | 大小 | 说明 |
|---|---|---|
| `super.img` | 9728 MB | 动态分区（system/vendor/odm/product），**未随仓库发布，见第五节** |
| `recovery.img` | 128 MB | Recovery（boot image v2），**见第五节** |
| `md1img.img` | 61.98 MB | 基带固件 |
| `boot.img` | 32 MB | 内核 4.14.186+ |
| `cam_vpu3/2/1.img` | 14.35 / 13.94 / 4.71 MB | 相机 VPU |
| `oppo20615.bin` | 8 MB | 平台配置 |
| `opporeserve2.img` | 5.05 MB | OPPO 保留分区（sparse） |
| `audio_dsp.img` | 3.62 MB | 音频 DSP |
| `userdata.img` | 2.28 MB | 用户数据（sparse，空） |
| `logo.bin` | 2 MB | 开机 Logo |
| `gz.img` | 1.90 MB | GZ / VM |
| `scp.img` | 1.82 MB | Tinysys SCP |
| `lk.img` | 1.25 MB | Little Kernel（bootloader） |
| `cdt_engineering.img` | 1 MB | CDT |
| `tee.img` | 0.95 MB | TEE / ATF |
| `sspm.img` | 0.73 MB | Tinysys SSPM |
| `preloader_oppo6893.bin` | 0.43 MB | Preloader |
| `mcupm.img` | 0.34 MB | Tinysys MCUPM |
| `dtbo.img` | 0.28 MB | DTBO |
| `spmfw.img`、`pi_img.img`、`dpm.img`、`cache.img` | < 0.15 MB | 其余小分区 |
| `vbmeta*.img`、`*.sig`、`auth_sv5.auth` | < 0.01 MB | AVB / 签名校验 |

---

## 四、使用方式（通用流程）

1. **装驱动**：先运行 `1.先点此处安装ADB驱动\一键安装安卓驱动.exe`；若提示缺少运行库，再装 `打不开安装这个（net4.7框架）.exe`。
2. **进 Fastboot**：设备关机后按住 **音量下 + 电源键** 进入 fastboot 模式，用数据线连接电脑；`ADB\fastboot.exe devices` 应能看到设备。
3. **刷机**：右键以**管理员身份**运行 `RealmeGT Neo线刷工具箱.exe`，按界面提示选择刷机模式与镜像。
4. **完成后**首次开机较慢（10–20 分钟属正常），不要中途断电。

> ⚠️ 若设备 bootloader 未解锁，fastboot 刷写会被拒绝；解锁会清空数据并可能失去保修。
> ⚠️ 刷入与本机不匹配的镜像（尤其是 preloader / lk / vbmeta）可能直接变砖，请先核对工程号 `20615` 与机型 `RMX3031`。

---

## 五、未包含的大文件

GitHub 单文件硬上限 100 MB，Release 附件上限 2 GB，以下两个文件因此**不在仓库内**：

| 文件 | 大小 | SHA256 | 获取方式 |
|---|---|---|---|
| `IMG/recovery.img` | 128 MB (134,217,728 B) | `777194211e8d7128230938e929cb1217d3863c09999efa1aa66060f71ef57a7f` | ✅ [Release v1.0 附件](../../releases/tag/v1.0) |
| `IMG/super.img` | 9,728 MB (9.5 GB) | `4c849cb5ce538db2bbe61f161022d2a545488ff41ea9d904e894d7b095d41cc7` | ❌ 无法提供 |

- **`recovery.img`**：Android boot image v2，kernel 段与 `boot.img` 完全一致（4.14.186+），ramdisk 32,585,298 字节（gzip）。因超过 GitHub 单文件 100 MB 限制，改由 Release 附件分发。
- **`super.img`**：动态分区镜像（system / vendor / odm / product），超过 Release 附件 2 GB 上限，GitHub 无法承载；请从原厂固件包获取，或用 `lpunpack` 自行处理。
- 校验方式：`certutil -hashfile IMG\recovery.img SHA256`，与上表或 `CHECKSUMS.sha256` 比对。

---

## 六、文件校验

仓库内所有文件（含上面两个大文件）的 SHA256 记录在 [`CHECKSUMS.sha256`](CHECKSUMS.sha256)：

```powershell
# Windows
Get-FileHash -Algorithm SHA256 IMG\boot.img

# Linux / macOS
sha256sum -c CHECKSUMS.sha256
```

---

## 七、第三方组件与版权

本仓库仅做镜像/工具集的整理与归档，**所有二进制与固件的版权归各自权利人**：

| 组件 | 版权/许可 |
|---|---|
| `adb.exe` / `fastboot.exe` / platform-tools | © Google，Apache License 2.0（见 `ADB/NOTICE.txt`，版本 33.0.2） |
| `zstd.exe` / `libzstd` | © Facebook / Meta，BSD-3-Clause |
| `7z.exe` / `7z.dll` | © Igor Pavlov，LGPL-2.1 |
| `aria2c.exe` | © Tatsuhiro Tsujikawa，GPL-2.0 |
| `sqlite3.exe` | Public Domain |
| `mke2fs.exe` / `make_f2fs.exe` | e2fsprogs（GPL-2.0）/ Android（Apache-2.0） |
| 驱动安装包 | 归原作者所有 |
| `RealmeGT Neo线刷工具箱.exe` | © **R+B玩机乐园**（v2.0.0.0，官方群 613471535） |
| 全部 `IMG/*` 固件与签名 | © realme / OPPO / MediaTek |

如权利人认为本仓库内容不当，请提 Issue 或联系仓库所有者，将立即删除。

---

## 八、免责声明（Disclaimer）

1. 本项目**仅供技术交流与学习研究**，作者不鼓励也不支持任何侵犯他人设备所有权、破坏计算机信息系统安全的行为。
2. 使用者须确认：设备为**本人所有**，或已取得设备所有者的**明确书面授权**。
3. 刷机属高风险操作，可能导致**设备损坏、数据丢失、保修失效、指纹/支付等安全功能异常**，作者**不承担任何直接或间接责任**。
4. 请遵守当地法律法规及设备厂商的用户协议；因使用本仓库内容产生的法律责任由使用者自负。
5. 本仓库不含任何破解、绕过付费、绕过账号锁或规避 DRM 的功能，亦不得用于此类目的。

**下载或使用本仓库任何内容，即表示你已阅读并同意以上全部条款。**
