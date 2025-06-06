# 驱动修复

## 什么是驱动修复

由于不同虚拟化平台在硬件抽象层、设备驱动模型及启动机制等方面存在较大差异，为确保主机能够在目标平台顺利启动，仅依赖云平台的接口调度远远不够。还需结合目标平台的实际环境，对操作系统执行包括驱动注入、启动项调整、网络适配等在内的多项兼容性配置操作，从而保障主机在迁移或切换后实现稳定运行。
驱动修复的核心目标，是确保主机能够成功引导至登录界面，并具备正常的网络通信能力。

## 限制条件

在实际部署过程中，我们已尝试采用多种高效且具性价比的自动化修复方案，以提升对各类操作系统的适配能力。然而，受限于部分操作系统本身的结构或配置差异，仍存在以下已知情形可能导致驱动修复失败，进而影响主机在目标平台上的启动能力。若源端系统符合以下任一情况，无论使用代理模式或无代理模式，均存在主机启动失败的风险：

### Windows 系统

* **不支持系统盘使用动态卷**：目前不支持系统盘采用动态卷（包括所有类型的动态卷），但数据盘使用动态卷不受影响。如需正常使用本产品，建议在数据同步前将系统盘转换为普通分区（例如 NTFS）。

* **不支持 ReFS 文件系统**：ReFS（Resilient File System）相比 NTFS 存在兼容性差、缺乏事务日志支持等问题，尤其在崩溃一致性快照模式下，可能导致文件系统不完整，最终恢复失败。如需确保兼容性，建议系统盘使用 NTFS 文件系统。

* **旧版本 Windows 系统兼容性问题**：某些版本的 Windows 2003/2008（32 位或 64 位）在特定云平台上可能无法正常启动，或启动后网卡驱动无法自动加载。建议在使用前向云平台厂商确认是否完全支持该操作系统版本。

#### 参考链接

* [Dynamic Disks are Deprecated](https://learn.microsoft.com/en-us/windows/win32/fileio/basic-and-dynamic-disks#dynamic-disks)
* [Do not use ReFS for Security Reasons](https://techcommunity.microsoft.com/discussions/sql_server/what-to-use-refs-or-ntfs/3046795)
* [Features and functionality removed in Windows client](https://learn.microsoft.com/en-us/windows/whats-new/removed-features)

### Linux 系统

* **分区结构异常**：若主机中存在不规范的分区结构，例如分区之间存在空间重叠，或挂载点无法访问，可能导致驱动修复失败。请确保源端分区结构规范后再尝试修复。

* **系统识别文件缺失或异常**：如 `/etc/*-release`、`/etc/issue` 等系统识别文件被删除或更名，会导致无法正确识别操作系统版本，进而导致修复失败。请还原相关文件后重试。

* **关键分区空间不足**：若 `/boot`、`/var` 或根分区空间不足（小于 100 MB），可能导致驱动修复过程无法完成。建议释放足够空间后再进行修复。

* **磁盘处于只读状态**：驱动修复过程中需对磁盘进行写操作，若源端主机磁盘处于只读状态，将导致修复失败。请先解除在生产环境中修改只读限制后重试。