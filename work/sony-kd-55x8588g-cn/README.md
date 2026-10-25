# Sony KD-55X8588G 清理与启动器部署记录

> 2026-10-25 · 电视 IP：192.168.31.185 · 平台：BRAVIA_UR2_4K_CN（Android 10, API 29）
> 固件：BRAVIA_UR2_4K_CN-user 10 QTG3.200305.006.S57 661321（中国版固件，无 GMS/Play 商店）

## 做了什么

1. **备份**（`..\..\backup\`）：全部 182 个包清单、已禁用包、Home 归属、系统属性、三项 settings，改动前留存。
2. **禁用 43 个预装应用**（禁用而非卸载，随时可恢复），按 6 批执行、每批截图（`..\..\shots\01-06`），重启后核对全部存活。详细清单见本目录 `debloat.txt`（含每批分组与原因）。
3. **构建并安装苹果 TV 风格启动器**（包名 `com.gathin.tvhome`，本项目 `launcher\` 源码），设为系统 Home。
4. **重启验证**：Home 仍归新启动器、前台焦点正确、禁用清单完整、Home 键可返回、方向键导航正常。
5. **收尾**：关闭 ADB 调试（见下文），保证不再有网络调试端口暴露。

## 保留了哪些（都没动）

| 类别 | 内容 |
|---|---|
| 看电视/输入 | 直播电视（com.sony.dtv.tvx）、HDMI 输入层、电视调谐、输入源菜单 |
| 系统必需 | 系统设置、权限管理、Play 服务类核心包、DRM（partnerkeycn）、Wi-Fi Direct 框架 |
| 固件更新 | com.sony.dtv.update 全家（用户选择保留） |
| 应用中心 | 当贝市场 com.sony.dangbeimarket（用户选择保留） |
| 投屏 | 当贝投屏 com.dangbei.screencast（用户只保留这一个；乐播、Miracast 已禁用） |
| 内容应用 | 腾讯视频、QQ音乐、哔哩哔哩、芒果TV、爱奇艺（银河奇异果，Dock 使用） |
| 兜底启动器 | 当贝桌面 com.dangbei.TVHomeLauncher——保留启用，随时可切回 |

## 禁用了哪些（43 个，禁用不卸载）

分组见 `debloat.txt`，要点：

- **遥测/诊断**：qterics、da.service、idd.agent、usercontext、bugreport、loglevel、mediatek agent/dmagent、京东服务
- **演示/帮助**：demosupport、demosystemsupport、multiscreendemo、braviashow（屏保内容）、customersupport、smarthelp、helpguidecontents、mysony
- **辅助/IoT**：livingfit（健康）、iot、appassistant、notificationcenter、serviceinformation、bivlinfo、leanbacklauncher（仅 Boot 接收器）、quicksetsdk
- **B2B/酒店模式**：8+4 个 b2b 包 + kidsprofileadapter（儿童模式适配）
- **语音助手**：com.sony.dtv.searchapp（腾讯 AI 语音，用户选择禁用）
- **内容/投屏**：cibn.tv、technology.newplayer（含神策采集 SDK）、乐播投屏、Miracast

## 启动器现状

- 底部 Dock：爱奇艺（奇异果TV）→ 腾讯视频（NewTV极光）→ 哔哩哔哩（云视听小电视）→ 芒果TV（自动补位）
- 顶部右侧：输入源 / 无线网络 / 设置；顶部左侧：问候语 + 时钟 + 日期
- 直播电视入口：Inputs 菜单（com.sony.dtv.tvx）
- 想改 Dock 顺序/换应用：编辑 `launcher\app\src\main\res\values\config.xml` 的 `dock_apps`（按包名排），重新构建安装即可
- 想换壁纸/主题：参考项目 `themes\` 目录说明
- 优酷TV 未预装：从当贝市场安装后，把它的包名加进 `dock_apps` 即可上 Dock

## 如何恢复

**恢复单个应用**（如语音助手）：
```
adb shell pm enable com.sony.dtv.searchapp
```
任意包名换用 `debloat.txt` 或 `backup\packages.txt` 中的名字。

**一键全部恢复**（回到改动前的状态：全部包恢复、Home 切回当贝桌面）：
```
tools\restore.ps1 -TvHost 192.168.31.185 -List work\sony-kd-55x8588g-cn\debloat.txt -Stock com.dangbei.TVHomeLauncher
```
恢复基于 `~/.cleanyourtv/` 的改动前记录回放，未改动过的状态不会被覆盖。

**切回原厂启动器（不恢复应用）**：
```
adb shell cmd package set-home-activity com.dangbei.TVHomeLauncher/com.dangbei.leard.leradlauncher.ui.sony.main.MainActivity
adb shell input keyevent KEYCODE_HOME
```

## 如何重新构建/安装启动器

构建环境（本机已就绪）：JDK 17（`C:\AndroidSetup\jdk\jdk-17.0.20.1+1`）、Android SDK（platform-34 + build-tools 34.0.0）。
```
cd launcher
$env:JAVA_HOME = "C:\AndroidSetup\jdk\jdk-17.0.20.1+1"
# 如需联网下载依赖，用本机代理：
.\gradlew.bat assembleRelease "-Dorg.gradle.jvmargs=-Xmx2g -Dfile.encoding=UTF-8 -Dhttp.proxyHost=127.0.0.1 -Dhttp.proxyPort=7897 -Dhttps.proxyHost=127.0.0.1 -Dhttps.proxyPort=7897"
```
APK 产物：`launcher\app\build\outputs\apk\release\app-release.apk`（必须 < 576KB）。
安装并接管 Home（当贝桌面作兜底，保留启用）：
```
cd launcher
.\build-and-install.ps1 -TvHost 192.168.31.185 -StockLauncher none
```

## 收尾：关闭 ADB（已完成时按此操作）

安全要求：完成后关闭调试并确认 5555 端口不再监听（该端口为明文协议，开着有风险）。

1. 电视上进入：设置 → 设备偏好设置 → 关于 → 开发者选项
2. 关闭「USB 调试」与「网络调试/ADB 调试」
3. 「撤销 USB 调试授权」
4. 重启电视
5. 验证端口已关：
```
tools\adb-closed.py 192.168.31.185
```
看到端口关闭即完成。之后如需远程管理，重新打开开发者选项里的调试开关即可（新启动器与禁用状态不受影响）。

## 截图索引（..\..\shots\）

| 文件 | 内容 |
|---|---|
| 00-before.png | 改动前（网络设置页） |
| 01-06-after-*.png | 每批禁用后的电视画面 |
| 07-after-reboot.png | 全部禁用完成、重启后 |
| 08-launcher-home.png | 新启动器首次安装后 Home |
| 09-after-reboot-home.png | 重启后新启动器 Home（最终状态） |
| 10-nav-test.png | 方向键导航测试 |
