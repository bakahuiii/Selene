# SELENE

[简体中文](README.md) | [English](README.en.md)

开发、扩展 Android/Windows 采集器、修改运动阈值或事件协议前，请先阅读
[开发者指南](docs/DEVELOPER_GUIDE.zh-CN.md)；英文版为
[Developer Guide](docs/DEVELOPER_GUIDE.md)。

SELENE 是 HYPERION 的独立时间线采集端，包含 Android 和 Windows 两个平台。
它只在本地采集经过授权的非文本背景，并写成不可变快照供 HYPERION 直接导入。
SELENE 不读取聊天数据库，也不上传或解释聊天内容。

## Windows 命令行安装

Windows 桌面版目前是唯一支持命令行构建的桌面平台。在 PowerShell 中执行；首次
安装 `winget` 包后请重新打开 PowerShell，再继续执行后续命令：

```powershell
winget install Git.Git
winget install Microsoft.DotNet.SDK.9
git clone https://github.com/bakahuiii/SELENE.git
cd SELENE
dotnet run --project desktop\SELENE.Windows\SELENE.Windows.csproj
```

Android 0.5.4 已内置经过校验的 Syncthing 原生核心。Windows SELENE 会生成 5 分钟
有效的一次性二维码并自动批准扫码手机；首次同局域网配对后，后续可跨网络自动补传，
不需要自建服务器或再次扫码。完整说明见
[一次配对同步](docs/P2P_SYNC.zh-CN.md)（[English](docs/P2P_SYNC.md)）。

## 平台

- Android：采集屏幕使用、前台应用时段、日历、设备状态、网络状态和可选
  的后台持续移动轨迹、速度与最近位置兜底。
- Windows：默认采集 SELENE 运行期间的前台进程名及使用时段、空闲时长、电源
  与网络状态；用户可逐项开启窗口标题、可执行路径和当前前台浏览器 URL，并会被
  明确标为敏感字段。

两个平台都使用 selene-context-events/v1，每次采集都会创建新的：

~~~text
SELENE-v1-20260806T185439123Z/context-events.json
~~~

快照不会被打开、合并或改写。远程同步模式下，Android 只有收到 Windows 持久归档的
逐文件 SHA-256 ACK 且快照超过 24 小时才删除手机副本；Windows Archive 不自动删除。
HYPERION 请选择 Windows 持久归档父目录。

## 一次配对远程同步

1. Windows SELENE 选择同步收件箱，建议开启登录后启动。
2. 点击“生成一次性配对二维码”；缺少 Syncthing 时会通过 winget 安装官方包。
3. 手机 SELENE 点击“扫描 Windows 配对二维码”，或粘贴配对码。
4. 两端显示成功后即可离开同一网络。Android 会写入应用私有 Send Only 目录，
   Windows 以 Receive Only 接收，再原子复制到独立持久归档并把归档路径写入用户级
   `HYPERION_SELENE_INBOX`。双方界面每 10 秒显示连接状态。

Windows 归档成功后会通过独立 `selene-ack-v1` 反向确认。Android 逐文件验证路径、大小
和 SHA-256 后，自动清理一天前的手机快照；任何离线、缺失或不匹配都会保留数据。
从旧版覆盖更新会自动补建 ACK 文件夹，不需要再次扫码。

Windows `0.5.4` 还会在持久归档的 `Merged` 子目录生成按日双端合并 JSON。它保留
Android 和 Windows 原始事件，按稳定 ID 去重；原始快照仍完整保留。详见
[按日合并 JSON](docs/DAILY_MERGE.zh-CN.md)（[English](docs/DAILY_MERGE.md)）。

Windows 还会从合并 JSON 在本地生成可读的中文日摘要（JSON 与 Markdown）。它使用确定性
规则翻译常见应用名、整理活动、位置和移动信息，不调用网络或远程模型，且不会改写原始数据。
详见[本地日摘要](docs/DAILY_NARRATIVE.zh-CN.md)（[English](docs/DAILY_NARRATIVE.md)）。

Android 8+ 要求低重要性的前台服务通知；SELENE 不会用规避系统规则的方式隐藏它。
Android APK 支持 `arm64-v8a` 和 `armeabi-v7a` 真机 ABI。

## Android 持续运动记录

Android `0.5.4` 在开启自动采集和后台位置后，会使用前台定位服务记录已经确认的
持续移动。它会输出轨迹点、各点的大致速度、距离和一次行程汇总；室内走几步、
单个噪声点、陈旧位置、低精度定位和不合理跳点不会作为移动导出。

完整的开通步骤、权限、过滤规则、字段、隐私边界、耗电与重启限制见
[Android 持续运动记录](docs/ANDROID_MOVEMENT.zh-CN.md)。英文版见
[ANDROID_MOVEMENT.md](docs/ANDROID_MOVEMENT.md)。导出协议中英文对照见
[EXPORT_LAYOUT.md](docs/EXPORT_LAYOUT.md) 和
[EXPORT_LAYOUT.zh-CN.md](docs/EXPORT_LAYOUT.zh-CN.md)。

## Windows 快速开始

1. 解压 SELENE-0.5.4-windows-x64.zip。
2. 运行 SELENE.Windows.exe。
3. 选择导出父目录。
4. 开启自动采集并选择周期。
5. 关闭窗口后程序会停留在系统托盘；需要完全退出时使用托盘菜单。

Windows 版无需管理员权限，发布版也不需要另装 .NET。详细说明见
[Windows 桌面版文档](docs/WINDOWS_DESKTOP.zh-CN.md)。内置组件来源和许可证见
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。

## Android 构建

构建 Android 版需要 Android SDK Platform 35、JDK 17 与 Gradle 8.9 或兼容版本。
请在本机配置 `JAVA_HOME`、`ANDROID_HOME` 和 Gradle 后执行构建；完整发布步骤见
[RELEASE_PROCESS.zh-CN.md](docs/RELEASE_PROCESS.zh-CN.md)，英文版见
[RELEASE_PROCESS.md](docs/RELEASE_PROCESS.md)。
